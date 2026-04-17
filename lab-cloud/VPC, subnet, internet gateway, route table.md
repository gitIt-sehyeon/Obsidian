## 1) 개념 정리

### VPC (Virtual Private Cloud)

- AWS 안에 만드는 **내 전용 가상 네트워크**
    
- 내가 정하는 것
    
    - **IP 대역(CIDR)**: 예) `10.0.0.0/16`
        
    - 서브넷 구성(구역 나누기)
        
    - 라우팅(어디로 보내는지)
        
    - 보안 경계(보안그룹/NACL 등)
        

비유: **건물 단지 전체**

---

### Subnet

- VPC 안을 **더 작은 네트워크 구역**으로 나눈 것
    
- 각 Subnet은 **반드시 1개의 AZ(가용영역)** 에 속함
    
- 용도별로 나누는 게 표준
    
    - **Public Subnet**: 인터넷과 직접 연결되는 구역(웹/ALB 등)
        
    - **Private Subnet**: 인터넷 직접 연결 X (DB/내부 API 등)
        

비유: **단지 안의 동네/구역**

---

### Internet Gateway (IGW)

- VPC가 **인터넷과 통신할 수 있게 해주는 “출입문”**
    
- IGW는 **VPC에 Attach(연결)** 해야 동작
    
- 단, IGW만 붙인다고 Public Subnet이 되는 게 아니라,
    
    - **Route Table에서 0.0.0.0/0 → IGW** 라우트가 있어야 Public 성격이 됨
        

비유: **단지 정문**

---

### Route Table

- “**이 목적지(Destination)로 가는 트래픽은 어디(Target)로 보내라**”를 정의
    
- 핵심 라우트 2개를 기억하면 됩니다.
    
    - `VPC CIDR → local` : VPC 내부 통신(기본값, 자동 존재)
        
    - `0.0.0.0/0 → igw-…` : 인터넷으로 나가는 길(이게 있으면 Public 가능)
        

비유: **길 안내 지도(교통 표지판)**

---

## 2) 설계 예시(권장 구조)

처음 학습/프로젝트용으로 가장 표준적인 구조는 아래입니다.

- VPC: `10.0.0.0/16`
    
- Public Subnet (2개, 서로 다른 AZ)
    
    - `10.0.1.0/24` (ap-northeast-2a)
        
    - `10.0.2.0/24` (ap-northeast-2c)
        
- Private Subnet (2개, 서로 다른 AZ)
    
    - `10.0.11.0/24` (ap-northeast-2a)
        
    - `10.0.12.0/24` (ap-northeast-2c)
        

이렇게 하면 **고가용성(AZ 분산)** 과 **보안 분리(공개/비공개)** 를 동시에 만족합니다.

---

## 3) AWS 콘솔에서 설정 순서(정석)

아래 순서대로 하면 “인터넷 되는 Public Subnet + 내부 Private Subnet” 환경이 완성됩니다.

---

### Step 0. 리전 확인

- 오른쪽 상단에서 리전을 확인하세요. (예: **서울 ap-northeast-2**)
    
- VPC/Subnet/AZ는 리전 단위로 만들어집니다.
    

---

### Step 1. VPC 생성

1. AWS 콘솔 → **VPC** 서비스
    
2. 좌측 메뉴 **Your VPCs** → **Create VPC**
    
3. 설정
    

- Name tag: 예) `my-vpc`
    
- IPv4 CIDR block: `10.0.0.0/16`
    
- Tenancy: Default
    

4. Create
    

결과: “단지(네트워크) 큰 틀” 생성

---

### Step 2. Subnet 생성 (Public/Private)

1. 좌측 메뉴 **Subnets** → **Create subnet**
    
2. VPC 선택: `my-vpc`
    
3. 서브넷 4개 생성 (권장)
    

- Public-A: AZ = ap-northeast-2a, CIDR = `10.0.1.0/24`
    
- Public-C: AZ = ap-northeast-2c, CIDR = `10.0.2.0/24`
    
- Private-A: AZ = ap-northeast-2a, CIDR = `10.0.11.0/24`
    
- Private-C: AZ = ap-northeast-2c, CIDR = `10.0.12.0/24`
    

핵심: **서브넷은 AZ에 종속**됩니다.

---

### Step 3. Internet Gateway 생성 및 VPC에 연결

1. 좌측 메뉴 **Internet Gateways** → **Create internet gateway**
    

