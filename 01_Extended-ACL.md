# 01. Extended ACL (Numbered)

## 📌 개요

**Numbered Extended ACL** 은 출발지/목적지 IP 주소 + 포트번호 + 프로토콜 번호를 사용하여
특정 애플리케이션 단위로 트래픽을 정밀하게 제어할 수 있는 ACL 이다.

| 항목 | 내용 |
| --- | --- |
| 범위 | 100 ~ 199 |
| 제어 기준 | SA + DA + Protocol + Port |
| 적용 위치 | 출발지에 가까운 인터페이스 (권장) |

### 형식
```
access-list [100-199] [permit/deny] [protocol] [SA] [W/M] [DA] [W/M]
access-list [100-199] [permit/deny] [protocol] [SA] [W/M] eq [SP] [DA] [W/M] eq [DP]
```

---

## 🧪 EX1) R2 내부 네트워크 보호

### 요구 조건
- `150.1.13.4` (R4) → `13.13.12.0/24` 로의 **Telnet** 차단
- `150.3.13.5` (R5) → `13.13.12.0/24` 로의 **HTTP** 차단
- `13.13.11.0/24` → `13.13.12.0/24` 로의 **ICMP** 차단
- 이외 모든 트래픽 허용
- R2 에서 설정

### R2 설정 (Numbered ACL)
```cisco
access-list 100 deny   tcp  host 150.1.13.4 13.13.12.0 0.0.0.255 eq telnet
access-list 100 deny   tcp  host 150.3.13.5 13.13.12.0 0.0.0.255 eq 80
access-list 100 deny   icmp 13.13.11.0 0.0.0.255 13.13.12.0 0.0.0.255
access-list 100 permit ip   any any
!
interface serial1/0.123
 ip access-group 100 in
```

### R2 설정 (Named ACL)
```cisco
ip access-list extended ACL
 deny   tcp  host 150.1.13.4 13.13.12.0 0.0.0.255 eq telnet
 deny   tcp  host 150.3.13.5 13.13.12.0 0.0.0.255 eq 80
 deny   icmp 13.13.11.0 0.0.0.255 13.13.12.0 0.0.0.255
 permit ip   any any
!
interface serial1/0.123
 ip access-group ACL in
```

---

## ✅ 검증

### R4 → R2 Telnet (차단)
```
R4#telnet 13.13.12.2
Trying 13.13.12.2 ...
% Destination unreachable; gateway or host down
```

### R5 → R2 HTTP (차단)
```
R5#telnet 13.13.12.2 80
Trying 13.13.12.2, 80 ...
% Destination unreachable; gateway or host down
```

### R1 → R2 ICMP (차단)
```
R1#ping 13.13.12.2 source 13.13.11.1

Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 13.13.12.2, timeout is 2 seconds:
Packet sent with a source address of 13.13.11.1
U.U.U
Success rate is 0 percent (0/5)
```

---

## 💡 포인트

- `host` 키워드는 `0.0.0.0` 와일드카드 마스크와 동일
- ACL 끝에는 암묵적 `deny any any` 가 존재하므로 마지막에 `permit ip any any` 필요
- Extended ACL 은 **출발지에 가까운 곳에 적용** 하는 것이 권장됨
- Named ACL 은 시퀀스 번호로 개별 항목 수정/삭제 가능
