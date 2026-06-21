# 09. Time-range (absolute)

## 📌 개요

`absolute` 키워드는 ACL 이 **특정 시작 시점부터 특정 종료 시점까지만** 동작하도록 한다.
이벤트 기간, 점검 기간, 임시 정책 등에 활용된다.

### 명령어 형식
```
time-range [이름]
 absolute start HH:MM DD MMM YYYY end HH:MM DD MMM YYYY
```

- `start` 만 지정 시 → 해당 시점부터 무한 활성
- `end` 만 지정 시 → 즉시 시작, 해당 시점까지 활성
- `start` + `end` → 지정 구간만 활성

---

## 🧪 EX3) 특정 기간만 허용

### 요구 조건
- R5 `13.13.5.5` 로의 Telnet, HTTP, ICMP 접속/통신은
  **2026년 7월 1일 09:30 ~ 2026년 8월 28일 18:30** 동안만 가능

### R5 설정
```cisco
time-range R5_ACL
 absolute start 09:30 01 july 2026 end 18:30 29 august 2026
!
access-list 103 permit udp  any eq 520 any eq 520
access-list 103 permit tcp  any host 13.13.5.5 eq 23 time-range R5_ACL
access-list 103 permit tcp  any host 13.13.5.5 eq 80 time-range R5_ACL
access-list 103 permit icmp any host 13.13.5.5       time-range R5_ACL
access-list 103 deny   tcp  any host 13.13.5.5 eq 23
access-list 103 deny   tcp  any host 13.13.5.5 eq 80
access-list 103 deny   icmp any host 13.13.5.5
access-list 103 permit ip   any any
!
interface fastethernet0/0
 ip access-group 103 in
```

---

## ✅ 검증

### 기간 내 (2026/07/27 14:30) - 접속/통신 허용
```
R5#clock set 14:30:00 27 july 2026

R2#telnet 13.13.5.5       [접속 O]
R2#ping 13.13.5.5         [통신 O]
```

### 기간 외 (2026/08/30 14:30) - 접속/통신 차단
```
R5#clock set 14:30:00 30 august 2026

R2#telnet 13.13.5.5       [접속 X]
R2#ping 13.13.5.5         [통신 X]
```

---

## 💡 포인트

- **ACL 구조 설계 패턴** : `permit (time-range)` + `deny` + `permit ip any any`
  - 활성 시간 → 첫 번째 permit 매칭
  - 비활성 시간 → permit 스킵 → deny 매칭으로 차단
  - 다른 트래픽 → 마지막 permit ip any any 로 통과
- `absolute` 와 `periodic` 을 **같은 time-range 안에 함께 사용 가능**
  ```
  time-range EXAMPLE
   absolute start 00:00 01 jan 2026 end 23:59 31 dec 2026
   periodic weekdays 09:00 to 18:00
  ```
- 시스템 시간이 정확해야 정상 동작 → **NTP 권장**
