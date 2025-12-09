# **Query Optimization Pipeline**

쿼리 최적화는 아래 네 단계로 이루어진 **파이프라인**이라고 보면 된다.

---

# 1) **Transformation of Relational Expressions**

### → "쿼리의 논리적 구조를 바꿔서 더 좋은 형태로 변환하는 단계"

쿼리를 **논리적으로 같은 의미를 유지하면서** 여러 형태로 바꾼다.

### 여기서 하는 일:

### 🔹 Equivalence rules (동등 변환 규칙)

예를 들어:

- selection pushdown  
    σ(A>5)(R ⨝ S) → (σ(A>5)(R)) ⨝ S
    
- 조인 교환  
    R ⨝ S = S ⨝ R
    
- 조인 결합  
    (R ⨝ S) ⨝ T = R ⨝ (S ⨝ T)
    

이런 규칙들을 이용해서 **동일한 의미의 여러 쿼리 형태** 만들기.

### 🔹 Enumeration (열거)

만들 수 있는 모든 동등한 쿼리 형태를 생성한다.

예:  
3개의 테이블 A, B, C를 조인한다고 하면 가능한 조인 순서:

- (A ⨝ B) ⨝ C
    
- (A ⨝ C) ⨝ B
    
- A ⨝ (B ⨝ C)
    
- …
    

→ 옵티마이저는 이런 “동등 표현들”을 모두 고려한다.

---

# 2) **Estimating Statistics of Expression Results**

### → "각 후보 계획이 얼마나 많은 데이터를 생성하는지 예측"

옵티마이저는 조인/셀렉션 등을 실행했을 때  
**얼마나 많은 튜플이 나오는지** 추정한다.

여기서 쓰는 정보:

### 🔹 Database-system catalog

- nr = 테이블의 총 튜플 수
    
- V(A,r) = 어떤 속성의 distinct 수
    
- fr = blocking factor
    
- br = block 수 등
    

### 🔹 Size estimation formulas

- selection selectivity
    
- join selectivity
    
- disjunction / conjunction 추정
    
- foreign-key/primary-key case
    
- histogram 사용 가능
    

즉, **이번 작업에서 실행 비용을 계산할 근거 데이터**를 준비하는 단계.

---

# 3) **Choice of Evaluation Plans**

### → "가능한 실행 계획 중에서 가장 비용이 낮은 것 선택"

여기서 진짜로 “어떤 방식으로 수행할까” 결정함.

### 옵티마이저 방식 두 가지:

### 🔹 Cost-based optimizer

각 계획의 비용을 숫자로 계산해서  
**가장 비용이 낮은 실행 계획을 선택**

예)

- nested loop join
    
- hash join
    
- sort-merge join
    
- full table scan
    
- index scan  
    등의 비용을 비교
    

### 🔹 Heuristics (경험적 규칙 기반)

"가벼운 쿼리"에서는 전체 탐색하면 오히려 비용이 크므로  
경험적 규칙으로 빠르게 결정

예)

- selection pushdown 먼저
    
- 가장 작은 테이블부터 조인
    
- projection early  
    등
    

---

# 4) **Materialized Views**

### → "쿼리 최적화를 더 빠르게 만드는 고급 기능"

여기는 고급 단계이고 두 가지를 포함한다.

---

### 🔹 4-1. Incremental View Maintenance

Materialized View 의 값을  
데이터 변경 시 빠르게 업데이트하는 법

- selection insert/delete 유지
    
- join insert/delete 유지
    
- projection count 방식 유지
    
- aggregation(sum, count, avg, min, max)
    
- set operations(intersection, union, diff)
    

이제 이 정보도 비용 평가에 사용될 수 있음

---

### 🔹 4-2. Materialized view and index selection

"자주 쓰이는 쿼리를 빠르게 하려면 어떤 뷰를 저장해 둘까?"  
"어떤 인덱스를 자동으로 만들까?"

→ 즉, DB 자체가 **미리 캐싱할 전략을 선택**

---

# 🔥 전체 프로세스를 한 문장으로 요약하면

> DBMS 옵티마이저는  
> **쿼리 → 여러 형태로 변환 → 결과 크기 추정 → 비용 비교 → 가장 빠른 실행 계획 선택 → 필요시 Materialized View 활용**  
> 을 수행한다.

---

# 📌최종 요약표

1) Logical Transformation    - Equivalence rules    - Create all logically equivalent expressions  
2) Statistics Estimation    - System catalog info    - Selectivity, join size estimation, histograms  
3) Plan Choice    - Cost-based optimization    - Heuristics for cheap queries  
4) Materialized Views    - Incremental maintenance    - MV + index selection