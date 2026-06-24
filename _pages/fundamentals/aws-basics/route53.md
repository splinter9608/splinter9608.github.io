---
layout: page
title: "조건을 읽는 표지판을 달아봅시다"
permalink: /fundamentals/aws-basics/route53/
---

## 조건을 읽는 표지판을 달아봅시다

[앞선 글]({% post_url 2026-06-25-aws-route53 %})에서 Route 53의 표지판은 날씨, 도로 상태, 운전자 출발지를 보고 방향을 바꾼다고 했습니다. 이번엔 그 표지판을 직접 세워봅시다.

이 페이지에서는 호스팅 영역을 만들고 → ALB에 Alias 레코드를 연결하고 → 헬스 체크와 장애조치 레코드까지 설정합니다. 도메인이 없어도 콘솔 구성까지는 할 수 있습니다. dig로 확인하는 단계는 도메인이 있을 때만 해당됩니다.

---

## 준비물

- AWS 계정
- [ELB 레퍼런스](/fundamentals/aws-basics/elb/)에서 만든 ALB (`web-alb`)
- 도메인 (선택 — 없어도 Step 1~3은 진행 가능)
- 터미널 (`dig` 명령어 사용 가능한 환경)

---

## Step 1 — 호스팅 영역 만들기

호스팅 영역은 Route 53이 특정 도메인의 DNS를 관리하는 공간입니다. 도메인당 하나씩 만듭니다.

콘솔 경로: `서비스 > Route 53 > 호스팅 영역 > 호스팅 영역 생성`

```
도메인 이름: example.com (본인 도메인 또는 테스트용 아무 이름)
유형: 퍼블릭 호스팅 영역
```

`호스팅 영역 생성` 클릭.

```
예상 결과:
example.com | 퍼블릭 | 레코드 수: 2

자동 생성된 레코드:
NS  example.com  ns-xxx.awsdns-xx.com 등 4개
SOA example.com  ns-xxx.awsdns-xx.com ...
```

NS 레코드 4개가 자동으로 생성됩니다. 도메인 등록 대행사(가비아, Namecheap 등)에서 이 4개 주소로 네임서버를 변경해야 Route 53이 해당 도메인의 DNS를 관리하게 됩니다.

> **P.S.** Route 53에서 직접 도메인을 구매하면 네임서버 설정이 자동으로 됩니다. 외부에서 구매한 도메인은 NS 레코드의 4개 주소를 대행사 설정 페이지에 직접 입력해야 합니다.

---

## Step 2 — ALB에 Alias A 레코드 연결하기

ALB는 고정 IP가 없어서 일반 A 레코드를 쓸 수 없습니다. Alias 레코드로 연결합니다.

콘솔 경로: `호스팅 영역 > example.com > 레코드 생성`

```
레코드 이름: (비워두면 루트 도메인 example.com에 적용)
레코드 유형: A
별칭: 활성화
트래픽 라우팅 대상:
  - Application/Classic Load Balancer에 대한 별칭
  - 리전: ap-northeast-2 (서울)
  - 로드 밸런서: web-alb 선택
라우팅 정책: 단순 라우팅
```

`레코드 생성` 클릭.

```
예상 결과:
A  example.com  별칭: web-alb-xxx.ap-northeast-2.elb.amazonaws.com
```

도메인이 있고 네임서버 설정을 완료했다면 dig로 확인해봅시다:

```bash
dig example.com A
```

```
;; ANSWER SECTION:
example.com.    60  IN  A  52.xx.xx.xx
example.com.    60  IN  A  13.xx.xx.xx
```

ALB의 IP 주소들이 응답으로 옵니다. ALB가 자동으로 IP를 변경해도 Alias 레코드가 알아서 따라갑니다.

---

## Step 3 — 헬스 체크 만들기

장애조치 라우팅에 앞서 헬스 체크를 먼저 만들어야 합니다. Route 53이 서버 상태를 모르면 장애조치가 작동하지 않습니다.

