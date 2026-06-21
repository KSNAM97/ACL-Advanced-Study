# 03. ICMP 단방향 필터링

## 📌 개요

ICMP 프로토콜은 **Request (echo)** 와 **Reply (echo-reply)** 가 모두 같은 ICMP 로 분류된다.
ACL 에서 `icmp` 만 차단하면 **양방향 모두 차단**되므로,
내부 → 외부 ICMP 는 허용하고 외부 → 내부 ICMP 만 차단하려면
**echo / echo-reply 키워드를 활용**해야 한다.

---

## 🧪 EX2) 외부 → 내부 ICMP 차단

### 요구 조건
- R4 내부 네트워크 (`13.13.4.0/24`, `13.13.14.0/24`) 로의 모든 ICMP 차단
- 그 외 트래픽은 허용

### R4 설정
```cisco
ip access-list extended ACL
 deny   icmp any 13.13.4.0  0.0.0.255
 deny   icmp any 13.13.14.0 0.0.0.255
 permit ip any any
!
interface fastethernet0/0
 ip access-group ACL in
```

### ⚠️ 문제점
- ICMP 를 단순히 deny 하면 **내부 → 외부 ICMP 응답도 차단** 된다
- 즉, R4 에서 외부로 ping 을 보내도 echo-reply 가 차단되어 통신 불가

---

## 🧪 EX2-2) 내부 발생 ICMP 응답만 허용

### 요구 조건
- 외부 → 내부 ICMP 차단
- 단, 내부에서 외부로 보낸 ICMP 의 응답 (echo-reply) 은 허용
- 그 외 모든 트래픽 허용

### R4 설정
```cisco
ip access-list extended ACL
 permit icmp any 13.13.4.0  0.0.0.255 echo-reply
 permit icmp any 13.13.14.0 0.0.0.255 echo-reply
 deny   icmp any 13.13.4.0  0.0.0.255
 deny   icmp any 13.13.14.0 0.0.0.255
 permit ip   any any
!
interface fastethernet0/0
 ip access-group ACL in
```

---

## ✅ 검증

### 외부 → R4 (차단)
```
R1#ping 13.13.4.4 source 13.13.1.1
Sending 5, 100-byte ICMP Echos to 13.13.4.4, timeout is 2 seconds:
Packet sent with a source address of 13.13.1.1
UUUUU
Success rate is 0 percent (0/5)
```

### R4 → 외부 (허용 - echo-reply 통과)
```
R4#ping 13.13.3.3 source 13.13.4.4
Sending 5, 100-byte ICMP Echos to 13.13.3.3, timeout is 2 seconds:
Packet sent with a source address of 13.13.4.4
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 100/118/124 ms
```

---

## 💡 포인트

- ICMP Type 키워드
  - `echo` : ICMP Request (Type 8)
  - `echo-reply` : ICMP Reply (Type 0)
  - `unreachable`, `time-exceeded` 등도 사용 가능
- **ACL 순서가 중요**: permit 을 먼저 정의해야 echo-reply 가 통과
- 이런 방식은 **단방향 ICMP 정책**의 기본
- 더 안전한 방법은 **Reflective ACL** 사용 (다음 챕터 참조)
