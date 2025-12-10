# 🔥 1️⃣ 왜 중첩 서브쿼리(특히 EXISTS/IN)가 문제인가?

중첩 서브쿼리는 다음과 같은 형태:

`select name from instructor where exists (     select *     from teaches     where instructor.ID = teaches.ID       and teaches.year = 2007 );`

이 형태의 문제점:

### ❌ 중첩된 쿼리를 "튜플마다 일일이 평가"할 수도 있다

(특히 optimizer가 변환 실패할 경우)

즉 instructor의 레코드가 10000개면  
→ 10000번 teaches를 검색할 수 있음 → 비효율적 (nested-loop with subquery)

### ❌ optimizer가 항상 잘 변환하지 못한다

모든 DB는 서로 다르고,  
특정 조건식이 복잡하면 optimizer가 서브쿼리를 JOIN으로 변환 못할 때도 있다.

그래서 강의 슬라이드가 말하는 문장이 나오는 것:

`it is best to avoid using complex nested subqueries (where possible)`

가능하면 복잡한 형태 쓰지 말라는 뜻.

---

# 🔥 2️⃣ optimzer가 항상 decorrelation을 성공시키는 건 아니다

몇몇 DBMS는 서브쿼리를 JOIN 형태로 자동 변환한다.

하지만 복잡한 조건식, aggregate 포함, 상관 서브쿼리(correlated subquery),  
OR/DISTINCT와 섞여 있으면 변환을 못할 때가 있다.

그래서 **성능이 DBMS마다 예측 불가능해짐**.

---

# 🔥 3️⃣ Decorrealation(디코릴레이션)이란 무엇인가?

### 👉 **중첩 서브쿼리(correlated subquery)를 JOIN 형태로 바꿔서 평평한(flat)한 쿼리로 만드는 과정.**

슬라이드에서 예시:

### 원래 형태 (correlated subquery)

`select name 
from instructor 
where exists 
(     select *     
	from teaches    
	 where instructor.ID = teaches.ID       
	 and teaches.year = 2007 
);`

여기서 문제는:

- teaches.year=2007 조건 + instructor.ID=teaches.ID 조건
    
- **instructor의 각 튜플마다 teaches를 다시 검색함**
    

---

# 🔥 4️⃣ Decorrelated 형태 (JOIN 형태)

슬라이드에서 변환한 형태는:

`create table t1 as     
	select distinct ID     
	from teaches     
	where year = 2007;  
select name 
from instructor, t1 
where t1.ID = instructor.ID;`

### 이 형태의 장점:

### ✔ 조인 최적화가 훨씬 쉬운 형태가 됨

- 중첩 루프 형태가 아니라 조인 형태
    
- 조인 순서 최적화 알고리즘이 적용됨
    
- 인덱스 사용 쉬움
    
- 중간 결과를 materialize 할 수도 있음
    

### ✔ 쿼리 계획이 DBMS마다 훨씬 일관적

- decorrelation 실패 → 최악의 성능
    
- decorrelation 성공 → 조인 기반 최적화로 빠름
    

즉, decorrelation은 “서브쿼리를 평탄화해서 조인 문제로 바꾸는 기술”.

---

# ⭐ 왜 교수님이 강조하는가?

조인 최적화는 decades 동안 DB 연구의 핵심이고  
대부분의 DBMS가 최적화 기법을 잘 제공하지만,

**nested subquery 최적화는 DBMS별로 편차가 크고 신뢰성을 보장할 수 없다.**

그래서 교수님이 아래 문구를 강조한 것:

`It is best to avoid using complex nested subqueries. We cannot be sure the optimizer succeeds at converting them.`

→ 즉, 직접 JOIN으로 바꾸거나  
→ WITH(CTE)나 VIEW로 평탄한 형태를 추천.

---

# 🧠 쉽게 비유하면:

서브쿼리는:

> "학생을 하나하나 읽으면서  
> 해당 학생이 2007년에 수업을 가르쳤는지  
> teaches 테이블을 그때그때 다시 검색하는 방식"

JOIN으로 바꾸면:

> "2007년에 가르친 사람 목록을 먼저 하나로 만들어 놓고  
> 그 목록과 instructor를 조인"

훨씬 빠르다.

---

# 🔚 한 문장 요약

> **Decorrelation = 중첩 correlated subquery 를 JOIN 기반의 평평한 쿼리로 바꿔서  
> 성능을 예측 가능하고 최적화하기 쉬운 형태로 만드는 기술.**