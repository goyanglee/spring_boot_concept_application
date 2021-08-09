### 소개

#### 스프링 웹 MVC 

스프링 웹 MVC란 Spring 프레임워크에서 제공하는 웹 모듈이다. 클래스들을 모델/뷰/컨트롤러로 구분하여 프로젝트를 구현.

[참조레퍼런스] https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#spring-web

<br/>

#### 스프링부트 MVC

```java
@WebMvcTest(UserController.class) //웹 계층을 테스트할 수 있는 어노테이션
public class UserControllerTest {
  
    //웹 MVC를 테스트할 때 주로 사용하는 객체
    //@WebMvcTest가 MocMvc를 자동으로 빈으로 만들어주기 때문에 꺼내서 사용하면 된다.
    @Autowired
    MockMvc mockMvc;

    @Test
    public void hello() throws Exception {
        mockMvc.perform(get("/hello"))
                .andExpect(status().isOk())
                .andExpect(content().string("hello"));
    }
```

```java
@RestController
public class UserController {

    @GetMapping("/hello")
    public String hello() {
        return "hello";
    }
}
```

스프링부트에서는 별도 설정없이 간단한 어노테이션으로 스프링 MVC 개발이 가능하다.

별도 설정없이 구현할 수 있는 이유는, 스프링부트가 제공해주는 기본설정 덕분이다. 

<br/>

**< WebMvcAutoConfiguration 자동설정 파일 >**

