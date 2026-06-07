# 03. OSPF Neighbor 상태 7단계

OSPF는 Neighbor와 완전한 인접 관계(Full)를 형성하기까지 7단계를 거칩니다.

| 단계 | 설명 |
| --- | --- |
| **Down** | 초기 단계. Hello를 수신하지 못한 상태 |
| **Init** | 상대방 Hello를 받았으나 내 Router-ID가 상대의 Hello에 포함되지 않은 상태 |
| **2-Way** | 양방향 Hello 교환 완료. Neighbor Table에 등록됨. (Multi-Access에서는 DR/BDR 선출) |
| **ExStart** | Master/Slave 결정. Router-ID가 더 큰 쪽이 Master |
| **Exchange** | DBD를 교환하여 LSDB 요약 정보 비교 |
| **Loading** | LSR로 누락된 LSA 요청 → LSU로 수신 |
| **Full** | LSDB 동기화 완료. 정상 인접 관계 |

### 추가 상태

| 상태 | 설명 |
| --- | --- |
| **Attempt** | NBMA 환경에서 `neighbor` 명령으로 수동 지정한 이웃에 응답이 없을 때 |

---

## 📌 인접 관계 디버깅

```cisco
R1# debug ip ospf adj
OSPF adjacency events debugging is on

R1# clear ip ospf process
Reset ALL OSPF processes? [no]: y

*Mar 1 00:15:44: OSPF: 2 Way Communication to 2.2.2.2 on Serial1/0, state 2WAY
*Mar 1 00:15:44: OSPF: NBR Negotiation Done. We are the SLAVE
*Mar 1 00:15:44: OSPF: Rcv DBD from 2.2.2.2 ... state EXCHANGE
*Mar 1 00:15:44: %OSPF-5-ADJCHG: Process 100, Nbr 2.2.2.2 on Serial1/0 
                                  from LOADING to FULL, Loading Done
```

---

## 📌 Multi-Access 환경의 인접 관계

DR/BDR이 선출되는 Broadcast/NBMA 환경에서:

- **DR ↔ 모든 라우터**: `FULL/DR`
- **BDR ↔ 모든 라우터**: `FULL/BDR`
- **DROTHER ↔ DROTHER**: `2-WAY/DROTHER` (Full로 진행하지 않음)

```cisco
R1# show ip ospf neighbor
Neighbor ID   Pri   State           Dead Time   Address        Interface
2.2.2.2         1   2WAY/DROTHER    00:00:37    192.168.1.2    Fa0/0
3.3.3.3         1   2WAY/DROTHER    00:00:39    192.168.1.3    Fa0/0
4.4.4.4         1   FULL/BDR        00:00:35    192.168.1.4    Fa0/0
5.5.5.5         1   FULL/DR         00:00:39    192.168.1.5    Fa0/0
```
