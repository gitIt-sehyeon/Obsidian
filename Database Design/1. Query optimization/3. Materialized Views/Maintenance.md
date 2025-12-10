# 📌 Materialized View 유지보수: 재계산 vs 증분 방식 비교

Materialized View는 **미리 계산된 결과를 물리적으로 저장해 두는 뷰**입니다. 이를 최신 상태로 유지하는 방식에는 **재계산(Recomputation)**과 **증분 뷰 유지보수(Incremental View Maintenance, IVM)** 두 가지가 있습니다.

아래에서 두 방식이 실제로 어떻게 다른지 **온라인 쇼핑몰 주문 데이터 시나리오**로 비교합니다.

---

# 📂 시나리오 개요

### ▸ 원본 테이블: `Orders`

|컬럼|설명|
|---|---|
|order_id|주문 ID|
|customer_id|고객 ID|
|order_date|주문 날짜|
|amount|주문 금액|

→ 수백만 건의 주문이 누적되어 있다고 가정.

---

### ▸ Materialized View: `DailySalesSummary`

`CREATE MATERIALIZED VIEW DailySalesSummary AS SELECT order_date, SUM(amount) AS total_daily_sales FROM Orders GROUP BY order_date;`

→ **날짜별 총 매출**을 저장하는 Materialized View

---

# 🔁 1. 재계산 방식 (Recomputation)

### ◼ 작동 방식

데이터 한 건이 바뀌어도 **전체 MV를 통째로 처음부터 다시 계산**함.

---

### ✔ 새 주문 1건 삽입 시 과정

1. Orders 테이블에 새 주문 1건 추가
    
2. Materialized View를 **완전히 삭제**
    
3. Orders의 **수백만 건** 전체 다시 스캔
    
4. 날짜별로 다시 **GROUP BY + SUM**
    
5. DailySalesSummary 전체 데이터를 다시 저장
    

---

### ❌ 비효율성

- **막대한 디스크 I/O**  
    → 전체 Orders 테이블 읽기
    
- **높은 CPU 사용량**  
    → 전체 그룹화 + 합산 재계산
    
- **긴 업데이트 시간**  
    → 그 동안 MV는 최신 상태가 아님
    
- **변경이 잦을수록 더 위험함**
    

---

# ⚡ 2. 증분 유지보수 (Incremental View Maintenance, IVM)

### ◼ 작동 방식

변경된 **그 데이터만** Materialized View에 반영.  
→ 전체 재계산 없음.

---

### ✔ 새 주문 1건 삽입 시 과정

예:  
`order_date = '2023-10-27'`, `amount = 10000` 주문 추가

1. 변경된 주문 **한 건만 확인**
    
2. Materialized View에서 `2023-10-27` 날짜 행을 찾음
    
    - 해당 날짜가 이미 존재하면 → `총합 + 10000`
        
    - 존재하지 않으면 → 새로운 행 삽입
        

---

### ✅ 효율성

- **I/O 최소**  
    → 변경된 1건만 읽고 반영
    
- **CPU 최소**  
    → 단순 덧셈/뺄셈만 수행
    
- **업데이트 속도 매우 빠름**
    
- **MV가 거의 즉시 최신 상태 유지**
    

---

# 📊 두 방식 비교 요약

|기준|재계산 (Recomputation)|증분 유지보수 (IVM)|
|---|---|---|
|처리 방식|전체 MV를 처음부터 다시 계산|변경된 데이터만 반영|
|I/O 비용|매우 높음|매우 낮음|
|CPU 비용|전체 집계 재계산|단순 연산|
|갱신 속도|느림|매우 빠름|
|대용량/빈번한 업데이트|매우 비효율적|최적|
|MV 최신성|지연 가능|거의 즉시 갱신|

---

# 📌 핵심 결론

재계산 방식은 **변경 1건에도 전체 데이터를 다시 스캔**하는 매우 비효율적인 방식입니다.  
반면 증분 유지보수는 **변경된 부분만 업데이트하므로** 빠르고, CPU·I/O 비용이 극적으로 낮아집니다.

따라서 **대규모 데이터베이스 시스템에서는 IVM이 사실상 필수**이며,  
현대 DBMS는 대부분 자동으로 증분 유지보수를 지원합니다.