콘솔 경로: `Route 53 > 헬스 체크 > 헬스 체크 생성`

```
이름: web-alb-health
모니터링 대상: 엔드포인트
프로토콜: HTTP
도메인 이름: example.com (또는 ALB DNS 이름)
경로: /
포트: 80
요청 인터벌: 30초
실패 임계값: 3
```

`헬스 체크 생성` 클릭 후 1~2분 대기.

```
예상 결과:
web-alb-health | 정상
```

상태가 `정상`이 되면 Route 53이 이 엔드포인트를 주기적으로 확인하기 시작한 것입니다.

---

## Step 4 — 장애조치 레코드 설정하기

메인 서버가 죽으면 자동으로 백업 페이지로 전환하는 구조를 만들어봅시다. 백업은 S3 정적 페이지를 씁니다.

> **S3** — 거의 무한 용량의 파일 저장소입니다. 정적 웹사이트 호스팅도 지원합니다. 자세한 내용은 S3 편에서 다룹니다.

**기존 A 레코드 삭제 후 아래 두 레코드로 교체합니다.**

**Primary 레코드:**
```
레코드 이름: (루트 도메인)
레코드 유형: A
별칭: ALB 선택
라우팅 정책: 장애 조치
장애 조치 레코드 유형: 기본
헬스 체크: web-alb-health
```

**Secondary 레코드:**
```
레코드 이름: (루트 도메인)
레코드 유형: A
별칭: S3 웹사이트 엔드포인트 선택 (또는 다른 ALB)
라우팅 정책: 장애 조치
장애 조치 레코드 유형: 보조
헬스 체크: 없음 (보조는 항상 대기)
```

평소엔 Primary(ALB)로 트래픽이 가고, 헬스 체크가 3번 연속 실패하면 TTL이 지난 후 Secondary로 자동 전환됩니다.

---

## 이렇게 됐으면 됩니다

아래가 확인되면 완료입니다.

- [ ] 호스팅 영역이 생성되고 NS/SOA 레코드 2개가 자동 생성됨
- [ ] ALB를 가리키는 Alias A 레코드가 생성됨
- [ ] 헬스 체크 상태가 `정상`으로 표시됨
- [ ] (도메인 있는 경우) `dig example.com A`로 ALB IP 주소 응답 확인

[이미지: Route 53 호스팅 영역 — Alias A 레코드와 헬스 체크 연결된 장애조치 레코드 목록]

---

## 다음

도로 표지판까지 달았습니다. 이제 전국 배달망 — 전 세계 어디서 접속해도 빠르게 응답하는 구조를 만들어봅시다.

→ [CloudFront — 전 세계 엣지에 콘텐츠 뿌리기](/fundamentals/aws-basics/cloudfront/) _(작성 예정)_

---

## 부록 — 라우팅 정책 비교

| 정책 | 언제 쓰나 | 핵심 |
|------|-----------|------|
| 단순 | 서버 하나, 조건 없음 | 헬스 체크 연동 불가 |
| 가중치 | 카나리 배포, A/B 테스트 | 숫자 비율로 트래픽 분배 |
| 지연시간 | 글로벌 서비스 | 가장 빠른 리전으로 |
| 장애조치 | Primary/Secondary 구성 | 헬스 체크 필수 |
| 지리적 | 국가별 다른 콘텐츠 | 위치 기반, 속도와 무관 |

---

## 부록 — 자주 쓰는 레코드 패턴

**루트 도메인 → ALB**
```
A  example.com  Alias → web-alb
```

**www 서브도메인 → 루트 도메인**
```
CNAME  www.example.com  example.com
```

**API 서브도메인 → 별도 ALB**
```
A  api.example.com  Alias → api-alb
```

**이메일 설정 (Google Workspace 예시)**
```
MX  example.com  1 aspmx.l.google.com
MX  example.com  5 alt1.aspmx.l.google.com
```
