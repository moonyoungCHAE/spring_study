# 스프링부트 개념과 활용 - 백기선

### Thymeleaf

- 템플릿 엔진은 view만 만드는데 쓰는게 아니다. 주로 View를 만드는데 사용한다.
- 코드 제네레이션, 이메일 템플릿 등에서 사용한다.
- 정적 컨텐츠를 사용하기 위해 사용한다.
- 템플릿 엔진의 종류
    - FreeMarker
    - Groovy
    - Thymeleaf
        - [1시간 같은 5분]([https://www.thymeleaf.org/doc/articles/sayhelloextendingthymeleaf5minutes.html](https://www.thymeleaf.org/doc/articles/standarddialect5minutes.html))
    - Mustache
- JSP를 권장 하지 않음
    - 자동설정을 지원하지 않음.
    - Jar는 사용하지 못함. War만 가능
    - Undertow는 JSP를 지원하지 않음.

- Thymeleaf 사용하기
    - 의존성추가

```java
implementation 'org.springframework.boot:spring-boot-starter-thymeleaf'
```

- 기본적으로 resources → template에서 찾게 됨

```java
import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.web.bind.annotation.RequestMapping;

@Controller
public class SampleController {

    @RequestMapping("/hello")
    public String hello(Model model){
        model.addAttribute("name", "rutgo"); // view에게 전달할 값, 맵이라고 생각하면 됨.
        return "hello";
    }
}
```

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
@WebMvcTest(SampleController.class)
public class SampleControllerTest {
    @Autowired
    MockMvc mockMvc;

    @Test
    public void hello() throws Exception {
        // 요청 "/hello"
        // 응답 모델 -> name : "rutgo"
        // 응답 뷰 -> "hello"

        mockMvc.perform(get("/hello"))
            .andExpect(status().isOk())
            .andExpect(view().name("hello"))
            .andExpect(model().attribute("name", "rutgo"));
    }
}
```

```java
<!doctype html>
<html lang="ko" xmlns:th="http://www.w3.org/1999/xhtml">
<head>
    <meta charset="UTF-8">
    <meta name="viewport"
          content="width=device-width, user-scalable=no, initial-scale=1.0, maximum-scale=1.0, minimum-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>Document</title>

</head>
<body>
    <span th:text="${name}">Name</span>
</body>
</html>
```

- MockMvc는 타임리프 엔진은 컨트롤러에 독립적인 엔진이기 때문에 body에서 본문을 확인할 수 있다.
- `Name`이라는 값을 주는 이유가 값이 넘어 오지 않으면 Name을 출력하고, 값이 넘어오면 넘어오는 값을 주라는 이유

### HtmlUnit

```java
testImplementation 'org.seleniumhq.selenium:htmlunit-driver'
    testImplementation 'net.sourceforge.htmlunit:htmlunit'
```

- html 단위 테스트 툴
- webClient로 사용하는데, htmlPage를 여러 방식으로 가져와서 테스트를 도와주는 도구이다.
- 이때 webClient는 Webflux가 아니다.

```java
import static org.assertj.core.api.Assertions.*;

import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.autoconfigure.web.servlet.WebMvcTest;
import org.springframework.test.context.junit4.SpringRunner;

import com.gargoylesoftware.htmlunit.WebClient;
import com.gargoylesoftware.htmlunit.html.HtmlHeading1;
import com.gargoylesoftware.htmlunit.html.HtmlPage;

@RunWith(SpringRunner.class)
@WebMvcTest(SampleController.class)
public class SampleControllerTest {
    @Autowired
    WebClient webClient;

    @Test
    public void hello() throws Exception {
        HtmlPage htmlPage = webClient.getPage("/hello");
        HtmlHeading1 h1 = htmlPage.getFirstByXPath("//h1");
        assertThat(h1.getTextContent()).isEqualTo("rutgo");
    }
}
```

### ExceptionHandler

- `BasicErrorController`가 HTML과 JSON응답을 지원한다.

```java
package com.inflearn.springboot.controller;

import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.ExceptionHandler;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class SampleController {
    @GetMapping("/hello")
    public String hello() {
        throw new SampleException("No Hello");
    }

    @ExceptionHandler(SampleException.class)
    public AppError sampleError(SampleException e){
        AppError appError = new AppError();
        appError.setReaons("error.app.key");
        appError.setMessge(e.getMessage());

        return appError;
    }

}
```

```java
package com.inflearn.springboot.controller;

public class AppError {
    private String messge;
    private String reaons;

    public String getMessge() {
        return messge;
    }

    public void setMessge(String messge) {
        this.messge = messge;
    }

    public String getReaons() {
        return reaons;
    }

    public void setReaons(String reaons) {
        this.reaons = reaons;
    }
}
```

```java
package com.inflearn.springboot.controller;

public class SampleException extends RuntimeException{
    public SampleException(String message) {
        super(message);
    }
}
```

- 커스텀 에러 템플릿
    - static → error → html 파일
    - 이때 html 파일이 상태코드 값과 똑같아야 한다. Ex) 4xx, 400, 5xx, 500....
    - 해당 status code가 발생했을 경우 해당 html 파일을 불러온다.
- ErrorViewResolver를 이용해 커스텀 에러 템플릿을 커스텀 할 수 있다.

### Spring HATEOAS

```java
implementation 'org.springframework.boot:spring-boot-starter-hateoas'
```

- 리소스를 제공할때, 연관된 링크들을 같이 제공하고, 연관된 링크 정보를 바탕으로 리소스에 접근한다.
- Hypermedia As The Engine Of Application State
- **Rel**ation
- **H**ypertext **Ref**erence

- ObjectMapper 제공
    - Jackson
    - Jackson2ObjectMapperBuilder

```java
package com.inflearn.springboot.controller;

import static org.springframework.test.web.servlet.request.MockMvcRequestBuilders.*;
import static org.springframework.test.web.servlet.result.MockMvcResultHandlers.*;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.*;

import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.autoconfigure.web.servlet.WebMvcTest;
import org.springframework.test.context.junit4.SpringRunner;
import org.springframework.test.web.servlet.MockMvc;

@RunWith(SpringRunner.class)
@WebMvcTest(SampleController.class)
public class SampleControllerTest{
    @Autowired
    MockMvc mockMvc;

    @Test
    public void hello() throws Exception {
        mockMvc.perform(get("/hello"))
            .andExpect(status().isOk())
            .andExpect(jsonPath("$._links.self").exists())
            .andDo(print());
    }
}
```

```java
package com.inflearn.springboot.controller;

import static org.springframework.hateoas.mvc.ControllerLinkBuilder.*;

import org.springframework.hateoas.Resource;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class SampleController {
    @GetMapping("hello")
    public Resource<Hello> hello(){
        Hello hello = new Hello();
        hello.setPrefix("Hey,");
        hello.setName("rutgo");

        Resource<Hello> helloResource = new Resource<>(hello);
        helloResource.add(linkTo(methodOn(SampleController.class).hello()).withSelfRel());
        return helloResource;
    }
}
```

```java
package com.inflearn.springboot.controller;

public class Hello {
    private String prefix;

    private String name;

    public String getPrefix() {
        return prefix;
    }

    public void setPrefix(String prefix) {
        this.prefix = prefix;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    @Override
    public String toString() {
        return prefix + " " + name;
    }
}
```

- LinkDiscovers 제공
    - 클라이언트 쪽에서 링크 정보를 Rel 이름으로 찾을 때 사용할 수 있는 XPath 확장 클래스

### CORS

- Single-Origin Policy
- Cross-Origin Resource Sharing
- [링크]([https://velog.io/@wlsdud2194/cors](https://velog.io/@wlsdud2194/cors))
- Origin?
    - URI 스키마 (http, https)
    - hostname(localhost)
    - 포트(8080, 18080)

- Bean 설정을 Spring Mvc에 제공하는 `@CrossOrigin`을 사용하면 자동으로 해준다.
- 서버

```java
package com.inflearn.springboot.controller;

import org.springframework.web.bind.annotation.CrossOrigin;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class SampleController {
    @CrossOrigin(origins = "http://localhost:18080")
    @GetMapping("/hello")
    public String hello() {
        return "Hello";
    }
}
```

```java
package com.inflearn.springboot.controller;

public class Hello {
    private String prefix;

    private String name;

    public String getPrefix() {
        return prefix;
    }

    public void setPrefix(String prefix) {
        this.prefix = prefix;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    @Override
    public String toString() {
        return prefix + " " + name;
    }
}
```

- 클라이언트

```java
<!doctype html>
<html lang="ko">
<head>
    <meta charset="UTF-8">
    <meta name="viewport"
          content="width=device-width, user-scalable=no, initial-scale=1.0, maximum-scale=1.0, minimum-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>Document</title>
</head>
<body>
<script>
    fetch("http://localhost:8080/hello").then(
      text => {
        return text.text()
      }
    ).then(text => alert(text))
      .catch(error => alert(error));
</script>
</body>
</html>
```

```java
server.port=18080
```

- 가능하게 해기 위해서는

```java
package com.inflearn.springboot.controller;

import org.springframework.context.annotation.Configuration;
import org.springframework.web.servlet.config.annotation.CorsRegistry;
import org.springframework.web.servlet.config.annotation.WebMvcConfigurer;

@Configuration
public class WebConfig implements WebMvcConfigurer{
    @Override
    public void addCorsMappings(CorsRegistry registry) {
        registry.addMapping("/**")
            .allowedOrigins("http://localhost:18080");
    }
}
```

```java
@CrossOrigin(origins = "http://localhost:18080")
```

### 인메모리 데이터베이스

- 지원하는 인메모리 데이터베이스
    - H2(권장)
    - HSQL
    - Derby
- Spring-JDBC가 클래스패스에 있으면 자동 설정이 필요한 빈을 설정해 줍니다.
    - DataSoruce
    - JdbcTemplate

```java
package com.inflearn.springboot;

import java.sql.Connection;
import java.sql.Statement;

import javax.sql.DataSource;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.ApplicationArguments;
import org.springframework.boot.ApplicationRunner;

public class H2Runner implements ApplicationRunner {
    @Autowired
    DataSource dataSource;

    @Override
    public void run(ApplicationArguments args) throws Exception {
        try (Connection connection = dataSource.getConnection();
        ) {
            System.out.println(connection.getMetaData().getURL());
            System.out.println(connection.getMetaData().getUserName());

            Statement statement = connection.createStatement();
            String sql = "CREATE TABLE USER(ID INTEGER NOT NULL, name VARCHAR(255), PRIMARY KEY(id))";
            statement.executeUpdate(sql);
        }
    }
}
```

- H2 콘솔 사용 방법
    - Spring dev tools

    ```java
    spring.h2.console.enabled=true
    ```

```java
package com.inflearn.springboot;

import java.sql.Connection;
import java.sql.Statement;

import javax.sql.DataSource;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.ApplicationArguments;
import org.springframework.boot.ApplicationRunner;
import org.springframework.jdbc.core.JdbcTemplate;

public class H2Runner implements ApplicationRunner {
    @Autowired
    DataSource dataSource;

    @Autowired
    JdbcTemplate jdbcTemplate;

    @Override
    public void run(ApplicationArguments args) throws Exception {
        try (Connection connection = dataSource.getConnection();
        ) {
            System.out.println(connection.getMetaData().getURL());
            System.out.println(connection.getMetaData().getUserName());

            Statement statement = connection.createStatement();
            String sql = "CREATE TABLE USER(ID INTEGER NOT NULL, name VARCHAR(255), PRIMARY KEY(id))";
            statement.executeUpdate(sql);
        }
        jdbcTemplate.execute("INSERT INTO USER VALUES (1, 'rutgo')"); // 위에 있는 코드를 아래와 같이 한 줄로 줄일수 있다.
    }
}
```

### MySQL 설정하기

- DBCP(DB Connection Pool)
    - DBCP에서 Connection을 만드는 과정이 여러가지 과정을 거치게 되는데, 미리 Connection을 여러개 만들어 놓고, 사용하는 것이다. 또한, DB에 대한 여러가지 설정을 할 수 있다.
    - 기본값은 10개이다.
    - 커넥션이 많다고 해서 다 한번에 실행할 수 있는 것은 아니다. 한번에 커넥션을 수행할 수 있는 것은 서버 코어 수다.
    - HikariCP가 기본값이다.
    - JDBC에서 문서에서 기본값들을 확인할 수 있다.

```java
spring.datasource.DBCPName.maximum-poop-size=4;
```

- MySQL 사용

```java
implementation 'mysql:mysql-connector-java'
```

```java
spring.datasource.url=
spring.datasource.username=
spring.datasource.password=
```

- 특정 버전 이상에서 SSL을 권장하는데, 이때 url에 쿼리에다가 useSSL=false 또는 useSSL=true and provide truststore을 사용하면 된다.