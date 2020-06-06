# 스프링 부트 프로젝트 생성하기

1. spring initializer
2. java project 생성 후 의존성 추가
3. 온라인 initializer

[Spring Initializr](https://start.spring.io/)

# 의존성 관리 이해

'io.spring.dependency-management' 가 Spring 의존성 버전 관리를 해준다.

```
plugins {
    id 'org.springframework.boot' version '2.2.2.RELEASE'
    **id 'io.spring.dependency-management' version '1.0.8.RELEASE'
		// cf. maven에서는 parent**
    id 'java'
}
.
.
.
dependencies {
    **implementation 'org.springframework.boot:spring-boot-starter-web'
    implementation 'org.springframework.boot:spring-boot-starter-data-jpa'
		// spring.dependency-management 덕분에 version을 명시하지 않고 사용할 수 있다.
		// 이 의존성은 또 다른 라이브러리들을 추가해준다.
}
```

1. 'io.spring.dependency-management'에는 각 의존성에 따른 버젼을 설정해두었다.
2. 사용자는 version을 명시하지 않아도 의존성을 추가할 수 있다.

    하지만 version을 명시하는 것이 좋다. (server에 배포할 때 다른 version이 사용될 수 있기 때문에)

3. Spring boot에서 의존성 관리를 해주면서 개발자는 관리할 의존성이 별로 없어졌다. 

---

```java
repositories {
    mavenCentral()
    jcenter()
}
```

`repositories` 는 각종 의존성(라이브러리)들을 어떤 원격 저장소에서 받을지를 정합니다. 기본적으로 mavenCentral을 많이 사용하지만, 최근에는 **라이브러리 업로드 난이도** 때문에 jcenter도 많이 사용합니다.

mavenCentral은 이전부터 많이 사용하는 저장소지만, 본인이 만든 라이브러리를 업로드하기 위해 많은 과정과 설정이 필요하다보니 개발자들이 직접 만든 라이브러리를 업로드하는 것이 힘들어 점점 공유가 안되는 상황이 발생했습니다.

이런 문제점을 개선하여 최근에 나온 것이 jcenter입니다. 그리고 jcenter에 라이브러리를 업로드하면 mavenCentral에도 업로드될 수 있도록 자동화를 할 수 있습니다. 여기서는 mavenCentral, jcenter 둘 다 등록해서 사용하겠습니다.

---

## 자동 설정 이해

@SpringBootApplication  = @SpringBootConfiguration + @ComponentScan + @EnableAutoConfiguration

Bean은 사실 두 단계로 나눠서 읽힘

- 1단계: @ComponentScan
- 2단계: @EnableAutoConfiguration

### ComponentScan

1. @Component 을 scan해준다.

    Component: @Configuration @Repository @Service @Controller @RestController

2. Sacn 받지 않도록 설정해준 Component는 제외하고 Scan한다.

```java
@ComponentScan(excludeFilters = { @Filter(type = FilterType.CUSTOM, classes = TypeExcludeFilter.class),
		@Filter(type = FilterType.CUSTOM, classes = AutoConfigurationExcludeFilter.class) })
```

3. ComponentScan을 시작하는 곳 하위 package만 Scan하는 것 주의할 것

### EnableAutoConfiguration

- @EnableAutoConfiguration에 등록된 Annotation(@XXXConfiguration)이 있다면 Bean 등록해준다.
    - gradle 설정을 통해 EnableAutoConfiguation 라이브러리를 받았다. (external lib)
    - 그 라이브러리의 meta file인 spring.factories에, @EnableAutoConfiguration을 통해 등록할 annotation 목록이 있다.
    - @XXXConfiguartion 역시 @Configuration이다.
    - 조건이 있어 조건에 따라 Bean 등록을 하게 된다.
---
### 내장 웹 서버 이해
부트는 서버가 아니다.
1. 톰캣 객체 생성
2. 포트 설정
3. 톰캣에 컨텍스트 추가
4. 서블릿 만들기
5. 톰캣에 서블릿 추가
6. 컨텍스트에 서블릿 맵핑
7. 톰캣 실행 및 대기

이 모든 과정을 보다 상세히 또 유연하게 설정하고 실행해주는게 바로 스프링 부트의 자동 설정

* ServletWebServerFactoryAutoConfiguration(서블릿 웹 서버 생성)
    * TomcatServletWebServerFactoryCustomizer(서버 커스터마이징)
* DispatcherServletAutoConfiguration
    * 서블릿 만들고 등록
---
## HTTPS, HTTP2
### HTTPS
(1) Key store 생성
```
$ keytool -genkey -alias tomcat -storetype PKCS12 -keyalg RSA -keysize 2048 -keystore keystore.p12 -validity 4000
application.yml
```
명령어 수행한 위치에 키스토어가 생성된다. 
(2) Key 등록
```
[application.yml]
server:
ssl:
  key-store: keystore.p12
  key-store-type: PKCS12
  key-store-password: 123456
  key-alias: tomcat
```
이렇게 SSL 키를 등록하고 스프링부트 애플리케이션을 실행하면, localhost:8080으로 접근이 불가하다. 앞으로 애플리케이션으로의 모든 접근은 https로 해야한다.

(3)
추가적으로 http 접근도 가능하게 설정하려면 아래와 같이 애플리케이션 코드에 http 요청을 받기 위한 커넥터를 추가해주면 된다. 대신 application.yml에서 https의 포트를 변경해준다.

```
 @Bean
   public ServletWebServerFactory serverFactory() {
       TomcatServletWebServerFactory tomcat = new TomcatServletWebServerFactory();
       tomcat.addAdditionalTomcatConnectors(createStandardConnector());
       return tomcat;
  }
​
   private Connector createStandardConnector() {
       Connector connector = new Connector("org.apache.coyote.http11.Http11NioProtocol");
       connector.setPort(8080);
       return connector;
  }
```
---
### 독립적으로 실행 가능한 JAR
**의존성 packing을 통해 받은 외부 라이브러리를 통해 JAR 파일 하나로 독립적으로 실행 할 수 있다.**

- gradle run (run하기 위해 필요한 파일 생성, jar 생성X )
- gradle boot jar (jar 생성)
- build (run + gardle boot jar에 필요한 파일 생성)
    - gradle boot jar 명령어를 통해 jar 파일을 만드려면 gradle에 main class 명시하고, plugin에 application 추가해주어야 함
    - gradle :  mf파일을 통해 실행한다.
    - jar : arFile, Launcher를 사용해서 실행한다.
- jar 안에 외부 라이브러리 jar로 가지고 있다.

### SpringApplication 실행

다음과 같은 방법으로 SpringApplication의 설정을 custom하여 실행할 수 있다.

```java
// 방법 1
SpringApplication app = new SpringApplicaiton(SpringInitApplication.class);
app.run(args);

// 방법 2
new SpringAppplicationBuilder().sources(SpringInitApplication.class).run(args);
```

----

### Event Listener

https://docs.spring.io/spring-boot/docs/current/reference/html/boot-features-spring-application.html#boot-features-application-events-and-listeners



Spring Container가 실행 된 후 Event Listener

```JAVA
public class Listener implements ApplicationListener<ApplicationStartedEvent> {
    @Override
    public void onApplicationEvent(ApplicationStartedEvent event) {
        
    }
}
```



Spring Container가 실행 되기 전 Event Listener

```JAVA
public class Listener implements ApplicationListener<ApplicationStartingEvent> {
    @Override
    public void onApplicationEvent(ApplicationStartingEvent event) {
        
    }
}
```

**이슈: Event가 Application Context 가 실행되기 전/후 중 언제 실행되는 지 유의할 것!**

Application Context가 실행 되기 전, StartedEvent :: 따로 Listener로 등록해주어야 한다.

```java
public static void main(String args) {
	SpringApplication app = new SpringApplication(SpringInitApplication.class);
	app.addListener(new Listener());
	app.run(args);
}
```

----

### Application Args

```
public static void main(String args) {
	// args : Application Argument
}
```

Application Argument

* Spring에서 Bean으로 등록해놓았다. 
* 가져다 쓰면 된다.

### ApplicaionRunner

```
@Component
public class Runner implements ApplicationRunner {
    @Override
    public void run(ApplicationArguments args) throws Exception {
		// Applicaion Argument Bean을 사용하는 예시
    }
}
```

Spring이 실행되면 위 run 메서드로 간다.

---

## 외부 설정

### 외부 설정 파일이란?

* 애플리케이션에서 사용하는 여러가지 설정 값들을 애플리케이션의 밖이나 안에 정의할 수 있는 기능을 말한다.

### application.properties 

* 이 파일은 스프링부트가 애플리케이션을 구동할 때 자동으로 로딩하는 파일이다.
* key - value 형식으로 값을 정의하면 애플리케이션에서 참조하여 사용할 수 있다.

```
tommy.name = tommy
tommy.age = ${random.int}
```

* 융퉁성 있는 바인딩
  * (케밥) tommy.full-name
  * (언드스코어) tommy.full_name
  * (캐멀) tommy.fullName
  * -> 알아서 tommy.fullName으로 바인딩 된다.
* 데이터 바인딩
  * properties에는 데이터 타입 정의되어 있지 않다.
  * 내가 바인딩할 곳에서 정의한 data type으로 바인딩 된다.

```
public class TestPropertyConfiguration {
	@NotBlank
    private String name; 
    // config 나 value : 사용할 곳에서 정의한 data type으로 바인딩된다.
    private int age;
```



cf.

```
heumsi.sessionTimeout = 20s
```

위와 같이 `s`, `m`, `d` 등과 같은 suffix 로 `Duration` 타입임을 명시하고 바인딩시켜줄 수 있다.



### 사용 방법

1. Value로 접근

```JAVA
 @Value("${tommy.name}")
 String name
```

* @Value로 접근 시 변수 값에 대한 Validation을 할 수 없다.

  

2. Property 통으로 접근
   * Validation 可
   * getter, setter 정의 해주어야 한다.

```JAVA
@Component
@ConfigurationProperties("tommy")
@Validated
public class TestPropertyConfiguration {
	@NotBlank
    private String name;
    private int age;

    public String getName() {
        return name;
    }
    
 	// ...	
}
```





Property는 어디에 정의하느냐, 어디에 저장하느냐에 따라 우선 순위가 달라진다.

**프로퍼티 우선 순위**

1. 유저 홈 디렉토리에 있는 spring-boot-dev-tools.properties
2. 테스트에 있는 @TestPropertySource
3. @SpringBootTest 애노테이션의 properties 애트리뷰트
4. 커맨드 라인 아규먼트
5. SPRING_APPLICATION_JSON (환경 변수 또는 시스템 프로티) 에 들어있는 프로퍼티
6. ServletConfig 파라미터
7. ServletContext 파라미터
8. java:comp/env JNDI 애트리뷰트
9. System.getProperties() 자바 시스템 프로퍼티
10. OS 환경 변수
11. RandomValuePropertySource
12. JAR 밖에 있는 특정 프로파일용 application properties
13. JAR 안에 있는 특정 프로파일용 application properties
14. JAR 밖에 있는 application properties
15. JAR 안에 있는 application properties
16. @PropertySource
17. 기본 프로퍼티 (SpringApplication.setDefaultProperties)

**application.properties 우선 순위 (높은게 낮은걸 덮어 씁니다.)**

1. file:./config/
2. file:./
3. classpath:/config/
4. classpath:/



cf. 테스트와 property

* 테스트 코드에 있는 property로 테스트가 진행된다.

  테스트 실행 = production code + resource (application.properties) 컴파일 => test code + resource (application.properties) 컴파일

* application-test.properties로 production code에서와 다른 이름의 property 파일을 저장하는 게 좋다.



cf. 3rd Party Configuration: Component 사용할 수 없다. (외부에서 정의한 )

@Bean은 가능, @ConfiguartionProperties -> 규칙

---

## Profile

### Profile이란 ? 

dev, prod, test : 스프링 부트에서는 프로파일(Profile)을 통해 스프링 부트 애플리케이션의 런타임 환경을 관리할 수 있습니다. 예로들어 어플리케이션 작동 시 테스트 환경에서 실행할 지 프로덕션 환경에서 실행할 지를 프로파일을 통해 관리할 수 있죠.



### 사용 방법

1. 어떤 property를 사용할지 명시한다. (at application.properties)
   ```
   [application.properties]
   spring.profiles.active=prod
   ```
2. 특정 Profile에 사용할 Bean을 명시한다.

   ```JAVA
   @Profile("prod")
   @Configuration
   public class BaseConfiguration {
   
       @Bean
       public String hello(){
           return "hello production";
       }
   }
   ```

* 결과

```JAVA
@Profile("test")
@Configuration
public class TestConfiguration {

    @Bean // 함수 Bean으로 등록 可
    public String hello(){ 
        return "hello test";
    }
}

@Component
public class AppRunner implements ApplicationRunner {

    @Autowired
    private String hello;

    @Override
    public void run(ApplicationArguments args) throws Exception {
        System.out.println("=============");
        System.out.println(hello); // prod로 Profile을 설정했으면 hello production이 출력된다.
        System.out.println("=============");
    }
}
```



### application-{프로파일}.properties 파일을 통한 프로파일 관리

**application-{프로파일}.properties** 파일을 생성하여 관리하게 되면 **application.properties** 보다 우선순위가 높게 외부 설정값이 관리되므로 보다 편하게 프로파일을 관리할 수 있습니다. 

```
[application.properties]
spring.profiles.active=prod
# application-prod.properties 가 우선순위가 높으므로 
# 아래 값은 다른 값으로 오버라이딩 된다.
saelob.name=power

[application-prod.properties]
saelobi.name=saelobi prod
# 추가할 프로파일 설정 -- properties에서 다른 profile 추가 가능하다.
spring.profiles.include=saelobi 

[application-saelobi.properties]
saelobi.fullName=KBS
```



<h2> 로깅 </h2>

```JAVA
@Component
public class AppRunner implements ApplicationRunner {
	// slf4j 로깅 파사드를 통해 logback 로깅 모듈을 지원
	private Logger logger = LoggerFactory.getLogger(AppRunner.class);

	@Override
	public void run(ApplicationArguments args) throws Exception {
		logger.info("=============");
		logger.info("This is Spring Boot App");
		logger.info("============="); // 기본적으로 출력된다.
        logger.debug("debug mode"); // 별도의 설정을 해야 출력된다.
        // [application.properties] logging.level.project.Runner=DEBUG
	}
}
```



### application.properties를 통한 로깅 설정

**application.properties를 통해 다음과 같이 로깅 설정을 할 수 있습니다.**

```
[application.properties]

spring.output.ansi.enabled=always
# 로그 메세지가 저장되는 로그 디렉터리 (새로 디렉토리가 생성되고, 여기에 log 정보가 저정되어 있다.)
logging.path=logs
# logging.level.{패키지 경로}를 통해 로깅 레벨을 결정할 수 있다.
# 원래 출력 안되던 logger.debug까지 출력된다.
logging.level.project.AppRunner=DEBUG
```
# 테스트

spring-boot-start-test

- test를 위한 기본
- gradle에 추가해주기

spring test에는 크게 2가지 종류가 있다.

1. 통합 테스트 (실제 서버 환경에서 테스트하기)
2. 슬라이싱 테스트 (실제 서버 환경 X에서 테스트하기)

## @SpringBootTest

- **실제 Spring 서버 구동 환경으로 하거나 (port 존재) / 그와 비슷한 환경 (Mock)에서 테스트한다**.
- @RunWith(SpringRunner.class) 와 같이 써준다.
- 실행하면 @SpringApplication 찾아간다
    - 모든 bean 스캔한다.
    - mock bean 설정하면 그 mock bean만 교체된다.
    - mock bean은 테스트할 때마다 reset된다.
### SpringBootTest  webEnvironment 1. MOCK (default)

- **내장 톰캣을 구동하지 않는다.**
    - servlet container를 띄우지 않는다.
    - spring 환경과 ""비슷""한 상태이다. (mockup 되어있다.)

    **→ MockMvc 같이 사용해준다.**

```java
@RunWith(SpringRunner.class)
@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.MOCK)
@AutoConfigureMockMvc
public class Test {

    @Autowired
    MockMvc mockMvc;

    @org.junit.Test
    public void name() {
        mockMvc.perform(get("/hello"))
                .andExpect(status().isOk())
                .andExpect(content().string("hello keesun"))
                .andDo(print());

    }
}
```

### SpringBootTest  webEnvironment 2. Port 지정

- **실제로 내장 톰캣이 구동된다.**
    - Random_PORT or PORT 지정
        - Random_PORT로 테스트하는 게 더 좋다.
- TestRestTemplate : sync 동기
- **WebTestClinet** : async 비동기
    - API 테스트할 때 더 좋다. (성능상 이득)
    - gradle에 추가해야 한다.

```java
@RunWith(SpringRunner.class)
@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
@AutoConfigureMockMvc
public class Test {
    @Autowired
    TestRestTemplate restTemplate;

    @MockBean
    SampleService sampleService;
    
    @Test
    public void name() {
        String result = restTemplate.getForObject("/hello", String.class);
    }
}
```

## Slice Test

실제 Spring 환경이 아닌, 원하는 Component만 사용하고 싶을 때

### **@JsonTest**

- `@JsonTest`는 JSON의 직렬화, 역직렬화를 수행하는 라이브러인 Gson과 Jackson의 테스트를 제공합니다.

```
@RunWith(SpringRunner.class)
@JsonTest
public class BookJsonTest {

    @Autowired
    private JacksonTester<Book> json;

    @Test
    public void json_test() throws IOException {

        final Book book = new Book("title", 1000D);

        String content= "{\n" +
                "  \"id\": 0,\n" +
                "  \"title\": \"title\",\n" +
                "  \"price\": 1000\n" +
                "}";

        assertThat(json.parseObject(content).getTitle()).isEqualTo(book.getTitle());

        assertThat(json.write(book)).isEqualToJson("/test.json");
    }
}
```

### @WebMVCTest

**Client Test에 적합하다**.

web과 관련된 component만 등록된다.

### @OutputCapture

log 메세지에 찍히는 것으로 테스트할 수 있다.

---

**SpringDevTool 설정하면 Reload할 수 있다.**

### Restart vs Reload

써드 파티 안의 jar 파일은 base classloader로 읽어들이고, 사용자가 만드는 클래스는 restart classloader로 읽어들입니다. 이러한 접근은 재시작이 _cold start(완전히 껏다가 다시 켜는 것)_보다 훨씬 빨리 일어나게 도와줍니다.

아직 재시작(restart)가 느리게 느껴진다면, JRebel 같은 reloading 기술을 이용할 수도 있습니다. JRebel은 릴로딩이 편하도록 클래스를 다시 만듭니다.

---

# Spring Web MVC

WebMVC (Test) 자동설정이 있다. 

**SpringBoot MVC : 자동설정**을 제공한다.

**Spring MVC 프레임워크**는 디커플된 웹 애플리케이션 개발 방법을 제공한다. Dispatcher Servlet, ModelAndView, View Resolver 과 같은 단순개념을 이용해서 웹 애플리케이션 개발을 쉽게 할 수 있도록 해준다.

**[출처]** [Spring Boot vs. Spring MVC vs. Spring 의 비교](http://blog.naver.com/sthwin/221271008423)|**작성자** [멋진태혁](http://blog.naver.com/sthwin)

Spring MVC 확장: @Configuration + @WebMVC Configurer

Spring MVC 재정의: @Configuration + @EnableWebMvc

## HttpMessgaeConvertors

Http 요청 ( Request/Response Body ) ↔ Object

```java
@PostMapping("/user")
public @ResponseBody User create
```

## VeiwResolver

Request Accetp Header에 따라 결과가 달라진다.

1. 모든 view type에 따른 결과를 만든다. (by. HttpMessageConvertor)
    - json, xml 등 format
        - (만약 convertor에서 해당 type을 지원하지 않는다면 gradle에 추가해줄 것)
    - view: jsp, html 같은 타입?? 아니면 json, xml 같은 타입?? 헷갈림!
2. accept header에 맞는 결과를 돌려준다. (by. ViewResolver)

ViewResolver가 Convertor에 특정 type으로 만들라고 시킨다 ??

### 정적 리소스

[localhost:8080/me.html](http://localhost:8080/me.html) → static resoure file

서버에서 어떤 작업을 한 후 데이터를 제공하는 게 아니라 원래 있던 데이터를 제공하는 것

- 기본 리소스 위치가 있고 변경할 수도 있다.

- 브라우저: 자기가 받은 리소스 last modified 있음, 받으려는 파일의 last modified와 비교해서 변화가 있다면 다시 받고 아니라면 원래 있던 리소스를 재사용한다. (=304 응답)
- ResourceHttpRequestHandler가 처리한다.
    - Handler: 어떤 요청이 왔을 때 처리를 지정한다.
        - [localhost:8080/me.html](http://localhost:8080/me.html) → static resoure file **(기본)**
        - **/m/something에 대한 요청을 처리를 추가할 수 있다.**

    ```java
    @Configuration
    public class WebConfig implements WebMvcConfigurer {
        @Override
        public void addResourceHandlers(ResourceHandlerRegistry registry) {
            registry.addResourceHandler("/m/**")
                    .addResourceLocations("classpath:/m/")
                    .setCachePeriod(20);
        }
    }
    ```

### 웹 JAR

[https://velog.io/@jayjay28/웹-JAR](https://velog.io/@jayjay28/%EC%9B%B9-JAR)

**Client 측에서 사용하는 웹 JAR (ex. jQuery)를 Server Gradle에 추가해서 사용할 수 있다 !!**

```java
[Gradle]
implementation (group: 'org.webjars.bower', name: 'jquery', version: '3.3.1')

[html]
<script src="/webjars/jquery/3.3.1/dist/jquery.min.js"></script>
<script>
    $(function() {
        alert("ready!");
    })
</script>
```