![그림1](https://github.com/goyanglee/spring_boot_concept_application/blob/main/4_%EC%8A%A4%ED%94%84%EB%A7%81_%EB%B6%80%ED%8A%B8_%ED%99%9C%EC%9A%A9/image/MVC_img_1.png)

<br/>

```java

@Configuration(
    proxyBeanMethods = false
)
@ConditionalOnWebApplication(
    type = Type.SERVLET //웹 어플리케이션
)
@ConditionalOnClass({Servlet.class, DispatcherServlet.class, WebMvcConfigurer.class}) //클래스패스에 해당 클래스들 존재
@ConditionalOnMissingBean({WebMvcConfigurationSupport.class}) //해당 클래스가 없으면
@AutoConfigureOrder(-2147483638)
@AutoConfigureAfter({DispatcherServletAutoConfiguration.class, TaskExecutionAutoConfiguration.class, ValidationAutoConfiguration.class}) //사용
public class WebMvcAutoConfiguration {
    public static final String DEFAULT_PREFIX = "";
    public static final String DEFAULT_SUFFIX = "";
    private static final String SERVLET_LOCATION = "/";

    public WebMvcAutoConfiguration() {
    }

    @Bean
    @ConditionalOnMissingBean({HiddenHttpMethodFilter.class})
    @ConditionalOnProperty(
        prefix = "spring.mvc.hiddenmethod.filter",
        name = {"enabled"}
    )
    public OrderedHiddenHttpMethodFilter hiddenHttpMethodFilter() {
        return new OrderedHiddenHttpMethodFilter();
    }
    ...
```

- HttpMessageConverterAutoConfiguration
- 뷰리졸버 설정 제공

<br/>

#### 스프링 MVC 확장

스프링부트가 제공해주는 MVC 기능을 전부 사용하면서 추가적인 설정을 더 주고 싶다면 'WebMvcConfigurer'를 구현한다. 

```java
@Congiguration
public class WebConfig implements WebMvcConfigurer {
  //다양한 콜백 제공
  ...
}
```

<br/>

#### 스프링 MVC 재정의

'@EnableWebMvc' 어노테이션을 사용하게되면 스프링부트가 제공하는 모든 MVC 기능은 사라지게되어 자체 구현해줘야 한다. 웹 MVC 설정은 스프링부트가 제공하는 것을 사용하자

```java
@Configuration
@EnableWebMvc
public class WebConfig implements WebMvcConfigurer {
  ...
}
```

<br/>

### HttpMessageConverters

스프링 프레임워크에서 제공하는 인터페이스로, 스프링부트 MVC에서 제공하는 기능

- HTTP 요청본문으로 들어오는 정보를 객체로 변환하거나, 객체를 HTTP 응답본문으로 변환할 때 사용

  ```
  {"username":"goyanglee", "password":"20161114"} <=> User
  ```

- 스프링부트 어노테이션 중 @RequestBody, @ResponseBody로 메시지 컨버터를 적용함

<br/>

```java
//컨버터가 사용되는 예
@RestController 
public class UserController {
  @PostMapping("/user")
  public @ResponseBody User create(@RequestBody User user) {
    return user; 
  }
}
```

- 요청 형태, 응답 형태에 따라 사용되는 HttpMessageConverter가 달라진다. 

  - **@RequestBody**

    요청바디=Json형태 : JsonMessageConverter가 Json 형태의 본문을 객체로 변환해준다.

  - **@ResponseBody**

    리턴타입=User : 기본적으로 JsonMessageConverter를 사용하여 객체를 변환해준다. (HTTP메시지는 문자이기 때문에 객체자체 리턴X)

    리턴타입=String : StringMessageConverter가 사용된다. 

    리턴타입=int : 정수값 자체를 String으로 변환할 수 있기 때문에 StringMessageConverter가 사용된다. 

<br/>

#### 참고

```java
//case1
@RestController
public class UserController {
    @GetMapping("/hello")
    public String hello() {
        return "hello";
    }
}

//case2
@Controller
public class UserController {
    @GetMapping("/hello")
    public @ResponseBody String hello() {
        return "hello";
    }
}
```

- **@RestController를 사용하면 @ResponseBody를 생략해도 된다.**
  - MessageConverter가 적용되어 반환값을 응답본문으로 바꾼다. 
- **@Controller를 사용할 경우에는 @ResponseBody를 붙여줘야 반환 값에 메시지컨버터가 적용이 된다.**
  - @ResponseBody를 선언하지 않으면 BeanNameViewResolver에 의해서 ViewName에 해당하는 뷰를 찾으려고 치도한다. 

<br/>

### ContentNegotiatingViewResolver

뷰를 찾기 위해 요청 URL의 AcceptHeader를 사용하는 ViewResolver 

- 하나의 URI를 통해 다양한 contentType 으로 응답을 할 수 있도록 도와준다. 

- 응답을 만들 수 있는 모든 뷰를 찾고 최종적으로 Accept header와 뷰의 타입을 비교하여 리턴

  - Accept Header: 클라이언트가 어떤 타입의 응답을 원하는 지 서버에게 알려주는 것

    - Accept Header를 제공하지 않는 요청도 많다. 이런 경우에 대비해서 "foramt"이라는 매개변수를 사용

      ```
      /path?format=xml
      /path?format=html
      ```

<br/>

**< ContentNegotiatingViewResolver 동작확인을 위해 응답을 XML로 받아보기 >**

(1) 테스트 코드 작성

```java
@Test
public void createUser_JSON() throws Exception {
  String userJson = "{\"username\":\"goyanglee\",\"password\":\"20161114\"}";
  mockMvc.perform(post("/users/create")
                  .contentType(MediaType.APPLICATION_JSON)
                  .accept(MediaType.APPLICATION_XML)
                  .content(userJson))
    .andExpect(status().isOk())
    .andExpect(xpath("/User/username").string("goyanglee"))
    .andExpect(xpath("/User/password").string("20161114"));
}
```

- HttpMediaTypeNotAcceptableException 에러가 발생한다. 

  - HttpMediaType을 처리하는 HTTP 메시지 컨버터가 없어서 발생

  - HttpMessageConverterAutoConfiguration 

    ![그림2](https://github.com/goyanglee/spring_boot_concept_application/blob/main/4_%EC%8A%A4%ED%94%84%EB%A7%81_%EB%B6%80%ED%8A%B8_%ED%99%9C%EC%9A%A9/image/MVC_img_2.png)


    - XmlMapper 클래스가 클래스패스에 있을 때만 컨버터가 등록이 되도록 설정되어 있음 

      => 의존성 추가 필요
<br/>

(2) pom.xml 의존성 추가

```xml
<dependency>
  <groupId>com.fasterxml.jackson.dataformat</groupId>
  <artifactId>jackson-dataformat-xml</artifactId>
  <version>2.9.6</version>
</dependency>
```
<br/>

3) 정리
- json response : 기본으로 사용하기 때문에 설정필요 없음
- xml response : 의존성 추가 필요

<br/>

### 정적 리소스 지원

웹 MVC 기본설정에서 정적리소스 지원 기능을 제공한다.  웹 브라우저나 클라이언트에서 요청이 들어왔을 때 이미 만들어진 적절한 리소스를 그대로 보내준다. 

#### 정적 리소스 제공방법

```
# 정적 리소스 맵핑 "/**"
# 기본 리소스 위치
	- classpath:/static
	- classpath:/public
	- classpath:/resources
	- classpath:/META-INF/resources
```

- 기본적으로 위 4가지 위치에 있는 리소스가 "/**" 요청에 매핑되어 제공된다. 
- 예) /hello.html 요청 => /static/hello.html 파일을 보내줌
- ResourceHttpRequestHandler가 처리

