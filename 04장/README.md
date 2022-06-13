# Chapter 04

## 프로젝트의 와이어프레임

- 웹 프로젝트를 구성할 때는 와이어프레임(화면 설계서)을 제작하고 진행하는 것이 좋다.
- 화면의 URI와 전달되는 파라미터 등을 미리 결정할 수 있다.
- DB 설계에 필요한 칼럼들을 미리 파악하는데 도움이 된다.

![image](https://user-images.githubusercontent.com/43159295/173421722-60e69a33-897f-4ec1-8b4d-72b6f07712c5.png)

화면을 먼저 만들면 다음과 같은 상세 기능도 만들기 편하다.

| 기능 | URL                 | GET/POST | 기능             | Redirect URL    |
| ---- | ------------------- | -------- | ---------------- | --------------- |
| 목록 | /guestbook/list     | GET      | 목록/페이징/검색 |                 |
| 등록 | /guestbook/register | POST     | 등록 처리        | /guestbook/list |

프로젝트의 구조

- 계층형 아키텍처 구조
  [계층형 아키텍처 구조](https://jojoldu.tistory.com/603)
  1. 관심 범위 축소 (관심사 분리)
  2. 모듈 교체의 용이성
  3. 좀 더 용이한 테스트

![image](https://user-images.githubusercontent.com/43159295/173421870-d4aee002-b855-48c7-acdd-38d35ce343f1.png)

DTO를 통한 데이터 전달

- 엔터티 객체를 영속 계층(Persistence Layer = Repository 계층) 바깥쪽에서 사용하기 보다는 DTO를 사용하는게 좋다.
- DTO의 목적 자체가 데이터의 전달이므로 읽고, 쓰는 것이 모두 허용되고 일회용이다.
- [https://tecoble.techcourse.co.kr/post/2020-08-31-dto-vs-entity/](https://tecoble.techcourse.co.kr/post/2020-08-31-dto-vs-entity/)

![image](https://user-images.githubusercontent.com/43159295/173421938-2fc8bbed-885b-4826-8c65-2e176a53011b.png)

### 자동으로 처리되는 날짜, 시간 설정

- `@MappedSuperclass` 어노테이션을 사용하면 테이블로 생성되지 않는다.
- JPA에서는 엔터티 객체의 변화를 감지하는 리스너가 있다.
- JPA 내부에서 엔터티 객체가 생성, 변경되는 것을 감지하는 역할을 하는 `AuditingEntityListener`
- `AuditingEntityListener` 를 활성화시키기 위해 `@EnableJpaAuditing` 설정을 추가해야 함

```java
@MappedSuperclass
@EntityListeners(value = {AuditingEntityListener.class})
@Getter
public class BaseEntity {

    @CreatedDate
    @Column(name = "regdate", updatable = false)
    private LocalDateTime regDate;

    @LastModifiedDate
    @Column(name = "moddate")
    private LocalDateTime modDate;

}

@Entity
@Getter
@Builder
@AllArgsConstructor
@NoArgsConstructor
@ToString
public class Guestbook extends BaseEntity {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long gno;

    @Column(length = 100, nullable = false)
    private String title;

    @Column(length = 1500, nullable = false)
    private String content;

    @Column(length = 50, nullable = false)
    private String writer;

    public void changeTitle(String title) {
        this.title = title;
    }

    public void changeContent(String content) {
        this.content = content;
    }
}

@SpringBootApplication
@EnableJpaAuditing
public class GuestbookApplication {

    public static void main(String[] args) {
        SpringApplication.run(GuestbookApplication.class, args);
    }

}
```

### 동적 쿼리 처리를 위한 Querydsl 설정

- 복잡한 조합을 이용하는 경우의 수가 많은 상황에서는 동적으로 쿼리를 생성해서 처리할 수 있는 기능이 필요
- Querydsl을 이용하면 복잡한 검색조건이나 조인, 서브 쿼리 등의 기능 구현 가능
- 엔터티 클래스를 ‘Q도메인’으로 바꿔서 이용해야 함

gradle 5.0 이상부터 다음과 같이 구성

- build.gradle

  [https://www.inflearn.com/questions/355723](https://www.inflearn.com/questions/355723)

  ```java
  buildscript {
      ext {
          queryDslVersion = "5.0.0"
      }
  }

  plugins {
      id 'org.springframework.boot' version '2.6.4'
      id 'io.spring.dependency-management' version '1.0.11.RELEASE'
      id 'java'
  	    id 'com.ewerk.gradle.plugins.querydsl' version '1.0.10' // 추가
  }

  group = 'org.zerock'
  version = '0.0.1-SNAPSHOT'
  sourceCompatibility = '11'

  configurations {
      compileOnly {
          extendsFrom annotationProcessor
      }
  }

  repositories {
      mavenCentral()
  }

  dependencies {
      implementation 'org.springframework.boot:spring-boot-starter-data-jpa'
      implementation 'org.springframework.boot:spring-boot-starter-thymeleaf'
      implementation 'org.springframework.boot:spring-boot-starter-web'
      compileOnly 'org.projectlombok:lombok'

      //querydsl 추가
      implementation "com.querydsl:querydsl-jpa:${queryDslVersion}"
      implementation "com.querydsl:querydsl-apt:${queryDslVersion}"

      developmentOnly 'org.springframework.boot:spring-boot-devtools'
      annotationProcessor 'org.projectlombok:lombok'
      testImplementation 'org.springframework.boot:spring-boot-starter-test'
      // https://mvnrepository.com/artifact/org.mariadb.jdbc/mariadb-java-client
      implementation group: 'org.mariadb.jdbc', name: 'mariadb-java-client', version: '3.0.3'
      // https://mvnrepository.com/artifact/org.thymeleaf.extras/thymeleaf-extras-java8time
      implementation group: 'org.thymeleaf.extras', name: 'thymeleaf-extras-java8time', version: '3.0.4.RELEASE'
  }

  tasks.named('test') {
      useJUnitPlatform()
  }

  def querydslDir = "$buildDir/generated/querydsl"

  querydsl {
      jpa = true
      querydslSourcesDir = querydslDir
  }

  sourceSets {
      main.java.srcDir querydslDir
  }

  compileQuerydsl {
      options.annotationProcessorPath = configurations.querydsl
  }

  configurations {  // 책이랑 다름
      compileOnly {
          extendsFrom annotationProcessor
      }
      querydsl.extendsFrom compileClasspath
  }
  ```

- Q도메인 생성 위치
  자동으로 생성됨

![image](https://user-images.githubusercontent.com/43159295/173422050-7c69bf47-a3e7-40fa-a634-53c8643c3aa5.png)

### Querydsl 사용법

- Querydsl을 이용하기 위해서 `QuerydslPredicateExecutor` 인터페이스를 상속한다

```java
public interface GuestbookRepository extends JpaRepository<Guestbook, Long>,
	QuerydslPredicateExecutor<Guestbook> {
}
```

- Querydsl의 사용법

  - `BooleanBuilder`를 생성한다.
  - 조건에 맞는 구문은 `Predicate`타입의 함수를 생성한다.
  - [http://querydsl.com/static/querydsl/4.1.4/apidocs/com/querydsl/core/types/Predicate.html](http://querydsl.com/static/querydsl/4.1.4/apidocs/com/querydsl/core/types/Predicate.html)
  - `BooleanBuilder`에 작성된 `Predicate`를 추가하고 실행한다.

- Querydsl 테스트

  1. 단일 항목 검색 테스트

  ```java
  @Test
  void testQuery1() {
      Pageable pageable = PageRequest.of(0, 10, Sort.by("gno").descending());

      QGuestbook qGuestbook = QGuestbook.guestbook;

      String keyword = "1";

      BooleanBuilder builder = new BooleanBuilder();

      BooleanExpression expression = qGuestbook.title.contains(keyword);

      builder.and(expression);

      Page<Guestbook> result = guestbookRepository.findAll(builder, pageable);

      result.stream().forEach(guestbook -> {
          System.out.println(guestbook);
      });
  }

  ```

  1. 다중 항목 검색 테스트
