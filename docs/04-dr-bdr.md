# 04. DR / BDR (Designated Router / Backup DR)

## 📌 DR/BDR 개념

Multi-Access(Broadcast, NBMA) 환경에서는 모든 라우터가 서로 Full 인접 관계를 형성하면 LSA Flooding이 비효율적입니다. 이를 해결하기 위해 **DR/BDR**을 선출하여 중앙 집중식으로 LSA를 교환합니다.

```
N개의 라우터가 모두 Full로 연결 → N(N-1)/2 개의 인접 관계 필요
DR/BDR 도입 → (N-1) 개의 인접 관계로 축소
```

---

## 📌 DR/BDR 선출 기준

1. **OSPF Priority가 가장 큰 라우터** → DR
2. **Priority가 같다면 Router-ID가 가장 큰 라우터** → DR
3. 두 번째로 큰 라우터 → BDR
4. 나머지는 **DROTHER**
5. **Priority = 0** 이면 DR/BDR 선출에서 제외 (영구 DROTHER)

> 📌 **Priority 기본값 = 1**, 범위 `0 ~ 255`

---

## 📌 Priority 설정

```cisco
interface FastEthernet0/0
 ip ospf priority 255      ! 가장 우선적으로 DR이 되도록
!
interface FastEthernet0/0
 ip ospf priority 0        ! DR/BDR 선출에서 제외
```

---

## 📌 DR/BDR 선출 특징

- **비선점(Non-Preemptive)**: 한번 선출된 DR/BDR은 다운되기 전까지 유지됩니다.
- **Wait 타이머 (40초)**: OSPF가 시작되면 40초 동안 다른 라우터를 기다린 후 선출
- 따라서 **이미 운영 중인 네트워크에 더 높은 Priority의 라우터가 들어와도 DR이 바뀌지 않습니다.**

### DR을 강제로 변경하려면

```cisco
clear ip ospf process     ! 모든 라우터에서 실행
```

---

## 📌 DR/BDR이 선출되지 않는 환경

| Network Type | DR/BDR 선출 |
| --- | :---: |
| Broadcast (Ethernet) | ✅ 선출 |
| Non-Broadcast (Frame-Relay) | ✅ 선출 |
| **Point-to-Point** | ❌ 미선출 |
| **Point-to-Multipoint** | ❌ 미선출 |
| **Loopback** | ❌ 미선출 |

---

## 📌 통신 멀티캐스트

| 방향 | 주소 |
| --- | --- |
| DR → DROTHER | **224.0.0.5** (All OSPF Router) |
| DROTHER → DR | **224.0.0.6** (All DR/BDR) |

---

## 📌 본 실습 적용 (ISP-2 / ISP-3 / ISP-4)

본 실습에서는 ISP-2, ISP-3, ISP-4가 ISP-SW로 연결된 Multi-Access 구간에서  
**ISP-3 = DR**, **BDR은 선출하지 않도록** 설정합니다.

```cisco
! ISP-3 (DR로 선출)
interface f0/0
 ip ospf priority 255

! ISP-2, ISP-4 (DROTHER로 만들기 - BDR 미선출)
interface f0/0
 ip ospf priority 0
```

### 검증

```cisco
ISP-3# show ip ospf neighbor
Neighbor ID   Pri   State           Dead Time   Address          Interface
2.2.2.2         0   FULL/DROTHER    00:00:32    210.116.41.66    Fa0/0
4.4.4.4         0   FULL/DROTHER    00:00:35    210.116.41.68    Fa0/0
```

`Backup Designated Router (ID) 0.0.0.0` 으로 표시되면 BDR이 선출되지 않은 것입니다.
