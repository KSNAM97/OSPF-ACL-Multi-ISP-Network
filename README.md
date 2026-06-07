# OSPF Multi-Area & ACL Lab (Cisco IOS / GNS3)

![OSPF](https://img.shields.io/badge/Protocol-OSPF-blue)
![Cisco](https://img.shields.io/badge/Cisco-IOS-orange)
![GNS3](https://img.shields.io/badge/Simulator-GNS3-green)
![License](https://img.shields.io/badge/License-MIT-lightgrey)

**OSPF Multi-Area, Virtual-Link, DR/BDR, ACL 종합 실습 (Cisco IOS / GNS3)**

---

## 📌 프로젝트 개요

GNS3 / Cisco IOS 환경에서 **6개의 ISP 백본 라우터**와 **3개의 고객사 라우터(R1, R2, R3)**를 OSPF로 구성하고, ISP-1에서 Extended ACL로 트래픽을 제어하는 종합 실습 프로젝트입니다.

| 항목 | 내용 |
| --- | --- |
| 라우팅 프로토콜 | OSPF (Process 100) |
| 라우터 수 | 9대 (ISP 6대 + 고객사 R1/R2/R3) |
| Area 구성 | Area 0 (백본), Area 100, Area 210 |
| 핵심 기술 | Multi-Area OSPF, DR/BDR 선출, Passive Interface, Loopback Point-to-Point, Extended ACL |
| 시뮬레이터 | GNS3 (Cisco IOS c3745) |

---

## 🗺️ 네트워크 토폴로지

![OSPF Multi-Area Topology](./topology/OSPF-ACL-Multi-ISP-Network.png)

- **OSPF Area 100**: ISP-1 + R1 + 내부 LAN (198.210.10.0/27)
- **OSPF Area 0 (Backbone)**: ISP-1 ~ ISP-6
- **OSPF Area 210**: ISP-5 + R2 + R3 + 내부 LAN (100.10.1.0/24, 100.10.2.0/24)
- **WAN 구간**: HDLC (64Kbps ~ 128Kbps, 210.116.41.x/30)
- **LAN 구간**: FastEthernet (SW1~SW4, SWX/SWY)
- **Multi-Access 구간**: ISP-2, ISP-3, ISP-4가 ISP-SW로 연결 (210.116.41.64/29) → DR/BDR 선출

---

## 🎯 실습 목표

1. **EX1** - 모든 라우터의 Loopback / IP 주소 / Telnet / Enable 패스워드 설정
2. **EX2** - LAN(Ethernet) / WAN(HDLC) 구간 IP 할당 및 Next-hop 통신 확인
3. **EX3** - OSPF Multi-Area 구성 (Area 0 / 100 / 210)
   - Router-ID 수동 지정
   - Passive-interface로 OSPF Packet 송신 제어
   - ISP-2/3/4 Multi-access 구간에서 **ISP-3 = DR, BDR 미선출** (Priority 조정)
   - ISP-1의 Loopback 1 → Area 100 / ISP-5의 Loopback 1 → Area 210
4. **EX4** - ISP-1에서 Extended ACL로 R1 → R2/R3 트래픽 차단
5. **EX5** - Routing Table / Database Table / LSA-Type 검증

---

## 📁 프로젝트 구조

```
OSPF-MultiArea-ACL-Lab/
├── topology/
│   └── ospf_acl_topology.png         # GNS3 토폴로지 이미지
├── docs/                              # OSPF 이론 문서
│   ├── 01-ospf-theory.md              # OSPF 개요, AD/Metric, Area
│   ├── 02-ospf-pdu.md                 # 5가지 PDU (Hello/DBD/LSR/LSU/LSAck)
│   ├── 03-ospf-neighbor-state.md      # 인접관계 7단계 상태
│   ├── 04-dr-bdr.md                   # DR/BDR 선출 메커니즘
│   ├── 05-lsa-type.md                 # LSA Type 1~5 정리
│   ├── 06-virtual-link.md             # Virtual-Link 구성
│   ├── 07-ospf-authentication.md      # Neighbor / Area 인증
│   ├── 08-acl-theory.md               # Standard / Extended ACL
│   └── 09-ex4-acl-implementation.md   # EX4 ACL 구현
├── verification/
│   └── verification-commands.md       # show / debug 명령어 모음
├── LICENSE
└── README.md
```

---

## 🚀 빠른 시작

### 1단계: 라우터 사전 설정 (콘솔/Telnet/Enable 패스워드)

```cisco
en
conf t
 no ip domain-lookup
 hostname R1
!
line con 0
 logging sync
 exec-timeout 0 0
!
line vty 0 4
 password ciscovty
 login
!
enable secret ciscoen
!
end
```

### 2단계: OSPF 구성 예시 (R2 - Area 210 + ABR)

```cisco
router ospf 100
 router-id 2.2.2.2
 passive-interface default
 no passive-interface s1/3
 no passive-interface f0/0
 network 100.10.1.0 0.0.0.255 area 210
 network 210.116.41.44 0.0.0.3 area 210
```

### 3단계: ISP-3 DR 고정 (BDR 미선출)

```cisco
! ISP-3 (DR이 될 라우터)
interface f0/0
 ip ospf priority 255
!
! ISP-2, ISP-4 (DROTHER로 만들기)
interface f0/0
 ip ospf priority 0
```

### 4단계: ISP-1에서 ACL 적용 (EX4)

```cisco
! ISP-1
access-list 101 deny tcp host 198.210.10.1 host 110.11.2.2 eq 23
access-list 101 deny tcp host 198.210.10.1 host 100.10.1.1 eq 80
access-list 101 deny icmp host 198.210.10.2 host 110.11.3.3
access-list 101 deny tcp host 198.210.10.2 host 200.20.2.2 eq 80
access-list 101 permit ospf any any
access-list 101 permit ip any any
!
interface s1/1
 ip access-group 101 in
```

---

## 📚 OSPF 핵심 이론 요약 (docs/)

자세한 내용은 `docs/` 폴더를 참고하세요.

| 문서 | 내용 |
| --- | --- |
| [01. OSPF 이론](./docs/01-ospf-theory.md) | Link-State 알고리즘, AD=110, Cost 계산법 |
| [02. OSPF 5가지 PDU](./docs/02-ospf-pdu.md) | Hello, DBD, LSR, LSU, LSAck |
| [03. Neighbor State](./docs/03-ospf-neighbor-state.md) | Down→Init→2-Way→ExStart→Exchange→Loading→Full |
| [04. DR/BDR](./docs/04-dr-bdr.md) | Priority 기반 선출, Multi-Access 환경 |
| [05. LSA Type](./docs/05-lsa-type.md) | Type 1~5 (Router/Network/Summary/ASBR-Summary/External) |
| [06. Virtual-Link](./docs/06-virtual-link.md) | Backbone 단절 시 우회 연결 |
| [07. OSPF 인증](./docs/07-ospf-authentication.md) | Plain Text / MD5, Neighbor / Area 인증 |
| [08. ACL 이론](./docs/08-acl-theory.md) | Standard(1~99) / Extended(100~199) |
| [09. EX4 ACL 구현](./docs/09-ex4-acl-implementation.md) | 실습 EX4 단계별 구현 |

### OSPF Cost 계산식

```
Cost = Reference-Bandwidth(기본 10^8) / 인터페이스 Bandwidth

[기준 대역폭 변경 시: auto-cost reference-bandwidth 1000]
- GigabitEthernet : 10^9 / 10^9     = 1
- FastEthernet    : 10^9 / 10^8     = 10
- Ethernet        : 10^9 / 10^7     = 100
- Serial (T1)     : 10^9 / 1.544M   = 647
```

### LSA Type 비교표

| LSA Type | 생성 주체 | 용도 | 전파 범위 | Routing Table 표기 |
| :---: | :---: | :--- | :--- | :---: |
| Type-1 | 모든 OSPF Router | 자기 Area 내부 정보 | 동일 Area | `O` |
| Type-2 | DR | DR이 속한 네트워크 정보 | DR이 포함된 Network | 표기 없음 |
| Type-3 | ABR | 다른 Area 정보 요약 | ABR이 속한 Area | `O IA` |
| Type-4 | ABR | ASBR 위치 광고 | ABR이 속한 Area | 표기 없음 |
| Type-5 | ASBR | 외부(재분배) 네트워크 | ASBR이 속한 Area | `O E1`, `O E2` |

---

## 🔍 검증 명령어

```cisco
show ip interface brief             # 인터페이스 상태
show ip ospf neighbor               # OSPF Neighbor 상태 (Full 확인)
show ip ospf interface <int>        # 인터페이스별 OSPF 정보
show ip ospf database               # LSDB 요약
show ip ospf database router        # Type-1 LSA 상세
show ip route                       # 라우팅 테이블
show ip route ospf                  # OSPF로 학습한 경로만
show access-list                    # ACL 매칭 카운트
show ip interface s1/1              # ACL 적용 여부 확인
debug ip ospf adj                   # 인접관계 디버그
debug ip ospf packet                # OSPF Packet 디버그 (인증 검증)
```

---

## 💡 학습 포인트

- **Multi-Area OSPF**: Area 0(백본)과 비-백본 Area의 ABR 역할 이해
- **DR/BDR 선출 제어**: Priority `0`(선출 제외) / `255`(우선 DR) 활용
- **Passive-interface default**: 보안과 효율을 위한 OSPF 송신 제어
- **Loopback Point-to-Point**: 기본 /32가 아닌 인터페이스 마스크 그대로 광고
- **Extended ACL**: 출발지/목적지 IP + Port + Protocol 조합 필터링
- **OSPF Protocol 89 허용**: ACL 적용 시 OSPF Neighbor가 끊기지 않도록 `permit ospf` 추가

---

## 🛠️ 사용 도구

| 도구 | 용도 |
| --- | --- |
| **GNS3** | 네트워크 시뮬레이션 |
| **Cisco IOS (c3745)** | 라우터 OS |
| **Cisco CLI** | 라우터 명령어 입력 |
| **MobaXterm / PuTTY** | 콘솔 접속 |
| **Git / GitHub** | 형상 관리 |

---

## 📄 License

[MIT License](./LICENSE)
