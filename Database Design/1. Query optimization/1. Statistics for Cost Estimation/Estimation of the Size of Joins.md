# 1️⃣ **카티전 곱(Cartesian Product)**

조인 조건이 없으면 단순 곱집합이 된다.

`|r × s| = n_r * n_s`

각 튜플의 크기는:

`l_r + l_s`

---

# 2️⃣ **조인 속성이 없을 때 (R ∩ S = ∅)**

조인은 카티전 곱과 동일:

`|r ⋈ s| = n_r * n_s`

---

# 3️⃣ **조인 속성이 R의 Key일 때**

조인 속성이 R에서 key라면 R에는 중복이 없다.

따라서 **S의 각 튜플은 R의 최대 1개와 조인된다.**

`|r ⋈ s| ≤ n_s`

즉, 결과 크기는 **S의 튜플 수 이상 증가할 수 없다.**

---

# 4️⃣ **조인 속성이 S의 Foreign Key일 때**

S의 foreign key가 R의 primary key를 참조하는 경우:

- S의 각 튜플은 R의 정확히 1개와 조인됨
    
- 결과 크기는 S와 동일
    

`|r ⋈ s| = n_s`

예:  
`student ⋈ takes` 에서 `takes.ID → student.ID` (foreign key)

따라서:

`|student ⋈ takes| = n_takes`

---

# 5️⃣ **조인 속성이 Key도 Foreign Key도 아닌 경우**

조인 속성이 서로 중복될 수 있는 일반적인 경우.

조인 속성 A에 대해

- R 쪽 distinct 개수: V(A, r)
    
- S 쪽 distinct 개수: V(A, s)
    

튜플이 섞이는 정도를 균등 분포(uniform)로 가정해 추정한다.

---

## ✔ 경우 1: R 기준 추정

R의 각 튜플 t가 **S에서 평균적으로** 매칭되는 개수:

`n_s / V(A, s)`

따라서 전체 조인 크기:

`n_r * (n_s / V(A, s))`

---

## ✔ 경우 2: S 기준 추정

S의 각 튜플 t가 **R에서 평균적으로** 매칭되는 개수:

`n_r / V(A, r)`

따라서 전체 조인 크기:

`n_s * (n_r / V(A, r))`

---

## ✔ 최종 선택: 두 값 중 더 작은 값 선택

`min( n_r * n_s / V(A, s) ,  n_s * n_r / V(A, r) )`

왜 더 작은 걸 선택하나?  
→ 중복을 과하게 가정한 쪽보다 보수적인(덜 과장) 추정이 실제에 더 가깝기 때문.

---



# 6️⃣ **예제: student ⋈ takes** -> foreign key

학생 5000명  
takes 테이블 10,000개 튜플  
distinct 값:
`V(ID, takes) = 2500 V(ID, student) = 5000`

두 가지 추정:

`1) 5000 * 10000 / 2500 = 20,000 
`n_r * (n_s / V(ID, takes))`

`2) 10000 * 5000 / 5000 = 10,000`

둘 중 더 작은 값 = **10,000**

→ 실제 foreign key 정보로 계산한 값과 동일