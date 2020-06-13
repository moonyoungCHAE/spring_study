# 스프링부트 개념과 활용 - 백기선

### 스프링 스큐리티

```java
package com.inflearn.springboot;

import org.springframework.context.annotation.Configuration;
import org.springframework.web.servlet.config.annotation.ViewControllerRegistry;
import org.springframework.web.servlet.config.annotation.WebMvcConfigurer;

@Configuration
public class webConfig implements WebMvcConfigurer {
    @Override
    public void addViewControllers(ViewControllerRegistry registry) {
        registry.addViewController("/hello").setViewName("hello");
    }
}
```

- 정적 페이지를 띄어줄때, Controller를 사용하지 않고 위와 같은 코드로도 사용이 가능하다.
- index.html

```java
<!doctype html>
<html lang="ko">
<head>
    <meta charset="UTF-8">
    <meta name="viewport"
          content="width=device-width, user-scalable=no, initial-scale=1.0, maximum-scale=1.0, minimum-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>welcome</title>
</head>
<body>
<a href="/hello">hello</a>
<a href="/my">my page</a>
</body>
</html>
```

- basic Authenticate를 인증하라고 하며,
- form Authenticate도 포함
- Accept에 따라 달라집니다.

```java
package com.inflearn.springboot;

import java.util.Arrays;
import java.util.Collection;
import java.util.Optional;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.security.core.GrantedAuthority;
import org.springframework.security.core.authority.SimpleGrantedAuthority;
import org.springframework.security.core.userdetails.User;
import org.springframework.security.core.userdetails.UserDetails;
import org.springframework.security.core.userdetails.UserDetailsService;
import org.springframework.security.core.userdetails.UsernameNotFoundException;
import org.springframework.security.crypto.password.PasswordEncoder;
import org.springframework.stereotype.Service;

@Service
public class AccountService implements UserDetailsService {
    @Autowired
    private AccountRepository accountRepository;

    @Autowired
    private PasswordEncoder passwordEncoder;

    public Account createAccount(String username, String password){
        Account account = new Account();
        account.setUsername(username);
        account.setPassword(passwordEncoder.encode(password));
        return accountRepository.save(account);
    }

    @Override
    public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {
        Optional<Account> byUsername = accountRepository.findByUsername(username);
        Account account = byUsername.orElseThrow(() -> new UsernameNotFoundException(""));
        return new User(account.getUsername(), account.getPassword(), authorities());
    }

    private Collection<? extends GrantedAuthority> authorities() {
        return Arrays.asList(new SimpleGrantedAuthority("ROLE_USER"));
    }
}
```

```
@Configuration
public class WebSecurityConfig extends WebSecurityConfigurerAdapter {
   @Override
   protected void configure(HttpSecurity http) throws Exception {
       http.authorizeRequests()
               .antMatchers("/", "/hello").permitAll()
               .anyRequest().authenticated()
               .and()
           .formLogin()
               .and()
           .httpBasic();
   }
}
```

- `@WithMockUser`를 사용하면, Security에서 MockUser를 통해 접근 가능합니다.

```java
package com.inflearn.springboot;

import static org.springframework.test.web.servlet.request.MockMvcRequestBuilders.*;
import static org.springframework.test.web.servlet.result.MockMvcResultHandlers.*;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.*;

import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.autoconfigure.web.servlet.WebMvcTest;
import org.springframework.http.MediaType;
import org.springframework.security.test.context.support.WithMockUser;
import org.springframework.test.context.junit4.SpringRunner;
import org.springframework.test.web.servlet.MockMvc;

@RunWith(SpringRunner.class)
@WebMvcTest(HomeController.class)
public class HomeControllerTest {
    @Autowired
    MockMvc mockMvc;

    @Test
    @WithMockUser
    public void hello() throws Exception {
        mockMvc.perform(get("/hello"))
            .andDo(print())
            .andExpect(status().isOk())
            .andExpect(view().name("hello"));
    }

    @Test
    public void hello_without_user() throws Exception {
        mockMvc.perform(get("/hello"))
            .andDo(print())
            .andExpect(status().isUnauthorized());
    }

    @Test
    public void my() throws Exception {
        mockMvc.perform(get("/my"))
            .andDo(print())
            .andExpect(status().isOk())
            .andExpect(view().name("my"));
    }
}
```

- PassWord Encoder

```java
@Bean
public PasswordEncoder passwordEncoder() {
	return NoOpPasswordEncoder.getInstance();
}
```

### 스프링 REST 클라이언트 1부 : RestTemplate과 WebClient

- RestTemplate
    - Blocking I/O 기반의 동기
    - Spring-web에 존재
- WebClinent
    - Non-Blocking I/O 기반의 비동기
    - spring-webflux에 존재

### 스프링 REST 클라이언트 2부 : 커스터 마이징

- RestTemplate
    - 기본적으로 HttpURLConnection을 사용

### 그 밖에 다양한 기술 연동

- 캐시
    - 메시징
    - Validation
    - 이메일 전송
    - JTA
    - 스프링 인티그레이션
    - 스프링 세션
    - JMX
    - 웹소켓
    - 코틀린

### 스프링 부트 Actuator 1부 : 소개

- 스프링 부트 운영중에 주시할 수 있는 여러가지 정보를 제공해주는데, 제공해주는 역할이 엔드 포인트이다.
- 다양한 엔드포인트를 제공한다.
- JMX 또는 HTTP를 통해 접근이 가능

```java
management.endpoints.enabled-by-default=
management.endpoint.info.enabled=true
```

### 스프링 부트 Actuator 2부 : JMX와 HTTP

- JConsole
- VisualVM
- `/actuator`

```java
management.endpoints.web.exposure.include=*
management.endpoints.web.exposure.exclude=env,beans
```

### 스프링 부트 Actuator 3부 : 스프링 부트 어드민

- [https://github.com/codecentric/spring-boot-admin](https://github.com/codecentric/spring-boot-admin)

```java
impletation de.codecentric:spring-boot-admin-starter-servce, version 2.0.1
impletation de.codecentric:spring-boot-admin-starter-client, version 2.0.1
```

- `@EnableAdminServer` 를 이용해 Client 에 대한 Server를 설정할 수 있다.
- 중요한 정보들이 많다 보니, 꼭 Security를 적용하는 것을 추천한다.