# Kubernetes 기본 개념 학습 노트 — Overview

---

## 1. 기본구조

| 개념 | 정의 |
|---|---|
| [Cluster](./.md) | 분산 아키텍처 단위입니다. k8s 에서는 컨트롤 플레인과, 워커 노드로 구성된 관리 단위입니다. 사용자는 클러스터 단위로 워크로드를 선언하고 배포합니다.  <br> 컨트롤 플레인 핵심 컴포넌트 <br> - **kube-apiserver**: 클러스터 통신의 단일 endpoint. 아래 다른 컴포넌트, Node 에 떠있는 kubelet들은 서로 직접 통신 없이 오직 apiserver와만 통신.  <br> - **kube-scheduler**: 새로 생성된 Pod를 어느 노드에 배치할지 스케줄링하는 컴포넌트. 실제 컨테이너 기동은 kubelet이 수행. <br> - **kube-controller-manager**: 여러 Controller들(Deployment/ReplicaSet/Node/Job 등)을 하나의 프로세스에 묶어 실행하는 컴포넌트. 각 Controller는 현재 상태(Current State)와 목표 상태(Desired State)를 비교하고, 그 차이를 좁히려고 합니다(= Reconciliation Loop). <br> - **etcd**: 클러스터의 모든 상태 데이터를 저장하는 분산 Key-Value Store. 클러스터의 Single Source of Truth이며, 백업/복구 설계 시 가장 중요한 요소 |
| [Node](./.md) | k8s의 배포 최소단위인 Pod가 실행되는 워커 머신, 즉 VM 또는 물리서버를 Node라고 합니다. 컨트롤 플레인의 스케줄링 대상이 되며, kubelet, container runtime, kube-proxy는 Node에서 데몬 형태로 실행됩니다. <br> Node에서 떠있는 k8s 데몬들<br> - kubelet: kube-apiserver와 통신하는 노드별로 하나씩 있는 Agent. apiserver로부터 "해당 노드에 이런 PodSpec을 실행해"라는 지시를 받아, container-runtime에게 컨테이너 실행을 위임하고, Probe 결과를 apiserver에 반환한다.<br> - container-runtime: 컨테이너를 기동하는 엔진(containerd). CRI(Container Runtime Interface) 표준 인터페이스를 통해 kubelet과 통신한다.<br> - kube-proxy: 노드의 네트워크 규칙을 관리하고, Service 추상화를 구현하는 컴포넌트. Service의 ClusterIP/NodePort로 들어온 트래픽을 Pod 중 하나로 분산/라우팅하는 규칙을 노드에 설치한다(iptables/IPVS). 단, AKS의 Azure CNI Overlay 또는 Cilium모드는 kube-proxy를 eBPF 기반으로 대체하기도 한다.|
| [Pod](./.md) | k8s 최소 배포 단위로 취급되는 컨테이너 그룹입니다. 1개 이상의 컨테이너가 같은 네트워크 네임스페이스·스토리지를 공유합니다. Pod가 죽으면, 죽은 Pod가 복구되거나 되살려지는 게 아니라 새로 생성됩니다. |
| [Namespace](./.md) | 클러스터 내부를 논리적으로 분리하는 가상 경계입니다. 이름 충돌, RBAC, ResourceQuota의 적용 범위가 됩니다. 주로 팀/도메인 단위로 클러스터를 격리할때 사용합니다. |

---

## 2. 워크로드

| 개념 | 정의 |
|---|---|
| [Deployment ](./.md) | Stateless Pod 집합의 desired state를 선언적으로 관리하는 리소스입니다. 내부적으로 ReplicaSet을 만들어 롤링 업데이트·롤백·스케일링을 자동화한다. |
| [StatefulSet ](./.md) |  |
| [DaemonSet ](./.md) |  |
| [Job ](./.md) |  |
| [CronJob ](./.md) |  |

---

## 3. 네트워크

| 개념 | 정의 |
|---|---|
| [Service ](./.md) |  |
| [Service - NodePort](./.md) |  |
| [Service - Client IP](./.md) |  |
| [Service - LoadBalancer](./.md) |  |
| [Ingress](./.md) |  |
| [Ingress Controller ](./.md) |  |
| [Network Policy ](./.md) |  |

---

## 4. 설정

| 개념 | 정의 |
|---|---|
| [ConfigMap ](./.md) |  |
| [Secret ](./.md) |  |

---

## 5. 스토리지

| 개념 | 정의 |
|---|---|
| [Volume ](./.md) |  |
| [Persistent Volume (PV)](./.md) |  |
| [Persistent Volume Claim (PVC)](./.md) |  |
| [StorageClass](./.md) |  |

---

## 6. 권한

| 개념 | 정의 |
|---|---|
| [ServiceAccount](./.md) |  |
| [RBAC ](./.md) |  |

---

## 7. 가용성

| 개념 | 정의 |
|---|---|
| [Rolling Update](./.md) |  |
| [Blue/Green ](./.md) |  |
| [Canary](./.md) |  |
| [Graceful Termination](./.md) |  |

---

## 8. 배포 전략

| 개념 | 정의 |
|---|---|
| [Probe](./.md) |  |
| [Liveness Probe](./.md) |  |
| [Readiness Probe](./.md) |  |
| [Horizontal Pod Autoscaler (HPA) ](./.md) |  |
| [Node Failover ](./.md) |  |
| [JMeter](./.md) |  |
