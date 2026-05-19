# Azure 기본 개념 학습 노트 — Overview

---

## 1. 가용성

| 개념 | 정의 |
|---|---|
| [Region](./01.가용성/region.md) | Azure가 전 세계에 배치한 데이터센터의 지리적 집합 단위. 각 Region은 자체 latency boundary를 가지며, 일부 Region은 재해 복구용 paired region(지역 쌍)을 갖는다. |
| [Availability Zone](./01.가용성/availability-zone.md) | 한 Region 내에서 전원/냉각/네트워크가 물리적으로 분리된 독립 데이터센터(또는 데이터센터 그룹)로, 최소 3개의 zone으로 구성된다. Region 내 다중 AZ 배치로 단일 데이터센터 장애에 견딘다. |

---

## 2. 네트워크

| 개념 | 정의 |
|---|---|
| [VNet (Virtual Network)](./02.네트워크/vnet.md) | Azure 내에서 사용자가 정의하는 사설 IP 주소 공간을 가진 격리된 가상 네트워크. 단일 region에 종속되며, peering으로 다른 VNet/region과 연결한다. |
| [CIDR](./02.네트워크/cidr.md) | IP 주소 범위를 10.0.0.0/16 형태로 표기하는 표준 방식. /N의 N이 네트워크 prefix 비트 수이며, 작을수록 큰 주소 공간(/16 = 65,536개, /24 = 256개). |
| [Subnet](./02.네트워크/subnet.md) | VNet의 CIDR을 더 작은 단위로 쪼갠 네트워크 segment. 리소스는 특정 Subnet에 배치되며, Subnet 단위로 NSG·UDR이 붙는다. |
| [Subnetting](./02.네트워크/subnetting.md) | VNet의 큰 CIDR을 용도별 Subnet으로 분할하는 설계 행위. AKS / AppGW / Private Endpoint / DB는 각자 별도 Subnet에 두는 게 권장 패턴. |
| [Application Gateway (AppGW)](./02.네트워크/application-gateway.md) | Layer 7(HTTP/HTTPS) 트래픽 처리하는 Azure managed reverse proxy / load balancer. URL/host 기반 라우팅, TLS termination, WAF 통합 제공. |
| [WAF (Web Application Firewall)](./02.네트워크/waf.md) | OWASP Top 10 등 알려진 웹 공격 패턴을 차단하는 L7 보안 기능. Azure에서는 AppGW WAF SKU 또는 Azure Front Door WAF로 제공. |
| [Listener](./02.네트워크/listener.md) | AppGW가 들어오는 트래픽을 어떤 조건(프로토콜·포트·호스트명·인증서)으로 받을지 정의하는 단위. Listener가 Rule을 통해 Backend Pool로 연결된다. |
| [Backend Pool](./02.네트워크/backend-pool.md) | AppGW가 트래픽을 전달할 backend 대상 집합. IP/FQDN/VMSS/App Service/AKS Pod(AGIC 경유) 등이 멤버. |
| [Rule](./02.네트워크/rule.md) | Listener와 Backend Pool을 연결하는 라우팅 규칙. Basic(단순 1:1)과 Path-based(URL path별 분기) 두 종류. |
| [Health Probe](./02.네트워크/health-probe.md) | Backend의 생존을 주기적으로 확인하는 헬스체크. 응답 코드/본문/타임아웃 기준으로 healthy 여부를 판정해 트래픽 송신 결정. |
| [Internal Load Balancer (ILB)](./02.네트워크/internal-load-balancer.md) | Public IP 없는 Layer 4(TCP/UDP) 부하분산기. VNet 내부에서만 접근 가능. AKS에서는 Service type=LoadBalancer + azure-load-balancer-internal: "true" annotation으로 사용. |
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
