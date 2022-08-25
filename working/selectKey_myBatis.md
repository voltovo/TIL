# selectKey

### 원인_대한항공 기내면세점 프로젝트
GNB 메뉴 적용 이력 테이블에 INSERT 후 GNB 메뉴 편집 테이블에 INSERT 할 때 적용 이력 테이블의 SEQ_NO를 HIST_SEQ_NO로 넣을 필요성이 발생.

### selectKey 란?
DB에서 지원하는 sequence와 scope_identity등을 활용해서 insert시에 추가한 데이터에 기본키를 반환한다.

#### 기본 사용법
```xml
<selectKey resultType="integer" keyProperty="idx" order="AFTER">
    SELECT LAST_INSERT_ID()
</selectKey>
```

#### 속성 정리   
|속성명|설명|
|---|------|
|keyProperty|selectKey구문의 결과가 셋팅될 대상|
|resultType|결과의 타입. 마이바티스는 이 기능을 제거할 수 있지만 추가해도 문제 X|
|order|BEFORE 또는 AFTER으로 셋팅 가능|

#### order 추가 설명
* BEFORE   
키를 먼저 조회하고 그 값을 keyProperty에 셋팅한 뒤 insert 구문을 실행

* AFTER   
insert 구문을 실행한 뒤 selectKey 구문을 실행

오라클 같은 데이터베이스에서는 insert구문 내부에서 일관된 호출 형태로 처리한다.

#### selectKey 적용 예시
* AUTO_INCREMENT가 적용되지 않은 테이블에 id를 계산해서 넣고 싶은 경우
* AUTO_INCREMENT가 적용된 테이블에 삽입된 데이터의 id를 바로 조회하여 바로 다른 테이블에 삽입하고 싶은 경우

```xml
<insert id="insertHobby" parameterType="hobby">
    /* order="BEFORE" 삽입 전에 조회 */
    /* selectKey 구문의 위치는 INSERT 쿼리 위, 아래 상관 없이 위치할 수 있습니다. */
    <selectKey keyProperty="hobbyId" resultType="int" order="BEFORE">
        SELECT MAX(hobby_id) + 1 FROM hobby
    </selectKey>

    /* #{hobbyId}에는 SelectKey 구문을 통해서 조회한 값이 저장되어 있습니다. */
    INSERT INTO hobby(hobbdy_id, hobby_name, user_id)
    VALUES (#{hobbyId}, #{hobbyName}, #{userId})
</insert>
```

#### 적용하고 싶었던 예시
TB_GNB_MENU_HIST 테이블에 insert 후 SEQ_NO를 파라미터로 넘긴 hashMap에 그대로 담아서 서비스단 로직에서 TB_GNB_MENU 테이블 insert 할 떄 사용하기

```xml
<insert id="insertGnbMenuHist" parameterType="map">
INSERT INTO .......
    <!-- as 를 안 붙이면 에러가 발생 -->
    <selectKey keyProperty="seqNo" resultType="int" order="AFTER">
        SELECT (SEQ_GNB_MENU_HIST.CURRVAL) AS SEQ_NO FROM DUAL
    </selectKey>
</insert>
```
```java
int result = boardMapper.insertGnbMenuHist(param);
String seqNo = param.get("seqNo").toString();
```

#### selectKey 와 멀티쓰레드
멀티쓰레드 환경에서 Sequence를 통해서 키 값을 사용할 때 문제가 발생할 수 있다.

* 문제 발생 예시   
1. 1번 채번 insert
2. 2번 채번 insert
3. 1번 채번 조회 : 2번 채번 값
4. 2번 채번 조회 : 1번 채번 값

**1번 채번 INSERT 후 가져온 해당 키 값이 2번 채번 값으로 가져와 오류를 발생시킬 수 있다.**
그래서 selectKey를 사용하면, 채번한 값을 객체에 저장한 후 그 값을 INSERT에 사용하므로, 멀티스레드 상황에서도 안전하게 값을 가져올 수 있다.