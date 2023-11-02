# sql-anti-patterns
SQL AntiPatterns - Bill Karwin


![스크린샷 2023-10-22 오후 11 23 50](https://github.com/isntkyu/sql-anti-patterns/assets/56504493/51ca2218-b0b0-4ef2-99b7-adf4eb99e831)


---

# 무단횡단

일대다 관계 테이블의 요구사항이 다대다로 바뀐 상황.
한 컬럼에 쉼표로 이어진 다중 값 속성을 사용한다.
> 건너기만 하면 된다는 무단횡단에 비유

이러면 안되는 이유
- 조회시 패턴매칭해야함 > 인덱스 안탐
- 집계함수 못씀
- 정렬도 안됨
- 수정이 복잡함
- 유효성 검증이 안됨
- 구분자를 절대 쓰이지 않을 문자로 해야함
- 길이(항목수) 제한이 있다

```
결론
중간 테이블을 써야함
```

--- 
# 순진한트리  

데이터의 재귀적 관계

계층 구조를 만들때는 모든자식을 한번에 조회할수있는지 체크해야한다

1. parent_id 컬럼을 추가하는 방법 (인접목록)
	1. 조인의 개수만큼의 뎁스의자식만 가져올 수 있다 (동적으로 조인추가가 안되니깐)
	2. select 컬럼이 늘어나야해서 집계함수 사용이 어렵다
	3. 노드 삭제가 어렵다(복잡 위험)

2. 재귀적 쿼리도 있음.

## 해법

### 경로열거

Path 라는 컬럼을 추가해서 경로를 입력 (디렉토리 구조 처럼)
하지만 검증 비용이 많고  애플리케이션 레벨에서 만들어지기 떄문에 의존적이며
무단횡단 < 의 단점을 많이 공유한다.

### 중첩집합

nsleft, nsright 컬럼추가
사용법: DFS 로 탐색하며 내려가면서nsleft값을 입력하고 다시 올라오면서 nsright 값을 입력한다.

강점: 부모노드를 삭제해도 할아버지 노드와 자식 노드가 새롭게 부모자식으로 이어진다

그 외에는 모든 사용법이 너무 복잡하다

### 클로저테이블

새로이 테이블을 생성한다. 
이 테이블은 조상 자손 관계 또는 경로까지 담고있다.

---

# 아이디가 필요해

pk 에 대한 이야기이다.  
모든 pk 를 가상키 id 로 이름짓게 되면,  
외래키 조인 시 USING 쿼리를 사용 불가함 (같은 컬럼명이면 유용한데, 외래키로 id 를 넣을 수 없다. 그 테이블에도 id가 있어서)

다대다 교차테이블에서 가상키 Pk 를 따로잡는 것도 안티패턴이라함 (자주 이렇게 사용하고있었다.)

꼭 가상키pk 가 필요하지 않다는 걸 설명하고 있다.  
자연키가 좋다고 하는데, 자연키가 인덱싱이 어려운 값일 경우엔 가상키가 좋다고 한다.

## 해법

### 상황에 맞추기

auto_increment랑 pk 는 독립적인 개념인 것 명확히 하기.  
인덱스를 지원하기만 하면 어느것이 pk 일 수 있다.

의미 있는 컬럼명을 짓자.

### 자연키와 복합키 포용

관례에 얽매이지 말자.

유일하고 not null이며 행 식별이 가능하다면 통념때문에 가상키를 고집하지 않아도 된다.  

다대다 교차테이블에서 양쪽의 pk 외래키를 복합pk로 하면 장점은 ?!  
조인을 안해도된다.  근데 fk 도 복합키가 다 들어가야함.

복잡한 것 같다.
