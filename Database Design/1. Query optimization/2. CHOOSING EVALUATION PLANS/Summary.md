# 🔥 1. Physical equivalence rules란?

논리적 쿼리 플랜(Logical plan)을  
**실제로 실행 가능한 물리적 쿼리 플랜(Physical plan)으로 바꾸는 규칙**을 의미해.

### 예를 들어 “Join”이라는 논리 연산은 물리적으로 여러 방식이 있음:

|논리 연산|물리 연산(알고리즘)|
|---|---|
|Join|Hash join|
|Join|Nested-loop join|
|Join|Sort-merge join|
|Selection|Index scan|
|Selection|Sequential scan|

즉,

> “Join”이라는 하나의 논리적 연산 → 여러 개의 실제 실행 알고리즘으로 변환할 수 있음.

이 변환 규칙이 **physical equivalence rules**야.

---

# 🔥 2. 왜 equivalence rules가 중요한가?

쿼리 옵티마이저는:

- 가능한 모든 **논리적 쿼리 변형**을 만들고 (join 순서 변경, selection pushdown 등)
    
- 거기에 적용할 수 있는 **물리적 알고리즘들**까지 조합함
    

하지만 이걸 "그냥" 하면 **조합 폭발(Combinatorial explosion)** 때문에  
수천~수백만 개의 후보 플랜이 나올 수 있어.

그래서 효율화를 위해 다음 기술들이 필요함.

---

# 🔥 3. Efficient optimizer가 필요로 하는 기술 4가지

슬라이드는 4개의 핵심 요소를 말함.

하나씩 쉽게 설명할게.

---

## ✅ (1) space-efficient representation

> “같은 서브쿼리/서브트리를 여러 번 복사하지 말고 한 번만 저장하자.”

예:

`(A ⋈ B) ⋈ C   A ⋈ (B ⋈ C)`

실제로는 `(A ⋈ B)` 같은 부분식이 여러 플랜에서 반복 사용됨 →  
**그래프 형태로 표현하면 중복 저장 방지 가능 (DAG 구조)**

이게 중요한 이유:

- 메모리 절약
    
- 서브플랜 재사용 가능
    
- 중복 계산 방지
    

---

## ✅ (2) detecting duplicate derivations

> “같은 결과를 내는 식을 다시 만들지 않도록 중복 계산을 탐지한다.”

예:

- `A ⋈ B`를 이미 계산했는데  
    다른 경로에서 또 `A ⋈ B`를 만들려고 하면 → 버림(pruning)
    

DB가 해야 하는 일:

- 식을 normalized form으로 변환하여 비교  
    (예: commutativity `A ⋈ B == B ⋈ A`)
    

이걸 해야 search space가 줄어듦.

---

## ✅ (3) dynamic programming + memorization

이게 핵심!!!!

### DBMS 쿼리 옵티마이저의 근본 원리:

> “하위 문제(subset)의 Best plan을 먼저 구한 뒤 저장해두고,  
> 상위 문제에서 재사용한다.”

여기서 memorization = 캐싱

즉:

|Subset|Best Plan 저장?|
|---|---|
|{A}|저장|
|{B}|저장|
|{A,B}|저장|
|{A,B,C}|저장|

그래서 상위 플랜 계산할 때 다시 계산하지 않음.

이게 없으면 **DP 없이 brute-force 탐색 → 수백만 플랜**이 생성됨.

---

## ✅ (4) Cost-based pruning

> “비용이 너무 커서 최적이 될 가능성이 없는 플랜은 조기 제거한다.”

예:

- A ⋈ B ⋈ C 를 계산할 때
    
- {A,B} 단계의 어떤 물리적 조인 알고리즘이 비용 1000이 나옴
    
- 하지만 이미 비용이 200인 {A,B} 다른 플랜이 있다면
    

→ 비용 1000짜리 플랜은 버림 (prune)

이렇게 pruning을 해야 search space가 폭발하지 않고 계산 가능해짐.

---

# 📌 전체 그림 요약

### Logical plan → 여러 Physical plan 후보 생성

(physical equivalence rules 사용)

하지만 후보 수가 너무 많기 때문에 옵티마이저는 다음을 사용:

- DAG 구조로 표현하여 space 절약
    
- 같은 서브식은 duplicate detection으로 제거
    
- DP + memorization로 하위 결과 재사용
    
- pruning으로 비용 높은 후보 early 제거
    

이 네 가지가 있어야 commercial DBMS가 **현실적인 시간 안에** 최적화 수행 가능함.

---

# ✔ 쉽게 비유

### ❌ 나쁜 방식:

모든 플랜을 매번 처음부터 끝까지 싹 다 만들어본다  
→ 미친 연산량 → 불가능

### ✔ 좋은 방식:

- 똑같은 조각은 재사용
    
- 비싸서 후보가 될 수 없는 건 초반부터 제거
    
- 조각별로 best plan을 캐싱
    
- 같은 조각은 그래프 형태로 공유
    

즉, **"계산할 것은 최소로, 재사용할 것은 최대한 재사용"** 하는 방식.