- Name: `my-igw`
    

2. 생성 후, 해당 IGW 선택 → **Actions → Attach to VPC**
    
3. VPC로 `my-vpc` 선택 → Attach
    

결과: VPC에 “정문(IGW)”을 달아둔 상태  
단, 아직 Public Subnet이 된 건 아닙니다. (라우팅이 남아있음)

---

### Step 4. Public Route Table 생성 및 라우팅 추가

1. 좌측 메뉴 **Route Tables** → **Create route table**
    

- Name: `rtb-public`
    
- VPC: `my-vpc`
    

2. 생성 후 `rtb-public` 클릭 → **Routes 탭 → Edit routes**
    
3. 라우트 추가
    

- Destination: `0.0.0.0/0`
    
- Target: `my-igw` (igw-xxxx)
    

4. Save changes
    

의미:

- “인터넷(전체)로 가는 길은 IGW로 보내라”
    
- 이것이 Public Subnet의 핵심 조건입니다.
    

---

### Step 5. Public Subnet에 Public Route Table 연결(Association)

1. `rtb-public` → **Subnet associations 탭 → Edit subnet associations**
    
2. Public 서브넷 2개 선택
    

- Public-A (`10.0.1.0/24`)
    
- Public-C (`10.0.2.0/24`)
    

3. Save
    

결과:

- 이 두 서브넷은 `0.0.0.0/0 → IGW` 라우트를 사용
    
- 즉, **Public Subnet 확정**
    

---

### Step 6. Private Route Table 생성 및 연결

Private는 “인터넷 직접 연결 없음”이 기본입니다.

1. **Route Tables** → Create route table
    

- Name: `rtb-private`
    
- VPC: `my-vpc`
    

2. 생성 후 **Subnet associations**에서
    

- Private-A (`10.0.11.0/24`)
    
- Private-C (`10.0.12.0/24`)  
    연결
    

3. `rtb-private`의 Routes는 기본적으로
    

- `10.0.0.0/16 → local` 만 있어도 정상입니다.
    

결과:

- Private Subnet은 인터넷으로 “직접” 못 나갑니다.
    

---

## 4) (선택) Private Subnet도 “나가기만” 가능하게 만들기: NAT Gateway

Private 인스턴스가 `yum update`, 패키지 다운로드 같은 걸 해야 하면 NAT가 필요합니다.

### NAT 구성 순서

1. **Elastic IP** 생성(필수)
    

- VPC 콘솔 → Elastic IPs → Allocate Elastic IP
    

2. **NAT Gateway 생성**
    

- NAT Gateways → Create NAT gateway
    
- Subnet: **Public Subnet 중 하나** 선택 (예: Public-A)
    
- Elastic IP: 위에서 만든 EIP 선택
    

3. `rtb-private` Routes에 추가
    

- Destination: `0.0.0.0/0`
    
- Target: `nat-xxxx`
    

의미:

- Private는 인터넷 “직접 연결”은 안 되지만,
    
- **NAT를 통해서만 아웃바운드**가 가능해집니다.
    
- 외부에서 Private로 “직접 인바운드”는 여전히 불가(보안 유지)
    

---

## 5) 자주 헷갈리는 포인트(실수 방지)

1. **IGW를 VPC에 붙였다고 Public이 되는 게 아님**
    

- Public이 되려면 **해당 Subnet이 사용하는 Route Table에**
    
    - `0.0.0.0/0 → IGW` 가 있어야 합니다.
        

2. **Public Subnet의 인스턴스는 Public IP가 있어야 외부에서 접속됨**
    

- EC2 만들 때 “Auto-assign public IPv4” 또는 퍼블릭 IP 할당 여부 확인 필요
    

3. **보안그룹은 별개**
    

- 라우팅이 열려도, 보안그룹 inbound가 막혀 있으면 접속 안 됩니다.
    
- 라우팅 = 길
    
- 보안그룹 = 출입 허가증
    

---

## 6) 빠른 점검 체크리스트

Public Subnet이 “인터넷 되는 상태”인지 확인:

- Public Subnet이 연결된 Route Table에  
    `0.0.0.0/0 → IGW` 존재
    
- EC2에 Public IP 존재
    
- Security Group inbound에 (예: 80/443/22) 허용
    
- NACL이 막고 있지 않음(기본 NACL은 대부분 허용)