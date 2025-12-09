[[Maintenance]]

# 🧱 **Materialized View란? (물질화 뷰)**

**쿼리 결과를 계산해두고, 그 결과를 실제 테이블처럼 디스크에 저장해둔 것.**

즉,

> 보통 View = **쿼리를 저장**  
> Materialized View = **쿼리 결과를 저장**

---

# 🔍 **일반 View와 비교해서 보면 딱 이해됨**

### ✔ **일반 View (Virtual View)**

- 정의만 저장됨 (`CREATE VIEW ... AS SELECT ...`)
    
- 매 조회마다 **원본 테이블에 쿼리를 다시 실행함**
    
- 저장 공간 거의 안 듦
    
- 하지만 **쿼리가 비싸면 조회할 때 매우 느림**
    

예:

`CREATE VIEW high_salary AS SELECT * FROM employee WHERE salary > 100000;`

조회할 때마다 employee 테이블을 다시 스캔함.

---

### ✔ **Materialized View**

- 뷰 정의뿐 아니라 **실제 데이터도 저장함**
    
- 조회할 때는 원본 테이블 접근 없이 **미리 계산된 결과만 사용 → 매우 빠름**
    
- 하지만 단점:
    
    - 원본 테이블이 변경되면 materialized view도 **갱신(refresh)** 해야 함
        
    - 저장 공간 많이 사용
        

예:

`CREATE MATERIALIZED VIEW high_salary_mv AS SELECT * FROM employee WHERE salary > 100000;`

employee가 수정되면 high_salary_mv를 새로 계산(refresh)해야 함.

---

# 📌 왜 Materialized View를 쓰는가?

## 🔥 1) 성능 향상

특히 다음과 같은 경우에 효과 폭발:

- Aggregate-heavy query  
    (SUM, AVG, GROUP BY)
    
- Join-heavy query  
    (여러 테이블 조인)
    
- 대규모 데이터 분석(OLAP)
    
- 반복 실행되는 쿼리  
    (대시보드, 리포트)
    

예를 들어:

`SELECT region, SUM(sales)  FROM big_sales  GROUP BY region;`

이걸 하루에 **수백 번** 실행해야 한다면 Materialized View 사용 → 즉시 조회 가능.

---

## 🔥 2) 캐싱(Caching)

Materialized View = **DB 내부에서 제공하는 강력한 캐시**  
Redis 같은 캐시지만 SQL 기반이고 자동 관리 가능.

---

# 📌 Materialized View 갱신(Refresh)

### ✔ **Manual Refresh**

개발자가 직접 갱신:

`REFRESH MATERIALIZED VIEW high_salary_mv;`

### ✔ **Automatic Refresh (DB마다 다름)**

- Oracle: Fast refresh / Complete refresh 지원
    
- PostgreSQL: 기본적으로 manual only
    
- BigQuery: 자동 materialized view
    
- Snowflake: 자동 유지
    

---

# 📌 Materialized View 예시로 이해하기

### 예: 하루 판매량 집계

원본 테이블: `orders` (수백만 건)

`CREATE MATERIALIZED VIEW daily_sales AS SELECT date(order_time) as day, SUM(price) FROM orders GROUP BY day;`

➡️ BI 대시보드에서 빠르게 조회 가능

원본 테이블에 insert/delete가 생기면  
**materialized view를 refresh**해야 최신 값 유지 가능.

---

# 📌 Materialized View는 언제 쓰면 안 될까?

- 실시간 데이터가 매우 중요할 때  
    (refresh 전까지는 outdated data)
    
- 원본 테이블 변경이 너무 잦을 때  
    (refresh 비용이 너무 큼)
    
- 저장 공간이 부족할 때
    

---

# ✔ 핵심 요약

| 개념     | View        | Materialized View |
| ------ | ----------- | ----------------- |
| 저장?    | ❌ 쿼리 정의만 저장 | ✔ 실제 결과를 저장       |
| 조회 속도  | 느림          | 매우 빠름             |
| 갱신 필요? | ❌           | ✔ 필요(Refresh)     |
| 쓰는 곳   | 일반 조회       | 집계/조인/대규모 분석      |