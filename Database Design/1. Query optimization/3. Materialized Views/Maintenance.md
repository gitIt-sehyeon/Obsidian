Materialized View(머티리얼라이즈드 뷰)는 데이터베이스에서 미리 계산하여 저장해 둔(computed and stored) 뷰를 의미합니다. 일반적인 뷰는 쿼리가 실행될 때마다 정의된 내용을 동적으로 계산하지만, Materialized View는 한 번 계산된 결과를 물리적으로 저장해 두므로, 이후 쿼리에서는 이 저장된 결과를 바로 사용할 수 있습니다.

# Materialized View의 정의 및 예시:

제공된 예시 create view department_total_salary(...)는 instructor 테이블에서 각 부서(dept_name)별 급여(salary)의 합계를 계산하는 뷰입니다.
이 뷰를 Materialized View로 만들면, dept_name별 total_salary 정보가 데이터베이스에 실제 테이블처럼 저장됩니다.
따라서 "부서별 총 급여"와 같이 자주 조회되는 복잡한 집계 쿼리에 대해 매번 원본 테이블에서 데이터를 읽고 계산하는 대신, 미리 계산된 결과를 바로 반환하여 성능을 크게 향상시킬 수 있습니다. 이는 여러 튜플을 찾아 금액을 합산하는 노력을 절약해 줍니다.


# Materialized View Maintenance (유지보수):

Materialized View의 핵심 과제 중 하나는 원본 데이터(underlying data)가 변경될 때 Materialized View의 내용을 **최신 상태로 유지(up-to-date)**하는 것입니다. 이를 Materialized View 유지보수라고 합니다.
유지보수 전략:
### 재계산 (Recomputation): 
가장 간단한 방법으로, 원본 데이터가 업데이트될 때마다 Materialized View의 내용을 처음부터 다시 계산하여 갱신하는 방식입니다. 데이터 양이 많거나 변경 빈도가 잦으면 비효율적일 수 있습니다.
### 증분 뷰 유지보수 (Incremental View Maintenance):
더 효율적인 방법으로, 원본 데이터의 변경 사항(삽입, 삭제, 업데이트 등)만을 추적하여 Materialized View에 반영하는 방식입니다.
이는 전체 뷰를 재계산하는 것보다 훨씬 적은 비용으로 뷰를 최신 상태로 유지할 수 있습니다.
과거에는 개발자가 이러한 증분 업데이트를 위해 수동으로 트리거(triggers)를 정의하거나 코드를 작성해야 했습니다.


## 현대 데이터베이스 시스템의 지원:
최신 데이터베이스 시스템은 Materialized View가 선언되면 그 내용을 계산하고, 원본 데이터가 변경될 때 자동으로 증분 업데이트하는 기능을 제공합니다.
이는 개발자가 복잡한 유지보수 로직을 직접 구현할 필요 없이, 선언적인 방식으로 Materialized View를 관리할 수 있게 해줍니다.


## 뷰 유지보수 시점:

### 즉시 유지보수 (Immediate View Maintenance): 
대부분의 데이터베이스 시스템에서 채택하는 방식으로, 원본 데이터가 업데이트되는 즉시, 해당 업데이트 트랜잭션의 일부로 Materialized View도 함께 갱신됩니다. 이는 뷰의 내용이 항상 최신 상태임을 보장합니다.
### 지연 유지보수 (Deferred View Maintenance): 
일부 데이터베이스 시스템은 뷰 유지보수를 나중으로 연기하여 특정 시간 간격이나 수동으로 트리거될 때 일괄적으로 갱신하는 방식을 지원합니다. 이는 원본 데이터 변경 시 트랜잭션 부하를 줄일 수 있지만, 뷰의 데이터가 잠시 동안 최신이 아닐 수 있습니다.



### 💬 증분 뷰 유지보수(Incremental View Maintenance) 방식이 재계산(Recomputation) 방식보다 어떻게 더 효율적인지 구체적인 데이터 변경 시나리오를 들어 설명해 주실 수 있나요?

구체적인 시나리오를 통해 설명해 드리겠습니다.
시나리오: 온라인 쇼핑몰의 일별 매출 합계 뷰

원본 테이블: Orders (주문 정보)

컬럼: order_id, customer_id, order_date, amount (주문 금액)
가정: 이 테이블에 수백만 건의 주문 데이터가 누적되어 있습니다.


Materialized View: DailySalesSummary (일별 총 매출)

