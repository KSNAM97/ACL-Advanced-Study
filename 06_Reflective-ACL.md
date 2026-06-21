# 06. Reflective ACL

## 📌 개요

**Reflective ACL** 은 내부 → 외부로 출력되는 트래픽을 추적하여
그 응답 트래픽만 허용하는 **Stateful ACL** 이다.

- `established` 가 TCP 의 ACK 비트만 검사하는 반면,
- Reflective ACL 은 **TCP / UDP / ICMP** 모두 지원
- 내부에서 발생한 트래픽 정보를 **State Table** 에 기록하고,
  외부 응답이 들어올 때 State Table 과 비교 후 허용

### 동작 방식
1. 내부 → 외부 트래픽 발생 시 검사
2. `reflect` 가 적용된 트래픽의 SA/DA/Protocol/Port 등 정보로 **임시 ACL 항목 자동 생성**
3. 외부에서 응답 트래픽이 들어오면 `evaluate` 명령으로 임시 ACL 과 비교
4. 정상 응답이면 허용, 임의 트래픽이면 차단

---

## 🧪 EX5) Reflective ACL 구성

### 요구 조건
- R1 내부 (`13.13.4.0/24`, `13.13.14.0/24`) → 외부 로의 TCP, UDP, ICMP **허용**
- 외부 → R1 내부로 임의 시작된 트래픽은 **차단**
- 단, RIP 라우팅 (UDP/520) 은 허용해야 함

### R1 설정
```cisco
ip access-list extended IN--->OUT
 permit tcp  any any reflect REFLECTIVE_ACL
 permit udp  any any reflect REFLECTIVE_ACL
 permit icmp any any reflect REFLECTIVE_ACL
!
ip access-list extended OUT--->IN
 permit udp any eq 520 any eq 520
 evaluate REFLECTIVE_ACL
!
interface serial1/0.123
 ip access-group IN--->OUT out
 ip access-group OUT--->IN in
!
interface serial1/1
 ip access-group IN--->OUT out
 ip access-group OUT--->IN in
```

---

## ✅ 검증

### 내부 → 외부 ICMP (정상)
```
R4#ping 13.13.5.5 source 13.13.4.4

Sending 5, 100-byte ICMP Echos to 13.13.5.5, timeout is 2 seconds:
Packet sent with a source address of 13.13.4.4
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 96/140/184 ms
```

### State Table 확인
```
R1#sh access-lists

Extended IP access list IN--->OUT
    10 permit tcp  any any reflect REFLECTIVE_ACL
    20 permit udp  any any reflect REFLECTIVE_ACL
    30 permit icmp any any reflect REFLECTIVE_ACL (5 matches)

Extended IP access list OUT--->IN
    10 permit udp any eq rip any eq rip (14 matches)
    20 evaluate REFLECTIVE_ACL

Reflexive IP access list REFLECTIVE_ACL
    permit icmp host 13.13.5.5 host 13.13.4.4 (10 matches) (time left 267)
```

---

## 💡 포인트

- **`reflect [이름]`** : State Table 생성하는 트래픽 정의
- **`evaluate [이름]`** : 외부 트래픽을 State Table 과 비교
- 자동 생성된 임시 ACL 은 **SA ↔ DA 가 뒤바뀐 형태**
- **Timeout**: 기본 300초, `ip reflexive-list timeout [sec]` 로 변경 가능
- **방향**: 내부 인터페이스 기준 `out` 에서 reflect, `in` 에서 evaluate
- 라우팅 프로토콜(RIP, EIGRP, OSPF Hello 등)은 별도 permit 필요

---

## 📊 Reflective ACL 흐름도

```
[내부 Client]                [R1 Gateway]                [외부 Server]
      │                            │                            │
      │── 1. SYN ──────────────────│                            │
      │                            │── reflect (State 생성) ────│
      │                            │── 2. SYN 전달 ─────────────│
      │                            │                            │
      │                            │── 3. SYN/ACK 수신 ◀────────│
      │                            │── evaluate (State 일치) ───│
      │◀── 4. SYN/ACK 전달 ────────│                            │
      │                            │                            │
      │── 5. ACK ──────────────────│── 6. ACK 전달 ─────────────│
```
