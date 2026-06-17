---
layout: posts
title: "PPP / PPP-RTK 유료 보정신호 서비스 업체 정리 (북미·유럽·아시아)"
categories: [study, gnss]
tags: [GPS, GNSS, PPP, PPP-RTK, 자율주행, 측위, 보정신호]
---

---

# PPP / PPP-RTK 유료 보정신호 서비스 업체 정리

> PPP(Precise Point Positioning)는 전 세계 어디서든 단일 수신기로 수 cm급 위치 정확도를 달성할 수 있는 기술이다. 최근 자율주행, 정밀농업, 드론 등 고정밀 측위 수요가 급증하면서 PPP 및 PPP-RTK 유료 서비스 시장이 빠르게 성장하고 있다.
>
> 이 글에서는 북미·유럽·아시아 권역의 주요 상용 보정신호 서비스를 기술 유형·정확도·수렴시간 기준으로 정리한다.

---

## 기술 개념 먼저

헷갈리기 쉬운 기술 용어를 먼저 정리하고 넘어가자.

| 기술 | 핵심 개념 | 정확도 | 수렴시간 |
|------|-----------|--------|----------|
| **PPP** | 위성 궤도·시계 보정값을 전 세계 단일 수신기에 브로드캐스트 | 5~20 cm | 1~30분 |
| **PPP-AR** | PPP + 정수 위상 모호 정수 해 (Ambiguity Resolution) | ~2.5 cm | <1분 |
| **PPP-RTK / SSR** | PPP 기반이지만 지역 대기 보정을 추가해 RTK 수준 달성 | 1~7 cm | 수 초~20초 |
| **Network RTK (NRTK)** | 기준국 망 기반 OSR 보정, 즉각 초기화 | ~2 cm | 수 초 (즉각) |

보정신호 전달 방식은 크게 두 가지다.

- **L-band 위성**: Inmarsat 등 정지궤도(GEO) 위성으로 인터넷 없이 전 세계 수신 가능. 해양·원격지에 강점.
- **IP(인터넷)**: NTRIP 프로토콜로 스마트폰/LTE 연결. 수렴이 더 빠르고 지상 기반 보완 데이터 활용 가능.

---

## 북미 (North America)

### Trimble Inc. (미국)

**CenterPoint RTX Fast**는 Trimble의 플래그십 PPP 서비스다. L-band와 IP를 동시 지원하며, ProPoint 엔진 사용 시 1분 이내 수렴, 전 세계 2 cm RMS 수평 정확도를 제공한다. 자율주행·건설·정밀농업 전반에서 사용된다.

**RangePoint RTX**는 농업용 저가 라인업이다. 수평 정확도 15~50 cm 수준으로 서브미터급 정밀농업을 타겟으로 한다.

