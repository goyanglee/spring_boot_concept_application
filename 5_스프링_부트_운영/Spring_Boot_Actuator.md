## Spring Actuator

Actuator는 endpoint 를 제공한다. 

인증 여부, 등록된 빈들, 어떤 자동설정이 어떻게 적용이 되었는지, 잘 구동 중인지, 메모리 사용 정보 등등(endpoint들)을 모니터링 할 수 있다.

단, shutdown은 앱을 종료시키는 거라서 비활성화가 기본값이다.

1. 의존성

```xml
<dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```

1. 페이지 접근

```xml
localhost:8080/actuator
```

1. 페이지 출력 (hateoas)

![Actuator01](https://github.com/goyanglee/spring_boot_concept_application/blob/main/5_%EC%8A%A4%ED%94%84%EB%A7%81_%EB%B6%80%ED%8A%B8_%EC%9A%B4%EC%98%81/images/Actuator01.png)

1. 옵션 활성화, endpoint 노출 여부 설정 

4.1 활성화 

```yaml
management.endpoint.{option_name}.enabled=true
management.endpoint.shutdown.enabled=true
```

4.2 endpoint 노출 여부 (전부 노출하면 위험하기 때문에 security 로 해당 endpoint들을 보안으로 막아두면 좋다)

```yaml
management.endpoints.web.exposure.include=*
//모든 endpoint 노출 
```

## JMX, HTTP

1. jconsole 입력 

![Actuator02](https://github.com/goyanglee/spring_boot_concept_application/blob/main/5_%EC%8A%A4%ED%94%84%EB%A7%81_%EB%B6%80%ED%8A%B8_%EC%9A%B4%EC%98%81/images/Actuator02.png)

1. 모니터링 하고 싶은 프로세스 선택 > connect > Insecure connection 

![Actuator03](https://github.com/goyanglee/spring_boot_concept_application/blob/main/5_%EC%8A%A4%ED%94%84%EB%A7%81_%EB%B6%80%ED%8A%B8_%EC%9A%B4%EC%98%81/images/Actuator03.png)

1. 메모리나 쓰레드 등의 정보를 확인할 수 있다.

![Actuator04](https://github.com/goyanglee/spring_boot_concept_application/blob/main/5_%EC%8A%A4%ED%94%84%EB%A7%81_%EB%B6%80%ED%8A%B8_%EC%9A%B4%EC%98%81/images/Actuator04.png)

1. 노출되어있는 endpoint 확인 

![Actuator05](https://github.com/goyanglee/spring_boot_concept_application/blob/main/5_%EC%8A%A4%ED%94%84%EB%A7%81_%EB%B6%80%ED%8A%B8_%EC%9A%B4%EC%98%81/images/Actuator05.png)

### VisualVM

자바10부터는 라이브러리에 포함되지 않는다.

1. 설치 

1. 실행 

![Actuator06](https://github.com/goyanglee/spring_boot_concept_application/blob/main/5_%EC%8A%A4%ED%94%84%EB%A7%81_%EB%B6%80%ED%8A%B8_%EC%9A%B4%EC%98%81/images/Actuator06.png)

1. 확인 

![Actuator07](https://github.com/goyanglee/spring_boot_concept_application/blob/main/5_%EC%8A%A4%ED%94%84%EB%A7%81_%EB%B6%80%ED%8A%B8_%EC%9A%B4%EC%98%81/images/Actuator07.png)

1. tools > plugin > available plugins > mbean 검색 후 설치 

![Actuator08](https://github.com/goyanglee/spring_boot_concept_application/blob/main/5_%EC%8A%A4%ED%94%84%EB%A7%81_%EB%B6%80%ED%8A%B8_%EC%9A%B4%EC%98%81/images/Actuator08.png)

### 인텔리제이

![Actuator09](https://github.com/goyanglee/spring_boot_concept_application/blob/main/5_%EC%8A%A4%ED%94%84%EB%A7%81_%EB%B6%80%ED%8A%B8_%EC%9A%B4%EC%98%81/images/Actuator09.png)

## 스프링 부트 어드민

공식은 아니고 누군가 올려놓은 오픈 라이브러리.

1. 어드민 서버 프로젝트 별도 생성 
    1.  의존성 

    ```xml
    <dependency>
        <groupId>de.codecentric</groupId>
        <artifactId>spring-boot-admin-starter-server</artifactId>
        <version>2.0.1</version>
    </dependency>
    ```

    b. 어노테이션 추가 

    ```java
    @SpringBootApplication
    @EnableAdminServer
    public class DemoserverApplication {

        public static void main(String[] args) {
            SpringApplication.run(DemoserverApplication.class, args);
        }

    }
    ```

2. 클라이언트 생성 
    1. 의존성

    ```xml
    <dependency>
        <groupId>de.codecentric</groupId>
        <artifactId>spring-boot-admin-starter-client</artifactId>
        <version>2.1.1</version>
    </dependency>
    ```

    b. 설정 추가 

    ```java
    spring.boot.admin.client.url=http://localhost:8080
    server.port=18080
    ```

    c. 트러블슈팅

     1) 2.0.1로 했을 때 스프링 부트 버전과 안맞아서 발생

![Actuator10](https://github.com/goyanglee/spring_boot_concept_application/blob/main/5_%EC%8A%A4%ED%94%84%EB%A7%81_%EB%B6%80%ED%8A%B8_%EC%9A%B4%EC%98%81/images/Actuator10.png)

    2) 2.1.1로 했을 때 발생 → 2.5.1 로 해서 해소

    ```
    org.springframework.beans.factory.BeanCreationException: Error creating bean with name 'adminHandlerMapping' defined in class path resource [de/codecentric/boot/admin/server/config/AdminServerWebConfiguration$ServletRestApiConfirguation.class]: Invocation of init method failed; nested exception is java.lang.StackOverflowError
    ```

3. [localhost:8080](http://localhost:8080) 으로 접근 

![Actuator11](https://github.com/goyanglee/spring_boot_concept_application/blob/main/5_%EC%8A%A4%ED%94%84%EB%A7%81_%EB%B6%80%ED%8A%B8_%EC%9A%B4%EC%98%81/images/Actuator11.png)
