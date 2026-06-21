# 04. IP Fragment 필터링

## 📌 IP Fragment 란?

통신 장비는 데이터 전송 시 **MTU Size (기본 1500 Byte)** 보다 큰 데이터를 그대로 전송할 수 없다.
전송할 데이터의 크기가 MTU 보다 클 경우, 데이터를 **MTU 단위로 분할(Fragment)** 하여 전송한다.

- 분할된 패킷은 수신측에서 재조립되어야 함 → CPU 부하 증가
- 공격자는 일부러 큰 IP 패킷을 보내 라우터/서버에 부하를 유발 (Fragment 기반 DoS)

---

## 🧪 EX3-1) ICMP Fragment DoS 차단

### 요구 조건
- R4 에 연결된 Server `13.13.4.4` 가 외부로부터의
  **ICMP IP Fragments 기반 DoS 공격 트래픽**을 처리하지 않도록 차단

### R4 설정
```cisco
ip access-list extended ACL
 deny   icmp any host 13.13.4.4 fragment
 permit ip any any
!
interface fastethernet0/0
 ip access-group ACL in
```

---

## ✅ 검증

### 일반 ICMP (정상 - 1500 이하)
```
R5#ping 13.13.4.4 source 13.13.5.5

Sending 5, 100-byte ICMP Echos to 13.13.4.4, timeout is 2 seconds:
Packet sent with a source address of 13.13.5.5
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 88/110/124 ms
```

### 큰 사이즈 ICMP (Fragment 발생 → 차단)
```
R5#ping 13.13.4.4 source 13.13.5.5 size 4000

Sending 5, 4000-byte ICMP Echos to 13.13.4.4, timeout is 2 seconds:
Packet sent with a source address of 13.13.5.5
.....
Success rate is 0 percent (0/5)
```

---

## 💡 포인트

- `fragment` 키워드는 **Non-initial fragment (두 번째 이후 조각)** 에 매칭됨
- 첫 번째 fragment 는 L4 정보(포트 등)를 가지고 있어 일반 ACL 규칙에 매칭
- 따라서 정상 통신은 영향받지 않고, **분할된 후속 패킷만 차단**
- Fragment 기반 공격: Ping of Death, Teardrop, ICMP Fragmentation Flood 등
