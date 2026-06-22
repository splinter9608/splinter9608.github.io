---
layout: page
title: "ASG와 ALB를 연결해봅시다"
permalink: /fundamentals/aws-basics/elb/
---

## ASG와 ALB를 연결해봅시다

[앞선 글]({% post_url 2026-06-24-aws-elb %})에서 ELB는 서버 목록이 바뀌어도 자동으로 따라간다고 했습니다. ASG와 연결해두면 서버가 자동으로 늘어도 ALB가 바로 트래픽을 넣어준다는 것이죠.

이번엔 그 연결을 직접 만들어봅시다. [HA 레퍼런스](/fundamentals/aws-basics/ha-scalability/)에서 ALB를 이미 만들어봤으니, 여기서는 ASG + ALB 연동에 집중합니다.

---

## 이 페이지에서 할 것

EC2 AMI로 시작 템플릿을 만들고 → ASG를 구성하고 → ALB 타겟 그룹에 연결합니다. 완료되면 인스턴스가 자동으로 ALB에 등록되고 해제되는 구조가 완성됩니다.

---

## 준비물

- [EC2 레퍼런스](/fundamentals/aws-basics/ec2/)에서 저장한 AMI (`my-webserver-ami`)
- ALB 하나 (없으면 [HA 레퍼런스](/fundamentals/aws-basics/ha-scalability/) Step 3 참조)
- ALB에 연결된 타겟 그룹 (`web-tg`)

---

## Step 1 — 시작 템플릿 만들기

ASG는 인스턴스를 새로 만들 때 "어떤 설정으로 만들지"를 시작 템플릿에서 읽어옵니다. AMI, 인스턴스 유형, 보안 그룹, User Data를 여기 정의해둡니다.

콘솔 경로: `서비스 > EC2 > 시작 템플릿 > 시작 템플릿 생성`

```
시작 템플릿 이름: web-launch-template
AMI: my-webserver-ami (내 AMI 탭에서 선택)
인스턴스 유형: t2.micro
키 페어: my-key
보안 그룹: HTTP(80) 허용하는 그룹
```

고급 세부 정보 > 사용자 데이터:
```bash
#!/bin/bash
systemctl start httpd
echo "<h1>$(hostname -f)</h1>" > /var/www/html/index.html
```

> hostname -f는 인스턴스 자신의 호스트명을 출력합니다. 새로고침할 때마다 다른 서버 이름이 뜨는 걸로 분산을 확인할 수 있습니다.

`템플릿 생성` 클릭.

```
예상 결과:
web-launch-template | 버전 1 | 기본 버전
```

---

## Step 2 — ASG 만들기

시작 템플릿을 기반으로 ASG를 만듭니다.

콘솔 경로: `서비스 > EC2 > Auto Scaling 그룹 > Auto Scaling 그룹 생성`

**1단계 — 시작 템플릿 선택**
```
Auto Scaling 그룹 이름: web-asg
시작 템플릿: web-launch-template
```

**2단계 — 네트워크 설정**
```
VPC: 기본 VPC
가용 영역: ap-northeast-2a, ap-northeast-2b 모두 선택
```

**3단계 — 로드 밸런서 연결**
```
기존 로드 밸런서에 연결
타겟 그룹: web-tg
헬스 체크 유형: ELB (기본값은 EC2 — ELB로 바꿔야 ELB 헬스 체크 결과를 반영합니다)
```

**4단계 — 그룹 크기**
```
원하는 용량: 2
최소 용량: 1
최대 용량: 4
```

**5단계 — 스케일링 정책 (선택)**
```
대상 추적 정책
  지표 유형: 평균 CPU 사용률
  목표값: 50
```

CPU가 50%를 넘으면 인스턴스를 추가하고, 여유가 생기면 줄입니다.

`Auto Scaling 그룹 생성` 클릭.

```
예상 결과 (2~3분 후):
web-asg | 원하는 용량: 2 | 최소: 1 | 최대: 4
인스턴스 2대가 자동 생성됨
```

---

## Step 3 — 타겟 그룹 등록 확인

ASG가 만든 인스턴스가 타겟 그룹에 자동 등록됐는지 확인합니다.

콘솔 경로: `서비스 > EC2 > 로드 밸런싱 > 대상 그룹 > web-tg > 대상`

```
예상 결과:
i-xxxxxxxxx | ap-northeast-2a | 포트 80 | 정상
i-yyyyyyyyy | ap-northeast-2b | 포트 80 | 정상
```

상태가 `정상`이 되면 ALB가 이 두 서버로 트래픽을 보내기 시작합니다.

ALB DNS로 접속하면:
```
첫 번째: ip-10-0-1-xxx.ap-northeast-2.compute.internal
새로고침: ip-10-0-2-yyy.ap-northeast-2.compute.internal
```

새로고침할 때마다 다른 인스턴스 이름이 뜨면 정상입니다.

---

## 이렇게 됐으면 됩니다

아래 세 가지가 확인되면 완료입니다.

- [ ] ASG 인스턴스 2대가 자동 생성되고 `실행 중` 상태
- [ ] 타겟 그룹에서 두 인스턴스 모두 `정상` 표시
- [ ] ALB DNS로 접속 시 새로고침할 때마다 다른 인스턴스 이름 표시

[이미지: 타겟 그룹 — ASG가 자동 등록한 인스턴스 2대가 정상 상태]

---

## 스케일링 확인 (선택)

실제로 인스턴스가 늘어나는지 테스트할 수 있습니다.

인스턴스 하나에 SSH로 접속 후 CPU를 인위적으로 올립니다:

```bash
# 부하 발생 (yes 명령어로 CPU 100% 점유)
yes > /dev/null &
```

5분 정도 기다리면 CloudWatch 알람이 울리고, ASG가 인스턴스를 추가합니다.

```
예상 결과:
web-asg | 활동 내역: "그룹의 원하는 용량이 2에서 3으로 증가"
```

테스트 후 부하 프로세스를 종료합니다:

```bash
kill %1
```

---

## 다음

ALB + ASG 조합으로 트래픽 변화에 자동으로 대응하는 구조가 완성됐습니다. 이제 이 서비스를 전 세계 사용자에게 빠르게 전달하는 방법을 알아봅시다.

→ [Route 53 — 도메인 연결하기](/fundamentals/aws-basics/route53/) _(작성 예정)_

---

## 부록 — ALB 경로 기반 라우팅

하나의 ALB로 URL 경로에 따라 다른 서버로 보내고 싶을 때 쓰는 설정입니다.

예: `/api/*` → API 서버 타겟 그룹, `/` → 웹 서버 타겟 그룹

콘솔 경로: `로드 밸런서 > web-alb > 리스너 > 규칙 보기/편집`

```
규칙 추가:
  조건: 경로 패턴 = /api/*
  작업: web-api-tg 로 전달

기본 규칙 (조건 없음):
  작업: web-tg 로 전달
```

규칙은 위에서 아래로 순서대로 평가됩니다. `/api/users` 요청이 들어오면 첫 번째 규칙에 걸려 `web-api-tg`로 가고, `/` 요청은 기본 규칙에 걸려 `web-tg`로 갑니다.

> 타겟 그룹마다 별도의 헬스 체크와 스케일링 정책을 가질 수 있습니다. API 서버와 웹 서버를 독립적으로 스케일 아웃할 수 있다는 뜻입니다.
