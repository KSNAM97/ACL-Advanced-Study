# ACL Advanced Study

[![Cisco](https://img.shields.io/badge/Cisco-IOS-1BA0D7?style=flat&logo=cisco&logoColor=white)]()
[![ACL](https://img.shields.io/badge/Security-ACL-red?style=flat)]()
[![Security](https://img.shields.io/badge/Network-Security-orange?style=flat)]()
[![Routing](https://img.shields.io/badge/Routing-RIP%20%7C%20EIGRP-blue?style=flat)]()
[![GNS3](https://img.shields.io/badge/GNS3-Lab-2E8B57?style=flat)]()
[![License](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)

**Cisco ACL 심화 (Extended / Named / Reflective / Dynamic / Time-range / TCP Intercept)**

> - **Part 1** Extended ACL 기본 (Numbered / Named / log-input / fragment / established)
> - **Part 2** Reflective ACL & Dynamic ACL (Lock & Key)
> - **Part 3** Time-range ACL (periodic / absolute)
> - **Part 4** TCP Intercept (Intercept / Watch / Drop Mode)

---

## 📖 개요

Cisco IOS 라우터에서 **Extended ACL** 을 기반으로 한
**Stateful 기반 보안 기능 (Reflective ACL, Dynamic ACL, TCP Intercept)** 과
**시간 기반 트래픽 제어 (Time-range)** 를 실습한 자료입니다.

Frame-Relay (R1·R2·R3 NBMA) 기반 토폴로지에서 RIPv2 / EIGRP 라우팅이 동작하는
환경 위에 다양한 ACL 시나리오를 적용하여 트래픽을 제어합니다.

| 항목 | 내용 |
| --- | --- |
| 장비 | Cisco IOS Router (c7200), Frame-Relay Switch |
| 주요 키워드 | Extended ACL, Named ACL, Reflective ACL, Dynamic ACL, Time-range, TCP Intercept, log-input, fragment, established |
| 환경 | GNS3 / Dynamips |
| 핵심 명령어 | `ip access-list extended`, `reflect`, `evaluate`, `time-range`, `periodic`, `absolute`, `ip tcp intercept` |

---

## 📂 디렉토리 구조

```
ACL-Advanced-Study/
├── README.md
├── 01_Extended-ACL.md
├── 02_Named-ACL_log-input.md
├── 03_ICMP-Filtering.md
├── 04_IP-Fragment-ACL.md
├── 05_Established-ACL.md
├── 06_Reflective-ACL.md
├── 07_Dynamic-ACL_Lock-Key.md
├── 08_Time-range_periodic.md
├── 09_Time-range_absolute.md
├── 10_TCP-Intercept_Intercept-Mode.md
├── 11_TCP-Intercept_Watch-Mode.md
└── images/
    └── topology.png
```

---

## 📚 챕터 (Chapters)

### Part 1 ─ Extended ACL 기본

| # | 제목 | 핵심 내용 |
| --- | --- | --- |
| 01 | [Extended ACL (Numbered)](01_Extended-ACL.md) | 100~199 번호 ACL, SA/DA + Port, Telnet/HTTP/ICMP 차단 |
| 02 | [Named ACL & log-input](02_Named-ACL_log-input.md) | 이름 기반 ACL, 차단 패킷 로그 기록 (수신 인터페이스) |
| 03 | [ICMP 단방향 필터링](03_ICMP-Filtering.md) | `echo` / `echo-reply` 구분, 내부→외부 ICMP 만 허용 |
| 04 | [IP Fragment 필터링](04_IP-Fragment-ACL.md) | ICMP Fragment 기반 DoS 차단 (`fragment` 옵션) |
| 05 | [Established ACL](05_Established-ACL.md) | TCP `ack` 비트 기반 단방향 세션 제어 |

### Part 2 ─ Stateful ACL

| # | 제목 | 핵심 내용 |
| --- | --- | --- |
| 06 | [Reflective ACL](06_Reflective-ACL.md) | `reflect` + `evaluate`, 내부 → 외부 응답만 허용 (State table) |
| 07 | [Dynamic ACL (Lock & Key)](07_Dynamic-ACL_Lock-Key.md) | Telnet 인증 후 일정 시간 동안 ACL 자동 허용 |

### Part 3 ─ Time-range ACL

| # | 제목 | 핵심 내용 |
| --- | --- | --- |
| 08 | [Time-range periodic](08_Time-range_periodic.md) | 평일/주말/요일별 ACL 활성화 |
| 09 | [Time-range absolute](09_Time-range_absolute.md) | 특정 기간 (시작~종료) 동안만 ACL 활성화 |

### Part 4 ─ TCP Intercept

| # | 제목 | 핵심 내용 |
| --- | --- | --- |
| 10 | [TCP Intercept ─ Intercept Mode](10_TCP-Intercept_Intercept-Mode.md) | Gateway 가 TCP 3-way 를 대행 (Default Mode) |
| 11 | [TCP Intercept ─ Watch Mode](11_TCP-Intercept_Watch-Mode.md) | Half-open 세션 감시 후 RST 전송 |

---

## 🌐 토폴로지

![acl_Topology](./topology/acl_Topologypng)

```

- **R1, R2, R3** : Frame-Relay NBMA (multipoint)
- **R4** : R1 외부 네트워크 (FastEthernet)
- **R5** : R3 외부 네트워크 (FastEthernet)
- **라우팅** : RIPv2 (Part1~3) / EIGRP 100 (Part4)

---

## ⚙️ 핵심 개념 요약

### Extended ACL 형식
```
access-list [100-199] [permit/deny] [protocol] [SA] [W/M] [DA] [W/M]
access-list [100-199] [permit/deny] [protocol] [SA] [W/M] eq [SP] [DA] [W/M] eq [DP]
```

### Reflective ACL
- 내부 → 외부 트래픽 전송 시 **State Table** 자동 생성
- 외부 → 내부 응답 트래픽은 `evaluate` 로 State Table 과 비교 후 허용
- TCP / UDP / ICMP 모두 적용 가능 (Established 는 TCP 만 가능)

### Dynamic ACL (Lock & Key)
- 외부 → 내부 모든 트래픽 차단
- 사용자가 라우터에 **Telnet 인증** 후 일시적으로 ACL 자동 추가
- `username ... autocommand access-enable host`

### Time-range
- `periodic` : 요일/시간대 반복 (weekdays / weekend / monday~sunday)
- `absolute` : 특정 시작/종료 시점 지정
- ACL 항목 뒤에 `time-range [이름]` 으로 연결

### TCP Intercept Mode 비교

| Mode | 동작 방식 |
| --- | --- |
| **Intercept** (Default) | Gateway 가 TCP 3-way 를 대신 수행 후 Server 와 연결 |
| **Watch** | Server-Client 세션을 감시, Timeout 시 RST 전송 |
| **Drop** | 1분 내 1100 개 Half-open 시 순차적 세션 삭제 |

---

## 🛠️ 사용 도구

| 도구 | 용도 |
| --- | --- |
| **GNS3** | 네트워크 토폴로지 시뮬레이션 |
| **Dynamips** | Cisco IOS 에뮬레이션 |
| **Cisco IOS (c7200)** | Router / FR-Switch |
| **MobaXterm** | 콘솔 접속 |
| **Git / GitHub** | 학습 기록 관리 |

---

## License

[MIT License](LICENSE)
