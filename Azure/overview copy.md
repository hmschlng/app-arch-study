# Azure 기본 개념 학습 노트 — Overview

---

## 1. 가용성

| 개념 | 정의 |
|---|---|
| [Region](./01.가용성/region.md) | Azure 글로벌 인프라의 지역별로 구축된 인프라(데이터센터) 단위입니다. <br>한국에는 현재 Korea Central과 Korea South Region이 있습니다. <br> - 지역쌍(Paired Region) : 리전별로 Azure에서 지정해놓은 쌍입니다. DR 관점에서의 몇가지 편의를 제공합니다. 현재 KC와 KS region이 지역쌍으로 지정이 되어 있습니다. |
| [Availability Zone](./01.가용성/availability-zone.md) | 고가용성(HA)을 위해, 한 Region 내 물리적으로 분산 구축된 독립 인프라 단위입니다. 보통 성숙한 region의 경우 최소 3개의 zone으로 구성되어 데이터센터 단위 장애에 견디도록 하여 SLA를 방어합니다. |

---

## 2. 네트워크

| 개념 | 정의 |
|---|---|
| [VNet (Virtual Network)](./02.네트워크/vnet.md) | 사용자가 정의하는 논리적 인프라에 지정되는 가상 네트워크입니다. VNet은 사설 IP 주소 공간 대역(CIDR)으로 정의되며, <br> Region에 종속되어 생성되기 때문에 Region이 다르면 VNet도 다릅니다. 저희는 랜딩존에 Spoke VNet에 구축하는 겁니다. |
| [CIDR](./02.네트워크/cidr.md) | IP 주소 범위 표기 방식입니다. 10.0.0.0/16 형태로 표기하며, /N 은 IP 주소 앞쪽에서 N bit만큼 네트워크 ID로 고정되고, 나머지 (32-N)비트가 호스트 주소로 할당 가능한 범위라는 뜻입니다. <br> 예시: (192.168.0.0/16 = 65,536개, 192.168.255.0/25 or 192.168.255.128/25 = 128개). |
| [Subnetting](./02.네트워크/subnetting.md) | - Subnet: VNet에서 지정한 CIDR을 더 작게 쪼갠 네트워크 공간 대역입니다. Vnet 에 정의한 Subnet에 각 리소스가 배치되며, 각 Subnet에는 NSG·UDR을 정의할 수 있습니다.<br> - Subnetting: VNet 내의 Subnet을 용도별로 분할 설계하는 작업입니다. AKS / AppGW / PE(Private Endpoint) / DB는 분리된 Subnet에 두는 게 권장 패턴이고, 랜딩존도 이를 제안하고 있습니다. |
| [Application Gateway (AppGW)](./02.네트워크/application-gateway.md) | L7 Layer 트래픽(HTTP/HTTPS)을 처리하는 reverse proxy 겸 load balancer 입니다. SKU에 따라 다르지만, TLS/mTLS 통신 지원, URL path / host 기반 라우팅(이게 일반적으로 reverse proxy 역할), TLS termination과 WAF, Health probe, URL Rewrite, Timeout 등을 통합 제공합니다. <br> AppGW로 들어온 요청의 흐름은 Listener → TLS Termination → WAF → Rule → Backend Pool 순서라고 볼 수 있습니다.|
| [Listener](./02.네트워크/listener.md) | AppGW가 들어오는 트래픽을 어떤 조건(프로토콜·포트·호스트명·인증서)으로 받을지 정의하는 단위.  <br> **Cardinality** <br>- 도메인 : Listener = 1:1 <br>- Listener : Rule = 1:1 <br> - Rule : Backend Pool = 1:N (Path-based 시) |
| [WAF (Web Application Firewall)](./02.네트워크/waf.md) | AppGW WAF SKU 선택시 지원하는 방화벽 기능입니다. OWASP Top10 등 알려진 공격패턴을 차단합니다. 그러나 탐지를 위해서는 페이로드를 검사해야 하기 때문에, AppGW에서 인증서로 TLS 복호화를 거쳐 요청이 평문으로 변환이 되어야(TLS Termination) 한다는 전제조건이 붙습니다. 만약 AppGW가 TLS passthrough모드일 경우, 페이로드 검사는 안하고 WAF 또한 pass합니다. |
| [Rule](./02.네트워크/rule.md) | Listener와 Backend Pool을 연결하는 라우팅 규칙. Basic(단순 1:1)과 Path-based(URL path별 분기) 두 종류. <br> **Rule 구성요소** <br> - Listener: 출발지 <br> - Backend Pool : 목적지 <br> - HTTP Settings : 백엔드와의 통신 설정(프로토콜, 포트, timeout, path/Header 변경, Health Probe(Custom Probe) 선택 등) <br> - URL Path Map(path-based route인 경우) : url 패턴별 분기 규칙 <br> - Rewrite Rule Set : url/헤더 가공 <br> - Redirect Configuration : 다른 Listener, 또는 외부 URL로 리다이렉트 |
| [Backend Pool](./02.네트워크/backend-pool.md) | AppGW가 트래픽을 전달할 backend 대상 그룹입니다. <br> **backend pool 추가가능한 멤버** <br> - IP 주소(백엔드 단 하나일때, 거의 쓸 일 없음) <br> - FQDN: DNS 상의 이름(appgw가 주기적으로 resolve). 외부 서비스 등 <br> - VMSS(VM Scale Set) <br> - Azure App Service 인스턴스 <br> - AKS Pod(AGIC 경유. AGIC가 Pod IP를 자동으로 등록) |
| [Health Probe](./02.네트워크/health-probe.md) | Backend의 생존을 주기적으로 확인하는 헬스체크 기능입니다. 응답 코드/본문/타임아웃 기준으로 healthy/unhealthy를 판단해, healthy로 판정된 멤버에만 트래픽이 분배되도록 제어합니다. |
| [Internal Load Balancer (ILB)](./02.네트워크/internal-load-balancer.md) | VNet 내부에서만 접근되는 Layer 4(TCP/UDP) Load Balancer입니다. HTTP AppGW와는 달리 non-Http TCP(ssh, mySQL, pgSQL, Redis 등등)나 UDP(RT(S)P 영상 프로토콜이나 MQTT 등) 통신을 전부 pass through할 수 있어서, 19000, 34000번대 포트 통신의 경우는 이 ILB를 엔트리포인트로 둘 수 있습니다. 그러나 말그대로 Internal LB이고, AppGW처럼 인터넷 외부 인입이 ILB를 직접 접근할 수 없기 때문에, 랜딩존 Hub + Spoke 규격에서 ILB 인입 흐름에 대해 확인할 필요가 있습니다. |
| [Private Endpoint](./02.네트워크/private-endpoint.md) | PaaS 서비스(Storage, Key Vault, ACR 등)를 public internet 거치지 않고 VNet 내부 사설 IP로 접근하게 하는 NIC. DNS도 privatelink.~ 사설 영역으로 매핑. |
| [Network Security Group (NSG)](./02.네트워크/nsg.md) | Subnet 또는 NIC 단위로 적용되는 stateful L3/L4 방화벽 규칙 집합. source/destination/port/protocol 기반 allow/deny. |
| [User Defined Route (UDR)](./02.네트워크/udr.md) | Subnet의 기본 라우팅을 사용자가 재정의하는 routing rule. 흔히 outbound를 firewall/NVA로 강제 우회시킬 때 사용. |

