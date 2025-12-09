# 1) Selectivity(선택도)

selectivity(θᵢ)는  
**튜플 하나가 조건 θᵢ를 만족할 확률**

- `sᵢ` : 조건 θᵢ를 만족하는 튜플 수
    
- `n_r` : relation r 의 전체 튜플 수
    

`selectivity(θᵢ) = sᵢ / n_r`

# 2) AND 조건 (Conjunction)

![[Pasted image 20251209214659.png]]
![[Pasted image 20251209214706.png]]

---

# 3) OR 조건 (Disjunction)

조건 θ₁ ∨ θ₂ ∨ … ∨ θₙ  
즉, 여러 조건 중 **하나라도 만족**하면 포함되는 경우.

원리:

- 먼저 “**모든 조건을 만족하지 않는 확률**”을 계산
    
- 이를 1에서 빼면 “하나라도 만족하는 확률”이 됨
    

결과 튜플 수 공식:

`n_r * ( 1 - (1 - s₁/n_r) * (1 - s₂/n_r) * ... * (1 - sₙ/n_r) )`
![[Pasted image 20251209214737.png]]

설명:

- (1 - sᵢ/nᵣ) = 조건 θᵢ를 **만족하지 않을 확률**
    
- 이것들을 전부 곱하면 “θ₁, θ₂, ..., θₙ 중 아무것도 충족하지 않을 확률”
    
- 마지막에 1 - (…) 하면 “하나라도 만족하는 확률”
    

---

# 4) NOT 조건 (Negation)

σ_{¬θ}(r)  
즉, 조건 θ 를 만족하지 않는 튜플의 수.

전체 nr에서 조건을 만족하는 튜플 개수를 빼면 됨.

`n_r - size(σ_θ(r))`

또는 selectivity를 이용하면:

`n_r * (1 - sel(θ))`

---

# 최종 요약 코드블록 (Notion 바로 붙여넣기)

# Selectivity
selectivity(θᵢ) = sᵢ / n_r

# Conjunction (AND 조건)
n_r * (s₁ / n_r) * (s₂ / n_r) * ... * (sₙ / n_r)
= n_r * (s₁ * s₂ * ... * sₙ) / (n_r^n)
= n_r * sel(θ₁) * sel(θ₂) * ... * sel(θₙ)

# Disjunction (OR 조건)
n_r * ( 1 - (1 - s₁/n_r) * (1 - s₂/n_r) * ... * (1 - sₙ/n_r) )

# Negation (NOT 조건)
n_r - size(σ_θ(r))
또는
n_r * (1 - sel(θ))
