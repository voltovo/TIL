# input 태그의 이전 작성값 기록하기

### 원인_대한항공 기내면세점 프로젝트
Jquery의 datepicker 위젯을 사용하는데, 조건문으로 체크해서 종료 날짜가 시작 날짜보다 빠른경우 선택하기 이전 종료 날짜로 변경하는 경우. 달력 위젯을 클릭해서 값을 변경하기 이전 값을 저장하는 방법이 필요.

#### input의 focusin 이벤트?
해당 요소가 포서크 상태가 됐을 경우
```javaScript
<input type="text" value="2022-02-02" id="endDtm">
<script>
    $('#endDtm').on('focusin', function(){
        console.log("saving value = ", $(this).val());
    });
</script>
```
반대 이벤트 = focusout

#### 이전 값 처리 방식
이전 값을 동일한 태그에 저장
```javaScript
<input type="text" value="2022-02-02" id="endDtm">
<script>
    $('#endDtm').on('focusin', function(){
        console.log("saving value = ", $(this).val());
        // 해당 태그에 data라는 속성을 만들어서 값을 저장
        $(this).data('val', $(this).val());
    });

    $('#endDtm').on('change', function(){
        var preVal = $(this).data('val');
        var current = $(this).val();
        console.log("PreVal value = ", preVal);
        console.log("new value = ", current);
    })
</script>
```
