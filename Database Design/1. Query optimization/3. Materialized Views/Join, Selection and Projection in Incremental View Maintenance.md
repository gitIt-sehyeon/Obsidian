# ✅ 1) Selection View 의 Incremental Maintenance

뷰 정의:

`v = σθ(r)`

선택(selection)은 **튜플 단위로 독립적**으로 결정되므로 관리가 매우 쉽다.

## ✔ INSERT (새 튜플 ir이 r에 들어옴)

`v_new = v_old ∪ σθ(ir)`

- 조건 θ 를 만족하면 결과 뷰에 **추가**
    
- 만족하지 않으면 **무시**
    

## ✔ DELETE (튜플 dr이 r에서 삭제됨)

`v_new = v_old − σθ(dr)`

- 조건 θ 를 만족하는 튜플만 삭제하면 됨
    
- 투영처럼 "다른 튜플 때문에 남아있어야 하는지" 고민할 필요 없음
    

➡️ selection 은 **튜플 개별로 판단 가능 → 가장 쉬운 뷰 유지**

---

# ✅ 2) Join View 의 Incremental Maintenance

뷰 정의:

`v = r ⨝ s`

새로 들어온 튜플은 **반대편 관계와만 조인해주면 됨.**

## ✔ INSERT (r에 ir이 삽입됨)

새 튜플 ir 이 들어오면:

`v_new = v_old ∪ ( ir ⨝ s )`

즉:

- ir과 s의 모든 매칭되는 튜플들을 조인해서
    
- 그 결과를 뷰에 추가
    

## ✔ DELETE (r에서 dr 삭제)

`v_new = v_old − ( dr ⨝ s )`

- 삭제된 튜플 dr 과 매칭되는 조인 결과만 제거하면 됨
    
- projection 처럼 “다른 튜플 때문에 남아 있어야 하는지” 따질 필요 없음
    

➡️ join 도 selection처럼 튜플 단위로 “독립적”이므로 간단.

---

# ⚠️ 3) Projection View 가 복잡한 이유

뷰 정의:

`v = π_A(r)`
A가 첫번째 열이라고 가정
### ✔ 문제의 핵심

특정 원본 튜플이 삭제되어도  
**projection 결과는 삭제되면 안 될 수도 있음.**

예:

`r = {(a,2), (a,3)} π_A(r) = {a}`

- (a,2) 삭제 → 여전히 (a,3)이 있으므로 `{a}` 유지해야 함
    
- (a,3)까지 삭제 → 이제서야 `{a}` 제거 가능
    

즉, projection 결과 **하나의 투영 튜플이 여러 원본 튜플에 의해 지지됨**

그래서 selection/join처럼 간단히

`v_old - π_A(dr)`

할 수 없음.

---

# ⭐ 해결: 투영 튜플에 대한 “count(지지 개수)” 유지

Projection view 를 유지하려면:

`(투영된 값 a) → r에 있는 (a로 시작하는 튜플)의 개수`

이 개수를 저장해야 함

## ✔ INSERT (ir이 r에 삽입됨)

1. ir을 투영 → a 라고 하자
    
2. 이미 a가 존재하면 count + 1
    
3. 없으면 새로 추가하고 count = 1
    

## ✔ DELETE (dr이 r에서 삭제)

1. dr을 투영 → a
    
2. count – 1
    
3. count = 0 이 되면 뷰에서 (a) 삭제
    

➡️ Projection은 **지지 개수를 유지해야만** incremental 유지가 가능하다.

---

# 🔥 전체 요약

---

## ✅ Selection View Maintenance

`v = σθ(r)  INSERT ir:     v_new = v_old ∪ σθ(ir) DELETE dr:     v_new = v_old − σθ(dr)`

✔ 독립적 → 단순

---

## ✅ Join View Maintenance

`v = r ⨝ s  INSERT ir:     v_new = v_old ∪ (ir ⨝ s) DELETE dr:     v_new = v_old − (dr ⨝ s)`

✔ 조인은 “상대 관계와의 매칭 결과만” 추가/삭제 → 단순

---

## ⚠️ Projection View Maintenance (복잡)

`v = π_A(r)`

Projection 튜플은 여러 원본 튜플에게 의해 만들어질 수 있으므로  
삭제 시 단순히 빼면 안 됨.

### ✔ 해결: count 유지

INSERT ir:

- a = π_A(ir)
    
- count(a)++
    
- count(a) = 1 이면 뷰에 추가
    

DELETE dr:

- a = π_A(dr)
    
- count(a)--
    
- count(a) = 0 이면 뷰에서 제거