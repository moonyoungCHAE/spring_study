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

