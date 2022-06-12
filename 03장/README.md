# ch03 MVC, Thymeleaf

Thymeleaf.

mvc 패턴의 view를 담당하는 템플릿 엔진.

spring boot에서는 jsp대신 thymeleaf를 권장하고 있으며 jsp와 비교시 다음의 장점을 가진다.

- model에 담긴 객체를 js로 처리.
- 개발 도구 이용시 .html 파일로 생성함에 문제가 없으며 별도의 확장자를 이용하지 않음.
- html코드의 원본을 최대한 유지.

// 개인적으로, 코딩의 관점에서 볼 때 3번 째 장점이 가장 돋보인다.

기존 jsp의 <% %>로 블록을 설정하는 문법은 불편하기도 하며 

무엇보다 가독성을 흐리고 html 원본을 망가뜨리는건 여간 신경쓰이는 일이 아니니까. //

이번 챕터의 예제를 위해 아래 4가지 의존성을 추가한다.

- Spring boot DevTools
- Lombok
- Spring Web
- Thymeleaf

// Spring boot DevTools를 통해 다음의 기능을 사용한다.

- Automatic Restart - class path에 존재하는 파일이 변경되면 어플리케이션을 재시작.
- Live Reload - 정적 자원(html, css, js)이 수정되면 새로고침 없이 적용.
- Property Defaults - Thymeleaf의 캐싱 기능을 개발 과정에서 false로 설정.

Lombok은 Entity와 같은 도메인 클래스의 필드에 접근하는 

getter, setter, toString등의 메서드 이용에 편리성을 제공한다.

개발자마다 호불호가 나뉘며, 나는 intelliJ의 자동완성을 통해 getter와 setter를 직접 생성하는 편을 추구한다.

직관적이기도 하고 어차피 빌드하면 클래스 파일에 생성될 메서드들이다.

Spring Web은 그냥 해당 프로젝트가 웹 어플리케이션이라는 소리다. //

생성된 프로젝트의 application.properties에 다음을 추가한다.

`spring.thymeleaf.cache=false`

// Thymeleaf는 성능 향상을 위해 캐싱 기능을 제공하지만 개발 단계에서 계속 사용하면

수정된 소스가 제대로 반영되지 않는 상황이 발생하기 때문에.

즉, Thymeleaf 파일을 수정 후 저장한 후에 변경된 결과를 브라우저에서 확인하기 위함이다. //

프로젝트 디렉토리에 controller를 생성하고 SampleController.java를 생성한다.

```java
import lombok.extern.log4j.Log4j2;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestMapping;

@Controller
@RequestMapping("/sample")
@Log4j2
public class SampleController {

    @GetMapping("/ex1")
    public void ex1() {
			log.info("ex1 yeah");
    }
}
```

// log를 남기는 버릇을 들이자.

나중에 배포 시 tomcat, nginx와 같은 was에서 로그가 존재해야 함은 필수적이다. //

thymeleaf는 프로젝트 생성 시 기본적으로 생성되는 resources.templates 폴더를 기본으로 사용한다.

해당 위치에 적당한 폴더를 하나 만들어주고 ex1.html을 생성한다.

```markup
<!DOCTYPE html>
<html lang="en" xmlns:th="http://www.thymeleaf.org">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
    <h1 th:text = "${'fuck you world'}"> </h1>s
</body>
</html>
```

여기서 중요한 것은 th:text = “${}” 부분이다.

기존의 html 속성 앞에 th: 를 붙여주고 속성값을 지정하면 된다.

// 일반적인 js 라이브러리들과 유사해서 별 어려움이 없다. //

프로젝트 디렉토리에 dto 디렉토리를 만들고 SampleDTO 클래스를 생성한다.

```java
import lombok.Builder;
import lombok.Data;
import java.time.LocalDateTime;

@Data
@Builder(toBuilder = true)
public class SampleDTO {
    private Long sno;
    private String first;
    private String last;
    private LocalDateTime regTime;
}
```

Data 어노테이션은 getter/setter toString equals hashCode 등을 자동으로 생성한다.

처음 생성했던 SampleController에 다음을 추가한다.

