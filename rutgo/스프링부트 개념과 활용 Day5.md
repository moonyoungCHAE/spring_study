# 스프링부트 개념과 활용 - 백기선

### 스프링 데이터 JPA 소개

```java
implementation 'org.springframework.boot:spring-boot-starter-data-jpa'
```

- ORM(Object-Relational Mapping)
    - 객체와 관계를 맵핑할때 발생하는 개념적 불일치를 해결하는 프레임 워크
    - 예제
        - 객체는 크기가 다양한데, 테이블은 크기가 한정적이다.
        - 테이블은 상속이 없는데, 객체는 상속이 있다.
        - 테이블에서는 식별자가 쉬운데, 객체는 HashCode 및 equlesTo
- 즉, JPA는 ORM은 자바를 위한 표준
- ORM이 대부분 하이버네이트 기준으로 되어 있다 보니, JPA는 하이버네이트 기준으로되어 있다.
- 스프링 데이터 JPA
    - JPA에 대한 표준 스펙을 쉽게 사용할 수 있도록 데이터로 추상화 시켜놓은 것이다.
    - SDJ → JPA → Hibernate → Datasoruce

### Spring-Data JPA 연동

- Repository와 관련된 Bean들만 등록해서 테스트를 하는 것이 `@DataJpaTest`
- `@dataJpaTest`를 위해서는 임베디드 DB가 있어야 한다.
    - 예를들어 H2

```java
spring.jpa.properties.hibernate.jdbc.non_contextual_creation=true
```

```java
import java.sql.Connection;
import java.sql.DatabaseMetaData;
import java.sql.SQLException;

import javax.sql.DataSource;

import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.autoconfigure.orm.jpa.DataJpaTest;
import org.springframework.jdbc.core.JdbcTemplate;
import org.springframework.test.context.junit4.SpringRunner;

@RunWith(SpringRunner.class)
@DataJpaTest // 슬라이싱 테스트
public class AccountRepositoryTest {
    @Autowired
    DataSource dataSource;

    @Autowired
    JdbcTemplate jdbcTemplate;

    @Autowired
    AccountRepository accountRepository;

    @Test
    public void di() throws SQLException {
        try (Connection connection = dataSource.getConnection()) {
            DatabaseMetaData metaData = connection.getMetaData();
            System.out.println(metaData.getURL());
            System.out.println(metaData.getDriverName());
            System.out.println(metaData.getUserName());
        }
    }
}
```

- 위 코드로 테스트 코드가 H2 데이터베이스를 돌아가고 있다는걸 알고 있다.
- 그리고 `SpringBootTest`를 이용하면 실제 DB가 돌아간다.
- 따라서 테스트용 DB는 인메모리 DB로 하는 것이 좋다.

```java
@SpringBootTest(value = {"application.properties "}) // 슬라이싱 테스트
```

- 이런식으로 테스트 DB만 다르게 줄 수 있다.
- 중간에 쿼리를 확인할 수 있고, 또한 Optional을 사용해서 반환할수도 있다.(find 메서드)
- 쿼리 메서드도 지원한다.

```java
package com.inflearn.springboot.account;

import java.util.Optional;

import org.springframework.data.jpa.repository.JpaRepository;

public interface AccountRepository extends JpaRepository<Account, Long> {
    Optional<Account> findByUsername(String username);
}
```

```java
package com.inflearn.springboot.account;

import static org.assertj.core.api.Assertions.*;

import java.sql.SQLException;

import javax.sql.DataSource;

import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.autoconfigure.orm.jpa.DataJpaTest;
import org.springframework.jdbc.core.JdbcTemplate;
import org.springframework.test.context.junit4.SpringRunner;

@RunWith(SpringRunner.class)
@DataJpaTest
public class AccountRepositoryTest {
    @Autowired
    DataSource dataSource;

    @Autowired
    JdbcTemplate jdbcTemplate;

    @Autowired
    AccountRepository accountRepository;

    @Test
    public void di() throws SQLException {
        Account account = new Account();
        account.setUsername("rutgo");
        account.setPassword("pass");

        Account newAccount = accountRepository.save(account);
        assertThat(newAccount).isNotNull();

        Account existingAccount = accountRepository.findByUsername(newAccount.getUsername());
        assertThat(existingAccount).isNotNull();

        Account nonExistingAccount = accountRepository.findByUsername("brown");
        assertThat(nonExistingAccount).isNull();
    }
}
```

