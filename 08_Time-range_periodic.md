# 08. Time-range (periodic)

## 📌 개요

**Time-range** 는 ACL 이 특정 날짜나 시간에만 동작하도록 제한하는 기능이다.

- 라우터/스위치에 설정된 ACL 은 관리자가 삭제하기 전까지 24시간 동작
- Time-range 와 연결하면 **업무시간 / 야간 / 주말** 등 지정된 시간에만 활성화
- `periodic` : 요일/시간대 반복
- `absolute` : 절대 기간 (다음 챕터)

### 명령어 형식
```
time-range [이름]
 periodic [요일] HH:MM to HH:MM
```

### 요일 키워드
| 키워드 | 의미 |
| --- | --- |
| `monday`~`sunday` | 특정 요일 |
| `weekdays` | 평일 (월~금) |
| `weekend` | 주말 (토~일) |
| `daily` | 매일 |

---

## 🧪 EX1) 평일/주말 시간별 ACL

### 요구 조건
- R4 `13.13.4.4` 로의 Telnet, HTTP 접속을
  - **평일** 오전 8시 ~ 오후 6시
  - **주말** 오전 10시 ~ 오후 4시
  에만 허용

### R4 설정
```cisco
time-range R3_HOST_13.13.4.4
 periodic weekdays 08:00 to 18:00
 periodic weekend  10:00 to 16:00
!
access-list 101 permit udp any eq 520 any eq 520
access-list 101 permit tcp any host 13.13.4.4 eq 23 time-range R3_HOST_13.13.4.4
access-list 101 permit tcp any host 13.13.4.4 eq 80 time-range R3_HOST_13.13.4.4
!
interface fastethernet0/0
 ip access-group 101 in
```

---

## ✅ 검증

### Time-range 비활성 시간대
```
R4#show access-lists
Extended IP access list 101
    10 permit udp any eq rip any eq rip
    20 permit tcp any host 13.13.4.4 eq telnet time-range R3_HOST_13.13.4.4 (inactive)
    30 permit tcp any host 13.13.4.4 eq www    time-range R3_HOST_13.13.4.4 (inactive)

R5#telnet 13.13.4.4
Trying 13.13.4.4 ...
% Destination unreachable; gateway or host down
```

### 시간 변경 후 (활성 시간대)
```
R4#clock set 17:00:00 june 18 2026

R4#show access-lists
Extended IP access list 101
    10 permit udp any eq rip any eq rip (3 matches)
    20 permit tcp any host 13.13.4.4 eq telnet time-range R3_HOST_13.13.4.4 (active)
    30 permit tcp any host 13.13.4.4 eq www    time-range R3_HOST_13.13.4.4 (active)

R5#telnet 13.13.4.4
Trying 13.13.4.4 ... Open
User Access Verification
Password: 
R4>
```

---

## 🧪 EX2) 요일별 + 프로토콜별 Time-range

### 요구 조건
- R5 `13.13.5.5` 로의
  - **Telnet/HTTP**: 평일 월·수·금 (07:00~16:00) / 화·목 (10:00~19:00) / 주말 (13:00~17:00)
  - **ICMP**: 주말 (13:00~17:00) 만

### R5 설정
```cisco
time-range ACL-TCP
 periodic monday wednesday friday 07:00 to 16:00
 periodic tuesday thursday        10:00 to 19:00
 periodic weekend                 13:00 to 17:00
!
time-range ACL-ICMP
 periodic weekend 13:00 to 17:00
!
access-list 102 permit udp  any eq 520 any eq 520
access-list 102 permit tcp  any host 13.13.5.5 eq 23 time-range ACL-TCP
access-list 102 permit tcp  any host 13.13.5.5 eq 80 time-range ACL-TCP
access-list 102 permit icmp any host 13.13.5.5       time-range ACL-ICMP
!
interface fastethernet0/0
 ip access-group 102 in
```

---

## ✅ 검증

### 평일 월요일 14:30 - Telnet 허용
```
R5#clock set 14:30:00 27 July 2026
R2#telnet 13.13.5.5     [접속 O]
```

### 평일 월요일 20:30 - Telnet 차단
```
R5#clock set 20:30:00 27 July 2026
R2#telnet 13.13.5.5     [접속 X]
```

### 주말 일요일 13:30 - ICMP 허용
```
R5#clock set 13:30:00 26 July 2026
R2#ping 13.13.5.5       [통신 O]
```

### 주말 토요일 18:30 - ICMP 차단
```
R5#clock set 18:30:00 7 February 2026
R2#ping 13.13.5.5       [통신 X]
```

---

## 💡 포인트

- 라우터 시간은 NTP 동기화 권장 (`ntp server [IP]`)
- 요일 키워드는 **여러 개 나열 가능** (e.g. `monday wednesday friday`)
- 동일 time-range 에 `periodic` 여러 줄 등록 가능 (OR 조건)
- `show access-lists` 에서 `(active)` / `(inactive)` 로 현재 상태 확인 가능