```java
@GetMapping({"/ex2"})
public void exModel(Model model) {
    List<SampleDTO> list = 
			IntStream.rangeClosed(1, 20).asLongStream().mapToObj(i -> {
        SampleDTO dto = SampleDTO.builder()
                .sno(i)
                .first("First.." + i)
                .last("Last.." +i)
                .regTime(LocalDateTime.now())
                .build();
        return dto;
    }).collect(Collectors.toList());

    model.addAttribute("list", list);
}
```

SampleDTO 타입 객체 20개를 만들어 List로 model에 담아 하는 메서드다.

GetMapping의 value를 {}로 처리하면 하나 이상의 url을 지정할 수 있다.

// 람다식을 사용하는건 테스트 또는 임시적이기 때문이 아닐까 싶다. 무지성 람다식 남발 금지.

sno는 시리얼 넘버, 중복되지 않는 고유 값을 나타내는 것 같다. 

그런거라면 일반적으로 seq로 쓰는 경우가 많다.

위의 controller에서 전송한 model객체를 view에서 써볼 차례다.

ex02.html을 생성한다.

```markup
<!DOCTYPE html>
<html lang="en" xmlns:th="http://www.thymeleaf.org">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
  <ul>
    <li th:each="dto, state : ${list}">
      [[${state.index}]] -- [[${dto}]]
    </li>
  </ul>
</body>
</html>
```

`th:each = “var : ${data}”`

Thymeleaf에서 반복은 th:each 속성을 사용한다.

따라서 위에서는 dto라는 변수명으로, Model로 전달된 데이터는 ${list}를 이용해서 처리한다.

[[ ]]는 인라인 표현식으로 별도의 태그를 지정하지 않고 사용할 때 유용하다.

// react의 <> </> 와 비슷한?

dto뒤에 붙는 state는 상태 객체이다.

이를 이용해 인덱스, 홀수/짝수 등을 지정할 수 있다.

state.index, state.count 등을 사용할 수 있는데, index와 count는 각 각 시작 번호가 0, 1 이라는 차이가 있다.

thymeleaf의 제어문은 크게 두 가지 방식이 있다.

th:if ~ unless / 삼항 연산자.

th:if ~ unless 는 else if 구문과 비슷하지만 단독으로 처리한다는 차이가 있다.

ex / sno가 5의 배수이면 sno만 출력, 아닌 경우 first를 출력하는 경우

```markup
<ul>
  <li th:each="dto : ${list}" >
    <span th:if="${dto.sno % 5 == 0}" th:text="${'--------' + dto.sno}"> </span>
    <span th:unless="${dto.sno % 5 == 0}" th:text="${dto.first}"> </span>
  </li>
</ul>
```

위와 동일한 ex에 대해 삼항 연산자를 적용한다면

```markup
<li th:each="dto : ${list}" 
	th:text="${dto.sno % 5 == 0} ? ${'--------' + dto.sno} : ${dto.first}"> </li>
```

Thymeleaf의 속성 중 개발에 많은 도움을 주는 기능으로 inline 속성이 있다.

주로 javaScript처리에서 유용하다.

SampleController에 다음을 추가한다.

```java
@GetMapping({"/exInline"})
public String exInline(RedirectAttributes redirectAttributes) {

log.info("exInline ######## ");

    SampleDTO dto = SampleDTO.builder()
            .sno(100L)
            .first("First..100")
            .last("Last..100")
            .regTime(LocalDateTime.now())
            .build();
    redirectAttributes.addFlashAttribute("result", "success");
    redirectAttributes.addFlashAttribute("dto", dto);

    return "redirect:/sample/ex3";
}

@GetMapping("/ex3")
public void ex3() {
	log.info("ex3");
}
```

대단한건 없다.

RedirectAttributes를 이용하여 내부적으로 result, dto 두 데이터를 담아 ex3으로 리다이렉션 한다.

리다이렉트 되는 페이지 ex3.html을 생성한다.

```markup
<!DOCTYPE html>
<html lang="en" xmlns:th="http://www.thymeleaf.org">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
<h1 th:text = "${result}"> </h1>
<h1 th:text = "${dto}"> </h1>

<script th:inline="javascript">
    var msg= [[${result}]];
    var dto= [[${dto}]];
</script>
</body>
</html>
```

여기서 중요한건 이 부분이다.

```markup
<script th:inline="javascript">
    var msg= [[${result}]];
    var dto= [[${dto}]];
</script>
```

inline의 속성 값으로 javascript가 사용된다.

