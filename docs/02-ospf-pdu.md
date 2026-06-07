# 02. OSPF 5가지 PDU

OSPF는 인접 관계 형성과 LSDB 동기화를 위해 5가지 PDU를 사용합니다.

| PDU | 역할 |
| --- | --- |
| **Hello** | Neighbor 발견 및 유지 |
| **DBD (Database Description)** | LSDB 요약 정보 교환 |
| **LSR (Link-State Request)** | 누락된 LSA 요청 |
| **LSU (Link-State Update)** | LSA 전송 |
| **LSAck** | LSA 수신 확인 |

---

## 📌 1. Hello Packet

이웃 발견과 인접 관계 유지를 위한 주기적 PDU.

### Hello에 포함된 필드 (★ = 일치해야 인접 관계 형성)

| 필드 | 일치 필요 |
| --- | :---: |
| Area-ID | ★ |
| Hello / Dead Interval | ★ |
| SubnetMask | ★ |
| MTU size | ★ |
| Authentication (인증) | ★ |
| Stub Area Flag | ★ |
| Router-ID | - |
| OSPF Priority | - |
| DR / BDR | - |

> 위 5가지 항목이 일치해야 Neighbor Table에 등록되고 인접 관계로 진행됩니다.

---

## 📌 2. DBD (Database Description)

- 자신의 LSDB 요약본을 상대방에게 전송
- 상대방이 가지지 않은 LSA를 확인하기 위함
- Master/Slave가 결정된 후 진행 (Router-ID가 더 큰 쪽이 Master)

---

## 📌 3. LSR (Link-State Request)

- DBD 교환 후 자신에게 없거나 더 오래된 LSA가 확인되면 LSR로 해당 LSA를 요청

---

## 📌 4. LSU (Link-State Update)

- LSR 요청에 대한 응답
- 실제 LSA(Link-State Advertisement)를 캡슐화하여 전송
- 토폴로지 변경 시에도 LSU로 변경 정보를 광고

---

## 📌 5. LSAck (Link-State Acknowledge)

- DBD, LSR, LSU 수신 확인 응답
- 신뢰성 있는 전송 보장
