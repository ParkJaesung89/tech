# AWS 기본 VPC 구성

## 1. VPC(Virtual Private Cloud)
VPC란
사용자가 정의한 가상 네트워크, Private Network 활용하여 네트워크망을 구성하며 내부의 각종 리소스들을 탑재할 수 있는 서비스(EC2, ELB, RDS, Seculity Group, Network ACL 등)
vpc에 들어가는 리소스들은 고유의 사설 ip 와 interface를 반드시 갖게 되며, 외부에 공개될 리소스의 경우, 공인 ip를 보유할 수 있다.

### 사설 IP

AWS 관리 IP로서 사용 불가능한 AWS 예약 주소
```
10.0.1.0 : 네트워크 주소(network ID)
10.0.1.1 : VPC 라우터용 예약 주소(Default Gateway)
10.0.1.2 : DNS 서버의 IP 주소(기본 VPC 네트워크 범위에 2를 더한 주소)
           - CIDR 블록이 여러개인 VPC 경우, DNS 서버 IP 주소가 기본 CIDR에 위치함.
10.0.1.3 : AWS에서 앞으로 사용하려고 예약한 주소...
10.0.1.255 : 네트워크 브로드캐스트 주소. VPC에서는 브로드캐스트 지원을 하지 않음.
```

- VPC 생성시 허용되는 블록 크기는 /28 ~ /16

<vpc 네트워크 예시>

---AWS------------------------------------------------------
|    ----region------------------------------------------- |
|    |    ----VPC - 10.0.0.0 /16-----------------------  | |
|    |    |  --AZ1-----------    --AZ2--------------  |  | |
|    |    |  |               |   |                 |  |  | |
|    |    |  | --subnet1---  |   | --subnet2---    |  |  | |
|    |    |  | |          |  |   | |          |    |  |  | |
|    |    |  | |[instance]|  |   | |[instance]|    |  |  | |
|    |    |  | |__________|  |   | |__________|    |  |  | |
|    |    |  |_______________|   |_________________|  |  | |
|    |    |___________________________________________|  | |
|    |___________________________________________________| |
|__________________________________________________________|

### VPC 생성 방법

1. VPC 에서 VPC 생성 클릭
2. 이름 작성                    ex) TEST-Jaesung-VPC
   IPv4 CIDR 블록               ex) 10.20.0.0/16 ~ /28
   IPv6 CIDR 블록               ex) 없음으로 체크
   테넌시                       ex) 기본값
   태그                         ex) Name - TEST-Jaesung-VPC

## 2. Subnet
각 subnet은 한개의 AZ(Availability Zone)에 속함.
*한개의 AZ에 여러 서브넷이 존재할 수 있지만, 한개의 서브넷에 여러개의 AZ가 존재할 수 없다.*

- 같은 서브넷에 존재하는 instance들 끼리 다른 서비스를 통하지 않고 다이렉트로 통신 가능.
- 다른 서브넷에 존재하는 instance들 끼리 통신을 위해서는 라우팅 테이블을 필요로 함.

### subnet 생성 방법
1. vpc - subnet 에서 subnet 생성 클릭
2. VPC ID                       ex) TEST-Jaesung-VPC
   서브넷 이름                  ex) TEST-Jaesung-PRD-PUBLIC-A
   가용 영역                    ex) 아시아 태평양 (서울) / ap-northeast-2a
   IPv4 CIDR 블록               ex) 10.10.0.0/24
   새 서브넷 추가               // 여러 서브넷 생성시 추가


## 3. Routing table
VPC 내에 각 Subnet은 각기 다른 네트워크 대역을 갖고있음.
Routing table에 서브넷들을 등록하여 테이블에 등록된 서브넷끼리 통신가능.

### Routing table 생성 구성 방법
1. vpc - routing table - 생성 클릭
2. 서브넷 연결 - 명시적 서브넷 연결에서 서브넷 연결 편집 클릭
3. 원하는 서브넷을 체크 후 연결 저장

## 4. Internet Gateway
VPC안의 instance와 외부 네트워크간 통신을 가능하게 함.
외부통신을 하려고 하는 트래픽을 가진 VPC의 라우팅 테이블에 Internet Gateway 대상을 연결하여 퍼블릭 주소가 할당 된 instance에 대해 NAT를 수행하여 외부통신을 할 수 있도록 함.

### Internet Gateway 생성 및 구성 방법
1. VPC - 인터넷 게이트웨이 - 인터넷 게이트웨이 생성
2. 이름 태그                    ex) TEST-Jaesung-IGW
3. 생성한 IGW에 들어가서 작업 - VPC에 연결
4. 라우팅 테이블 - 라우팅 - 라우팅 편집 => 0.0.0.0/0 - igw-xxxxxxxxx 추가 후 변경 사항 저장

## 5. NAT Gateway
외부접근에 대한 차단을 위해 private subnet에 존재하는 instance가 외부로 통신을 해야 될 경우 public subnet에 NAT Gateway를 생성해두고 private subnet이 외부로 통신할 경우에만 사용하도록 라우팅을 추가함.
private subnet의 서비스들은 외부 인터넷으로 나갈때 공인 IP로 NAT되어서 나감.
- 내부에서 외부로의 접근만 가능하고 외부에서 내부 접근은 불가능함.

### NAT Gateway 종류
- public
  private 서브넷의 인스턴스는 퍼블릭 NAT Gateway를 통해 인터넷에 연결할 수 있지만 인터넷에서 원치 않는 인바운드 연결을 수신할 수 없다.
  
- private