---

## 3. 스토리지

| 개념 | 정의 |
|---|---|
| [Storage Account](./03.스토리지/storage-account.md) | Azure Storage의 모든 데이터 객체(Blob/File/Queue/Table/Disk)를 담는 namespace 컨테이너. 이름이 글로벌 unique한 endpoint(<name>.blob.core.windows.net 등)가 되며, replication·access tier·보안 설정이 이 단위로 묶인다. |
| [Blob Storage](./03.스토리지/blob-storage.md) | 대용량 비정형 객체(이미지·영상·로그·백업)를 저장하는 object storage. Block blob(파일 일반) / Append blob(로그) / Page blob(VHD) 세 타입이 있고, access tier(Hot/Cool/Cold/Archive)로 비용 최적화. |
| [Azure Files](./03.스토리지/azure-files.md) | SMB 또는 NFS 프로토콜로 마운트 가능한 fully managed file share. 기존 NAS/파일서버 직접 대체용으로 가장 유사하지만 프로토콜 선택에 트레이드오프가 크다. |

---

## 4. 보안

| 개념 | 정의 |
|---|---|
| [Key Vault](./04.보안/key-vault.md) | Secret(연결문자열·API 키 등), Key(암호화 키), Certificate(TLS 인증서) 세 종류를 저장·관리하는 Azure managed 비밀 저장소. Standard와 Premium(HSM-backed) tier가 있고, 권한 모델은 access policy(legacy) 또는 RBAC을 선택. |

---

## 5. AKS

| 개념 | 정의 |
|---|---|
| [AKS Cluster](./05.AKS/aks-cluster.md) | Azure가 control plane(API server, etcd, scheduler, controller-manager)을 관리해주는 managed Kubernetes 클러스터. 사용자는 worker node와 워크로드만 책임진다. |
| [Node Pool (VMSS)](./05.AKS/node-pool.md) | 같은 VM SKU·OS·설정을 공유하는 worker node 그룹. 내부적으로 VMSS(Virtual Machine Scale Set)로 구현되어 cluster-autoscaler·rolling upgrade·zone spread가 가능하다. |
| [Managed Identity](./05.AKS/managed-identity.md) | Azure 리소스가 자체 identity를 Entra ID(구 Azure AD)에 등록·관리하여, 코드에 자격증명을 두지 않고 다른 Azure 서비스에 인증하게 하는 메커니즘. System-assigned(리소스 수명과 동기) / User-assigned(독립 수명) 두 종류. |
| [RBAC](./05.AKS/rbac.md) | "누가(principal) / 무엇을(role) / 어디서(scope)" 형태로 권한을 부여하는 모델. AKS는 두 층위 RBAC이 공존한다: Azure RBAC(클러스터 자체 관리 권한)과 Kubernetes RBAC(클러스터 내부 리소스 권한). |
| [CNI Overlay](./05.AKS/cni-overlay.md) | Pod에 VNet IP가 아닌 별도 overlay CIDR의 IP를 할당하는 네트워킹 모드. Pod 간 통신은 overlay 위에서, pod에서 외부로는 노드 IP로 SNAT되어 나간다. |