- Optional 테스트

```java
import static org.assertj.core.api.Assertions.*;

import java.sql.SQLException;
import java.util.Optional;

import javax.sql.DataSource;

import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.autoconfigure.orm.jpa.DataJpaTest;
import org.springframework.jdbc.core.JdbcTemplate;
import org.springframework.test.context.junit4.SpringRunner;

@RunWith(SpringRunner.class)
@DataJpaTest
public class AccountRepositoryTest {
    @Autowired
    DataSource dataSource;

    @Autowired
    JdbcTemplate jdbcTemplate;

    @Autowired
    AccountRepository accountRepository;

    @Test
    public void di() throws SQLException {
        Account account = new Account();
        account.setUsername("rutgo");
        account.setPassword("pass");

        Account newAccount = accountRepository.save(account);
        assertThat(newAccount).isNotNull();

        Optional<Account> existingAccount = accountRepository.findByUsername(newAccount.getUsername());
        assertThat(existingAccount).isNotEmpty();

        Optional<Account> nonExistingAccount = accountRepository.findByUsername("brown");
        assertThat(nonExistingAccount).isEmpty();
    }
}
```

### 데이터베이스 초기화

- JPA를 사용한 데이터베이스 초기화

```java
spring.jpa.hibernate.ddl-auto=create // 실행할때마다 새로 생성

none: 아무것도 실행하지 않는다 (대부분의 DB에서 기본값이다)
create-drop: SessionFactory가 시작될 때 drop및 생성을 실행하고, SessionFactory가 종료될 때 drop을 실행한다 (in-memory DB의 경우 기본값이다)
create: SessionFactory가 시작될 때 데이터베이스 drop을 실행하고 생성된 DDL을 실행한다
update: 변경된 스키마를 적용한다
validate: 변경된 스키마가 있다면 변경점을 출력하고 애플리케이션을 종료한다
```

```java
spring.jpa.generate-ddl=true // 기본값이 false인데, ddl을 사용하기 위해 true로 변경
```

```java
spring.jpa.show-sql=true // 스키마가 생성되는것을 볼 수 있음
```

- 운영 DB일때

```java
spring.jpa.hibernate.ddl-auto=validate
spring.jpa.generate-ddl=false
```

- hibernate.ddl-auto가 활성화 되어있으면 generate-dll 설정은 무시함

- schema.sql을 실행한 후에 data.sql을 실행한다.
- sechema-${platform}.sql
- data-${platform}.sql
- ${platform} 값은 sprung.datasource.platform 으로 설정 가능

### 데이터베이스 마이그레이션

- Flyway, Liquibase
- Flyway는 기본적으로 SQL을 사용함.
- 이번 예제는 Flyway를 사용할 것임

```java
implementation 'org.flywaydb:flyway-core'
```

- V1__init.sql

```java
drop table if exists account;
drop sequence if exists hibernate_sequence;
create sequence hibernate_sequence start with 1 increment by 1;
create table account (id bigint not null, email varchar(255), password varchar(255));
```

- 마이그레이션 디렉토리
    - db/migration 또는 db/migration/{vendor}
    - spring.flyway.locations 변경 가능
- 마이그레이션 파일 이름
    - V숫자__이름.sql
    - V는 꼭 대문자로
    - 숫자는 순차적으로 (타입스탬프 권장)
    - 숫자와 이름 사이에 언더바 두 개.
    - 이름은 가능한 서술적으로.

- 데이터 마이그레이션을 왜 써야 하는가?
- 그리고 SQL이 변경될때마다 새로운 파일을 계속 만들어 줘야 하는데 이게 좋은것인가