<br/>

#### 정적 리소스 동작

ResourceHttpRequestHandler는 'last modified'와 'If-Modified-Since'라는 헤더 정보를 비교하여 상황에 따라 304 응답을 내려준다. 파일 변경없이 요청이 들어오면 304에러를 주면서 리소스 자체를 내려주지는않는다. 이러한 캐싱동작으로 빠른 응답이 가능하다. 

```
(!)
last modified : 파일이 변경되어 요청이 들어왔을 때 업데이트 됨
If-Modified-Since : 요청이 들어왔을 때마다 업데이트 됨
```

<br/>

#### 리소스 매핑URL 변경

- 기본적으로 리소스들은 전부 루트부터 매핑되어 있다. 

  URL 패턴 : http://localhost:8080/hello.html

- application.properties 설정을 통해 매핑을 변경할 수 있다. 

  ```
  spring.mvc.static-path-pattern=/static/**
  > http://localhost:8080/static/hello.html
  ```

<br/>

#### 리소스 위치 재설정

- application.properties 설정을 통해 위치를 커스터마이징 할 수 있다. 이 방법은 기본 리소스 로케이션을 사용하지 않게 되어서 권장하지 않는다. 

  ```
  spring.mvc.static-locations
  ```

- 추천하는 방법 

  - 스프링부트가 기존에 제공하는 리소스핸들러는 그대로 유지하면서 추가하고자하는 리소스핸들러만 커스터마이징

  ```java
  //WebMvcConfigurer의 addResourceHandlers로 커스터마이징
  @Configuration
  public class WebConfig implements WebMvcConfigurer {
      @Override
      public void addResourceHandlers(ResourceHandlerRegistry registry) {
          registry.addResourceHandler("/m/**")  
            			//m으로 시작하는 이러한 요청이 오면
                  .addResourceLocations("classpath:/m/") 
            			//이 요청을 클래스패스 기준으로 m 디렉토리 밑에서 제공을 하겠다 
            			//맨 뒤 '/'붙여줘야 매핑 제대로 동작
                  .setCachePeriod(20); 
        			//단위:초
        			//캐시전략 직접설정
      }
  }
  ```

<br/>

### 웹 JAR

스프링부트는 웹 자르에 대한 기본 매핑을 지원한다. 

```
WebJars
- WebJars 는 클라이언트에서 사용하는 웹라이브러리(jquery 와 bootstrap) 를 JAR 파일 안에 패키징한 것이다.
출처: https://java.ihoney.pe.kr/428 [허니몬(Honeymon)의 자바guru]
```

메이븐 저장소에 업로드되어있기 때문에 pom.xml에 추가해서 사용하면 된다. 

예) jquery

```xml
<!-- https://mvnrepository.com/artifact/org.webjars.bower/jquery 
org.webjars.bower 적용이 안 됨 -->
<dependency>
  <groupId>org.webjars</groupId>
  <artifactId>jquery</artifactId>
  <version>3.6.0</version>
</dependency>
```

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Sample</title>
</head>
<body>
    Hello, static resource
<script src="/webjars/jquery/3.6.0/jquery.min.js"></script>
<script>
  $(function () {
    console.log("jQuery ready");
  });
</script>
</body>
</html>
```

<br/>

#### 버전 생략하는 방법

```xml
<dependency>
  <groupId>org.webjars</groupId>
  <artifactId>webjars-locator-core</artifactId>
  <version>0.40</version>
</dependency>
```

```
<script src="/webjars/jquery/jquery.min.js"></script>
```

<br/>

### 인덱스 페이지(웰컴 페이지)

- http://localhost:8080 에서 보여주는 루트 페이지
- 2가지 방법
  - 정적 페이지를 보여주는 방법
    - 정적리소스를 지원하는 4가지 위치 중 선택해서 index.html을 생성
  - 동적 페이지를 보여주는 방법
    - TODO

<br/>

### 파비콘 교체

- 리소스 위치 아래에 favicon.ico 파일 추가
- 업데이트 안되는 경우 브라우저 캐싱문제일 수 있다. 
  - 파비콘 아이콘 직접 요청(http://localhost:8080/favicon.ico) => Refresh!! => 모든브라우저 종료 후 재요청
