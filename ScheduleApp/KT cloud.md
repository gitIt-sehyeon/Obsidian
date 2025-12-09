## 1. KT cloud 어떤 회사 느낌이냐?

KT에서 분사한 **전문 클라우드 회사**라서, 사업 자체가  
“클라우드 인프라 팔고, 컨테이너·DB·보안·운영 자동화 서비스 제공하는 것”이야.[KT Cloud+1](https://cloud.kt.com/?utm_source=chatgpt.com)

실제 **신입/IT직군 공채 공고**를 보면:

- **클라우드 서비스 개발 (Cloud Native, AI, DevOps 분야)**
    
- **클라우드 자동화 플랫폼 개발**
    
- **대규모 웹 어플리케이션 및 서버 개발**  
    이런 일을 한다고 써있고[KT Cloud 채용+1](https://career.ktcloud.com/recruit.html?utm_source=chatgpt.com)
    

인프라 쪽은

- **OpenStack, Kubernetes(컨테이너), SDN/SDS 기반 IaaS 설계/운영**
    
- **대규모 Public Cloud 엔지니어링**[케이티클라우드](https://www.ktcloud.com/static/pdf/common/kt%20cloud%20%2724.2H%20%EA%B2%BD%EB%A0%A5%EC%A7%81%20%EC%B1%84%EC%9A%A9%20%EC%83%81%EC%84%B8%20%EC%A7%81%EB%AC%B4%EC%86%8C%EA%B0%9C_1.%20TECH.pdf?utm_source=chatgpt.com)
    

그리고 자체 **Kubernetes 플랫폼(K2P), 서버리스 컨테이너(KCI, FlyingCube)** 같은 것도 직접 서비스 중.[KT Cloud+4Cloud 매뉴얼+4Cloud 매뉴얼+4](https://manual.cloud.kt.com/kt/k2p-ent-info?utm_source=chatgpt.com)

=> **즉, “클라우드 인프라 + K8s + 자동화 + 모니터링”을 이해하고, 그걸 코드로 다룰 줄 아는 사람**을 좋아할 확률이 매우 높음.

---

## 2. 그래서 어떤 프로젝트가 좋냐? (KT cloud 맞춤형)

### 🎯 컨셉:

> **“KT cloud 스타일의 인프라 자동 관리 + 모니터링 플랫폼”**
> 
> - 그 위에서 돌아가는 **심플한 웹 서비스(Todo/작업관리)**
>     

기능은 너무 복잡할 필요 없고,  
**핵심은 ‘인프라를 어떻게 클라우드 네이티브하게 설계·자동화했는지’**야.

---

## 3. 프로젝트 한 줄 정의

> **KT Cloud Resource Manager & Todo Platform**  
> “Todo 웹 서비스를 운영하면서, 그 인프라 자체를  
> ▶ 자동으로 배포하고  
> ▶ 리소스를 모니터링하고  
> ▶ 부하에 맞춰 자동으로 스케일링하는 플랫폼”

이 프로젝트 하나에 **KT cloud가 좋아할 키워드** 다 들어가:

- Cloud Native
    
- Kubernetes (K2P / K8s Service)[Cloud 매뉴얼+2Cloud 매뉴얼+2](https://manual.cloud.kt.com/kt/k2p-ent-info?utm_source=chatgpt.com)
    
- 서버리스 컨테이너(KCI)[kt cloud Tech Blog+1](https://tech.ktcloud.com/entry/%EA%B5%AD%EB%82%B4-%EC%B5%9C%EC%B4%88-%EC%84%9C%EB%B2%84%EB%A6%AC%EC%8A%A4-%EC%BB%A8%ED%85%8C%EC%9D%B4%EB%84%88-%EC%84%9C%EB%B9%84%EC%8A%A4-KCI-K2P-2%ED%8E%B8?utm_source=chatgpt.com)
    
- CI/CD
    
- Terraform (IaC)
    
- Observability(모니터링/로그)
    
- 자동화 플랫폼 성격
    

---

## 4. 어떤 기능을 가진 “웹 서비스”로 할까?

앱보다는 **웹**이 훨씬 좋아.  
인프라, 배포, 쿠버네티스 연계가 웹 + API 구조가 제일 편해서.

### ✅ 사용자 기능 (웹 Todo 서비스)

프론트 기준, 유저가 볼 수 있는 건 딱 이 정도면 충분:

1. **회원가입 / 로그인**
    
    - 이메일 + 비밀번호 (JWT)
        
2. **Todo 관리**
    
    - Todo 생성/수정/삭제
        
    - 마감일, 우선순위, 태그(예: 공부/운동/개발)
        
3. **대시보드**
    
    - 오늘 해야 할 일
        
    - 이번 주 완료 개수
        
    - 태그별 Todo 통계 간단 그래프
        

여기까지는 그냥 평범한 웹앱.

진짜 포인트는 **이걸 어떻게 클라우드에 올려서 자동으로 잘 돌게 만들었냐** 쪽이야.

---

## 5. 백엔드 구조 (MSA로 살짝 나누기)

### 📌 서비스 분리 예시

1. **auth-service**
    
    - 로그인/회원가입/JWT
        
2. **todo-service**
    
    - Todo CRUD, 태그, 통계용 데이터 쿼리
        
3. **stats-service**
    
    - 주간/월간·태그별 통계 계산 (배치/비동기)
        

이렇게 3개 정도로 쪼개두면  
“Kubernetes에서 여러 마이크로서비스 운영”이라는 스토리가 생김.

### 📌 추천 백엔드 스택

- **Java + Spring Boot** (이미 잘함, KT도 많이 씀)
    
- Spring Security + JWT
    
- DB: MySQL(or MariaDB)
    
- 통계/배치 쪽은
    
    - 그냥 Spring Batch
        
    - 또는 메시지 큐 붙이고 싶으면 Kafka/RabbitMQ (옵션)
        

---

## 6. 인프라/클라우드 쪽이 진짜 메인

여기가 KT cloud 포폴 핵심.

### ✅ 1) Kubernetes (KT Cloud K2P or 직접 K8s 클러스터)

KT cloud는 **K2P(Kubernetes Pack)**, **Kubernetes Service**, **KCI/FlyingCube** 같은 컨테이너 서비스로 컨테이너 환경을 자동 구성해줘.[kt cloud Tech Blog+4Cloud 매뉴얼+4Cloud 매뉴얼+4](https://manual.cloud.kt.com/kt/k2p-ent-info?utm_source=chatgpt.com)

너 포폴에서는:

- 각 서비스(auth/todo/stats)를 Docker로 이미지 빌드
    
- **Kubernetes Deployment, Service, Ingress**로 배포
    
- **Horizontal Pod Autoscaler(HPA)**로 CPU/메모리 기준 자동 스케일링
    

까지 하면,  
“Cloud Native 서비스 운영 경험 있음”이라고 말할 수 있음.

---

### ✅ 2) IaC: Terraform으로 전체 인프라 생성

KT cloud도 Terraform으로 인프라 자동화 많이 씀(요즘 CSP 기본).[KT Cloud 채용+1](https://career.ktcloud.com/work.html?utm_source=chatgpt.com)

포폴에서 할 수 있는 내용:

- `terraform/` 디렉토리 만들어서
    
    - VPC / Subnet
        
    - K8s 클러스터(K2P or K8s용 VM)
        
    - DB(MySQL 인스턴스)
        
    - Object Storage 버킷
        
    - LB / 보안그룹(ACG)
        
- 전부 코드를 통해 생성
    

README에

> “`terraform apply` 한 번으로 인프라가 한 번에 세팅된다”  
> 이 한 줄만 있어도 면접에서 눈 반짝임.

---

### ✅ 3) CI/CD (DevOps 포인트)

KT cloud 공채에서도 **클라우드 자동화 플랫폼 개발, DevOps 분야**를 강조함.[KT Cloud 채용+1](https://career.ktcloud.com/recruit.html?utm_source=chatgpt.com)

그래서 CI/CD는 꼭 넣자.

- **GitHub Actions** or Jenkins
    
- 파이프라인 흐름:
    
    1. main 브랜치에 push/merge
        
    2. 백엔드/프론트 각각 테스트 & 빌드
        
    3. Docker 이미지 빌드 → 컨테이너 레지스트리 푸시
        
    4. `kubectl apply` or `helm upgrade`로 K8s 배포 자동화
        

YAML 파일 그대로 깃허브에 올리면

> “이 사람은 CI/CD 개념이 아니라, 실제로 운영해본 사람이다”

로 보임.

---

### ✅ 4) Observability (모니터링/로그)

KT/KT cloud는 아예 **Observability 엔지니어**를 별도 포지션으로 뽑을 정도로 중요하게 봄.[원티드+1](https://www.wanted.co.kr/wd/206433?utm_source=chatgpt.com)

여기서 먹힐 요소:

- **Prometheus**:
    
    - 각 서비스의 메트릭(API 응답 시간, 에러율, 요청 수 등) 수집
        
- **Grafana**:
    
    - 대시보드 만들어서
        
        - Todo 생성/조회 트래픽
            
        - 서비스별 에러율
            
        - Pod CPU/메모리 사용량
            
- **로그 스택(Loki or ELK)**:
    
    - K8s Pod 로그를 중앙 수집
        
    - 에러 로그 필터링 / 검색
        

포폴 문서에  
**“운영 중 장애를 가정하고, 로그·메트릭으로 원인 분석한 시나리오”**까지 적어두면 진짜 좋다.

---

### ✅ 5) Serverless(KCI or Function)로 자동화 작업 추가

KT cloud는 **서버리스 컨테이너(KCI)**, 서버리스 기능을 강조하고 있음.[kt cloud Tech Blog+1](https://tech.ktcloud.com/entry/%EA%B5%AD%EB%82%B4-%EC%B5%9C%EC%B4%88-%EC%84%9C%EB%B2%84%EB%A6%AC%EC%8A%A4-%EC%BB%A8%ED%85%8C%EC%9D%B4%EB%84%88-%EC%84%9C%EB%B9%84%EC%8A%A4-kt-cloud-container-1%ED%8E%B8?utm_source=chatgpt.com)

이걸 이렇게 쓰면 예쁨:

- 매일 09:00에
    
    - “오늘 마감인 Todo 리스트”를 계산하고
        
    - 이메일 or 슬랙 Webhook으로 보내는 **알림 작업**
        
- 이걸 **서버리스 Function**으로 구현:
    
    - 스케줄링(이벤트 룰) + 컨테이너/함수 실행
        
    - DB에서 Todo 조회 → 메일 발송
        

=> “일정 주기/이벤트 기반 자동화” 경험 = **클라우드 자동화 플랫폼 감성** 살리기 딱 좋음.

---

## 7. 최종 스택 정리 (KT cloud 노리고 가는 버전)

### 🌐 Frontend

- React + TypeScript
    
- 간단한 대시보드/통계 페이지
    

### 🧠 Backend

- Java 21 + Spring Boot
    
- Spring Security + JWT
    
- MySQL(or MariaDB)
    
- 통계/배치용 별도 모듈 or 서비스
    

### ☁ Infra & DevOps

- Docker
    
- Kubernetes(K2P or 일반 K8s)
    
- Terraform (VPC, DB, K8s, LB, Storage 등)
    
- GitHub Actions or Jenkins CI/CD
    
- Prometheus + Grafana, (Loki/ELK)
    
- Serverless(Function/KCI) for scheduled tasks
    

---

## 8. 이 프로젝트로 면접에서 할 수 있는 말

- “단순 CRUD가 아니라, **클라우드 네이티브 아키텍처**를 설계하고 직접 운용해봤다.”
    
- “Kubernetes 위에서 MSA를 굴리고, HPA/Ingress/Service 구조를 이해하고 있다.”
    
- “Terraform으로 인프라를 코드로 정의하고, CI/CD와 연동했다.”
    
- “로그·메트릭 기반으로 서비스 상태를 모니터링하고, 장애 상황을 가정해 대응 플로우를 설계했다.”
    
- “서버리스(Function/KCI)로 반복 작업을 자동화해서 **클라우드 자동화 플랫폼** 개념을 직접 구현해봤다.”
    

= 딱 KT cloud 공채/경력 JD에 맞는 키워드들임.[KT Cloud 채용+2케이티클라우드+2](https://career.ktcloud.com/recruit.html?utm_source=chatgpt.com)