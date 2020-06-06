# 0606 테스트, SpringMVC

## @SpringBootTest

모든 Bean을 호출한 뒤 @MockBean으로 선언된 컴포넌트만 mock으로 대체

### webEnviroment

```java
@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment./*서버 종류*/)
```

MOCK

디폴트 값!  
실제 서버를 띄우지 않음 Mock 서버 이용  
mockMvc 사용 시에만 가능한 환경  

RANDOM_PROT

임의 포트를 이용해 서버를 생성 후 실제 요청 실행

## 테스트 도구

### MockMvc

mock 서버를 이용해 가상의 요청을 실행

```java
@Autowired
MockMvc mockMvc;

@Test
public void hello() throws Exception {
		mockMvc.perform(get("/hello"))
				.andExpect(status().isOk())
			  .andExpect(content().string("hello"))
			  .andDo(print());
}
```

### TestRestTemplate

param은 getForObject() 메소드의  인자로 뒤에 추가

```java
@Autowired
TestRestTemplate template;

@Test
public void hello() {
		String result = template.getForObject("/hello", String.class);
		assertThat(result).isEqualTo("hello");
}
```

### WebTestClient

spring-boot-starter-webflux 의존성 추가 필요

**비동기 방식!**

```java
@Autowired
WebTestClient client;

@Test
public void hello() {
		client.get().uri("/hello")
		    .exchange()
		    .expectStatus().isOk()
		    .expectBody(String.class).isEqualTo("hello");
}
```

## Slice Test

모든 Bean을 호출하기 때문에 테스트가 무거워진다!  
계층별 테스트를 위한 Slice Test가 별도로 존재한다. 테스트의 최적화  
⇒ @WebMvcTest, @WebFluxTest .....

Slice Test 사용 시 추가적으로 다른 Bean을 사용하기 위해서는 MockBean을 추가하면 된다

## OutputCaptureRule

console에 찍히는 문자열을 확인할 수 있다.

```java
@Rule
OutputCaptureRule capture = new OutputCaptureRule();
~
//in test method
assertThat(capture.toString())
		.contains("example"); // 콘솔의 모든 문자를 가져와서 확인
```

## Spring MVC 확장

```java
@Configuration
//@EnableWebMvc 쓰지 마세요
public class Config implements WebMvcConfigurer {
~
```

기존의 WebMvcConfigurer를 상속받아야 Spring에서 제공하는 기본적인 설정을 사용할 수 있다.  
@EnableWebMvc을 사용하면 직접 다 설정해야 한다!

## Spring MVC / HttpMessageConverter, ViewResolver

```java
@Test
public void createUser_JSON() throws Exception {
    String userJson = "{\"username\":\"name\", \"password\":\"1234\"}";
    mockMvc.perform(post("/users/create")
        .contentType(MediaType.APPLICATION_JSON) // 서버가 받을 데이터 형식
        .accept(MediaType.APPLICATION_JSON) // view한테 보낼 데이터 형식
        .content(userJson)) // httpMessageConverter가 처리함
        .andExpect(status().isOk())
        .andExpect(jsonPath("$.username", is(equalTo("name"))))
        .andExpect(jsonPath("$.password", is(equalTo("1234"))));
}
```

Converter가 request를 object로 변환하거나 request header의 accpet에 맞게 object를 HTTP response로 변환한다.  
Converter가 변환한 리턴값을 이용해 Resolver가 뷰를 생성한다.

원하는 데이터 형식에 따라 의존성 추가가 필요할 수도 있다.

## 정적 리소스 자원

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
resource/static & template로 기본 설정되어있는 리소스 자원의 위치를 임의로 추가할 수 있다.