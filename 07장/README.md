# Chapter 07

영화와 회원 사이에 다음 명제가 존재.

- 한 편의 영화는 여러 회원의 평가가 행해질 수 있다.
- 한 명의 회원은 여러 영화에 대해서 평점을 줄 수 있다.

## M:N(다대다) 관계의 특징

---

M:N 관계는 논리적인 설계와 실제 테이블 설계가 다르게 된다.

영화와 회원은 모두 독립적인 엔터티로 설계가 가능하다.

회원 입장에서 여러 영화를 평가한다는 가능하다.

영화 입장에서 여러 회원이 존재한다는 가능하다.

![image](https://user-images.githubusercontent.com/43159295/179396669-54718c33-6848-472e-acd0-39a7401e3f8b.png)

문제는 M:N 관계를 실제 테이블로 설계할 수 없다.

테이블은 고정된 개수의 칼럼을 가지고 있기 때문이다.

다음과 같이 테이블을 만들면 안된다.

![image](https://user-images.githubusercontent.com/43159295/179396678-b361c94b-87ed-41b7-aadb-7547a7473b2d.png)

M:N을 해결하기 위해서 실제 테이블 설계에서는 매핑 테이블을 사용한다.

![image](https://user-images.githubusercontent.com/43159295/179396691-fdfb125a-b66a-4f5d-a3ad-37a4e270ef99.png)

매핑 테이블의 특징은 다음과 같다.

- 매핑 테이블의 작성 이전에 다른 테이블들이 먼저 존재해야 한다.
- 매핑 테이블은 명사가 아닌 동사나 히스토리에 대한 데이터를 보관하는 용도이다.
- 매핑 테이블은 중간에서 양쪽의 PK를 참조하는 형태로 사용한다.

## JPA에서 M:N 처리

---

1. `@ManyToMany` 이용 (쓰면 안된다)
2. 별도의 엔터티 설계 후 `@ManyToOne` 사용

- `@ManyToMany` 경우 양방향 참조를 이용한다.
- 양방향 참조는 현재 메모리상의 엔터티 객체들의 상태와 데이터베이스의 상태를 동기화 시키는 것이 가장 중요하다.
- 하나의 객체를 수정하는 경우에 다른 객체의 상태를 매번 일치하도록 변경하는 작업은 쉽지 않다.
- 가능하면 `@ManyToOne`(단방향) 을 사용한다.

## 실전 예제

---

```java
public class Member extends BaseEntity {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long mid;
    private String email;
    private String pw;
    private String nickname;
}

public class Movie extends BaseEntity {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long mno;
    private String title;
}

public class Review extends BaseEntity {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long reviewnum;

    @ManyToOne(fetch = FetchType.LAZY)
    private Movie movie;

    @ManyToOne(fetch = FetchType.LAZY)
    private Member member;

    private int grade;

    private String text;

}
```

- Review 테이블을 매핑 테이블로 새로 만든다.
- 매핑 테이블은 동사나 히스토리를 의미하는 테이블이다.
    - 회원이 영화에 대해 평점을 준다. (평점을 준다는 행위가 매핑 테이블이 필요한 부분)
    - ManyToMany의 경우 관계를 설정할 수 있지만 두 엔터티 간의 추가적인 데이터를 기록할 수 없다.

## N + 1 문제

---

- 1번의 쿼리로 N 개의 데이터를 가져왔는데 N 개의 데이터를 처리하기 위해서 필요한 추가적인 쿼리가 각 N개에 대해서 수행되는 상황

```java
public class MovieImage {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long inum;
    private String uuid;
    private String imgName;
    private String path;

    @ManyToOne(fetch = FetchType.LAZY)
    private Movie movie;
}

public class Movie extends BaseEntity {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long mno;
    private String title;
}

```

### N + 1 문제가 발생하지 않는 경우

```java
@Query("SELECT m, mi, avg(COALESCE(r.grade, 0)), COUNT(DISTINCT r) FROM Movie m " +
        "LEFT OUTER JOIN MovieImage mi ON mi.movie = m " +
        "LEFT OUTER JOIN Review r ON r.movie = m GROUP BY m")
Page<Object[]> getListPage(Pageable pageable);

@Test
void testListPage() {
    PageRequest pageRequest = PageRequest.of(0, 10, Sort.by(Sort.Direction.DESC, "mno"));

    Page<Object[]> result = movieRepository.getListPage(pageRequest);

    for (Object[] objects : result.getContent()) {
        System.out.println(Arrays.toString(objects));
    }
}
```

sql 작동 코드

```sql
Hibernate: 
    select
        movie0_.mno as col_0_0_,
        movieimage1_.inum as col_1_0_,
        avg(coalesce(review2_.grade,0)) as col_2_0_,
        count(distinct review2_.reviewnum) as col_3_0_,
        movie0_.mno as mno1_1_0_,
        movieimage1_.inum as inum1_2_1_,
        movie0_.moddate as moddate2_1_0_,
        movie0_.reg_date as reg_date3_1_0_,
        movie0_.title as title4_1_0_,
        movieimage1_.img_name as img_name2_2_1_,
        movieimage1_.movie_mno as movie_mn5_2_1_,
        movieimage1_.path as path3_2_1_,
        movieimage1_.uuid as uuid4_2_1_ 
    from
        movie movie0_ 
    left outer join
        movie_image movieimage1_ 
            on (
                movieimage1_.movie_mno=movie0_.mno
            ) 
    left outer join
        review review2_ 
            on (
                review2_.movie_mno=movie0_.mno
            ) 
    group by
        movie0_.mno 
    order by
        movie0_.mno desc limit ?

Hibernate: 
    select
        count(movie0_.mno) as col_0_0_ 
    from
        movie movie0_ 
    left outer join
        movie_image movieimage1_ 
            on (
                movieimage1_.movie_mno=movie0_.mno
            ) 
    left outer join
        review review2_ 
            on (
                review2_.movie_mno=movie0_.mno
            ) 
    group by
        movie0_.mno
```

### N + 1 문제가 발생하는 경우

```java
@Query("SELECT m, MAX(mi), avg(COALESCE(r.grade, 0)), COUNT(DISTINCT r) FROM Movie m " +
        "LEFT OUTER JOIN MovieImage mi ON mi.movie = m " +
        "LEFT OUTER JOIN Review r ON r.movie = m GROUP BY m")
Page<Object[]> getListPage(Pageable pageable);

@Test
void testListPage() {
    PageRequest pageRequest = PageRequest.of(0, 10, Sort.by(Sort.Direction.DESC, "mno"));

    Page<Object[]> result = movieRepository.getListPage(pageRequest);

    for (Object[] objects : result.getContent()) {
        System.out.println(Arrays.toString(objects));
    }
}
```

sql 작동 코드

```sql
Hibernate: 
    select
        movie0_.mno as col_0_0_,
        max(movieimage1_.inum) as col_1_0_,
        avg(coalesce(review2_.grade,
        0)) as col_2_0_,
        count(distinct review2_.reviewnum) as col_3_0_,
        movie0_.mno as mno1_1_,
        movie0_.moddate as moddate2_1_,
        movie0_.reg_date as reg_date3_1_,
        movie0_.title as title4_1_ 
    from
        movie movie0_ 
    left outer join
        movie_image movieimage1_ 
            on (
                movieimage1_.movie_mno=movie0_.mno
            ) 
    left outer join
        review review2_ 
            on (
                review2_.movie_mno=movie0_.mno
            ) 
    group by
        movie0_.mno 
    order by
        movie0_.mno desc limit ?

Hibernate: 
    select
        movieimage0_.inum as inum1_2_0_,
        movieimage0_.img_name as img_name2_2_0_,
        movieimage0_.movie_mno as movie_mn5_2_0_,
        movieimage0_.path as path3_2_0_,
        movieimage0_.uuid as uuid4_2_0_ 
    from
        movie_image movieimage0_ 
    where
        movieimage0_.inum=?
Hibernate: 
    select
        movieimage0_.inum as inum1_2_0_,
        movieimage0_.img_name as img_name2_2_0_,
        movieimage0_.movie_mno as movie_mn5_2_0_,
        movieimage0_.path as path3_2_0_,
        movieimage0_.uuid as uuid4_2_0_ 
    from
        movie_image movieimage0_ 
    where
        movieimage0_.inum=?
Hibernate: 
    select
        movieimage0_.inum as inum1_2_0_,
        movieimage0_.img_name as img_name2_2_0_,
        movieimage0_.movie_mno as movie_mn5_2_0_,
        movieimage0_.path as path3_2_0_,
        movieimage0_.uuid as uuid4_2_0_ 
    from
        movie_image movieimage0_ 
    where
        movieimage0_.inum=?
Hibernate: 
    select
        movieimage0_.inum as inum1_2_0_,
        movieimage0_.img_name as img_name2_2_0_,
        movieimage0_.movie_mno as movie_mn5_2_0_,
        movieimage0_.path as path3_2_0_,
        movieimage0_.uuid as uuid4_2_0_ 
    from
        movie_image movieimage0_ 
    where
        movieimage0_.inum=?
Hibernate: 
    select
        movieimage0_.inum as inum1_2_0_,
        movieimage0_.img_name as img_name2_2_0_,
        movieimage0_.movie_mno as movie_mn5_2_0_,
        movieimage0_.path as path3_2_0_,
        movieimage0_.uuid as uuid4_2_0_ 
    from
        movie_image movieimage0_ 
    where
        movieimage0_.inum=?
Hibernate: 
    select
        movieimage0_.inum as inum1_2_0_,
        movieimage0_.img_name as img_name2_2_0_,
        movieimage0_.movie_mno as movie_mn5_2_0_,
        movieimage0_.path as path3_2_0_,
        movieimage0_.uuid as uuid4_2_0_ 
    from
        movie_image movieimage0_ 
    where
        movieimage0_.inum=?
Hibernate: 
    select
        movieimage0_.inum as inum1_2_0_,
        movieimage0_.img_name as img_name2_2_0_,
        movieimage0_.movie_mno as movie_mn5_2_0_,
        movieimage0_.path as path3_2_0_,
        movieimage0_.uuid as uuid4_2_0_ 
    from
        movie_image movieimage0_ 
    where
        movieimage0_.inum=?
Hibernate: 
    select
        movieimage0_.inum as inum1_2_0_,
        movieimage0_.img_name as img_name2_2_0_,
        movieimage0_.movie_mno as movie_mn5_2_0_,
        movieimage0_.path as path3_2_0_,
        movieimage0_.uuid as uuid4_2_0_ 
    from
        movie_image movieimage0_ 
    where
        movieimage0_.inum=?
Hibernate: 
    select
        movieimage0_.inum as inum1_2_0_,
        movieimage0_.img_name as img_name2_2_0_,
        movieimage0_.movie_mno as movie_mn5_2_0_,
        movieimage0_.path as path3_2_0_,
        movieimage0_.uuid as uuid4_2_0_ 
    from
        movie_image movieimage0_ 
    where
        movieimage0_.inum=?
Hibernate: 
    select
        movieimage0_.inum as inum1_2_0_,
        movieimage0_.img_name as img_name2_2_0_,
        movieimage0_.movie_mno as movie_mn5_2_0_,
        movieimage0_.path as path3_2_0_,
        movieimage0_.uuid as uuid4_2_0_ 
    from
        movie_image movieimage0_ 
    where
        movieimage0_.inum=?

Hibernate: 
    select
        count(movie0_.mno) as col_0_0_ 
    from
        movie movie0_ 
    left outer join
        movie_image movieimage1_ 
            on (
                movieimage1_.movie_mno=movie0_.mno
            ) 
    left outer join
        review review2_ 
            on (
                review2_.movie_mno=movie0_.mno
            ) 
    group by
        movie0_.mno
```
