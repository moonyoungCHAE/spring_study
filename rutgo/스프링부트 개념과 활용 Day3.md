# 스프링부트 개념과 활용 - 백기선

## 4부. 스프링 부트 활용하기

### 테스트
- 테스트할 코드
```java
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class SampleController {
    private final SampleService sampleService;

    public SampleController(SampleService sampleService) {
        this.sampleService = sampleService;
    }

    @GetMapping("/hello")
    public String hello() {
        return "hello" + sampleService.getName();
    }
}
```
- 테스트 코드를 작성하기 위해서는 의존성을 추가해야한다.
```groovy
testImplementation 'org.springframework.boot:spring-boot-starter-test'
```
- 저 의존성 안에는 mockito, assertj 등 여러가지 테스트 도구들이 들어있다.
- MOCK TEST
```java
import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.autoconfigure.web.servlet.AutoConfigureMockMvc;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.test.context.junit4.SpringRunner;
import org.springframework.test.web.servlet.MockMvc;

import static org.springframework.test.web.servlet.request.MockMvcRequestBuilders.get;
import static org.springframework.test.web.servlet.result.MockMvcResultHandlers.*;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.*;

import junit.framework.TestCase;

@RunWith(SpringRunner.class)
@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.MOCK)
@AutoConfigureMockMvc
public class SampleControllerTest extends TestCase {

    @Autowired
    MockMvc mockMvc;

    @Test
    public void hello() throws Exception {
        mockMvc.perform(get("/hello"))
            .andExpect(status().isOk())
            .andExpect(content().string("hello rutgo"))
            .andDo(print());
    }
}
```
- SpringBootTest default값은 MOCK으로, 실제 환경이 돌아가는 것 처럼 servelt을 모킹하는 방법이다.
- 실제 웹 서버가 돌아가면서 테스트 하는 방법
```java
import static org.assertj.core.api.Assertions.*;

import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.boot.test.web.client.TestRestTemplate;
import org.springframework.test.context.junit4.SpringRunner;

@RunWith(SpringRunner.class)
@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
public class SampleControllerTest {

    @Autowired
    TestRestTemplate testRestTemplate;

    @Test
    public void hello() {
        String result = testRestTemplate.getForObject("/hello", String.class);
        assertThat(result).isEqualTo("hello럿고");
    }
}
```
- 이때 실제 웹 서버가 실행하는 테스트를 하게 된다면 테스트 범위가 너무 넓어지기 때문에 `mockBean`을 사용해서 범위를 줄일 수 있다.
```java
package com.inflearn.springboot.sample;

import static org.assertj.core.api.Assertions.*;
import static org.mockito.Mockito.*;

import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.boot.test.mock.mockito.MockBean;
import org.springframework.boot.test.web.client.TestRestTemplate;
import org.springframework.test.context.junit4.SpringRunner;

@RunWith(SpringRunner.class)
@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
public class SampleControllerTest {

    @Autowired
    TestRestTemplate testRestTemplate;

    @MockBean
    SampleService sampleService;

    @Test
    public void hello() {
        when(sampleService.getName()).thenReturn("브라운");
        String result = testRestTemplate.getForObject("/hello", String.class);
        assertThat(result).isEqualTo("hello브라운");
    }
}
```
- 위 코드에서 `sampleService`는 실제 실행되지 않고 `브라운`값을 반환한다.

- Srping5부터 추가된 `WebTestClient` 사용
- 이것은 `asyn/await`(비동기) 방식으로 응답이 오면 콜백을 받아서 이벤트를 실행할 수 있다.
- 이 테스트를 사용하기 위해서는 `webflux`를 의존성에 추가해야 한다.
```groovy
implementation 'org.springframework.boot:spring-boot-starter-webflux'
```
```java
package com.inflearn.springboot.sample;

import static org.assertj.core.api.Assertions.*;
import static org.mockito.Mockito.*;

import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.boot.test.mock.mockito.MockBean;
import org.springframework.boot.test.web.client.TestRestTemplate;
import org.springframework.test.context.junit4.SpringRunner;
import org.springframework.test.web.reactive.server.WebTestClient;

@RunWith(SpringRunner.class)
@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
public class SampleControllerTest {

    @Autowired
    WebTestClient webTestClient;

    @MockBean
    SampleService sampleService;

    @Test
    public void hello() {
        when(sampleService.getName()).thenReturn("브라운");
        webTestClient.get().uri("/hello").exchange()
            .expectStatus().isOk()
            .expectBody(String.class).isEqualTo("hello브라운");
    }
}
```
- SpringApplication을 실제 실행하면서 테스트를 한다면 테스트 비용이 엄청나게 높다.
- 왜냐하면 사용하지 않는 Bean들도 모두 불러오기 떄문이다.
- 이때 사용하는 것이 슬라이싱 테스트이다.
    - `@JsonTest`
        - 도메인이 json으로 변환할때 변환한 값에 대한 테스트이다.
    - `@webMvcTest`
        - 컨트롤러 하나만 테스트를 하는 것으로, 서비스나 Repository는 빈으로 등록되지 않는다. 따라서 mock으로 관리해야 한다.
        - webMvcTest에서 추가되는 빈은 @Controller, @ControllerAdvice, @JsonComponent, Filter, WebMvcConfigurer and HandlerMethodArgumentResolver 이것들이다.
