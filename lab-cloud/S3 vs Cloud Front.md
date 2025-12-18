![[Pasted image 20251218170047.png]]
# CloudFront 사용 유무에 따른 웹 사이트 응답 시간 비교 실험

## 1. 실험 목적

S3 정적 웹 사이트에 대해  
**CloudFront(CDN)를 사용할 때와 사용하지 않을 때의 웹 리소스 로딩 시간 차이**를 비교하여  
CloudFront의 **캐싱 및 엣지 서버 효과**를 직접 확인한다.

---

## 2. 실험 환경 구성

### 공통 구성

- 정적 파일: `car.jpg` (약 8.7MB)
    
- Origin: Amazon S3 (정적 웹 호스팅 활성화)
    
- 측정 도구: 브라우저 Network 탭
    
- HTTP Status: 200 OK
    

---

### (1) CloudFront 미사용

`User → Internet → S3 (Static Website Endpoint)`

- S3 웹 사이트 엔드포인트 직접 접근
    
- 리전: AWS us-east-1 (N. Virginia)
    
- 캐시 없음 (요청마다 S3로 직접 접근)
    

---

### (2) CloudFront 사용

`User → Internet → CloudFront Edge → S3`

- CloudFront Distribution 생성
    
- Origin: S3
    
- Cache policy: CachingOptimized (정적 콘텐츠용)
    
- Edge Location에서 캐시 응답
    

---

## 3. 실험 결과 (응답 시간 비교)

### 🔹 CloudFront 미사용 (S3 직접 접근)

|요청|응답 시간|
|---|---|
|1|11.10 s|
|2|3.49 s|
|3|3.86 s|
|4|4.26 s|
|5|4.25 s|

**특징**

- 최초 요청이 매우 느림
    
- 모든 요청이 S3 원본까지 직접 도달
    
- 사용자와 S3 리전 간 거리(latency) 영향 큼
    

---

### 🔹 CloudFront 사용

|요청|응답 시간|
|---|---|
|1|357 ms|
|2|603 ms|
|3|420 ms|
|4|239 ms|
|5|242 ms|

**특징**

- 모든 요청이 1초 미만
    
- Edge 서버에서 캐시된 응답 제공
    
- 네트워크 지연 거의 없음
    

---

## 4. 결과 비교 요약

|항목|S3 직접 접근|CloudFront 사용|
|---|---|---|
|평균 응답 시간|약 4~5초|약 0.3~0.6초|
|최초 요청|매우 느림|빠름|
|이후 요청|매번 S3 접근|캐시 Hit|
|사용자 체감|느림|매우 빠름|
|서버 부하|S3 집중|Edge 분산|

👉 **CloudFront 사용 시 응답 시간이 약 10배 이상 단축됨**

---

## 5. 왜 이런 차이가 발생하는가?

### ① S3 직접 접근 시

- 모든 요청이 **미국 리전(us-east-1)** 까지 이동
    
- 네트워크 왕복 시간(RTT) 큼
    
- 캐시 없음 → 매번 동일한 파일 재전송
    

---

### ② CloudFront 사용 시

- 사용자의 위치와 가까운 **Edge Location**에서 응답
    
- 최초 1회만 S3 접근 (Cache Miss)
    
- 이후 요청은 **Edge Cache Hit**
    
- 전송 거리 감소 + 캐싱 효과
    

---

## 6. 캐시 관점에서의 해석

- `car.jpg`는 **정적 리소스**
    
- 요청 내용이 매번 동일
    
- CloudFront의 Cache Key 기준에서 동일 요청으로 판단
    
- 따라서 캐시 효율이 매우 높음
    

---

## 7. 실험을 통해 얻은 결론

1. **정적 웹 사이트 + 대용량 파일**은 CloudFront 사용 효과가 매우 큼
    
2. CloudFront는 단순한 “중계”가 아니라 **캐시 서버** 역할을 수행
    
3. 사용자 경험(UX)과 성능 개선에 직접적인 영향
    
4. 실제 서비스 환경에서는 **CloudFront + S3 조합이 사실상 표준 구조**