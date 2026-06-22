---
layout: page
title: "설계도 한 장으로 서버를 찍어냅시다"
permalink: /fundamentals/aws-basics/ec2/
---

## 설계도 한 장으로 서버를 찍어냅시다

[앞선 글]({% post_url 2026-06-23-aws-ec2 %})에서 AMI를 건물 설계도에 비유했습니다. 이번엔 그 설계도로 실제 건물을 하나 지어보고, 그 건물을 내 설계도로 저장해봅시다.

이 페이지에서는 EC2 인스턴스 시작 → SSH 접속 → 내 AMI 저장까지 한 사이클을 직접 해봅니다. ASG, ELB 연동은 [ELB 레퍼런스](/fundamentals/aws-basics/elb/)에서 이어집니다.

---

## 준비물

- AWS 계정 (프리 티어 가능)
- 터미널 (Mac/Linux 기본 포함, Windows는 PowerShell 또는 WSL)
  - **WSL** — Windows에서 Linux 환경을 쓸 수 있게 해주는 도구입니다. [공식 설치 가이드](https://learn.microsoft.com/ko-kr/windows/wsl/install)

---

## Step 1 — EC2 인스턴스 시작하기

AWS 콘솔에서 인스턴스를 직접 만들어봅시다.

콘솔 경로: `서비스 > EC2 > 인스턴스 > 인스턴스 시작`

설정값:

```
이름: my-first-server
AMI: Amazon Linux 2023 (프리 티어 사용 가능)
인스턴스 유형: t2.micro
키 페어: 새 키 페어 생성 → 이름: my-key → .pem 파일 다운로드
보안 그룹: 새 보안 그룹 생성
  - SSH (22번 포트): 내 IP에서만 허용
  - HTTP (80번 포트): 어디서든 허용 (0.0.0.0/0)
```

고급 세부 정보 > 사용자 데이터에 아래 스크립트를 붙여넣으면 인스턴스가 뜨자마자 웹서버가 자동 실행됩니다:

```bash
#!/bin/bash
yum update -y
yum install -y httpd
systemctl start httpd
systemctl enable httpd
echo "<h1>내 첫 번째 EC2 서버</h1>" > /var/www/html/index.html
```

`인스턴스 시작` 버튼 클릭 후 1~2분 대기합니다.

```
예상 결과:
my-first-server | 실행 중 | 퍼블릭 IP: xxx.xxx.xxx.xxx
```

브라우저에서 퍼블릭 IP로 접속하면 "내 첫 번째 EC2 서버"가 뜨면 됩니다.

[이미지: EC2 콘솔 — 인스턴스 `실행 중` 상태와 퍼블릭 IP 확인 화면]

---

## Step 2 — SSH로 인스턴스 접속하기

서버 안에 직접 들어가봅시다. 다운받은 `.pem` 키 파일이 필요합니다.

**키 파일 권한 설정** (처음 한 번만):

```bash
chmod 400 ~/Downloads/my-key.pem
```

**SSH 접속**:

```bash
ssh -i ~/Downloads/my-key.pem ec2-user@<퍼블릭-IP>
```

```
예상 결과:
   ,     #_
   ~\_  ####_        Amazon Linux 2023
  ~~  \_#####\
  ~~     \###|
  ~~       \#/ ___   https://aws.amazon.com/linux/amazon-linux-2023
   ~~       V~' '->
    ~~~         /
      ~~._.   _/
         _/ _/
       _/m/'
[ec2-user@ip-xxx-xxx-xxx-xxx ~]$
```

프롬프트가 바뀌면 서버 안에 들어온 겁니다. `exit`으로 나옵니다.

> **P.S.** `Permission denied (publickey)` 오류가 나면 키 파일 경로나 사용자 이름을 확인해봅시다. Amazon Linux는 `ec2-user`, Ubuntu는 `ubuntu`입니다.

---

## Step 3 — 내 AMI로 저장하기

이 상태의 인스턴스를 설계도로 찍어두면, 나중에 같은 환경을 바로 복사할 수 있습니다.

콘솔에서: `인스턴스 선택 → 작업 → 이미지 및 템플릿 → 이미지 생성`

```
이미지 이름: my-webserver-ami
이미지 설명: httpd 설치된 Amazon Linux 2023
재부팅 없음: 체크 해제 (기본값 그대로)
```

`이미지 생성` 클릭 후 `AMI` 메뉴에서 확인합니다.

```
콘솔 경로: 서비스 > EC2 > AMI

예상 결과:
my-webserver-ami | 사용 가능 | ami-xxxxxxxxxxxxxxxxx
```

상태가 `사용 가능`이 되면 이 AMI로 언제든 같은 서버를 새로 뽑을 수 있습니다.

[이미지: AMI 목록 — `사용 가능` 상태의 내 AMI]

---

## 이렇게 됐으면 됩니다

아래 세 가지가 확인되면 완료입니다.

- [ ] EC2 인스턴스가 `실행 중` 상태이고, 퍼블릭 IP로 브라우저 접속 시 페이지가 뜬다
- [ ] SSH로 인스턴스에 접속했다 (프롬프트 변경 확인)
- [ ] AMI가 `사용 가능` 상태로 저장됐다

---

## 다음

이제 건물 하나를 짓고 설계도도 저장했습니다. 다음은 그 앞에 안내데스크(ELB)를 세우고, 트래픽에 따라 건물을 자동으로 늘리는(ASG) 구조를 만들어봅시다.

→ [ELB — 수백 대 서버 앞에서 교통정리하기](/fundamentals/aws-basics/elb/) _(작성 예정)_

---

## 부록 — 자주 참조하는 설정

### 인스턴스 유형 선택 기준

병목이 어디서 오느냐에 따라 패밀리를 고릅니다.

| 병목 | 패밀리 | 대표 유형 | 쓰임새 |
|------|--------|-----------|--------|
| 균형잡힌 범용 | t | t3.micro, t3.small | 웹서버, 개발환경, 소규모 앱 |
| CPU | c | c5.large, c6i.xlarge | 배치처리, ML 추론, 고성능 웹서버 |
| 메모리(RAM) | r | r5.large, r6g.xlarge | 인메모리 DB, 실시간 분석 |
| 디스크 I/O | i | i3.large | 고성능 DB, 데이터 웨어하우스 |

처음이라면 t3.micro나 t3.small로 시작하고, 병목이 생기면 그때 바꿔도 됩니다.

---

### 구매 옵션 한눈 비교

| 옵션 | 할인율 | 언제 쓰나 | 주의 |
|------|--------|-----------|------|
| On-Demand | — | 개발, 예측 불가 트래픽 | 가장 비쌈 |
| Savings Plans | 최대 66% | 안정화된 서비스, 유형 변경 가능성 있음 | 금액 약정 |
| Reserved | 최대 72% | 유형/리전 고정, 장기 운영 서버 | 유형 묶임 |
| Spot | 최대 90% | 배치처리, 중단 허용 작업 | 2분 경고 후 종료 |
| Dedicated Host | — | 컴플라이언스 요구사항 | 가장 비쌈 |

---

### 자주 쓰는 보안 그룹 설정 예시

**웹서버 (기본)**

| 유형 | 프로토콜 | 포트 | 소스 |
|------|----------|------|------|
| SSH | TCP | 22 | 내 IP만 (`내 IP` 선택) |
| HTTP | TCP | 80 | 0.0.0.0/0 |
| HTTPS | TCP | 443 | 0.0.0.0/0 |

**ALB 뒤에 있는 서버 (ALB에서만 트래픽 받기)**

| 유형 | 프로토콜 | 포트 | 소스 |
|------|----------|------|------|
| SSH | TCP | 22 | 내 IP만 |
| HTTP | TCP | 80 | ALB 보안 그룹 ID |

> SSH 22번 포트를 `0.0.0.0/0`(전체 허용)으로 열어두면 안 됩니다. 반드시 내 IP만 허용합니다.