```java
package com.inflearn.springboot.sample;

import static org.mockito.Mockito.*;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.*;
import static org.springframework.test.web.servlet.request.MockMvcRequestBuilders.*;

import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.autoconfigure.web.servlet.WebMvcTest;
import org.springframework.boot.test.mock.mockito.MockBean;
import org.springframework.test.context.junit4.SpringRunner;
import org.springframework.test.web.servlet.MockMvc;

@RunWith(SpringRunner.class)
@WebMvcTest(SampleController.class)
public class SampleControllerTest {

    @MockBean
    SampleService sampleService;

    @Autowired
    MockMvc mockMvc;

    @Test
    public void hello() throws Exception {
        when(sampleService.getName()).thenReturn("브라운");
        mockMvc.perform(get("/hello"))
            .andExpect(content().string("hello브라운"));
    }
}
```

### 테스트 유틸
- OutputCapture
    - 이것은 Junit에 있는 rule을 확장해서 만든 것이다.
    - 이떄 `public`으로 객체를 만들어 줘야 한다.
    - `Logger`나 `system.out.println()`을 모두 캡처한다고 생각하면 된다.

    ```java
    import static org.assertj.core.api.Assertions.*;
    import static org.mockito.Mockito.*;
    import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.*;
    import static org.springframework.test.web.servlet.request.MockMvcRequestBuilders.*;

    import org.junit.Rule;
    import org.junit.Test;
    import org.junit.runner.RunWith;
    import org.springframework.beans.factory.annotation.Autowired;
    import org.springframework.boot.test.autoconfigure.web.servlet.WebMvcTest;
    import org.springframework.boot.test.mock.mockito.MockBean;
    import org.springframework.boot.test.rule.OutputCapture;
    import org.springframework.test.context.junit4.SpringRunner;
    import org.springframework.test.web.servlet.MockMvc;

    @RunWith(SpringRunner.class)
    @WebMvcTest(SampleController.class)
    public class SampleControllerTest {

        @Rule
        public OutputCapture outputCapture = new OutputCapture();

        @MockBean
        SampleService sampleService;

        @Autowired
        MockMvc mockMvc;

        @Test
        public void hello() throws Exception {
            when(sampleService.getName()).thenReturn("브라운");
            mockMvc.perform(get("/hello"))
                .andExpect(content().string("hello브라운"));
            assertThat(outputCapture.toString())
                .contains("우테코")
                .contains("우아한테크코스");
        }
    }
    ```

- TestPropertyValues
- TestRestTemplate
- ConfigFileApplicationContextInitializer

### Spring Boot Devtools

- 스프링 부트를 기본적으로 적용되는 것이 아니라 의존성을 추가해야한다. 
- 부가적인 툴이다.

```groovy
implementation 'org.springframework.boot:spring-boot-devtools'
```

