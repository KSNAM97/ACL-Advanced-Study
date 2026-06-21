# 07. Dynamic ACL (Lock & Key)

## 📌 개요

**Dynamic ACL** (Lock & Key) 은 외부에서 내부로 들어오는 모든 트래픽을 기본적으로 차단하고,
**Telnet 인증을 거친 사용자에게만 일정 시간 동안 동적으로 ACL 항목을 추가** 하는 보안 기능이다.

### 동작 흐름
1. 외부 사용자가 라우터에 Telnet 접속 시도
2. 라우터에서 `username/password` 인증
3. 인증 성공 시 `autocommand` 로 Telnet 세션 자동 종료
4. 사용자 IP 주소에 대한 임시 ACL 항목이 자동 추가됨
5. Timeout 동안 내부 네트워크 접근 허용

---

## 🧪 EX) Dynamic ACL 구성

### 요구 조건
- 외부 사용자는 R3 에 Telnet 인증 후
- 내부 네트워크 `13.13.15.0/24` 로 5분간 접근 가능

### R3 설정
```cisco
username soladmin privilege 15 password solcisco
username soladmin autocommand access-enable host timeout 5
!
line vty 0 4
 login local
!
access-list 105 permit udp any eq 520 any eq 520
access-list 105 permit tcp any host 13.13.3.3 eq 23
access-list 105 dynamic SOL_DYNAMIC timeout 5 permit ip any 13.13.15.0 0.0.0.255
!
interface serial1/0.123
 ip access-group 105 in
!
interface serial1/1
 ip access-group 105 in
```

---

## ✅ 검증

### ACL 상태 (초기)
```
R3#show access-list
Extended IP access list 105
    10 permit udp any eq rip any eq rip (3 matches)
    20 permit tcp any host 13.13.3.3 eq telnet
    30 Dynamic SOL_DYNAMIC permit ip any 13.13.15.0 0.0.0.255
```

### 인증 전 (외부 → 내부 차단)
```
R4#ping 13.13.15.5 source 13.13.14.4
Sending 5, 100-byte ICMP Echos to 13.13.15.5, timeout is 2 seconds:
Packet sent with a source address of 13.13.14.4
U.U.U
Success rate is 0 percent (0/5)

R4#telnet 13.13.15.5 /source-interface fastEthernet 0/1
Trying 13.13.15.5 ...
% Destination unreachable; gateway or host down
```

### R3 에 Telnet 인증
```
R4#telnet 13.13.3.3 /source-interface fastEthernet 0/1
Trying 13.13.3.3 ... Open

User Access Verification

Username: soladmin
Password: 

[Connection to 13.13.3.3 closed by foreign host]
```

### 인증 후 (외부 → 내부 허용)
```
R4#ping 13.13.15.5 source 13.13.14.4
Sending 5, 100-byte ICMP Echos to 13.13.15.5, timeout is 2 seconds:
Packet sent with a source address of 13.13.14.4
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 44/64/88 ms

R4#telnet 13.13.15.5 /source-interface fastEthernet 0/1
Trying 13.13.15.5 ... Open

User Access Verification
Password: 
R5>
```

---

## 💡 포인트

- `autocommand access-enable host` : 인증 즉시 ACL 항목 활성화 후 세션 종료
- `host` 키워드: 인증한 **사용자 IP 만** 허용 (없으면 모든 IP 허용 위험)
- `timeout [분]` : Dynamic ACL 활성 시간 (Idle timeout)
- `absolute-timeout` : 절대 시간 제한 (선택적)
- VPN 이 없던 시절의 원격 접근 보안에 유용했던 기능
- 인증 트래픽 자체는 통과해야 하므로 `permit tcp any host [R3] eq 23` 필수
