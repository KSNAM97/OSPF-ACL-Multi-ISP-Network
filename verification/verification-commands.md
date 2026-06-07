# 검증 명령어 모음 (Verification Commands)

## 📌 인터페이스 / 기본 확인

```cisco
show ip interface brief              ! 인터페이스 IP / 상태 요약
show running-config                  ! 현재 동작 중인 설정
show running-config | section ospf   ! OSPF 설정만 추출
show ip protocols                    ! 동작 중인 라우팅 프로토콜
```

## 📌 OSPF 확인

```cisco
show ip ospf                         ! OSPF 프로세스 정보 (Router-ID, Area 등)
show ip ospf interface               ! 인터페이스별 OSPF 정보
show ip ospf interface brief         ! 간략 형식
show ip ospf neighbor                ! Neighbor 상태 (Full 확인)
show ip ospf neighbor detail         ! 상세 정보
show ip ospf database                ! LSDB 요약
show ip ospf database router         ! Type-1 LSA 상세
show ip ospf database network        ! Type-2 LSA 상세
show ip ospf database summary        ! Type-3 LSA 상세
show ip ospf database external       ! Type-5 LSA 상세
show ip ospf virtual-links           ! Virtual-Link 상태
show ip ospf border-routers          ! ABR/ASBR 목록
```

## 📌 라우팅 테이블

```cisco
show ip route                        ! 전체 라우팅 테이블
show ip route ospf                   ! OSPF 학습 경로만
show ip route 100.10.1.0             ! 특정 네트워크 상세
```

## 📌 ACL 확인

```cisco
show access-list                     ! 모든 ACL + 매칭 카운트
show ip access-list                  ! IP ACL만
show ip access-list 101              ! 특정 ACL
show ip interface s1/1               ! 인터페이스에 적용된 ACL 확인
clear access-list counters           ! 매칭 카운터 초기화
```

## 📌 디버그 명령어 (운영 환경 주의!)

```cisco
debug ip ospf adj                    ! 인접 관계 형성 과정
debug ip ospf hello                  ! Hello Packet 송수신
debug ip ospf packet                 ! OSPF Packet 전체 (인증 검증)
debug ip ospf events                 ! 일반 이벤트
debug ip packet                      ! IP 패킷 디버그 (부하 주의)
undebug all                          ! 모든 debug 종료
```

## 📌 OSPF 재시작

```cisco
clear ip ospf process                ! OSPF 프로세스 재시작
                                      ! Reset ALL OSPF processes? [no]: y
```

## 📌 통신 테스트

```cisco
ping 100.10.1.1                                    ! 기본 ping
ping 100.10.1.1 source loopback 0                  ! 소스 지정 ping
ping 100.10.1.1 source 198.210.10.2 repeat 100     ! 반복 횟수 지정

telnet 110.11.2.2                                  ! Telnet 접속
telnet 110.11.2.2 80                               ! 포트 지정 (HTTP 테스트)
telnet 110.11.2.2 /source-interface loopback 0     ! 소스 인터페이스 지정

traceroute 100.10.1.1                              ! 경로 추적
```
