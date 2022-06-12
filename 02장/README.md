# Chapter 02

Spring Data JPA 라이브러리 추가시 오류 발생

- 오류 내용

  ```java
  Error starting ApplicationContext. To display the conditions report re-run your application with 'debug' enabled.
  2022-05-28 00:40:06.959 ERROR 10324 --- [  restartedMain] o.s.b.d.LoggingFailureAnalysisReporter   :

  ***************************
  APPLICATION FAILED TO START
  ***************************

  Description:

  Failed to configure a DataSource: 'url' attribute is not specified and no embedded datasource could be configured.

  Reason: Failed to determine a suitable driver class

  Action:

  Consider the following:
  	If you want an embedded database (H2, HSQL or Derby), please put it on the classpath.
  	If you have database settings to be loaded from a particular profile you may need to activate it (no profiles are currently active).

  Process finished with exit code 0
  ```

- 해결 방법
  datasource와 관련된 url 설정과 관련된 오류, 프로젝트에 JPA 라이브러리가 추가되었기 때문에 관련된 설정은 추가되었으나 구체적인 값이 지정되지 않아서 발생하는 문제이다.

  - MariaDB JDBC 드라이버 추가
  - 스프링 부트 프로젝트 내 MariaDB 설정
    의존성 추가

  ```java
  dependencies {
      ...
      // https://mvnrepository.com/artifact/org.mariadb.jdbc/mariadb-java-client
      implementation group: 'org.mariadb.jdbc', name: 'mariadb-java-client', version: '2.7.0'

  }
  ```

  [application.properties](http://application.properties) 설정

  ```java
  spring.datasource.driver-class-name=org.mariadb.jdbc.Driver
  spring.datasource.url=jdbc:mariadb://localhost:3307/bootex
  spring.datasource.username=bootuser
  spring.datasource.password=bootuser
  ```

![image](https://user-images.githubusercontent.com/43159295/173236628-68399852-6094-4e64-8840-4493f9a15b73.png)

HikariPool이란?

스프링 부트가 기본적으로 이용하는 커넥션 풀 (HickariCP)

[https://github.com/brettwooldridge/HikariCP](https://github.com/brettwooldridge/HikariCP)

![image](https://user-images.githubusercontent.com/43159295/173236310-de3e63c7-3330-4bf8-9e02-ec7a35410267.png)

### ORM

ORM : 객체지향 패러다임을 관계형 DB에 보존하는 기술, 객체지향 패러다임을 관계형 패러다임으로 매핑해주는 개념

[패러다임 불일치](https://jgrammer.tistory.com/76)

```java
// Member 클래스

public class Member {
	private String id;
	private String pw;
	private String name;
}
```

Member 테이블

| id   | String |
| ---- | ------ |
| pw   | String |
| name | String |

1. 클래스와 테이블은 유사하다
   1. 새로운 테이블에 칼럼을 정의하고 칼럼에 맞는 데이터 타입을 지정해서 데이터를 보관한다는 점이 유사
2. 인스턴스와 Row도 유사하다.
   1. 객체지향에서는 클래스에서 인스턴스를 생성해서 인스턴스라는 공간에 데이터를 보관
   2. 테이블에서는 하나의 Row에 데이터를 저장
3. 관계와 참조라는 의미도 유사하다.
   1. RDB는 테이블 사이의 관계를 통해서 구조적인 데이터를 표현한다.
   2. 객체지향에서는 참조를 통해서 관계를 맺고 있는지를 표현한다.

## JPA

JPA는 Java Persistence API의 약어로 ORM을 Java 언어에 맞게 사용하는 스펙이다.

JPA는 단순한 스펙이기 때문에 해당 스펙을 구현하는 구현체마다 이름이 다르다. 가장 유명한 것은 Hibernate 이다.

![image](https://user-images.githubusercontent.com/43159295/173236712-2b0e75bc-d38d-4735-8b5a-6e6ee336c369.png)

스프링 부트는 JPA의 구현체 중에서 Hibernate를 이용한다.

### Spring Data JPA

프로젝트 생성 시 추가한 Spring Data JPA는 Hibernate를 스프링 부트에서 쉽게 사용할 수 있는 추가적인 API들을 제공한다.

스프링 프레임워크 자체는 대부분의 다른 프레임워크와의 호환성을 위한 라이브러리를 제공하는데 Spring Data JPA 역시 그러한 예이다.

Spring Data JPA를 이용하면 다음과 같은 구성을 이용하게 된다.

![image](https://user-images.githubusercontent.com/43159295/173236807-43ad72d4-295f-4dd1-be7c-fd140dc66171.png)

### 엔터티 클래스

```java
@Entity
@Table(name = "tbl_memo")
@ToString
@Getter
@Builder
@AllArgsConstructor
@NoArgsConstructor
public class Memo {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long mno;

    @Column(length = 200, nullable = false)
    private String memoText;

}
```

`[application.properties](http://application.properties)` JPA 관련 설정 추가

```java
...
spring.jpa.hibernate.ddl-auto=update # update 설정
spring.jpa.properties.hibernate.format_sql=true # SQL 포맷팅
spring.jpa.show-sql=true # SQL 출력
```

- ddl-auto
  [https://smpark1020.tistory.com/140?category=857916](https://smpark1020.tistory.com/140?category=857916)
  프로젝트 실행 시에 자동으로 DDL(create, alter, drop 등)을 생성할 것인지를 결정하는 설정
  - create: drop → create
  - create-drop: drop → crate → drop
  - update: 데이터 변경시에 alter 실행, 테이블이 없으면 create 실행
  - validate: 엔터티와 테이블이 정상 매핑되어있는지만 확인

### JpaRepository

Spring Data JPA는 JPA의 구현체인 Hibernate를 이용하기 위한 여러 API를 제공한다. 그중에서 가장 많이 사용할 것이 바로 JpaRepository라는 인터페이스다.

![image](https://user-images.githubusercontent.com/43159295/173236827-3dffa969-7b79-4ea1-81d9-eb3f70b08838.png)

JpaRepository를 상속한 인터페이스 선언만으로도 자동으로 스프링 빈으로 등록된다.

```java
public interface MemoRepository extends JpaRepository<Memo, Long> {
}
```

Repository 인터페이스 테스트 시 자동으로 생성되는 Proxy 클래스

[https://velog.io/@prayme/Dynamic-Proxy와-Spring-Data-JPA](https://velog.io/@prayme/Dynamic-Proxy%EC%99%80-Spring-Data-JPA)

![image](https://user-images.githubusercontent.com/43159295/173236854-3420e34f-5cc3-46a8-8060-4238ec2dad4d.png)

- Repository Test

  ```java
  // @SpringBootTest
  @DataJpaTest
  @AutoConfigureTestDatabase(replace = AutoConfigureTestDatabase.Replace.NONE)
  class MemoRepositoryTests {

      @Autowired
      MemoRepository memoRepository;

      @Test
      void testClass() {
          System.out.println(memoRepository.getClass().getName());
      }

      @Test
      void testInsertDummies() {
          IntStream.rangeClosed(1, 100).forEach(i -> {
              Memo memo = Memo.builder().memoText("Sample..." + i).build();
              memoRepository.save(memo);
          });
      }

      @Test
      void testSelect() {
          Long mno = 100L;

          Optional<Memo> result = memoRepository.findById(mno);

          System.out.println("=====================");

          if (result.isPresent()) {
              Memo memo = result.get();
              System.out.println(memo);
          }
      }

      @Transactional
      @Test
      void testSelect2() {
          Long mno = 100L;

          Memo memo = memoRepository.getOne(mno);

          System.out.println("=====================");

          System.out.println(memo);
      }

      @Test
      void testUpdate() {
          Memo memo = Memo.builder().mno(100L).memoText("Update Text").build();
          System.out.println(memoRepository.save(memo));
      }

      @Test
      void testDelete() {
          Long mno = 100L;
          memoRepository.deleteById(mno);
      }

  }
  ```

### 페이징/정렬 처리

JPA는 내부적으로 Dialect 를 이용해서 각 데이터베이스에 맞는 쿼리를 처리한다.

Spring Data JPA에서 페이징 처리는 findAll() 메서드를 이용한다.

파라미터로 전달하는 Pageable 타입의 객체에 의해서 쿼리를 결정한다.

주의 사항은 리턴 타입을 Page<T> 타입으로 지정하는 경우에는 반드시 파라미터를 Pageable 타입을 이용해야 한다.

Page 타입은 목록만 가져오지 않고, 실제 페이지 처리에 필요한 전체 데이터의 개수를 가져오는 쿼리 역시 같이 처리한다.

### Pageable 인터페이스

Pageable 인터페이스는 페이지 처리에 필요한 정보를 전달하는 용도의 타입으로, 구현체로는PageRequest 클래스를 이용한다.

```java
@Test
void testPageDefault() {
    Pageable pageable = PageRequest.of(0, 10);
    Page<Memo> result = memoRepository.findAll(pageable);
    System.out.println(result);

    System.out.println("---------------------------");

    System.out.println("Total pages: " + result.getTotalPages());
    System.out.println("Total Count: " + result.getTotalElements());
    System.out.println("Page number: " + result.getNumber());
    System.out.println("Page size: " + result.getSize());
    System.out.println("has next page? " + result.hasNext());
    System.out.println("first page? " + result.isFirst());

    System.out.println("---------------------------");
    for (Memo memo : result.getContent()) {
        System.out.println(memo);
    }
}

@Test
void testSort() {
    Sort sort1 = Sort.by("mno").descending();
    Sort sort2 = Sort.by("memoText").ascending();
    Sort sortAll = sort1.and(sort2);
    Pageable pageable = PageRequest.of(0, 10, sortAll);
    Page<Memo> result = memoRepository.findAll(pageable);
    result.get().forEach(memo -> {
        System.out.println(memo);
    });
```

### 쿼리 메서드 기능과 @Query

특정한 범위의 Memo 객체를 검색하거나, like 처리가 필요한 경우, 여러 검색 조건이 필요한 경우 Spring Data JPA는 다음과 같은 방법을 제공한다.

- 쿼리 메서드 : 메서드의 이름 자체가 쿼리의 구문으로 처리되는 기능
- `@Query` : SQL과 유사하게 엔터티 클래스의 정보를 이용해서 쿼리를 작성하는 기능
- Querydsl 등의 동적 쿼리 처리 기능

### 쿼리 메서드

쿼리 메서드는 메서드의 이름 자체가 쿼리문이 되는 기능이다.

[https://docs.spring.io/spring-data/jpa/docs/current/reference/html/#appendix.query.method.subject](https://docs.spring.io/spring-data/jpa/docs/current/reference/html/#appendix.query.method.subject)

```java
public interface MemoRepository extends JpaRepository<Memo, Long> {
    List<Memo> findByMnoBetweenOrderByMnoDesc(Long from, Long to);

    Page<Memo> findByMnoBetween(Long from, Long to, Pageable pageable);

    void deleteMemoByMnoLessThan(Long num);
}
```

### @Query 어노테이션

조인이나 복잡한 조건을 처리해야 하는 경우에는 and, or 등이 사용된다.

일반적인 경우 간단한 처리에만 쿼리 메서드를 이용하고, @Query를 이용하는 경우가 더 많다.

@Query의 value는 JPQL로 작성한다. @Query를 이용해서는 다음과 같은 작업을 실행할 수 있다.

- 필요한 데이터만 선별적으로 추출하는 기능이 가능
- 데이터베이스에 맞는 순수한 SQL을 사용하는 기능
- insert, update, delete와 같은 select가 아닌 DML 등을 처리하는 기능 (@Modifying 사용)

JPQL은 테이블 대신 엔터티 클래스를 이용하고, 테이블의 칼럼 대신에 클래스에 선언된 필드를 이용해서 작성한다.

```java
@Query("select m from Memo m order by m.mno desc")
List<Memo> getListDesc();
```

### @Query의 파라미터 바인딩

where 구문과 그에 맞는 파라미터들은 다음과 같은 방식을 사용한다.

- ‘?1, ?2’ 와 1부터 시작하는 파라미터의 순서를 이용하는 방식
- ‘:xxx’ 와 같이 ‘:파라미터 이름’ 을 활용하는 방식
- ‘:#{}’ 과 같이 자바 빈 스타일을 이용하는 방식

```java
@Transactional
@Modifying
@Query("update Memo m set m.memoText = :memoText where m.mno = :mno ")
int updateMemoText(@Param("mno") Long mno, @Param("memoText") String memoText);

@Transactional
@Modifying
@Query("update Memo m set m.memoText = :#{#param.memoText} where m.mno = :#{#param.mno}")
int updateMemoText(@Param("param") Memo memo);
```

### @Query와 페이징 처리

Pageable 타입의 파라미터를 적용하면 페이징 처리와 정렬에 대한 부분을 작성하지 않아도 된다.

리턴 타입을 Page<T>로 지정하는 경우에는 count를 처리하는 쿼리를 적용할 수 있다.

따라서 @Query를 이용할 때는 별도의 countQuery라는 속성을 적용해주고 Pageable 타입의 파라미터를 전달하면 된다.

```java
@Query(value = "select m from Memo m where m.mno > :mno",
	    countQuery = "select count(m) from Memo m where m.mno > :mno")
Page<Memo> getListWithQuery(Long mno, Pageable pageable);
```

### Object[ ] 리턴

쿼리 메서드의 경우에는 엔터티 타입의 데이터만을 추출하지만, @Query를 이용하는 경우에는 현재 필요한 데이터만을 Object[ ]의 형태로 선별적으로 추출할 수 있다는 점이다.

JPQL을 이용할 때 join이나 group by 등을 이용하는 경우가 있는데, 이럴 때는 적당한 엔터티 타입이 존재하지 않는 경우가 많기 때문에 Object[ ] 타입을 리턴 타입으로 지정할 수 있다.

JPQL에서는 CURRENT_DATE, CURRENT_TIME, CURRENT_TIMESTAMP와 같은 구문을 통해서 현재 데이터베이스의 시간을 구할 수 있는데, 다음과 같은 형태가 된다.

```java
@Query(value = "select m.mno, m.memoText, CURRENT_DATE from Memo m where m.mno > :mno",
            countQuery = "select count(m) from Memo m where m.mno > :mno")
Page<Object[]> getListWithQueryObject(Long mno, Pageable pageable);
```

### Native SQL 처리

복잡한 JOIN 구문 등을 처리하기 위해 사용한다.
