# 📌 Aggregation Operations(집계 연산)의 Incremental View Maintenance

Aggregation view 를 유지할 때 중요한 핵심은:

> **aggregation 결과는 원본 튜플 여러 개가 함께 만들어내는 값**  
> → projection처럼 “하나의 튜플”에 의존하지 않음  
> → 증분 유지가 필요

---

# ✅ 1) COUNT Aggregation

### ✔ 정의

`v = Agcount_B(r)`

- 그룹별로 B 값에 해당하는 튜플 개수(count)를 유지
    

---

## ✔ INSERT(ir)

ir의 각 튜플 t에 대해:

1. t.A(그룹 key)를 확인
    
2. 이미 해당 그룹이 존재하면 count += 1
    
3. 그룹이 없으면 count = 1로 새로운 그룹 생성
    

---

## ✔ DELETE(dr)

dr의 각 튜플 t에 대해:

1. 해당 그룹의 count -= 1
    
2. count가 0이면 → 그룹을 결과에서 삭제
    

---

# 🔥 count 가 필요한 이유

Projection과 같은 원리로,

**집계 결과 하나는 원본 튜플 여러 개로부터 파생될 수 있음.**

따라서 그룹을 삭제해도 되는지 판단하려면  
그 그룹에 속한 튜플의 **실제 개수(count)**를 알아야 한다.

---

# ✅ 2) SUM Aggregation

### ✔ 정의

`v = Agsum_B(r)`

- 그룹별로 B 값의 합계를 유지
    

---

## ✔ INSERT(ir)

`sum[group] += t.B / count[group] += 1   (반드시 함께!)`

## ✔ DELETE(dr)

`sum[group] -= t.B / count[group] -= 1 if count == 0 → 그룹 삭제`

---

# ❗ 왜 “합계가 0인지 보면 그룹이 비었는지 판단할 수 없을까?”

예)

|B|
|---|
|-5|
|5|

합계 = 0  
하지만 튜플은 **2개 존재함**

→ sum = 0 이 되는 것은 “그룹이 비었다”는 의미가 아님  
→ 따라서 반드시 **count를 유지해야만** 그룹 존재 여부 판단 가능

---

# ✅ 3) AVG Aggregation

AVG는 따로 유지할 수 없고,

`avg = sum / count`

따라서 업데이트 시:

`sum += t.B -> count += 1 -> avg = sum / count`

---

# ✅ 4) MIN / MAX Aggregation

### ✔ 정의

`v = Agmin_B(r) / v = Agmax_B(r)`

---

## ✔ INSERT(ir)

간단함.

`if t.B < current_min → min = t.B`

또는 max는 반대로.

---

## ✔ DELETE(dr) — 어려운 부분 ⚠

삭제된 튜플이 **그룹의 최소값 또는 최대값이 아니면** → 아무 문제 없음.

하지만,

### ❗ 삭제된 튜플이 현재 min/max 였다면?

→ 해당 그룹의 나머지 모든 튜플을 다시 스캔해야 함  
→ 새로운 min/max를 찾아야 함  
→ 매우 비용이 큼

---

# 🧊 왜 count처럼 간단하게 해결할 수 없는가?

min/max는 수학적으로 "여러 줄에서 하나만 가져오는" 집계라서  
“invertible(역전)” 연산이 아님.

즉:

`min({a,b,c}) - a = ?`

을 계산하려면 b, c를 체크해야 함 (모든 값 스캔해야 함)

그래서 delete가 비싸다.

---

# ✅ 5) Set Intersection (교집합)

정의:

`v = r ∩ s`

---

## ✔ r에 튜플 삽입

튜플 t가 r에 들어옴 →  
s에도 t가 존재한다면:

`v_new = v_old ∪ {t}`

---

## ✔ r에서 튜플 삭제

튜플 t가 삭제됨 →  
t가 v에 있다면:

`v_new = v_old − {t}`

---

## ✔ s가 업데이트될 때도 동일(대칭)

---

# 🔥 다른 집합 연산(update 방식)

### ✔ 합집합 union

`v = r ∪ s`

INSERT(t in r):  
→ v에 추가 (중복은 허용 안 됨)

DELETE(t in r):  
→ s에 존재한다면 유지  
→ s에 없으면 제거

---

### ✔ 차집합 difference

`v = r − s`

INSERT(t in r):  
→ t가 s에 없으면 v에 추가

DELETE(t in r):  
→ v에서 삭제

---

전체적으로 보면,

# ⭐ 결과 요약표 (Notion-friendly)

|연산|Insert 처리|Delete 처리|난이도|
|---|---|---|---|
|Selection|조건 만족 시 추가|조건 만족 시 제거|쉬움|
|Join|ir ⨝ s 추가|dr ⨝ s 제거|쉬움|
|Projection|count 유지 필요|count=0 되면 삭제|보통|
|Count|count++|count-- → 0이면 제거|쉬움|
|Sum|sum+=B, count++|sum-=B, count--|보통|
|Avg|sum, count 유지|sum/count로 계산|보통|
|Min/Max|min/max 조정|삭제 시 재계산 필요|어려움|
|Intersect|s에 있으면 추가|v에 있으면 제거|쉬움|