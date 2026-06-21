# 05. Established ACL

## 📌 개요

**`established`** 키워드는 TCP 헤더의 **ACK 또는 RST 비트가 설정된 패킷만 매칭** 한다.
즉, 이미 연결된 TCP 세션의 응답 패킷만 허용하고, 새로 시작되는 TCP 연결(SYN)은 차단할 수 있다.

| 옵션 | 동작 |
| --- | --- |
| `ack` | TCP ACK 비트가 설정된 패킷만 매칭 (응답 트래픽) |
| `established` | ACK 또는 RST 비트가 설정된 패킷만 매칭 |
| 미지정 | 모든 TCP 트래픽 (SYN 포함) 매칭 |

> ⚠️ **한계**: TCP 에만 적용 가능 (UDP, ICMP 불가) → 이를 보완한 것이 **Reflective ACL**

---

## 🧪 EX4-1) 단방향 TCP 세션 허용

### 요구 조건
- R2 외부 네트워크 → R2 내부 (`13.13.2.0/24`, `13.13.12.0/24`) 로의 **TCP 트래픽 차단**
- 단, R2 내부 `13.13.12.0/24` 에서 **외부로의 TCP 접근은 허용** (응답 트래픽도 허용)
- 그 외 모든 트래픽 허용

### R2 설정
```cisco
ip access-list extended TCP_DENY
 permit tcp any 13.13.2.0  0.0.0.255 ack
 permit tcp any 13.13.12.0 0.0.0.255 ack
 deny   tcp any 13.13.2.0  0.0.0.255
 deny   tcp any 13.13.12.0 0.0.0.255
 permit ip  any any
!
interface serial1/0.123
 ip access-group TCP_DENY in
```

---

## ✅ 검증

### 내부 → 외부 Telnet (허용)
```
R2#telnet 13.13.4.4 /source-interface loop 0
Trying 13.13.4.4 ... Open
```

### 외부 → 내부 Telnet (차단)
```
R4#telnet 13.13.2.2 /source-interface loop 0
Trying 13.13.2.2 ...
% Destination unreachable; gateway or host down
```

---

## 💡 포인트

- 내부에서 외부로 TCP 연결 시 응답 패킷의 ACK 비트가 ON → `ack` 매칭으로 허용됨
- 외부에서 시작하는 SYN 패킷은 ACK 비트 OFF → 차단됨
- 단점: **상태 추적(Stateful) 없음**. 공격자가 임의로 ACK 비트만 설정한 패킷을 보내면 통과 가능
- 더 안전한 방법: **Reflective ACL** 또는 **Zone-Based Firewall**

---

## 📊 Established vs Reflective ACL

| 항목 | Established | Reflective |
| --- | --- | --- |
| 적용 프로토콜 | TCP only | TCP, UDP, ICMP |
| 상태 추적 | ❌ (단순 비트 검사) | ✅ (State Table) |
| 위조 방지 | ❌ | ✅ |
| 설정 복잡도 | 단순 | 중간 |
