# 스프링부트 개념과 활용 - 백기선

## 2부. 스프링 부트 시작하기

### 스프링 부트 소개
- 운영(제품) 수준에 애플리케이션을 빠르게 만들수 있도록 도와준다.
- 스프링부트가 가진 Convention -> Opinionated View
- 즉, 사용자가 하나하나 설정할 필요 없이 기본 설정을 제공해준다.
- 또한 원해는 대로 커스텀 설정이 가능하다.
- 기본 설정을 제공해주는 것은 스프링 플랫폼과 thrid-party까지이다.
- thrid-party의 대표적인 예가 톰켓이다.
- 비지니스로직을 구현하는데 필요한 기능뿐만 아니라, 다른 기능들도 폭넓게 제공한다.
- XML을 사용하지 않는다.

### 스프링 부트 시작하기
- spring boot 버전 관리(maven에서 parant와 동일하다.)
```groovy
plugins {
    id 'java'
    id 'org.springframework.boot' version '2.0.3.RELEASE'
    id 'io.spring.dependency-management' version '1.0.9.RELEASE'
}
```
- maven은 parant에 들어가면 어떤 버전을 관리해주는지 알 수 있지만 gradle은 홈페이지에서 직접 확인해야한다.
- [링크](https://docs.spring.io/spring-boot/docs/current/reference/html/appendix-dependency-versions.html)

### 스프링 부트 프로젝트 생성기
- https://start.spring.io

### 스프링 부트 프로젝트 구조
- SpringBoot main class는 본인이 사용하고 있는 최상위에 위치하는 것을 권장하는데 `@SpringBootApplication`이 Bean을 스캔할 때 그 하위 패키지에서 컴포넌트 스캔을 하기 때문이다.
- java 패키지 바로 아래 있다면 모든 패키지에 대해 컴포넌트 스캔을 하게 되는데, 그것은 위험하기 때문에 디폴트 패키지를 만든 후 하는 것을 추천한다.
- 메이븐 기본 프로젝트 구조와 동일
    - 소스 코드 (src\main\java)
    - 소스 리소스 (src\main\resource)
    - 테스트 코드 (src\test\java)
    - 테스트 리소스 (src\test\resource)
- 메인 애플리케이션 위치
    - 기본 패키지

## 3부. 스프링 부트 원리

### 의존성 관리 이해
- `spring-boot-starter-parent`가 인코딩, 플로그인, 자바 버전 명시 등 여러가지 기본설정을 해준다.
- gradle에서는 위를 참고하면 된다.
- `dependencyManagement`는 `parent`에 비해 적인 설정이 들어있다.

### 의존성 관리 응용

### 자동설정 이해
- `@SpringBootApplication`
    - `@ComponentScan`
    - `@EnableAutoConfiguration`
    - `@springBootConfiguration`
- 이때 Bean은 두번에 나눠서 등록하게 된다.
    1. `@ComponentScan`
    2. `@EnableAutoConfiguration`
- static 메서드가 아닌 거스텀을 하고 싶다면 아래와 같이 하면 된다.
- 아래는 AutoConfiguration을 사용하고 싶지 않을때이다.
```java
   SpringApplication application = new SpringApplication(Application.class);
   application.setWebApplicationType(WebApplicationType.NONE);
   application.run(args);
```
- 위와 같이한다면 웹 서버로는 동작하지 않는다.
- `@EnableAutoCOnfiguration`에 있는 빈들을 확인하고 싶다면 `spring.factories`를 확인해보면 된다.
- `@Conditaional~~`은 조건에 따라 빈을 등록하겠다는 애노테이션이다.

### 자동설정 만들기 1부
- 자동설정 파일을 만들기 위해 의존성을 추가해야한다.
```groovy
    implementation 'org.springframework.boot:spring-boot-autoconfigure'
    implementation 'org.springframework.boot:spring-boot-autoconfigure-processor'
```
- `@Configuration` 파일을 작성한다.
- resource 폴더에 META-INF 폴더를 만들고 spring.factories 파일을 추가해 자동 설정 파일을 추가한다.
```groovy
org.springframework.boot.autoconfigure.EnableAutoConfiguration=\
  springboot.starter.inflearn.HoloManConfiguration
```
- jar파일과 로컬에 저장하기 위해 gradle에 다음과 같이 추가해야 한다.
```groovy
apply plugin: 'maven'

bootJar.enabled = false
jar.enabled = true

repositories {
    mavenCentral()
    mavenLocal()
}
```
- gradle install
- 그러면 .m2 파일에 생성된것을 확인할 수 있을 것이다.

### 자동 설정 만들기 2부 : `@ConfigurationProperties`
- `@ConditionalOnMissingBean`
    - 덮어쓰기 방지 애노테이션으로 Component Scan으로 Bean이 등록되어 있다면 또 빈으로 등록하지 말라는 애노테이션이다.
- properties 파일에 있는 값으로 setter를 사용하는 방법을 소개한다.
```java
import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.stereotype.Component;

@Component
@ConfigurationProperties("holoman")
public class HoloManProperties {
    private String name;
    private int howLong;

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public int getHowLong() {
        return howLong;
    }

    public void setHowLong(int howLong) {
        this.howLong = howLong;
    }
}
```

```java
package springboot.starter.inflearn;

import org.springframework.boot.autoconfigure.condition.ConditionalOnMissingBean;
import org.springframework.boot.context.properties.EnableConfigurationProperties;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
@EnableConfigurationProperties(HoloManProperties.class)
public class HoloManConfiguration {
    @Bean
    @ConditionalOnMissingBean
    public HoloMan holoMan(HoloManProperties holoManProperties) {
        HoloMan holoMan = new HoloMan();
        holoMan.setName(holoManProperties.getName());
        holoMan.setHowLong(holoManProperties.getHowLong());
        return holoMan;
    }
}
```
- 이때 주의해야 하는 것은 애노테이션을 읽기 위해 의존성을 추가해줘야 한다.
```groovy
    implementation 'org.springframework.boot:spring-boot-autoconfigure'
    implementation 'org.springframework.boot:spring-boot-autoconfigure-processor'
    implementation 'org.springframework.boot:spring-boot-configuration-processor'
    annotationProcessor 'org.springframework.boot:spring-boot-configuration-processor'
```

### 내장 웹서버 이해
- 스프링 부트는 서버가 아니다. Spring Boot는 톰켓을 기본적으로 내장 웹서버로 사용하고 있다.
- 자바로 톰캣을 직접 실행할 수 있지만, 이미 자동 설정으로 되어 있기 때문에 따로 아래와 같은 코드로 작성할 일은 거의 없다.
```java
package com.inflearn.springboot;

import java.io.IOException;
import java.io.PrintWriter;
import java.nio.file.Files;

import javax.servlet.ServletException;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

import org.apache.catalina.Context;
import org.apache.catalina.LifecycleException;
import org.apache.catalina.startup.Tomcat;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class Application {
    public static void main(String[] args) throws LifecycleException, IOException {
        Tomcat tomcat = new Tomcat();
        tomcat.setPort(8080);

        String docBase = Files.createTempDirectory("tomcat-basedir").toString();

        Context context = tomcat.addContext("/", docBase);
        HttpServlet servlet = new HttpServlet() {
            @Override
            protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws
                ServletException,
                IOException {
                PrintWriter writer = resp.getWriter();
                writer.println("<html><head><title>");
                writer.println("Hey, Tomcat");
                writer.println("</title></head>");
                writer.println("<body><h1>hello Tomcat</h1></body>");
                writer.println("</html>");
            }
        };
        String servletName = "helloServlet";
        tomcat.addServlet("/", servletName, servlet);
        context.addServletMappingDecoded("/hello", servletName);

        tomcat.start();
        tomcat.getServer().await();
    }
}
```
- 위와 같은 코드가 이미 자동설정으로 되어 있다.
- `ServletWebServerFactoryAutoConfiguration` : 서블릿 웹 서버 생성
    - TomcatServletWebServerFactoryCustomizer (서버 커스터마이징)
- DispatcherServletAutoConfiguration : 서블릿 만들고 등록

### 내장 웹 서버 응용 1부 : 컨테이너와 서버 포트
- 우리는 기본적으로 톰캣을 웹 서버로 사용한다. 그렇다면 톰캣을 말고 다른 컨테이너를 사용하려면 어떻게 해야할까? 아래와 같은 의존성 변경이 필요하다.
```groovy
implementation ('org.springframework.boot:spring-boot-starter-web') {
        exclude group: 'org.springframework.boot', module: 'spring-boot-starter-tomcat'
    }
implementation 'org.springframework.boot:spring-boot-starter-jetty'
```
- spring-boot-web에는 기본적으로 톰캣이 있는데, 이걸 exclude하는 방법이다.
- 기본적으로 스프링부트는 웹 애플리케이션을 만들려고 시도하는데, 이걸 설정할 수 있다.
```properties
spring.main.web-application-type=none
```
- 포트 변경 방법
```properties
server.prot=7070
# 이때 값을 0으로 준다면 random port가 된다.
```
- 애플리케이션이 포트를 어떻게 사용하게 되는가?
- `ApplicationListener<ServletWebServerInitializedEvent>`
```java
package com.inflearn.springboot;

import org.springframework.boot.web.servlet.context.ServletWebServerApplicationContext;
import org.springframework.boot.web.servlet.context.ServletWebServerInitializedEvent;
import org.springframework.context.ApplicationListener;
import org.springframework.stereotype.Component;

@Component
public class PortListener implements ApplicationListener<ServletWebServerInitializedEvent> {
    @Override
    public void onApplicationEvent(ServletWebServerInitializedEvent event) {
        ServletWebServerApplicationContext servletWebServerApplicationContext = new ServletWebServerApplicationContext();
        System.out.println(servletWebServerApplicationContext.getWebServer().getPort());

    }
}
```

- server.compression.enabled=true
    - 응답을 압축해서 보내는 것이다.

### 내장 웹 서버 응용 2부 : HTTPS와 HTTP2
- key 생성 방법
```text
keytool -genkey -alias spring -storetype PKCS12 -keyalg RSA -keysize
 2048 -keystore keystore.p12 -validity 4000
```
- properties 설정
```properties
server.ssl.key-store=keystore.p12
server.ssl.key-store-password=123456
server.ssl.key-store-type=PKCS512
server.ssl.key-alias=tomcat
```
- 이렇게 하면 https를 적용시킬수 있다. 그러나 웹 브라우저는 경고창을 띄우게 되는데 실제 제대로 인증을 받지 않는 인증서이기 때문이다.
- https를 사용하면 http를 사용하지 못하는데 http connector는 기본적으로 하나만 있는데 https를 적용하면 connector가 https를 바라보기 떄문이다.
- 이때 connector를 새로 만들고 port를 다르게 준다면 둘다 사용이 가능하다
```java
    @Bean
    public ServletWebServerFactory serverFactory() {
        TomcatServletWebServerFactory tomcat = new TomcatServletWebServerFactory();
        tomcat.addAdditionalTomcatConnectors(createStandardConnector());
        return tomcat
    }

    private Connector createStandardConnector() {
        Connector connector = new Connector("org.apache.coyote.http11.http1NioProtocal");
        connector.setPort(8080);
        return connector;
    }
```
- http2 활성화 방법
```properties
server.http2.enabled=true
```

- tomcat 8.5 이하 버전에서는 http2를 사용하기 위해서는 여러가지 설정이 필요하기 때문에 힘들다.
- tomcat 9버전 이상부터는 위에 값과 jdk9 이상 버전을 사용하고 있다면 그냥 사용이 가능하다.
- 언더토우는 위에 값만 설정한다면 https만 적용되어 있다면 가능하다.
