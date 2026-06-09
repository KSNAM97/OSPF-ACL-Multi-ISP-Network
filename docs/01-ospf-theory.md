# 01. OSPF 이론 (Open Shortest Path First)

## 📌 OSPF 개요

| 항목 | 내용 |
| --- | --- |
| **Open** | 표준 공개 프로토콜 (모든 제조사 라우터 지원) |
| **Shortest Path First** | SPF 알고리즘(Dijkstra) 기반 Loop-free 최단 경로 계산 |
| **알고리즘** | Link-State |
| **AD (Administrative Distance)** | **110** |
| **Metric** | Cost (Bandwidth 기반) |
| **Protocol Number** | **89** |
| **Multicast Address** | 224.0.0.5 (All OSPF Router), 224.0.0.6 (DR/BDR) |
| **Classless** | VLSM, CIDR, Subnetting 지원 |

---

## 📌 Link-State 알고리즘 특징

- 인접 관계(Adjacency)가 형성된 이웃과만 정보 교환
- **주기적 업데이트 X** (LSDB 동기화 후에는 변경 시에만 부분 업데이트)
- 링크 상태 변화 시 **변화된 부분만** LSA로 전파
- 각 라우터가 전체 네트워크 토폴로지를 **LSDB(Link-State Database)**에 저장
- SPF 알고리즘으로 자기 자신을 루트로 한 최단 경로 트리(SPT) 계산

---

## 📌 AD (Administrative Distance) 비교

| 프로토콜 | AD |
| --- | :---: |
| Connected | 0 |
| Static | 1 |
| EIGRP (Internal) | 90 |
| **OSPF** | **110** |
| RIP | 120 |
| EIGRP (External) | 170 |

> 동일 목적지에 대해 여러 프로토콜이 학습된 경우, **AD가 낮은 경로가 라우팅 테이블에 등재**됩니다.

---

## 📌 OSPF Cost (Metric)

### 기본 계산 공식

```
Cost = Reference-Bandwidth / Interface-Bandwidth
     = 10^8 (기본 100Mbps) / 인터페이스 대역폭(bps)
```

### 기본 Reference-Bandwidth (10^8) 적용 시

| 인터페이스 | Bandwidth | 계산 | Cost |
| --- | --- | --- | :---: |
| FastEthernet | 100 Mbps | 10^8 / 10^8 | **1** |
| Ethernet | 10 Mbps | 10^8 / 10^7 | **10** |
| Serial (T1) | 1.544 Mbps | 10^8 / 1,544,000 | **64** |
| GigabitEthernet | 1 Gbps | 10^8 / 10^9 = 0.1 (최솟값 1로 강제) | **1** |

> ⚠️ **문제점**: FastEthernet과 GigabitEthernet 모두 Cost=1로 동일하게 인식됨

### Reference-Bandwidth 변경 (권장)

```cisco
router ospf 100
 auto-cost reference-bandwidth 1000   ! 단위: Mbps (1Gbps)
```

| 인터페이스 | 변경 후 Cost |
| --- | :---: |
| GigabitEthernet | 1 |
| FastEthernet | 10 |
| Ethernet | 100 |
| Serial (T1) | 647 |

> 📢 **주의**: 모든 OSPF 라우터에서 동일하게 설정해야 합니다.

---

## 📌 Router-ID 선정 우선순위

1. `router-id A.B.C.D` 명령어로 수동 지정 (가장 우선)
2. **Loopback interface** 중 가장 큰 IP
3. 활성 물리 인터페이스 중 가장 큰 IP

> ✅ 한번 선정된 Router-ID는 OSPF 프로세스가 재시작되지 않으면 변경되지 않습니다.

### Router-ID 변경 후 적용

```cisco
router ospf 100
 router-id 1.1.1.1
end
clear ip ospf process     ! Reset ALL OSPF processes? [no]: y
```

---

## 📌 OSPF Area 개념

OSPF는 LSA Flooding의 영향을 줄이고 SPF 계산 부하를 분산하기 위해 **Area** 단위로 LSDB를 동기화합니다.

### Single Area vs Multiple Area

| 구분 | 설명 |
| --- | --- |
| **Single Area** | 모든 라우터가 하나의 Area에 속함 (소규모) |
| **Multiple Area** | 둘 이상의 Area로 분리. 모든 Area는 **Backbone Area(Area 0)**에 연결되어야 함 |

### Area 종류별 라우터

| 라우터 종류 | 역할 |
| --- | --- |
| **Internal Router** | 단일 Area 내부에만 속한 라우터 |
| **Backbone Router** | Area 0에 속한 라우터 |
| **ABR (Area Border Router)** | 둘 이상의 Area에 걸쳐 있는 라우터. LSDB를 Area별로 분리 관리 |
| **ASBR (AS Boundary Router)** | 외부 프로토콜과 OSPF를 재분배(redistribute)하는 라우터 |

---

## 📌 OSPF 네트워크 타입

| Network Type | Hello / Dead | DR/BDR | 대표 프로토콜 |
| --- | :---: | :---: | --- |
| **Broadcast (BMA)** | 10 / 40 | 선출 O | Ethernet, FastEthernet |
| **Non-Broadcast (NBMA)** | 30 / 120 | 선출 O | Frame-Relay Multipoint |
| **Point-to-Point** | 10 / 40 | 선출 X | HDLC, PPP, F/R P2P |
| **Point-to-Multipoint** | 30 / 120 | 선출 X | 수동 설정 |
