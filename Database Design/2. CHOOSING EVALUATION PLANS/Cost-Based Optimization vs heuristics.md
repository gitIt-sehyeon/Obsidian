# 🔥 1️⃣ Cost-Based Optimization은 “엄청 비싸다”

DBMS가 쿼리 계획을 찾을 때 해야 할 일:

- 가능한 모든 조인 순서를 탐색
    
- 각 조인 순서마다
    
    - 조인 알고리즘(NLJ, Hash Join 등) 선택
        
    - 인덱스 사용 여부
        
    - push-down selection
        
    - 통계 기반 selectivity 계산
        
    - I/O cost, CPU cost 계산
        

이 과정은 계산량이 **조인할 릴레이션 수에 대해 O(n·2^n)** 이고,  
최악의 경우 조인 후보가 수천 개가 나올 수 있어.

즉:

### ✔ 조인 10개 들어간 쿼리 → 최적화 자체에 시간 오래 걸림

### ✔ SELECT * FROM 테이블 WHERE id = 5 같은 쿼리에는 필요 없음

그래서 cost-based optimization 은 “필요할 때만” 한다.

---

# 🔥 2️⃣ 그래서 옵티마이저는 휴리스틱(heuristics)을 사용한다

휴리스틱 = “정확하진 않아도, 대부분의 경우 효과가 좋은 규칙 기반 최적화”

## ✔ 휴리스틱은 언제 쓰냐?

### 👉 **빠르게 끝내야 하는, 가벼운 쿼리(cheap queries)**

예:

`select * from student where id = 10;`

또는

`select count(*) from small_table;`

이런 경우:

- cost-based 최적화를 하면 오히려 더 느려진다.
    
- 그래서 간단하고 직관적인 규칙으로 최적화한다.
    

---

# 🔥 3️⃣ 구체적으로 휴리스틱은 어떻게 사용되는가?

대표적인 휴리스틱 규칙들:

---

## ✔ (1) Selection pushdown

먼저 필터링해라

`select * from A, B where A.age > 30 and A.id = B.id`

DBMS는 무조건:

- A에서 age > 30 먼저 적용
    
- 그다음 B와 조인
    
- 그게 빠르다는 걸 알기 때문에 비용 계산 없이 바로 push down
    

**왜 heuristic인가?**  
항상 좋은 건 아니지만, 대부분 좋은 경우가 많으니까.

---

## ✔ (2) Projection pushdown

필요한 컬럼만 가져오고 나머지는 버려라

optimizer는 보기만 해도 pushdown 함.

---

## ✔ (3) Join commutativity / associativity 적용

다음 두 개는 비용 평가 없이도 같은 결과를 만든다:

`A ⋈ B == B ⋈ A (A ⋈ B) ⋈ C == A ⋈ (B ⋈ C)`

이걸 이용해 조인 순서를 빠르게 단순화할 수 있음.

---

## ✔ (4) 작은 테이블을 먼저 스캔

DB는 다음 규칙을 휴리스틱으로 적용:

- 작은 테이블을 먼저 읽는 것이 대부분 유리
    
- 작은 쪽을 driving table로 사용
    
- 인덱스가 있는 쪽을 먼저 탐색
    

이건 비용 기반으로 더 정확하게 할 수 있지만,  
어차피 크기 차이가 분명하면 heuristic으로도 충분히 맞음.

---

## ✔ (5) OR 조건 간단화 (predicate simplification)

`where x = 5 or x = 6 or x = 7`

→ IN (5,6,7) 으로 rewrite  
→ 인덱스 사용 용이

이런 rewrite도 heuristic.

---

## ✔ (6) 복잡한 subquery를 조인으로 변환하는 decorrelation

아까 설명한 것처럼:

`where exists (...)`

→ join 으로 rewrite  
(옵티마이저가 자동으로 하기도 되지만, heuristic 기반 rewrite이기도 함)

---

# 🔥 4️⃣ 비싼 쿼리(expensive queries)는 어떻게 처리되나?

비싼 쿼리 = 큰 조인 많이 있는 쿼리

예:

`select * from A, B, C, D, E where ...`

이때 optimizer는:

### ✔ exhaustive enumeration (가능한 모든 계획을 나열)

### ✔ dynamic programming으로 best plan 선택

즉, 진짜 cost-based 최적화의 풀버전을 적용한다.

이게 시간이 오래 걸리기 때문에:

- 복잡한 쿼리에서만 수행
    
- cheap query에는 하지 않음
    

---

# 🔥 5️⃣ 왜 “비싼 쿼리”에 대해서는 cost-based가 필요할까?

예를 들어 A, B, C 세 개를 조인한다고 할 때:

`(A ⋈ B) ⋈ C A ⋈ (B ⋈ C) (B ⋈ A) ⋈ C ...`

조인 순서 하나만 달라도 결과 시간이 **100배 이상** 차이날 수 있음.

즉:

### ✔ 작은 쿼리는 휴리스틱으로도 충분히 빠름

### ✔ 큰 쿼리는 휴리스틱만으로는 판단 못함 → 비용 계산 필수

---

# 🔥 6️⃣ 전체 슬라이드 한 문장 요약

> **쿼리 최적화는 비용이 비싸므로,  
> cheap query에는 빠른 휴리스틱을 쓰고  
> expensive query는 cost-based + exhaustive search를 사용한다.**