> 출처: [Trimble RTX 공식 페이지](https://positioningservices.trimble.com/en/rtx)

---

### Swift Navigation (미국)

**Skylark Cx**는 Swift Navigation의 PPP-RTK 서비스다. 인터넷(IP) 전용으로 **20초 이내** 고속 수렴이 특징이며, 수평 정확도 3~7 cm (95th percentile)를 달성한다. 북미·유럽 커버리지로 시작했으며 2024년 10월 KDDI 파트너십을 통해 **일본** 서비스도 개시했다.

자동차 OEM을 주요 타겟으로 하며, 구독형 API 방식으로 제공된다.

> 출처: [Swift Navigation 공식](https://www.swiftnav.com/skylark)

---

### NovAtel / Hexagon (캐나다)

NovAtel(Hexagon 계열)은 **TerraStar** 브랜드로 두 가지 서비스를 제공한다.

**TerraStar-C PRO**는 전 세계 커버리지의 PPP 서비스다. OEM7 수신기 기준 2.5 cm (95%) 수평, 5 cm (95%) 수직 정확도를 제공하며, FW 7.08.10+ 기준 수렴 시간은 약 3분이다. L-band와 IP 동시 지원.

**TerraStar-X**는 미국 한정 PPP-AR 서비스다. 마찬가지로 2.5 cm (95%) 수평, 5 cm (95%) 수직이지만, **1분 미만**으로 수렴 속도가 훨씬 빠르다. 미국 지역을 벗어나면 자동으로 TerraStar-C PRO로 fallback된다.

> 출처: [NovAtel TerraStar 비교 페이지](https://novatel.com/products/novatel-correct/terrastar)

---

### Point One Navigation (미국)

**Polaris**는 PPP-RTK 서비스로, 북미 중심의 고밀도 기준국 망을 기반으로 2~5 cm 수평 정확도를 제공한다. 자율주행 스타트업 생태계와의 통합에 강점이 있으며 IP 전용.

> 출처: [Point One Navigation 공식](https://pointonenav.com/polaris)

---

## 유럽 (Europe)

### Fugro (네덜란드)

해양 측량 전문 기업 Fugro는 **StarFix** 브랜드로 두 티어를 운영한다.

**StarFix.G4**는 PPP 서비스로 수평/수직 모두 10 cm (2σ) 이내 정확도를 제공한다. Inmarsat L-band와 IP 하이브리드 방식.

**StarFix.G4+**는 PPP-AR 강화 버전으로 수평 3 cm (2σ), 수직 6 cm (2σ) 정확도를 달성한다. 해양 플랫폼·해저 시공·에너지 인프라 등 산업용 타겟.

> 출처: [Fugro GNSS.space 공식](https://gnss.space/)

---

### Veripos / Hexagon (영국·스코틀랜드)

**Apex5**는 Veripos의 PPP-AR 서비스로 수평 3 cm (RMS) 정확도를 제공한다. L-band 전용으로 해양 시추·측량 분야에 특화. DP(Dynamic Positioning, 선박 자동위치유지) 시스템에도 활용된다.

> 출처: [Veripos 공식](https://veripos.com/products/apex5)

---

### HxGN SmartNet / Leica Geosystems (스위스·글로벌)

Hexagon의 지상 기준국 망(CORS) 기반 NTRK 서비스다. **RTK 기준 1~3 cm 수평, 2~5 cm 수직**으로 즉각 초기화가 가능하며, 유럽·북미·아시아에 걸쳐 광범위한 기준국 네트워크를 운영한다.

순수 PPP가 아닌 Network RTK 방식이지만 글로벌 측지 인프라로서 많이 참조된다.

> 출처: [HxGN SmartNet 공식](https://hxgnsmartnet.com)

---

### u-blox (스위스)

**PointPerfect Flex**는 PPP-RTK 서비스로, L-band(Inmarsat)와 IP를 동시 지원하는 하이브리드 방식이다. 수평 6 cm (RMS) 정확도, 수렴 시간 수십 초 수준. u-blox 모듈과 통합이 간편해 IoT·자동차 텔레매틱스·스마트 농업에 활용된다.

> 출처: [u-blox PointPerfect](https://www.u-blox.com/en/product/pointperfect)

---

### Sveaverken (스웨덴)

스웨덴 정밀농업 기업으로 **RTS(Real-Time Service)** 브랜드로 PPP-RTK 서비스를 제공한다. 전 세계 커버리지에서 ±2.5 cm 정확도를 목표로 하며, 트랙터·농기계 자동조향 솔루션과 통합 패키지로 판매된다.

> 출처: [Sveaverken 공식](https://www.sveaverken.com)

---

### Spaceopal / Galileo HAS (독일·EU)

**Galileo HAS(High Accuracy Service)**는 EU가 운영하는 무료 PPP 서비스지만, Spaceopal이 운영 인프라를 담당한다. Galileo 위성을 통해 직접 신호 수신(L-band 내장), 유럽 기준 20 cm 이내 정확도. 2023년 1월 서비스 개시.

> 출처: [ESA Galileo HAS](https://www.gsc-europa.eu/galileo/services/galileo-high-accuracy-service-has)

---

## 아시아 (Asia)

### Topcon (일본)

**TopNET Live Starpoint**는 Topcon의 PPP 서비스다. L-band와 NTRIP 복합 지원으로 수평 4~10 cm, 수렴 시간 20~30분이다. 측량·건설장비 분야에 통합되어 있으며 일본·동아시아 시장에 강세.

> 출처: [Topcon TopNET Live](https://www.topconpositioning.com/topnet-live)

---

### CHCNAV (중국)

**SWAS(SinoGNSS Wide Area Service)**는 CHCNAV의 PPP/PPP-RTK 상용 서비스다. L-band와 IP를 모두 지원하며, **수평 3 cm / 수직 6 cm** 정확도, **10초 이내** 초기화 시간이 특징이다. 전 세계 커버리지이며 사용량 기반(Usage-based) 유연한 요금제를 운영한다.

CHCNAV는 화웨이와의 협업을 통해 차량용 고정밀 측위 솔루션도 공급하고 있다.

> 출처: [CHCNAV SWAS 공식](https://www.chcnav.com/swas)

---

### Sixents Technology (중국)

**G-Earth RTK**는 중국 전역에 구축한 기준국 망 기반 Network RTK 서비스다. 수평 1~3 cm 정확도, 즉각 초기화. 드론·자율주행 농기계 분야에 강점. 중국 내 커버리지 중심.

> 출처: [Sixents Technology 공식](https://www.sixents.com)

---

### Geespace (中 吉利/Geely 계열, 중국)

**GeePPP**는 Geely 자동차 그룹 산하 Geespace가 운영하는 PPP-RTK 서비스다. 중국 자국 위성 시스템 BeiDou 기반으로 중국 내 수 cm 정확도를 목표로 하며, 자동차 OEM 납품 중심.

> 출처: [Geespace 공식](https://www.geespace.com)

---

## 서비스 요약 비교

| 권역 | 업체 | 서비스 | 기술 유형 | 수평 정확도 | 수렴 시간 | 전달 방식 |
|------|------|--------|-----------|-------------|-----------|-----------|
| 북미 | Trimble | CenterPoint RTX Fast | PPP | <2 cm (RMS) | <1분 | L-band / IP |
| 북미 | Trimble | RangePoint RTX | PPP | 15~50 cm | - | L-band / IP |
| 북미 | Swift Navigation | Skylark Cx | PPP-RTK | 3~7 cm (95%) | <20초 | IP |
| 북미 | NovAtel/Hexagon | TerraStar-C PRO | PPP | 2.5 cm (95%) | ~3분 | L-band / IP |
| 북미 | NovAtel/Hexagon | TerraStar-X | PPP-AR | 2.5 cm (95%) | <1분 | L-band |
| 북미 | Point One Nav | Polaris | PPP-RTK | 2~5 cm | - | IP |
| 유럽 | Fugro | StarFix.G4 | PPP | <10 cm (2σ) | - | L-band / IP |
| 유럽 | Fugro | StarFix.G4+ | PPP-AR | <3 cm (2σ) | - | L-band / IP |
| 유럽 | Veripos/Hexagon | Apex5 | PPP-AR | 3 cm (RMS) | - | L-band |
| 유럽 | Leica/Hexagon | HxGN SmartNet | NRTK | 1~3 cm | 즉각 | IP |
| 유럽 | u-blox | PointPerfect Flex | PPP-RTK | 6 cm (RMS) | 수십 초 | L-band / IP |
| 유럽 | Sveaverken | RTS | PPP-RTK | ±2.5 cm | - | IP |
| 아시아 | Topcon | TopNET Live Starpoint | PPP | 4~10 cm | 20~30분 | L-band / IP |
| 아시아 | CHCNAV | SWAS | PPP-RTK | 3 cm H / 6 cm V | <10초 | L-band / IP |
| 아시아 | Sixents | G-Earth RTK | NRTK | 1~3 cm | 즉각 | IP |
| 아시아 | Geespace | GeePPP | PPP-RTK | 수 cm | - | IP |

---

## 시장 트렌드 정리

PPP 서비스 시장에서 주목할 흐름 세 가지.

**① PPP-RTK로의 이동**: 순수 PPP(수렴 수십 분)에서 SSR 기반 PPP-RTK(수렴 수 초~20초)로 빠르게 옮겨가고 있다. Swift Skylark, PointPerfect Flex, CHCNAV SWAS 모두 이 방향.

**② L-band + IP 하이브리드**: 과거 해양용 L-band 전용이 주류였지만, 이제는 대부분의 서비스가 L-band와 IP를 동시 지원한다. 커버리지와 수렴 속도 양쪽을 잡기 위한 전략.

**③ 자동차 OEM 진입**: Trimble·Swift Navigation·Geespace 등이 자동차 OEM을 주요 타겟으로 명시하기 시작했다. 자율주행 레벨3 이상에서 고정밀 측위가 필수가 되면서 시장이 커지고 있다.

---

*본 글은 2026년 6월 기준 각 업체 공식 홈페이지 및 공개 자료를 바탕으로 정리했다. 가격·스펙은 변경될 수 있으며, 정확한 도입 조건은 업체에 직접 문의하는 것을 권장한다.*