위의 예제를 [localhost:8080/sample/exInline](http://localhost:8080/sample/exInline) 으로 확인하면 [localhost:8080/sample/ex](http://localhost:8080/sample/exInline)3 으로 넘어가며

이 페이지에서 콘솔창을 열어 script를 확인해보면

var msg = “success”

var dto = {”sno”:100, “first”:”First..100”, ….} 을 확인할 수 있는데,

처음 데이터를 넘겨줄 때, result는 String, dto는 SampleDTO 객체였다.

별도의 처리 없이 문자열은 문자열로, dto는 JSON 포맷 문자열이 되었다.

// 우리 리액트는 이런것도 지가 알아서 다 해준다. 자기가 자바스크립트 그 자체니까.

th:block은 때에 따라 유용한 기능이다.

별도의 태그가 필요하지 않기 때문에 반드시 태그에 붙어 th:text 또는 th:value 등을 써야하는 제약이 없다.

위에서 다뤘던 5의 배수 예제.

```markup
<ul>
  <li th:each="dto : ${list}" >
    <span th:if="${dto.sno % 5 == 0}" th:text="${'--------' + dto.sno}"> </span>
    <span th:unless="${dto.sno % 5 == 0}" th:text="${dto.first}"> </span>
  </li>
</ul>
```

이를 th:block을 이용한다면

```markup
<ul>
    <th:block th:each="dto : ${list}">
        <li th:text="${dto.sno % 5 == 0} ? ${dto.sno} : ${dto.first}"> </li>
    </th:block>
</ul>
```

와 같이 표현할 수 있다.

th:block 은 실제 화면에서는 html로 처리되지 않는다.

바로 위 예제와 같이 루프 등을 별도로 처리하는 데에 주로 사용된다.

Thymeleaf의 링크는 @{} 를 이용한다.

params를 전달하는 상황에서 가독성이 좋다는 특징이 있다.

// 굳이 장점을 말하자면 그렇다고

기존 SampleController의 exModel을 재사용 하도록 아래와 같이 수정한다.

```java
@GetMapping({"/ex2", "/exLink"})
public void exModel(Model model) {
    List<SampleDTO> list = IntStream.rangeClosed(1, 20).asLongStream().mapToObj(i -> {
        SampleDTO dto = SampleDTO.builder()
                .sno(i)
                .first("First.." + i)
                .last("Last.." +i)
                .regTime(LocalDateTime.now())
                .build();
        return dto;
    }).collect(Collectors.toList());

    model.addAttribute("list", list);
}
```

기존 `@GetMapping({"/ex2"})` 를

`@GetMapping({"/ex2", "/exLink"})` 로 수정하였다.

GetMapping()에 배열을 이용하면 하나 이상의 url을 처리할 수 있다.

위와 같은 방식으로 하여 sample/exLink 경로를 처리할 수 있는 구조가 된다.

exLink.html 을 생성한다.

```markup
<!DOCTYPE html>
<html lang="en" xmlns:th="http://www.thymeleaf.org">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
    <ul>
        <li th:each="dto : ${list}">
            <a th:href = "@{/sample/exView}"> [[${dto}]] </a>
        </li>
    </ul>
</body>
</html>
```

예제를 실행하면 <a href = "sample/exView” > 링크가 생성된다.

sample/exView 에 파라미터를 넘기기 위해 아래와 같이 수정한다.

```markup
<ul>
    <li th:each="dto : ${list}">
        <a th:href = "@{/sample/exView(sno = ${dto.sno})}"> [[${dto}]] </a>
    </li>
</ul>
```

url() 내에 key:value 형태를 추가한다.

실행 후 생성된 링크는

<a href = "sample/exView?sno=1” >

<a href = "sample/exView?sno=2” >

… 의 형태이다.

sno를 path로 사용하고 싶다면

```markup
<ul>
    <li th:each="dto : ${list}">
       <a th:href = "@{/sample/exView/{sno}(sno = ${dto.sno})}"> [[${dto}]] </a>
    </li>
</ul>
```

와 같이 사용하면 된다.

// 쿼리스트링은 기본적으로 url에 정보가 노출된다.

따라서 보안에 신경써야 하는 데이터는 쿼리스트링으로 전송하지 않는 것이 기본이다.

또한 이러한 방식은 전송하고자 하는 데이터에 특수문자가 있다면 URLEncode를 사용하여야 한다.

전송 데이터의 길이에도 제약이 있다. 

그러니 이 역시 무지성 남발 금지. //

Thymeleaf는 내부적으로 기본 객체를 지원한다.

기본 객체란 문자, 숫자, request, response, session 등이 그 예이다.

ex / sno를 모두 5자리로 만들어야 한다면

```markup
<ul>
    <li th:each="dto : ${list}">
       [[${#numbers.formatInteger(dto.sno, 5)}]]
    </li>
</ul>
```

로 구현할 수 있다.

내장 객체들이 많은 기능을 지원하지만 java8 에 등장한 LocalDate 타입이나 LocalDateTime에 대해서는

복잡한 방식으로 처리해야 하는 단점이 있다.

이를 보다 편리하게 처리하고 싶다면

[http://github.com/thymeleaf/thymeleaf-extras-java8time](http://github.com/thymeleaf/thymeleaf-extras-java8time) 을 이용하면 된다

build.gradle 에 다음 의존성을 추가한다. // maven을 사용한다면 [https://mvnrepository.com/](https://mvnrepository.com/) 에서 겟

```scheme
dependencies{
implementation 'org.springframework.boot:spring-boot-starter-thymeleaf'
    implementation 'org.springframework.boot:spring-boot-starter-web'
    compileOnly 'org.projectlombok:lombok'
    developmentOnly 'org.springframework.boot:spring-boot-devtools'
    annotationProcessor 'org.projectlombok:lombok'

    providedRunTime 'org.springframework.boot:spring-boot-starter-tomcat'
    testImplementation ('org.springframework.boot:spring-boot-starter-test'){
exclude group : 'org.junit.vintage', module : 'junit.vintage.engine'
}
compile group : 'org.thymeleaf.extras', name : 'thymeleaf-extras-java8time'
}
```

// 위 의존성을 그대로 사용했을 때

Could not find method providedRunTime() 또는 Could not find method compile() 을 마주할 수 있다.

gradle 7 이상부터는 compile 또는 testCompile 등을

implementation, testImplementation 으로 사용하고 있다.

인텔리제이를 사용한다면 터미널 - 루트 디렉토리에서 ./gradlew —v 로 gradle 버전을 확인할 수 있다.

7버전 이상이라면 아래 의존성을 사용하자.

이 때문에 과거 회사에서 고생했던 적이 있다. 역시 개발자의 성장 이벤트는 경험이다. //

```scheme
dependencies{
implementation 'org.springframework.boot:spring-boot-starter-thymeleaf'
    implementation 'org.springframework.boot:spring-boot-starter-web'
    compileOnly 'org.projectlombok:lombok'
    developmentOnly 'org.springframework.boot:spring-boot-devtools'
    annotationProcessor 'org.projectlombok:lombok'

    implementation 'org.springframework.boot:spring-boot-starter-tomcat'
    testImplementation ('org.springframework.boot:spring-boot-starter-test'){
exclude group : 'org.junit.vintage', module : 'junit.vintage.engine'
}
implementation group : 'org.thymeleaf.extras', name : 'thymeleaf-extras-java8time'
}
```

의존성 추가 후 exLink.html 을 수정한다.

```markup
<ul>
    <li th:each="dto : ${list}">
       [[${dto.sno}]] --- [[${#temporals.format(dto.regTime, 'yyyy-mm-dd')}]]
    </li>
</ul>
```

• 1 --- 2022-13-28

• 2 --- 2022-13-28 … 와 같은 결과를 확인할 수 있다.

// 당연한 얘기지만 포맷 패턴의 구분자는 /든 +든 무관하다.

Thymeleaf의 레이아웃 기능은 크게 2가지 형태로 사용 가능하다.

- JSP의 include와 같이 특정 부분을 내.외부에서 가져와 사용하는 형태
- 특정 부분을 파라미터로 전달하여 내용에 포함하는 형태

먼저 include 방식으로 처리하는 경우를 보자.

th:replace 와 th:insert 가 있다.

th:replace 는 기존 내용을 완전히 대체하는 방식이고

th:insert 는 기존 내용의 바깥 태그는 유지하면서 추가되는 방식이다.

SampleController에 다음을 추가한다.

```java
@GetMapping("/exLayout1")
public void exLayout1() {
	log.info("exLayout-----------");
}
```

template 폴더에 fragments 폴더를 생성하고 fragment1.html 을 생성한다.

```markup
<!DOCTYPE html>
<html lang="en" xmlns:th="http://www.thymeleaf.org">
<head>
  <meta charset="UTF-8">
  <title>Title</title>
</head>
  <body>
    <div th:fragment="part1">
      <h3> part 1</h3>
    </div>

    <div th:fragment="part2">
      <h3> part 2</h3>
    </div>

    <div th:fragment="part3">
      <h3> part 3</h3>
    </div>
  </body>
</html>
```

위의 th:fragment 로 구현된 부분을 가져다 쓸 exLayout1.html을 기존 templates/sample 폴더에 생성한다.

```markup
<!DOCTYPE html>
<html lang="en" xmlns:th="http://www.thymeleaf.org">
<head>
  <meta charset="UTF-8">
  <title>Title</title>
</head>
	<body>
	  <h2>Fragment test</h2>
	
	  <h3> Layout 1 - 1</h3>
	  <div th:replace = "~{/fragments/fragment1 :: part1}"> </div>
	
	  <h3> Layout 1 - 2</h3>
	  <div th:insert = "~{/fragments/fragment1 :: part2}"> </div>
	
	  <h3> Layout 1 - 3</h3>
	  <th:block th:replace = "~{/fragments/fragment1 :: part3}"> </th:block>
	
	</body>
</html>
```

실행 후 콘솔창을 열어보면 Layout 1 - 2에는 <div> 내에 <div> 가 하나 더 있는 것을 확인할 수 있다.

위의 설명을 다시 보자.

- th:replace 는 기존 내용을 완전히 대체하는 방식이고
- th:insert 는 기존 내용의 바깥 태그는 유지하면서 추가되는 방식이다.

th:insert 를 사용한 부분에서는 기존 <div>를 유지한 채 fragement part2 를 가져와 사용했기 때문에

<div>내에 <div>가 생성되었다.

th:replace 를 사용할 때 :: 뒤에 fragment의 이름을 생략하면 해당 파일의 전체 내용을 가져올 수 있다.

파일 전체를 사용하는 경우를 확인하기 위해 fragments 폴더 내에 fragment2.html 을 생성한다.

```markup
<!DOCTYPE html>
<html lang="en" xmlns:th="http://www.thymeleaf.org">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
	<body>
		<div>
		    <hr/>
		    <h3>Fragment2 File</h3>
		    <h3>Fragment2 File</h3>
		    <h3>Fragment2 File</h3>
		    <hr/>
		</div>
	</body>
</html>
```

exLayout1.html 을 수정한다.

```markup
<body>
  <h2>Fragment test</h2>

  <div>
    <th:block th:replace = "~{/fragments/fragment2}"> </th:block>
  </div>

  <h3> Layout 1 - 1</h3>
  <div th:replace = "~{/fragments/fragment1 :: part1}"> </div>

  <h3> Layout 1 - 2</h3>
  <div th:insert = "~{/fragments/fragment1 :: part2}"> </div>

  <h3> Layout 1 - 3</h3>
  <th:block th:replace = "~{/fragments/fragment1 :: part3}"> </th:block>

</body>
```

위에서

```markup
<div>
  <th:block th:replace = "~{/fragments/fragment2}"> </th:block>
</div>
```

부분을 보면 th:replace 에 :: 로 처리되는 부분이 없으므로 fragment2 의 전체 내용을 반영하게 된다.

이번에는 include가 아닌 파라미터로 전달하는 방식의 경우를 확인해본다.

SampleController의 exLayout1()을 재사용하기 위해 다음으로 수정한다.

```java
@GetMapping({"/exLayout1", "exLayout2"})
public void exLayout1() {
	log.info("exLayout-----------");
}
```

GetMapping을 배열로 바꾼게 전부다.

fragments폴더에 fragment3.html 을 생성한다.

```markup
<!DOCTYPE html>
<html lang="en" xmlns:th="http://www.thymeleaf.org">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
	<body>
		<div th:fragment="target(first, second)">
		    <style>
		        .c1 {
		            background-color: red;
		        }
		        .c2 {
		            background-color: blue;
		        }
		    </style>
		
		    <div class = "c1">
		        <th:block th:replace = "${first}"> </th:block>
		    </div>
		
		    <div class = "c2">
		        <th:block th:replace = "${second}"> </th:block>
		    </div>
		
		</div>
	</body>
</html>
```

target 부분에서 두 개의 파라미터를 받을 수 있도록 하며 이를 내부에서 th:block 으로 구성한다.

이 fragment를 사용할 exLayout2.html 을 templates/sample 폴더 내에 생성한다.

```markup
<!DOCTYPE html>
<html lang="en" xmlns:th="http://www.thymeleaf.org">

<th:block th:replace = 
"~{/fragments/fragment3 :: target(~{this:: #ulFirst}, ~{this:: #ulSecond} )}">

  <ul id = "ulFirst">
    <li>Fuck</li>
    <li>Fuck</li>
  </ul>

  <ul id = "ulSecond">
    <li>Damn</li>
    <li>Damn</li>
  </ul>

</th:block>
```

가장 큰 특징으로는 화면 구성과 관련된 기본 기능이 없다는 것이다.

대신 target에서 두 개의 파라미터를 사용한다.

(#ulFirst, #ulSecond는 CSS의 id 선택자이다.)

결과를 확인해보면 예상되는 결과 그대로가 출력된다.

레이아웃 전체를 하나의 페이지로 구성하고 필요한 부분을 파라미터로 전달하는 방식으로

공통의 레이아웃을 사용할 수 있다.

// 라고 본문에 적혀있다. 자칫 0개 국어로 보일 수 있는 설명이다.

풀어 말하면

예를 들어 데이터가 3개 있을 때, 이에 대해 페이지 3개를 만들게 아니라 

하나의 페이지에서 데이터 3개에 따라 다르게 보여주는 부분이 존재할 수 있다는 소리다.

리액트의 component 와 props 처럼 말이다. 이제 보니 리액트 진짜 개 깡패네ㅋㅋ //

위의 구조를 확인하기 위해 templates 폴더 내에 layout 폴더를 생성하고 이 안에 layout1.html을 생성한다.

```markup
<!DOCTYPE html>
<html lang="en" xmlns:th="http://www.thymeleaf.org">
<head>
  <meta charset="UTF-8">
  <title>Title</title>
</head>
	<body>
	  <style>
	    * {
	      margin : 0;
	      padding : 0;
	    }
	    .header {
	      width: 100vw;
	      height: 20vh;
	      background-color: lightgray;
	    }
	    .content{
	      width: 100vw;
	      height: 70vh;
	      background-color: white;
	    }
	    .footer{
	      width: 100vw;
	      height: 10vh;
	      background-color: grey;
	    }
	  </style>
	
	  <div class = "header">
	    <h3> Header</h3>
	  </div>
	
	  <div class = "content">
	    <h3> Content</h3>
	  </div>
	
	  <div class = "footer">
	    <h3> Footer</h3>
	  </div>
	</body>
</html>
```

평범한 html 파일이다.

가장 기본적인 Header - Content - Footer 의 구조로 이루어져 있다.

// 정적 파일이기 때문에 인텔리제이에서 해당 파일 우클릭 후 run layout1.html 하면 직접 실행할 수 있다. //

이제 여기서 저 가운데의 Content 영역만 다른 내용으로 변경되어야 한다.

// 실제 개발시 대부분이 그렇다. 어느 정신나간 회사도 Footer를 바꾸는 일은 없다.

Header는 로그인 여부나 경우에 따라 약간은 변경되지만 //

따라서 위의 파일을 아래와 같이 수정한다.

```markup
<!DOCTYPE html>
<html lang="en" xmlns:th="http://www.thymeleaf.org">
	<th:block th:fragment = "setContent(content)">
		<head>
		  <meta charset="UTF-8">
		  <title>Title</title>
		</head>
		<body>
		  <style>
		    * {
		      margin : 0;
		      padding : 0;
		    }
		    .header {
		      width: 100vw;
		      height: 20vh;
		      background-color: lightgray;
		    }
		    .content{
		      width: 100vw;
		      height: 70vh;
		      background-color: white;
		    }
		    .footer{
		      width: 100vw;
		      height: 10vh;
		      background-color: grey;
		    }
		  </style>
		
		  <div class = "header">
		    <h3> Header</h3>
		  </div>
		
		  <div class = "content">
		    <th:block th:replace = "${content}">
		
		    </th:block>
		  </div>
		
		  <div class = "footer">
		    <h3> Footer</h3>
		  </div>
		</body>
	</th:block>
</html>
```

`<th:block th:fragment = "setContent(content)">` 를 이용해 content 파라미터를 받고

```markup
<div class = "content">
  <th:block th:replace = "${content}">

  </th:block>
</div>
```

받은 파라미터를 여기에서 구현해준다.

SampleController의 exLayout1()의 GetMapping을 수정해준다.

`@GetMapping({"/exLayout1", "/exLayout2", "/exTemplate"})`

templates/sample 폴더에 exTemplate.html 을 생성한다.

```markup
<!DOCTYPE html>
<html lang="en" xmlns:th="http://www.thymeleaf.org">

<th:block th:replace = "~{/layout/layout1 :: setContent(~{this::content} )}">
    <th:block th:fragment="content">
        <h3>Template Page</h3>
    </th:block>
</th:block>
```

이를 실행하면 중간 content 부분만 변경된 것을 확인할 수 있다.

layout1.html은 코드 전체가 <th:block th:fragment = "setContent(content)"> 으로 감싸져 있다.

exTemplate.html은 layout1.html을 가져와서 content부분만을 변경해주는 것이다.

부트스트랩 템플릿을 적용해보자.

[https://startbootstrap.com/template/simple-sidebar](https://startbootstrap.com/template/simple-sidebar) 에 있는 템플릿을 사용해본다.

수상하게 생긴 Free Download를 누르면 zip파일을 받을 수 있다.

압축 파일 해제 후 생긴 폴더 내부의 폴더 및 파일들을 resources/static 에 넣는다.

templates/layout 에 basic.html 을 생성하고 resources/static의 index.html 내용을 복사한다.

basic.html 의 header를 다음으로 수정한다.

```markup
<!DOCTYPE html>
<html lang="en" xmlns:th="http://www.thymeleaf.org">
<th:block th:fragment = "setContent(content)">
<head>
    <meta charset="utf-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1, shrink-to-fit=no" />
    <meta name="description" content="" />
    <meta name="author" content="" />
    <title>Simple Sidebar - Start Bootstrap Template</title>
    <!-- Favicon-->
    <link rel="icon" type="image/x-icon" href="assets/favicon.ico" />
    <!-- Core theme CSS (includes Bootstrap)-->
    <link th:href="@{/css/styles.css}" rel="stylesheet" />

    <!-- Bootstrap core JS-->
    <script th:src="@{https://cdn.jsdelivr.net/npm/bootstrap@5.1.3/dist/js/bootstrap.bundle.min.js}"></script>
    <!-- Core theme JS-->
    <script th:src="@{/js/scripts.js}"></script>
</head>
```

thymeleaf 속성과 <th:block th:fragment = "setContent(content)"> 를 추가하고

link와 script를 thymeleaf 문법으로 수정한다.

(script는 하단에 위치한 것을 위로 올렸다.)

// stylesheet의 path는 최초 css/styles.css 로 되어있다.

basic.html의 위치는 static이 아니기 때문에 /css/styles.css 와 같이 상대경로로 지정해주어야 한다.

script의 src도 마찬가지. //

th:block 의 닫는 태그는 하단에 위치한다.

```markup
		</body>
	</th:block>
</html>
```

파라미터로 전송되어 적용될 부분은 basic.html의 중간 container-fluid 부분이다.

다음과 같이 수정한다.

```markup
<!-- Page content-->
<div class="container-fluid">
    <th:block th:replace = "${content}"> </th:block>
</div>
```

SampleController의 exLayout1()의 GetMapping을 수정한다.

`@GetMapping({"/exLayout1", "/exLayout2", "/exTemplate", "/exSidebar"})`

templates/sample 에 exSidebar.html을 생성한다.

```markup
<!DOCTYPE html>
<html lang="en" xmlns:th="http://www.thymeleaf.org">

<th:block th:replace = "~{/layout/basic :: setContent(~{this::content})}">

    <th:block th:fragment = "content">
        <h3> Sidebar page
        </h3>
    </th:block>

</th:block>
```

실행 후 부트스트랩이 적용된 페이지를 확인한다.