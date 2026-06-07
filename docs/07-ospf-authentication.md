# 07. OSPF 인증 (Authentication)

OSPF는 잘못된 라우팅 업데이트의 송수신을 방지하기 위해 인증 기능을 제공합니다.

| 종류 | 범위 |
| --- | --- |
| **Neighbor Authentication** | 인터페이스 단위 (1:1) |
| **Area Authentication** | Area 단위 (해당 Area 내 모든 라우터에 적용) |

| 인증 방식 | 보안성 | aut 값 |
| --- | --- | :---: |
| 인증 없음 | - | 0 |
| **Plain Text** | 낮음 (평문) | 1 |
| **MD5** | 높음 (해시) | 2 |

---

## 📌 1. Neighbor Authentication (Plain Text)

```cisco
! R1
interface serial 1/0
 ip ospf authentication
 ip ospf authentication-key cisco

! R2
interface serial 1/1
 ip ospf authentication
 ip ospf authentication-key cisco
```

### 검증

```cisco
R1# show ip ospf interface serial 1/0
  ...
  Simple password authentication enabled

R1# debug ip ospf packet
*Mar 1 ...: OSPF: rcv. v:2 t:1 l:48 rid:2.2.2.2
      aid:0.0.0.0 chk:E693 aut:1 auk: from Serial1/0
                                  ↑ aut:1 = Plain Text
```

---

## 📌 2. Neighbor Authentication (MD5)

```cisco
! R1
interface serial 1/0
 ip ospf authentication message-digest
 ip ospf message-digest-key 100 md5 cisco

! R2
interface serial 1/1
 ip ospf authentication message-digest
 ip ospf message-digest-key 100 md5 cisco
```

### 검증

```cisco
R1# debug ip ospf packet
*Mar 1 ...: OSPF: rcv. v:2 t:1 l:48 rid:2.2.2.2
      aid:0.0.0.0 chk:0 aut:2 keyid:100 seq:0x3C7EC86E from Serial1/0
                              ↑ aut:2 = MD5
```

---

## 📌 3. Area Authentication (Plain Text)

해당 Area 내 **모든 라우터의 모든 인터페이스**에 동일하게 설정해야 합니다.

```cisco
! R1, R2 (Area 0 양쪽 모두)
interface serial 1/0
 ip ospf authentication
 ip ospf authentication-key cisco
!
router ospf 100
 area 0 authentication
```

---

## 📌 4. Area Authentication (MD5)

```cisco
interface serial 1/0
 ip ospf authentication message-digest
 ip ospf message-digest-key 100 md5 cisco
!
router ospf 100
 area 0 authentication message-digest
```

> ⚠️ Area 0에 인증을 설정한 경우 **Virtual-Link에도 동일한 인증**을 반드시 적용해야 합니다.

```cisco
! R2
router ospf 100
 area 13 virtual-link 3.3.3.3 message-digest-key 100 md5 cisco

! R3
router ospf 100
 area 13 virtual-link 2.2.2.2 message-digest-key 100 md5 cisco
 area 0 authentication message-digest
```
