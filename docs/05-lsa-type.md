# 05. LSA Type (Link-State Advertisement)

OSPF는 LSDB를 Area 단위로 관리하며, LSA Type에 따라 정보를 분리해서 저장합니다.

## 📌 라우팅 테이블 표기

| 표기 | 의미 |
| --- | --- |
| `O` | Intra-Area (같은 Area 내부 네트워크) |
| `O IA` | Inter-Area (다른 Area의 네트워크) |
| `O E1` | External Type-1 (재분배 + ASBR까지의 경로 Cost 누적) |
| `O E2` | External Type-2 (재분배 + 외부 Metric만 사용, 기본값) |

---

## 📌 LSA Type 비교표

| LSA Type | 생성 주체 | 용도 | 전파 범위 | Routing Table | Database 표기 |
| :---: | :---: | --- | --- | :---: | --- |
| **Type-1** | 모든 OSPF Router | 자기 Area 내부 링크 정보 | 동일 Area | `O` | Router Links |
| **Type-2** | DR | DR이 속한 Multi-access 네트워크 정보 | DR이 포함된 Network | 표기 없음 | Net Link States |
| **Type-3** | ABR | 다른 Area 정보를 요약하여 광고 | ABR이 속한 Area | `O IA` | Summary Net Link States |
| **Type-4** | ABR | ASBR의 위치 광고 | ABR이 속한 Area | 표기 없음 | Summary ASB Link States |
| **Type-5** | ASBR | 재분배된 외부 네트워크 광고 | ASBR이 속한 Area (Stub 제외) | `O E1`, `O E2` | Type-5 AS External |

---

## 📌 LSDB 조회

```cisco
R2# show ip ospf database

            OSPF Router with ID (20.20.20.20) (Process ID 100)

                Router Link States (Area 0)
Link ID         ADV Router      Age   Seq#       Checksum Link count
10.10.10.10     10.10.10.10     246   0x80000002 0x007B9A 4
20.20.20.20     20.20.20.20     233   0x80000002 0x00CDD5 5
30.30.30.30     30.30.30.30     234   0x80000001 0x00F94E 4
```

### 상세 보기

```cisco
R2# show ip ospf database router

  Link State ID: 10.10.10.10
  Advertising Router: 10.10.10.10
  Number of Links: 4

    Link connected to: a Stub Network
     (Link ID) Network/subnet number: 172.16.1.0
     (Link Data) Network Mask: 255.255.255.0
       TOS 0 Metrics: 1                      ← Loopback (Cost 1)

    Link connected to: another Router (point-to-point)
     (Link ID) Neighboring Router ID: 20.20.20.20
     (Link Data) Router Interface address: 13.13.12.1
       TOS 0 Metrics: 647                    ← Serial 링크 (저속)
```

---

## 📌 Loopback Network Type 변경

기본적으로 Loopback은 **Network Type = LOOPBACK**으로 인식되어 `/32`로 광고됩니다.

```cisco
R1# show ip route
O    172.16.3.3/32 [110/129] via 13.13.12.2, 00:03:09, Serial1/0   ← /32로 광고됨
```

원래 서브넷 마스크대로 광고하려면:

```cisco
interface loopback 172
 ip ospf network point-to-point
```

변경 후:

```cisco
R1# show ip route
O    172.16.3.0/24 [110/648] via 13.13.12.2, ...   ← /24로 정상 광고
```