정의: 특정 날짜의 모든 주문 금액을 합산하여 총 매출을 보여주는 뷰입니다.
CREATE MATERIALIZED VIEW DailySalesSummary AS SELECT order_date, SUM(amount) AS total_daily_sales FROM Orders GROUP BY order_date;



이제 Orders 테이블에 새로운 주문 데이터가 추가될 때, 두 가지 유지보수 방식의 차이를 살펴보겠습니다.

1. 재계산 (Recomputation) 방식

작동 방식: Orders 테이블에 새로운 주문이 들어오면, DailySalesSummary Materialized View를 갱신하기 위해 뷰를 정의하는 쿼리(SELECT order_date, SUM(amount) FROM Orders GROUP BY order_date)를 처음부터 끝까지 다시 실행합니다.

단계별 과정 (새 주문 1건 추가 시):

새로운 주문 1건이 Orders 테이블에 삽입됩니다.
Materialized View 갱신 요청이 발생하면, 데이터베이스는 DailySalesSummary Materialized View의 기존 내용을 완전히 삭제합니다.
데이터베이스는 Orders 테이블의 수백만 건에 달하는 모든 기존 주문 데이터와 방금 추가된 1건의 새로운 주문 데이터를 전부 다시 스캔합니다.
스캔된 모든 데이터를 order_date별로 그룹화하고, 각 그룹의 amount를 합산하는 SUM 연산을 모든 날짜에 대해 다시 수행합니다.
새로 계산된 전체 DailySalesSummary 데이터를 Materialized View에 다시 저장합니다.


비효율성:

막대한 I/O 발생: Orders 테이블의 모든 데이터를 읽어야 하므로 디스크 I/O가 매우 많이 발생합니다.
높은 CPU 부하: 모든 주문 데이터를 다시 그룹화하고 합산해야 하므로 CPU 자원을 많이 소모합니다.
긴 갱신 시간: 데이터의 양이 많을수록 갱신에 소요되는 시간이 길어져, 뷰가 최신 상태를 유지하지 못하는 시간이 발생하거나 시스템 성능 저하를 유발할 수 있습니다.




2. 증분 뷰 유지보수 (Incremental View Maintenance, IVM) 방식

작동 방식: Orders 테이블에 변경(삽입, 삭제, 업데이트)이 발생하면, 해당 변경이 DailySalesSummary Materialized View에 미치는 영향만을 계산하여 뷰에 반영합니다.

단계별 과정 (새 주문 1건 추가 시):

새로운 주문 1건 (order_date = '2023-10-27', amount = 10000)이 Orders 테이블에 삽입됩니다.
데이터베이스는 이 새로운 주문 1건이 Materialized View에 미칠 영향만을 식별합니다.
DailySalesSummary 뷰에서 해당 order_date('2023-10-27')에 해당하는 레코드를 찾습니다.
만약 이미 '2023-10-27' 날짜의 매출이 기록되어 있다면 (예: total_daily_sales = 50000): 해당 레코드의 total_daily_sales 값을 50000 + 10000 = 60000으로 업데이트합니다.
만약 '2023-10-27' 날짜의 매출이 아직 기록되지 않았다면: 새로운 레코드 (order_date = '2023-10-27', total_daily_sales = 10000)를 DailySalesSummary 뷰에 삽입합니다.




효율성:

최소한의 I/O 발생: Orders 테이블에서는 변경된 레코드 1건만 읽고, DailySalesSummary 뷰에서는 해당 날짜의 레코드 1건만 읽거나 업데이트/삽입합니다. 전체 테이블을 스캔할 필요가 없습니다.
낮은 CPU 부하: 전체 데이터에 대한 그룹화 및 합산 연산 대신, 변경된 데이터에 대한 간단한 덧셈/뺄셈 연산만 수행합니다.
빠른 갱신 시간: 변경이 발생하자마자 거의 즉시 뷰가 갱신되므로, 뷰의 데이터가 항상 최신 상태를 유지하며 쿼리 성능에 미치는 영향이 최소화됩니다.




결론적으로, 재계산 방식은 데이터베이스에 변경이 일어날 때마다 "모든 것을 다시 시작"하는 반면, 증분 뷰 유지보수 방식은 "변경된 부분만 수정"합니다. 따라서 데이터의 양이 많고 변경이 빈번하게 일어나는 환경일수록 증분 뷰 유지보수가 압도적으로 효율적이며, Materialized View의 실질적인 성능 향상 효과를 가능하게 합니다.
