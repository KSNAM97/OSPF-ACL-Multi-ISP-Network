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

![OSPF Multi-Area Topology](./topology/ospf_acl_topology.png)

- **OSPF Area 100**: R1 + 내부 LAN (198.210.10.0/27)
- **OSPF Area 0 (Backbone)**: ISP-1 ~ ISP-6
- **OSPF Area 210**: R2 + R3 + 내부 LAN (100.10.1.0/24, 100.10.2.0/24)
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

Copy
OSPF-MultiArea-ACL-Lab/ ├── topology/ │ └── ospf_acl_topology.png # GNS3 토폴로지 이미지 ├── docs/ # OSPF 이론 문서 │ ├── 01-ospf-theory.md # OSPF 개요, AD/Metric, Area │ ├── 02-ospf-pdu.md # 5가지 PDU (Hello/DBD/LSR/LSU/LSAck) │ ├── 03-ospf-neighbor-state.md # 인접관계 7단계 상태 │ ├── 04-dr-bdr.md # DR/BDR 선출 메커니즘 │ ├── 05-lsa-type.md # LSA Type 1~5 정리 │ ├── 06-virtual-link.md # Virtual-Link 구성 │ ├── 07-ospf-authentication.md # Neighbor / Area 인증 │ └── 08-acl-theory.md # Standard / Extended ACL ├── preconfig/ # 단계별 사전 설정 (Pre-config) │ ├── 01-ip-preconfig.md # 인터페이스 IP 할당 │ ├── 02-loopback-preconfig.md # Loopback 0/1 생성 │ ├── 03-ospf-routing-preconfig.md # OSPF Area 0/100/210 구성 │ ├── 04-dr-bdr-priority-preconfig.md # ISP-3 DR 고정 설정 │ └── 05-acl-preconfig.md # ISP-1 Extended ACL 적용 ├── verification/ │ └── verification-commands.md # show / debug 명령어 모음 ├── LICENSE └── README.md

Copy
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
2단계: OSPF 구성 예시 (R2 - Area 210 + ABR)
Copyrouter ospf 100
 router-id 2.2.2.2
 passive-interface default
 no passive-interface s1/3
 no passive-interface f0/0
 network 100.10.1.0 0.0.0.255 area 210
 network 210.116.41.44 0.0.0.3 area 210
3단계: ISP-3 DR 고정 (BDR 미선출)
Copy! ISP-3 (DR이 될 라우터)
interface f0/0
 ip ospf priority 255
!
! ISP-2, ISP-4 (DROTHER로 만들기)
interface f0/0
 ip ospf priority 0
4단계: ISP-1에서 ACL 적용 (EX4)
Copy! ISP-1
access-list 101 deny tcp host 198.210.10.1 host 110.11.2.2 eq 23
access-list 101 deny tcp host 198.210.10.1 host 100.10.1.1 eq 80
access-list 101 deny icmp host 198.210.10.2 host 110.11.3.3
access-list 101 deny tcp host 198.210.10.2 host 200.20.2.2 eq 80
access-list 101 permit ospf any any
access-list 101 permit ip any any
!
interface s1/1
 ip access-group 101 in
📚 OSPF 핵심 이론 요약 (docs/)
자세한 내용은 docs/ 폴더를 참고하세요.

