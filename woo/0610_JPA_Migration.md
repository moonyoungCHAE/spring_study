# 0610 스프링 데이터 JPA / Migration

## ORM (객체 관계 매핑) / JPA (Java 영속성 API)

DB - Object를 매핑하는 인터페이스
JPA는 구현체! ⇒ 프레임워크

## Spring Data JPA

Spring Data에서 JPA를 이용해 쉽게 개발할 수 있도록 추상화 시킨 프레임워크

```java
@Entity 
@Getter @Setter //Lombook
public class Account {

    @Id 
    @GeneratedValue //(strategy = GenerationType.IDENTITY)
    private Long id;
    private String username;
    private String password;
}
```

```java
public interface AccountRepository extends JpaRepository<Account, Long> {
    Optional<Account> findByUsername(String username);
}
```

### @GeneratedValue(strategy = GenerationType.xxxx)

ID의 자동 생성 전략 선택 (IDENTITY, SEQUENCE, TABLE, AUTO)  
DB 종류에 따라 전략이 다르다.

TABLE은 DB에 키 생성에 대한 테이블을 만들어 DB 종류에 무관하게 사용할 수 있다.

AUTO는 DB 종류에 맞는 방식을 자동으로 채택하고 ID를 생성한다.

[Spring Data JPA 기본키 매핑하는 방법](https://ithub.tistory.com/24)

### 스키마 생성

```yaml
spring.jpa.hibernate.ddl-auto=update
```

@Entity에 따른 스키마 자동 생성 기능
- create : 시작 시 새로 생성
- create-drop : 시작 시 새로 생성 후 종료 시 삭제
- update : 기존 DB와 Entity를 비교해 기존 DB에 없는 컬럼 생성, 삭제는 하지 않음
- validate : 기존 DB와 Entity를 비교, 다르면 오류 발생

## 데이터베이스 마이그레이션

DB 변경 사항을 git처럼 로그화 시켜준다.

resource.db.migration 폴더에 V(n)__(name).sql의 형식으로 저장 (V1__init.sql)

V1 ~ n까지 DB 스키마의 변경 사항을 버전 별로 작성한다. 직접!

어플리케이션 실행 시 각 버전이 순차적으로 실행된다.

운영 중인 서비스에서 개발 DB와 서비스 DB의 차이가 발생했을 때, 순차적으로 DB 스키마 변경을 적용할 때 쓸 수 있을 것 같다.