# 10. TCP Intercept ─ Intercept Mode

## 📌 TCP Intercept 개요

**TCP Intercept** 는 **TCP SYN Flooding** 으로 인한 **DoS (Denial of Service)** 공격을 차단하는 기능이다.

### TCP SYN Flooding 공격 원리
1. 공격자가 도달 불가능한 IP 주소를 출발지로 위조하여 Server 에 TCP SYN 전송
2. Server 는 SYN/ACK 를 위조된 IP 로 전송 → 응답(ACK)을 무한정 대기
3. 세션이 **Half-open 상태**로 누적되면 Server 자원 고갈 → 정상 서비스 불가

### TCP Intercept 모드

| Mode | 동작 방식 |
| --- | --- |
| **Intercept** (Default) | Gateway 가 TCP 3-way 를 대신 수행, 검증 후 Server 와 연결 |
| **Watch** | 세션을 감시, Timeout 시 RST 전송하여 Half-open 정리 |
| **Drop** | 1분 내 1100 개 Half-open 시 순차적으로 세션 삭제 |

### 설정 형식
```
ip tcp intercept list [ACL 이름/번호]
ip tcp intercept mode [intercept | watch | drop]
ip tcp intercept connection-timeout [Sec]
```

---

## 📌 Intercept Mode 상세

### 정상 사용자 환경
1. Client → Server: TCP SYN 전송
2. **Router 가 가로채어 수신**
3. Router 가 Server IP 를 출발지로 위장하여 Client 에게 **SYN+ACK** 전송
4. Client → Server: ACK 전송
5. **Router 가 가로채어 수신** → 3-way handshake 1차 완료
6. Router 가 Client IP 를 출발지로 위장하여 Server 에게 SYN 전송
7. Server → Client: SYN+ACK (Router 가 수신, Client 에게 전달 X)
8. Router 가 Client IP 를 출발지로 위장하여 Server 에게 ACK 전송 → 2차 완료
9. Router 가 **Client-Router 세션과 Router-Server 세션을 연결(Splice)**

### 공격으로 간주되는 경우
1. Client → Router (Server 위장): SYN
2. Router → Client: SYN+ACK
3. **30초 (기본) 동안 ACK 미수신**
4. Router → Client: RST 전송 후 세션 강제 종료
5. **Server 는 SYN 자체를 받지 않음** → 자원 보호

---

## 🧪 EX1) Intercept Mode 구성

### 토폴로지
- **R2** = Server (`211.241.228.2`)
- **R1** = Gateway
- **R4** = Internet (Client/공격자)

### 요구 조건
- R1 은 R4 가 연결된 외부 네트워크에서 들어오는 TCP SYN Flooding 공격으로부터
- 내부 Server `211.241.228.2` 를 보호
- Gateway 인 R1 이 TCP 연결에 직접 관여

### R1 설정
```cisco
ip access-list extended TCP_INTERCEPT
 permit tcp any host 211.241.228.2
!
ip tcp intercept list TCP_INTERCEPT
ip tcp intercept mode intercept
ip tcp intercept connection-timeout 30
```

---

## ✅ 검증

### 정상 동작 (R4 → Server Telnet)
```
R1#debug ip tcp intercept
R4#telnet 211.241.228.2 /source-interface loopback 0

R1#
- Jun 19 07:50:00.051: INTERCEPT: new connection (13.13.4.4:51561 SYN -> 211.241.228.2:23)
- Jun 19 07:50:00.051: INTERCEPT(*): (13.13.4.4:51561 <- ACK+SYN 211.241.228.2:23)
- Jun 19 07:50:00.111: INTERCEPT: 1st half of connection is established 
                       (13.13.4.4:51561 ACK -> 211.241.228.2:23)
- Jun 19 07:50:00.115: INTERCEPT(*): (13.13.4.4:51561 SYN -> 211.241.228.2:23)
- Jun 19 07:50:00.147: INTERCEPT: 2nd half of connection established 
                       (13.13.4.4:51561 <- ACK+SYN 211.241.228.2:23)
- Jun 19 07:50:00.147: INTERCEPT(*): (13.13.4.4:51561 ACK -> 211.241.228.2:23)
- Jun 19 07:50:00.151: INTERCEPT(*): (13.13.4.4:51561 <- WINDOW 211.241.228.2:23)
```

### 비정상 동작 시뮬레이션 (ACK 미수신)

#### R1 에서 ACK 차단 설정 (테스트용)
```cisco
access-list 101 deny  tcp any host 211.241.228.2 ack
access-list 101 permit ip any any
!
interface fastethernet0/0
 ip access-group 101 in
```

#### 디버그 결과
```
R1#
- Jun 19 07:59:55.567: INTERCEPT: new connection (13.13.4.4:37032 SYN -> 211.241.228.2:23)
- Jun 19 07:59:55.571: INTERCEPT(*): (13.13.4.4:37032 <- ACK+SYN 211.241.228.2:23)
- Jun 19 07:59:56.571: INTERCEPT(*): SYNRCVD retransmit 1 (13.13.4.4:37032 <- ACK+SYN 211.241.228.2:23)
- Jun 19 07:59:58.571: INTERCEPT(*): SYNRCVD retransmit 2 (13.13.4.4:37032 <- ACK+SYN 211.241.228.2:23)
- Jun 19 08:00:02.571: INTERCEPT(*): SYNRCVD retransmit 3 (13.13.4.4:37032 <- ACK+SYN 211.241.228.2:23)
- Jun 19 08:00:10.571: INTERCEPT(*): SYNRCVD retransmit 4 (13.13.4.4:37032 <- ACK+SYN 211.241.228.2:23)
- Jun 19 08:00:26.571: INTERCEPT: SYNRCVD retransmitting too long 
                       (13.13.4.4:37032 <-> 211.241.228.2:23)
- Jun 19 08:00:26.571: INTERCEPT(*): (13.13.4.4:37032 <- RST 211.241.228.2:23)
```

---

## 💡 포인트

- **Intercept Mode 는 Default Mode**, 가장 강력한 보호
- Gateway 가 TCP 연결을 직접 대행 → CPU 부하 증가 (성능 트레이드오프)
- `connection-timeout` : 정상 연결된 세션의 Idle Timeout (기본 24시간)
- `watch-timeout` : SYN 수신 후 ACK 대기 시간 (기본 30초)
- ACL 매칭된 트래픽만 Intercept 동작 → 보호 대상 Server 만 지정 가능
