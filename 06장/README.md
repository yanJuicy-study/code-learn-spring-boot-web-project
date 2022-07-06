# Chapter06

### JSON과 Ajax로 댓글 처리

- `@RestController`를 사용하면 json 형식으로 리턴을 줄 수 있다.

```java
@RestController
@RequestMapping("/replies/")
@Log4j2
@RequiredArgsConstructor
public classReplyController{

    private finalReplyServicereplyService;

		@GetMapping(value = "/board/{bno}", produces =MediaType.APPLICATION_JSON_VALUE)
    publicResponseEntity<List<ReplyDTO>> getListByBoard(@PathVariable("bno")Longbno) {
        log.info("bno: " +bno);
        return new ResponseEntity<>(replyService.getList(bno),HttpStatus.OK);
    }

}

```

json 데이터 요청

![image](https://user-images.githubusercontent.com/43159295/177609624-4059de3f-284c-49bf-aeec-d253aad4ac6c.png)


- thymeleaf에서 javascript로 json 요청을 통해 데이터를 받는다.

```jsx
<script th:inline="javascript">
    $(document).ready(function () {
        var bno = [[${dto.bno}]];

        var listGroup = $(".replyList");

        function formatTime(str) {
            var date = new Date(str);

            return date.getFullYear() + '/' +
                (date.getMonth() + 1) + '/' +
                date.getDate() + ' ' +
                date.getHours() + ':' +
                date.getMinutes();
        }

        function loadJSONData() {
						// 여기서 요청 보냄
            $.getJSON('/replies/board/' + bno, function (arr) {
                console.log(arr);
                var str = "";
                $('.replyCount').html("Reply Count  " + arr.length);
                $.each(arr, function (idx, reply) {
                    console.log(reply);
                    str += ' <div class="card-body" data-rno="' + reply.rno + '"><b>' + reply.rno + '</b>';
                    str += ' <h5 class="card-title">' + reply.text + '</h5>';
                    str += ' <h6 class="card-subtitle mb-2 text-muted">' + reply.replyer + '</h6>';
                    str += ' <p class="card-text">' + formatTime(reply.regDate) + '</p>';
                    str += ' </div>'
                })
                listGroup.html(str);
            });
        }

        $(".replyCount").click(function () {
            loadJSONData();
        });

    });
</script>
```
