# 02. Named ACL & log-input

## 📌 개요

**`log-input`** 옵션은 ACL 에 의해 차단/허용된 트래픽에 대해
**수신한 인터페이스 정보까지 포함하여 로그를 출력**하는 옵션이다.

| 옵션 | 설명 |
| --- | --- |
| `log` | 매칭된 패킷의 로그 메시지만 출력 |
| `log-input` | 매칭된 패킷의 로그 + **수신 인터페이스 정보** 출력 |

---

## 🧪 EX1-2) Named ACL + log-input

### 요구 조건
- EX1 과 동일 조건
- 추가: 차단되는 패킷에 **수신 INTERFACE 정보와 LOG-MESSAGE** 를 출력

### R2 설정
```cisco
ip access-list extended ACL
 deny   tcp  host 150.1.13.4 13.13.12.0 0.0.0.255 eq telnet log-input
 deny   tcp  host 150.3.13.5 13.13.12.0 0.0.0.255 eq 80     log-input
 deny   icmp 13.13.11.0 0.0.0.255 13.13.12.0 0.0.0.255      log-input
 permit ip   any any
!
interface serial1/0.123
 ip access-group ACL in
```

---

## ✅ 검증

### 로그 출력 결과
```
- Jun 18 05:57:17.571: %SEC-6-IPACCESSLOGP: list ACL denied tcp 
                       150.3.13.5(18814) (Serial1/0.123 ) -> 13.13.12.2(80), 1 packet

- Jun 18 05:57:50.443: %SEC-6-IPACCESSLOGP: list ACL denied tcp 
                       150.1.13.4(44475) (Serial1/0.123 ) -> 13.13.12.2(23), 1 packet

- Jun 18 05:58:04.319: %SEC-6-IPACCESSLOGDP: list ACL denied icmp 
                       13.13.11.1 (Serial1/0.123 ) -> 13.13.12.2 (8/0), 1 packet
```

### 트래픽 테스트
```
R1#ping 13.13.12.2 source 13.13.11.1
Success rate is 0 percent (0/5)

R4#telnet 13.13.12.2
% Destination unreachable; gateway or host down

R5#telnet 13.13.12.2 80
% Destination unreachable; gateway or host down
```

---

## 💡 포인트

- `log-input` 은 **수신 인터페이스** 정보를 함께 표시하여 공격 출처 분석 시 유리
- 로그 메시지 형태 : `list [ACL이름] denied [proto] [SA]([SP]) ([INT]) -> [DA]([DP])`
- 단, 로그가 많이 발생할 경우 라우터 CPU 부하 증가 → 운영 환경에서는 주의 필요
