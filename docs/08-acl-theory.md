# 08. ACL (Access Control List) 이론

## 📌 ACL이란?

특정 네트워크 트래픽을 **허용(Permit) / 차단(Deny)**하는 필터링 기능. 보안 외에도 **Distribute-list, Offset-list, Route-map, NAT** 등 다양한 기능에서 트래픽 매칭 조건으로 활용됩니다.

---

## 📌 ACL의 종류

| 종류 | 번호 범위 | 특징 |
| --- | :---: | --- |
| **Standard Numbered** | 1 ~ 99 | Source IP만으로 필터링 |
| **Extended Numbered** | 100 ~ 199 | SA + DA + Protocol + Port 필터링 |
| **Standard Named** | - | 이름 기반 Standard ACL |
| **Extended Named** | - | 이름 기반 Extended ACL (개별 라인 수정 가능) |

---

## 📌 ACL 동작 원칙

1. **Top-down 처리**: 위에서부터 순차적으로 매칭 검사 → 첫 매칭 발견 시 해당 동작 수행 후 종료
2. **Implicit Deny**: 마지막에 자동으로 `deny any`가 적용됨 (명시 X)
3. **수정 제한**: Numbered ACL은 특정 라인을 부분 수정/삭제 불가
4. **적용 방향**: `in` (들어오는 트래픽), `out` (나가는 트래픽)
5. **인터페이스당 방향당 1개**의 ACL만 적용 가능

---

## 📌 Wildcard Mask

서브넷마스크의 **반대 개념**. 0 = 일치 필요, 1 = 무시.

| 표현 | 매칭 범위 |
| --- | --- |
| `0.0.0.0` | 정확히 일치 (= `host`) |
| `0.0.0.255` | /24 네트워크 전체 |
| `0.0.7.255` | /21 (8개 /24 네트워크) |
| `255.255.255.255` | 모두 일치 (= `any`) |

---

## 📌 Standard ACL

### 문법

```cisco
access-list <1-99> {permit | deny} <Source-IP> <Wildcard-Mask>
```

### 예시: 192.168.3.0/24만 허용

```cisco
access-list 1 permit 192.168.3.0 0.0.0.255
!
interface serial 1/0
 ip access-group 1 in
```

> 💡 Standard ACL은 **목적지에 가까운 인터페이스에 적용** (목적지 구분이 없으므로 너무 일찍 차단하면 다른 트래픽까지 영향)

---

## 📌 Extended ACL

### 문법

```cisco
! 일반
access-list <100-199> {permit | deny} <protocol> 
                     <SA> <SA-W/M> <DA> <DA-W/M>

! TCP/UDP + 포트
access-list <100-199> {permit | deny} <tcp|udp>
                     <SA> <SA-W/M> [eq <Src-Port>]
                     <DA> <DA-W/M> [eq <Dst-Port>]
```

### Protocol Type

| 키워드 | 설명 |
| --- | --- |
| `ip` | 모든 IP 트래픽 |
| `icmp` | Ping |
| `tcp` | Telnet(23), HTTP(80), FTP(20,21), HTTPS(443) |
| `udp` | DNS(53), TFTP(69), DHCP(67/68), RIP(520) |
| `ospf` | OSPF (Protocol 89) |
| `eigrp` | EIGRP (Protocol 88) |

### 예시: 3.3.3.3 → 1.1.1.1 Telnet 차단

```cisco
access-list 101 deny tcp host 3.3.3.3 host 1.1.1.1 eq 23
access-list 101 permit ip any any
!
interface serial 1/0
 ip access-group 101 in
```

> 💡 Extended ACL은 **출발지에 가까운 인터페이스에 적용** (불필요한 트래픽이 네트워크를 거치기 전 차단)

---

## 📌 Named ACL

### Standard Named

```cisco
ip access-list standard R1_ACL
 permit 1.1.1.0 0.0.0.255
 permit 13.13.10.0 0.0.0.255
!
interface s1/0
 ip access-group R1_ACL in
```

### Extended Named

```cisco
ip access-list extended R1_ACL_EXT
 permit tcp 1.1.1.0 0.0.0.255 13.13.30.0 0.0.0.255 eq telnet
 permit icmp 13.13.10.0 0.0.0.255 3.3.3.0 0.0.0.255
 permit ospf any any
!
interface s1/1
 ip access-group R1_ACL_EXT in
```

### 특정 라인 추가/삭제 (Named ACL의 장점)

```cisco
ip access-list extended R1_ACL_EXT
 25 permit tcp 2.2.0.0 0.0.255.255 3.3.3.0 0.0.0.255 eq 80    ! 25번 라인 삽입
 no 25                                                         ! 25번 라인 삭제
```

---

## 📌 라우팅 프로토콜과 ACL 공존

ACL을 적용하면 라우팅 프로토콜 패킷도 차단될 수 있으므로 **반드시 허용**해야 합니다.

| 프로토콜 | ACL 허용 명령 |
| --- | --- |
| **RIP v2** | `permit udp any any eq 520` |
| **EIGRP** | `permit eigrp any any` |
| **OSPF** | `permit ospf any any` |
| **BGP** | `permit tcp any any eq 179` |

### OSPF의 경우 구체적 허용

```cisco
! OSPF 멀티캐스트
access-list 101 permit ospf host <Neighbor-IP> host 224.0.0.5
access-list 101 permit ospf host <Neighbor-IP> host 224.0.0.6   ! Multi-Access
! Virtual-Link 통과 시
access-list 101 permit ospf host <Remote-RID> host <Local-RID>
```

---

## 📌 ACL 검증

```cisco
Router# show access-list                  ! ACL 목록 + 매칭 카운트
Router# show ip access-list               ! IP ACL만
Router# show ip interface s1/0            ! 인터페이스에 적용된 ACL 확인
   Outgoing access list is not set
   Inbound  access list is 101            ← 적용 확인
Router# clear access-list counters        ! 매칭 카운터 초기화
```