- [링크](https://github.com/spring-projects/spring-boot/blob/v2.0.3.RELEASE/spring-boot-project/spring-boot-devtools/src/main/java/org/springframework/boot/devtools/env/DevToolsPropertyDefaultsPostProcessor.java) 여기에 있는 설정들이 변경되는 것이다.
- 주로 캐시를 끄는 것과 연관되어 있다. 캐시가 켜져 있다면 변경 사항들이 빠르게 변경되지않는 경우들이 많다.
- restart 기능으로 코드를 변경하면 application restart를 해준다. 직접 껏다 켰다 하는거에 비해 빠르다.
- 스프링 부트는 reload 클래스 로더를 두개를 사용하는데, 하나는 의존성을 읽어 들이는 클래스 로더와 우리 application을 읽어 들이는 restart 클래스 로더가 있다.
- 변경한 다음에 저장후에 bulid project를 하면 다시 restart 한다.
- devtools가 있다면 ~/.spring-boot-devtools.properties가 1순위가 된다.
- liveload 기능을 사용하고 싶지 않다면 아래와 같은 설정을 추가해야 한다.

```groovy
spring.devtools.livereload.enabled=true
```

### 스프링 웹 MVC 1부 : 소개

```java
import static org.springframework.test.web.servlet.request.MockMvcRequestBuilders.*;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.*;

import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.autoconfigure.web.servlet.WebMvcTest;
import org.springframework.test.context.junit4.SpringRunner;
import org.springframework.test.web.servlet.MockMvc;

@RunWith(SpringRunner.class)
@WebMvcTest(UserController.class)
public class UserControllerTest {
    @Autowired
    MockMvc mockMvc;

    @Test
    public void hello() throws Exception {
        mockMvc.perform(get("/hello"))
            .andExpect(status().isOk())
            .andExpect(content().string("hello"));
    }
}
```

```java
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class UserController {

    @GetMapping("/hello")
    public String hello() {
        return "hello";
    }
}
```

- 스프링 부트에서 웹 MVC를 바로 사용할 수 있는데, spring-boot-autoconfigure에서 `spring.factroies에 있는 자동 설정 파일들이 실행되기 떄문에 가능한 것이다.
- 이때 가능하게 하는 설정파일이 `WebMvcAutoConfiguration`이다.
- 스프링 MVC 확장 ⇒ @Configuration + WebMvcAutoConfiguration

```java
import org.springframework.boot.autoconfigure.web.servlet.WebMvcAutoConfiguration;
import org.springframework.context.annotation.Configuration;
import org.springframework.web.servlet.config.annotation.WebMvcConfigurer;

@Configuration
public class WebConfig extends WebMvcAutoConfiguration {
}
```

- 스프링 MVC 재정의 ⇒ @Configuration + @EnableWebMvc

```java
import org.springframework.context.annotation.Configuration;
import org.springframework.web.servlet.config.annotation.EnableWebMvc;
import org.springframework.web.servlet.config.annotation.WebMvcConfigurer;

@Configuration
@EnableWebMvc
public class WebConfig implements WebMvcConfigurer {
}
```

### 스프링 웹 MVC 2부: HttpMessageConverters

- 이것은 인터페이스이며 MVC에 일부분이다.
- HTTP 요청 본문으로 들어오는 것을 객체로 변경하고 객체를 응답으로 변경하는 것을 한다.
- `@ReqeustBody` , `@ResposneBody`를 가지고 사용한다.

```java
@PostMapping("/user")
    public @ResponseBody User create(@RequestBody User user){
        return user;
    }
```

- 이때 사용하는 메시지 컨버터는 여러가지 종류가 많은데, 이때 어떤 것을 받고 보내는지에 따라 다르다.
- `Content-Type에 들어오는 것에 따라 다르게 실행된다.`
- 객체는 기본적으로 json 컨버터가 사용된다.
- String일때는 String 컨버터가 사용된다.
- int같은 것들도 String 컨버터가 사용된다.
- `@RestController`를 붙힐 경우에는 `@ResposneBody`를 생략해도 된다.
- 그냥 `Controller`는 뷰 리졸버를 사용한다.
- 이제 사용자를 만드는 생성하는 기능을 하는 것으로 예제를 작성해 볼것 이다.

```java
@Test
    public void createUser_JSON() throws Exception {
        String userJson = "{\"username\":\"rutgo\", \"password\":\"1234\"}";
        mockMvc.perform(post("/users/create")
            .contentType(MediaType.APPLICATION_JSON_VALUE)
            .accept(MediaType.APPLICATION_JSON_VALUE) // 응답으로 무엇을 원하는지
            .content(userJson))
            .andExpect(status().isCreated())
            .andExpect(jsonPath("$.username", is(equalTo("rutgo"))))
            .andExpect(jsonPath("$.password", is(equalTo("1234"))));
    }
```

```java
public class User {
    private Long id;
    private String username;
    private String password;

    public Long getId() {
        return id;
    }

    public String getUsername() {
        return username;
    }

    public String getPassword() {
        return password;
    }

    public void setId(Long id) {
        this.id = id;
    }

    public void setUsername(String username) {
        this.username = username;
    }

    public void setPassword(String password) {
        this.password = password;
    }
}
```
- 이때 DTO는 스프링 빈 규약에 의해 getter, setter가 필요하다.

```java
@PostMapping("/users/create")
    @ResponseStatus(HttpStatus.CREATED)
    public User create(@RequestBody User user){
        return user;
    }
```

### ViewResolver

- View Resolver를 설정하는 자동 설정이 존재한다.
- `ContentNegotiatingViewResolver`
    - 들어오는 Accpet Header에 따라 응답이 달라진다.
    - 클라이언트가 어떤 응답을 원하는지 보낸다.
    - 요청을 응답을 만들 수 있는 모든 View를 찾아서 accept 타입과 비교해서 최종적인 결과를 리턴한다.
- 응답을 XML로 출력해주기 위한 테스트 코드

```java
@Test
    public void createUser_XML() throws Exception {
        String userJson = "{\"username\":\"rutgo\", \"password\":\"1234\"}";
        mockMvc.perform(post("/users/create")
            .contentType(MediaType.APPLICATION_JSON_VALUE)
            .accept(MediaType.APPLICATION_XML_VALUE) // 응답으로 무엇을 원하는지
            .content(userJson))
            .andExpect(status().isCreated())
            .andExpect(xpath("/User/username").string("rutgo"))
            .andExpect(xpath("/User/password").string("1234"));
    }
```

- 이때 XML에 대한 viewResolverConverter가 없어서 에러가 발생한다.

```java
@ConditionalOnClass(XmlMapper.class)
```

- 여기서 XmlMapper가 클래스 패스에 추가되어 있지 않기 때문이다. 따라서 추가하기 위해 의존성을 추가해야 한다.

```groovy
implementation 'com.fasterxml.jackson.dataformat:jackson-dataformat-xml'
```

### 스프링 부트 웹 MVC 4부 : 정적 리소스 지원

- 정적 리소스라는 클라이언트에서 요청을 보냈을때 그냥 리소스를 건네주면 되는 것이다.
- 이미 만들어져있는 리소스가 있다고 생각하면 좋다.

- 정적 리소스 맵핑 “/**”
    - 기본 리소스 위치
        - classpath:/static
        - classpath:/public
        - classpath:/resources/
        - classpath:/META-INF/resources
    - 예) “/hello.html” => /static/hello.html

    ```html
    <!doctype html>
    <html lang="ko">
    <head>
        <meta charset="UTF-8">
        <meta name="viewport"
              content="width=device-width, user-scalable=no, initial-scale=1.0, maximum-scale=1.0, minimum-scale=1.0">
        <meta http-equiv="X-UA-Compatible" content="ie=edge">
        <title>hello</title>
    </head>
    <body>
    Hello
    </body>
    </html>
    ```

- [http://localhost:8080/hello.html](http://localhost:8080/hello.html) 을 들어가면 화면을 볼수 있다.

- 이런 기능을 제공하는 것이 `ResourceHttpRequestHandler`
- Last-Modified 헤더를 보고 304 응답을 보냄.
    - if-Modified-Since라는 것이 최근에 변경된 시간이다.  이것과 시간이 다르면 304를 던지고 새로운 리소스를 서버가 건내주지 않는다.
- 맵핍설정 변경

```properties
spring.mvc.static-path-pattern=/static/**
```

- 이렇게 하면 [http://localhost:8080/static/hello.html](http://localhost:8080/static/hello.html`)로 요청해야 한다.
- spring.mvc.static-locations: 리소스 찾을 위치 변경 가능
- WebMvcConfigurer의 addRersourceHandlers로 커스터마이징 할 수 있음

```java
@Override
public void addResourceHandlers(ResourceHandlerRegistry registry) {
	registry.addResourceHandler("/m/**")
	.addResourceLocations("classpath:/m/")
		.setCachePeriod(20);
}
```

### 스프링 웹 MVC 5부: 웹JAR

- 웹JAR는 클라이언트에서 사용하는 라이브러리를 jar파일로 추가할 수 있다.
- 웹 JAR들에 있는 자원들을 참조할 수있다.
- 웹JAR 맵핑 “/webjars/**”
- 버전 생략하고 사용하려면
    - webjars-locator-core 의존성 추가
    - 이게 가능한 것은 resource chaining으로 가능한 것이다.

```groovy
implementation (group: 'org.webjars.bower', name: 'jquery', version: '3.3.1')
```

```html
<script src="/webjars/jquery/3.3.1/dist/jquery.min.js"></script>
<script>
    $(function() {
        alert("ready!");
    })
</script>
```

### 스프링 웹 MVC 6부: index 페이지와 파비콘

- 웰컴 페이지(`/`)를 보여주는 방법이 있는데, 정적 페이지로 보여주는 방법은 `index.html`을 만드는 것이다.
- `index.html` 없이 `/`을 호출하면 에러 페이지를 보여준다. 이때 에러페이지는 톰켓이 만들어주는 것이 아니라 스프링 부트의 에러 페이지다. 에러 핸들러가 존재하는데, 에러 핸들러를 커스텀해서 자신만의 에러 페이지를 보여줄수도 있다.
- 파비콘
    - static 폴더에 favicon.ico를 만들고 html에 추가하면 된다.
    - 파비콘이 안 바뀔 때?
    1. Type in www.yoursite.com/favicon.ico (or www.yoursite.com/apple-touch-icon.png, etc.)
    2. Push enter
    3. ctrl + f5
    4. Restart Browser (IE, Firefox)