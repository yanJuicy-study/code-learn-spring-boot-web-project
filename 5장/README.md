# chapter05

jpa의 관계도.

회원과 게시물의 관계에 대해 다음의 명제를 성립할 수 있다.

- 한 명의 회원은 여러 게시물을 작성할 수 있음
- 하나의 게시물은 한 명의 회원에 의해 작성됨

두 번째 명제가 거짓은 아니지만 이 명제 그대로 데이터베이스를 모델링하면 파국으로 치닫는다.

한 명의 회원에 의해 여러개의 게시물이 생성됨은

회원 테이블의 PK가 게시물 테이블의 FK로 쓰인다는 뜻이다.

이를 ERD로 나타내면 대충 이렇다.

<img width="703" alt="스크린샷 2022-06-20 오전 12 41 28" src="https://user-images.githubusercontent.com/94847468/174489172-d60de793-765d-4315-ae05-5211e8e18974.png">

게시물과 댓글 역시 다음의 명제가 성립된다.

- 하나의 게시물은 여러 개의 댓글을 가질 수 있다.

그럼 이에 대해서도 ERD로 나타내면

<img width="701" alt="스크린샷 2022-06-20 오전 12 42 01" src="https://user-images.githubusercontent.com/94847468/174489204-1992f0ca-6573-4f81-8347-a24229b3796a.png">

회원 테이블을 기준으로 게시물 테이블은 1:N이 되고,

게시물 테이블을 기준으로 댓글 테이블도 1:N이 된다.

이를 jpa에서는 FK를 사용하는 Entity가 PK를 사용하는 Entity를 참조하는 구조로 설계한다.

회원 Entity를 다음으로 정의한다면

```java
@Entity
@Builder
@AllArgsConstructor
@NoArgsConstructor
@Getter
@ToString
@Table(name = "tbl_member")
public class Member extends BaseEntity {

    @Id
    private String email;

    private String password;

    private String name;

}
```

게시물 Entity는 다음과 같이 정의한다.

```java
@Entity
@Builder
@AllArgsConstructor
@NoArgsConstructor
@Getter
@ToString(exclude = "writer")
public class Board extends BaseEntity {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long bno;

    private String title;

    private String content;

    @ManyToOne
    private Member writer;

}
```

@ManyToOne 으로 PK에 대해 1:N을 명시한다.

@ManyToOne는 DB상에 FK 관계로 연결된 Entity class에 설정한다.

Eager/Lazy 조회

그런데 여기서 문제가 생긴다.

연관관계의 테이블을 조회할 때에는 join이 된다.

이렇게 설정된 Entity는 Board를 조회 시 Member도 가져온다.

(내부적으로 left ouuter join 처리)

Board를 참조하는 reply까지 있다면, reply을 가져올 때 board와 member까지 join하여 가져온다.

졸래리 낭비다.

이와 같이 한 번에 연관관계의 모든 Entity를 가져오는 조회를 Eager loading(즉시 로딩)라 한다.

여러 연관관계를 맺고 있거나 연관관계가 복잡할수록 조인으로 인한 성능 저하가 발생한다.

이를 방지하기 위해 Lazy loading(지연 로딩)을 사용한다.

@ManyToOne의 속성으로 ‘fetch 모드’를 사용하여 명시할 수 있다.

```java
@ManyToOne (fetch = FetchType.LAZY)
private Member writer;
```

…no session message

lazy 조회를 하게되면 다른 문제가 생긴다.

예를 들어 Board Entity를 통해 getWriter()를 호출하는 경우 에러가 발생한다.

… no session 라는 메세지를 확인할 수 있다.

writer는 Member table의 pk이므로 member table을 로딩해야 하는데

lazy조회를 통해 이미 db와의 연결은 끝난 상태이기 때문에 발생한 에러다.

```java
@Transactional
@Test
public void testRead1() {

    Board board = boardRepository.getOne(100L);

    System.out.println(board);
    System.out.println(board.getWriter());
}
```

@Transactional 어노테이션으로 해결 가능하다.

기본적으로 필요할 때 다시 DB와의 연결이 생성된다.

```java
@ToString(exclude = "writer")
public class Board extends BaseEntity {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long bno;

    private String title;

    private String content;

    @ManyToOne
    private Member writer;

}
```

@ToString은 해당 클래스의 모든 멤버 변수를 출력한다.

위의 Board객체에 @ToString을 주면 writer로 선언된 Member 객체도 출력해야 한다.

(이러면 Lazy loading을 쓰는 의미가 없어진다.)

@ToString(exclude = "writer")와 같이 exclude로 지정된 변수는 toString()에서 제외된다.

JPQL, left (outter) join

만약 위와 같은 구조에서 게시물과 댓글을 같이 가져오기 위해서는 하나의 Entity로 처리할 수 없다.

(그렇다고 Eager조회로 처리하면 프로그램 버려야 한다)

이런 경우 JPQL의 조인을 이용해 처리 가능하다.

(spring boot 2버전 이후의 jpa 버전은 Entity내에 연관관계가 없는 클래스도 조인이 가능하다)

위에서 Board 내부에 Member를 선언하고 연관관계를 맺고 있다.

이 때 Board의 writer를 이용해 조인을 처리한다.

```java
@Query("select b, w from Board b left join b.writer w where b.bno =:bno")
Object getBoardWithWriter(@Param("bno") Long bno);
```

getBoardWithWriter는 Board를 사용하고 있지만 Member를 같이 조회해야 하는 상황이다.

Board에서는 Member와 연관관계를 맺고 있으므로 b.writer와 같은 형태로 사용한다.

이처럼 내부에 있는 Entity를 이용할 때에는 left join 뒤에 on 을 사용하지 않는다.

연관관계가 없는 Entity의 조인을 처리하는 경우.

게시물과 댓글을 가져올 때, Reply은 @ManyToOne으로 Board를 참조하고 있으나

Board에서는 Reply객체를 참조하지 않는다.

이러한 경우 조인에 필요한 조건을 on을 사용하여 처리한다.
