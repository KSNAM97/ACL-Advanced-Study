# 11. TCP Intercept ─ Watch Mode

## 📌 Watch Mode 개요

**Watch Mode** 는 Intercept Mode 와 달리 **TCP 연결에 직접 개입하지 않는다.**
대신 Server-Client 간의 TCP 세션을 **감시 (Watch)** 하며,
지정된 시간 (기본 30초) 내에 3-way handshake 가 완성되지 않으면
**Server 에게 RST 패킷을 전송**하여 Half-open 세션을 강제 종료한다.

### Intercept vs Watch 차이

| 구분 | Intercept Mode | Watch Mode |
| --- | --- | --- |
| Gateway 개입 | 적극 (TCP 대행) | 소극 (감시만) |
| 처리 위치 | Gateway 가 3-way 수행 | Server 가 직접 3-way 수행 |
| CPU 부하 | 높음 | 낮음 |
| 보호 강도 | 강함 | 보통 |
| 동작 | SYN 시점부터 개입 | Timeout 시 RST 전송 |

---

## 🧪 EX) Watch Mode 구성

### 토폴로지
- **R2** = Server (`211.241.228.2`)
- **R3** = Gateway (Watch Mode)
- **R5** = Internet (Client/공격자)

### R3 설정
```cisco
ip access-list extended TCP_WATCH
 permit tcp any host 211.241.228.2
!
ip tcp intercept list TCP_WATCH
ip tcp intercept mode watch
ip tcp intercept connection-timeout 30
```

---

## ✅ 검증

### 정상 동작 (R5 → Server Telnet)
```
R3#debug ip tcp intercept
R5#telnet 211.241.228.2 /source-interface loopback 0

TCP intercept debugging is on
R3#
- Jun 19 08:08:55.883: INTERCEPT: new connection (13.13.5.5:48446 SYN -> 211.241.228.2:23)
- Jun 19 08:08:55.915: INTERCEPT: (13.13.5.5:48446 <- ACK+SYN 211.241.228.2:23)
- Jun 19 08:08:55.947: INTERCEPT: (13.13.5.5:48446 ACK -> 211.241.228.2:23)
```

> Watch Mode 는 패킷이 직접 통과하며, Gateway 는 감시만 수행

### 비정상 동작 시뮬레이션 (ACK 미수신)

#### R3 에서 ACK 차단 (테스트용)
```cisco
access-list 102 deny  tcp any host 211.241.228.2 ack
access-list 102 permit ip any any
!
interface fastethernet0/0
 ip access-group 102 in
```

#### 디버그 결과
```
R5#telnet 211.241.228.2 /source-interface loopback 0

R3#
- Jun 19 08:30:07.527: INTERCEPT: new connection (13.13.5.5:11270 SYN -> 211.241.228.2:23)
- Jun 19 08:30:07.555: INTERCEPT: (13.13.5.5:11270 <- ACK+SYN 211.241.228.2:23)
- Jun 19 08:30:09.547: INTERCEPT: server packet passed in SYNRCVD 
                       (13.13.5.5:11270 <- 211.241.228.2:23)
- Jun 19 08:30:37.527: INTERCEPT: SYNRCVD timing out (13.13.5.5:11270 <-> 211.241.228.2:23)
- Jun 19 08:30:37.527: INTERCEPT(*): (13.13.5.5:11270 RST -> 211.241.228.2:23)
```

> 30초 후 Timeout 발생 → **Server 에게 RST 전송하여 Half-open 정리**

---

## 💡 포인트

- Watch Mode 는 **Server 가 직접 3-way 처리**, Gateway 는 감시만
- Timeout 시 **Server 에게** RST 전송 (Intercept Mode 는 Client 에게)
- 부하가 적어 대용량 트래픽 환경에 적합
- 하지만 SYN Flooding 공격 자체는 Server 에 일부 도달하므로 강한 보호는 어려움
- 강한 보호: **Intercept Mode**, 가벼운 보호: **Watch Mode**

---

## 📊 TCP Intercept Mode 비교 요약

| 항목 | Intercept | Watch | Drop |
| --- | --- | --- | --- |
| 동작 시점 | SYN 도착 즉시 | Timeout 발생 시 | 임계치 초과 시 |
| Gateway 개입 | 직접 3-way | 감시만 | 세션 삭제만 |
| RST 전송 대상 | Client | Server | - |
| CPU 부하 | 높음 | 낮음 | 매우 낮음 |
| 보호 효과 | 매우 강함 | 보통 | 응급 처치 |
| 적용 환경 | 중요 서버 | 일반 서버 | 비상 상황 |

---

## 🛡️ 운영 권장사항

1. **평시**: Watch Mode (성능 우선)
2. **공격 감지 시**: Intercept Mode 로 전환
3. **극단적 공격 시**: Drop Mode 로 응급 처치
4. **현대 환경**: TCP Intercept 는 레거시 기능, **Zone-Based Firewall (ZBF)** 또는 **별도 IPS/방화벽** 권장

### 관련 명령어
```
show tcp intercept connections    ! 현재 추적 중인 세션 확인
show tcp intercept statistics      ! 통계 정보
clear tcp intercept                ! 통계 초기화
```
