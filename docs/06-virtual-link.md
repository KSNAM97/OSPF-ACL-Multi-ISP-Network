# 06. Virtual-Link

## 📌 Virtual-Link란?

OSPF에서 모든 Area는 **Backbone Area(Area 0)**에 직접 연결되어야 합니다.  
물리적으로 Area 0와 연결할 수 없는 경우, **Virtual-Link**를 사용하여 논리적으로 Area 0과 연결합니다.

```
[Area 100] --- R1 --- [Area 0] --- R2 --- [Area 13] --- R3 --- [Area 113]
                                                                    ↑
                                            Area 113은 Area 0과 직접 연결 X
                                            → Area 13을 가로질러 Virtual-Link 설정
```

---

## 📌 Virtual-Link 구성 조건

1. 두 ABR 사이를 잇는 **Transit Area**(중간 Area)는 Stub Area가 아니어야 함
2. Virtual-Link 양 끝의 라우터는 모두 **ABR**이어야 함
3. 양쪽 라우터에서 **상대방의 Router-ID**를 명시하여 설정

---

## 📌 설정 예시

```cisco
! R2 (ABR: Area 0 ↔ Area 13)
router ospf 100
 area 13 virtual-link 3.3.3.3

! R3 (ABR: Area 13 ↔ Area 113)
router ospf 100
 area 13 virtual-link 2.2.2.2
```

> ⚠️ `area <Transit-Area> virtual-link <상대 Router-ID>` 형식  
> Transit Area는 두 ABR이 모두 속한 중간 Area

---

## 📌 검증

```cisco
R2# show ip ospf virtual-link
Virtual Link OSPF_VL0 to router 3.3.3.3 is up
  Run as demand circuit
  DoNotAge LSA allowed.
  Transit area 13, via interface Serial1/0, Cost of using 64
  Transmit Delay is 1 sec, State POINT_TO_POINT,
  Timer intervals configured, Hello 10, Dead 40, Wait 40, Retransmit 5
    Adjacency State FULL (Hello suppressed)
```

```cisco
R2# show ip ospf neighbor
Neighbor ID   Pri   State        Dead Time   Address        Interface
3.3.3.3         0   FULL/  -         -        13.13.23.3     OSPF_VL0   ← Virtual-Link
1.1.1.1         0   FULL/  -    00:00:37     13.13.12.1     Serial1/1
3.3.3.3         1   FULL/DR     00:00:33     13.13.23.3     FastEthernet0/1
```

---

## 📌 Virtual-Link 인증

Virtual-Link는 논리적으로 Area 0에 속하므로, **Area 0에 인증이 설정된 경우 Virtual-Link에도 인증을 적용**해야 합니다.

```cisco
! R2
router ospf 100
 area 13 virtual-link 3.3.3.3 message-digest-key 100 md5 cisco

! R3
router ospf 100
 area 13 virtual-link 2.2.2.2 message-digest-key 100 md5 cisco
 area 0 authentication message-digest
```
