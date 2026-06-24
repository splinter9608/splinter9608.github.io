---
layout: page
title: "전국 편의점에 재고를 채워봅시다"
permalink: /fundamentals/aws-basics/cloudfront/
---

## 전국 편의점에 재고를 채워봅시다

[앞선 글]({% post_url 2026-06-26-aws-cloudfront %})에서 S3를 물류창고, CloudFront를 전국 편의점에 비유했습니다. 창고에 물건을 채우고, 편의점이 전국에 뿌리는 구조입니다.

이번엔 직접 만들어봅시다. S3 버킷에 HTML 파일을 올리고 → CloudFront로 전 세계에 배포하고 → 캐시 무효화까지 해봅니다.

---

## 준비물

- AWS 계정
- 터미널 (AWS CLI 선택 사항 — 콘솔로도 전부 가능)

> **S3** — 거의 무한 용량의 파일 저장소입니다. 자세한 내용은 S3 편에서 다루고, 여기서는 "파일을 올려두는 곳" 정도로 이해하면 됩니다.

---

## Step 1 — S3 버킷 만들고 파일 올리기

CloudFront의 오리진이 될 S3 버킷을 만듭니다. OAC를 쓸 것이므로 버킷은 퍼블릭으로 열지 않습니다.

콘솔 경로: `서비스 > S3 > 버킷 만들기`

```
버킷 이름: my-cloudfront-origin-[임의숫자]  (전 세계 유일해야 함)
AWS 리전: ap-northeast-2 (서울)
퍼블릭 액세스 차단: 모두 차단 (기본값 유지)
```

`버킷 만들기` 클릭.

버킷이 생성되면 테스트용 HTML 파일을 업로드합니다. 로컬에 `index.html` 파일을 하나 만듭니다:

```html
<!DOCTYPE html>
<html>
<body>
  <h1>CloudFront로 배달 왔습니다</h1>
</body>
</html>
```

콘솔에서 버킷 클릭 → `업로드` → 파일 선택 → `업로드`.

```
예상 결과:
버킷 > 객체 탭:
index.html | 업로드 완료
```

---

## Step 2 — CloudFront 배포 만들기 (OAC 포함)

이제 이 S3 버킷을 오리진으로 하는 CloudFront 배포를 만듭니다.

콘솔 경로: `서비스 > CloudFront > 배포 생성`

**오리진 설정:**
```
오리진 도메인: my-cloudfront-origin-[숫자].s3.ap-northeast-2.amazonaws.com
오리진 액세스: Origin access control settings (recommended) 선택
  → 제어 설정 생성 클릭
  → 이름: my-oac
  → 서명 동작: 서명 요청 (권장)
  → 생성
```

**기본 캐시 동작:**
```
뷰어 프로토콜 정책: Redirect HTTP to HTTPS
캐시 정책: CachingOptimized (기본값)
```

**설정:**
```
기본 루트 객체: index.html
```

`배포 생성` 클릭.

생성 직후 배너가 뜹니다:

```
"S3 버킷 정책을 업데이트해야 합니다"
→ [정책 복사] 버튼 클릭 → S3 버킷으로 이동 → 권한 탭 → 버킷 정책 편집 → 붙여넣기 → 저장
```

이 정책이 "CloudFront에서만 읽기 허용"을 S3에 설정하는 OAC 핵심 단계입니다.

```
예상 결과 (5~10분 소요):
배포 상태: 사용 가능
도메인 이름: xxxxxxxxxxxx.cloudfront.net
```

---

## Step 3 — CloudFront URL로 접근 확인

배포 상태가 `사용 가능`이 되면 CloudFront 도메인으로 접속합니다.

```
브라우저: https://xxxxxxxxxxxx.cloudfront.net
```

```
예상 결과:
CloudFront로 배달 왔습니다
```

S3 직접 URL로도 접근해봅시다:

```
브라우저: https://my-cloudfront-origin-[숫자].s3.ap-northeast-2.amazonaws.com/index.html
```

```
예상 결과:
AccessDenied
```

OAC가 정상 작동하는 것입니다. S3 직접 접근은 막히고, CloudFront 경유만 허용됩니다.

[이미지: 왼쪽 — CloudFront URL 접속 성공 / 오른쪽 — S3 직접 접속 AccessDenied]

---

## Step 4 — 파일 수정 후 캐시 무효화하기

파일을 바꿔도 캐시가 남아있으면 이전 버전이 보입니다. 무효화로 강제 갱신합니다.

`index.html`을 수정해서 다시 S3에 업로드합니다:

```html
<h1>업데이트된 버전입니다</h1>
```

S3에 업로드 후 바로 CloudFront URL로 접속하면 아직 이전 버전이 뜹니다. 캐시가 남아있기 때문입니다.

무효화 실행:

콘솔 경로: `CloudFront > 배포 선택 > 무효화 탭 > 무효화 생성`

```
객체 경로: /*
```

`무효화 생성` 클릭 → 상태가 `완료`로 바뀌면 브라우저에서 새로고침합니다.

```
예상 결과:
업데이트된 버전입니다
```

> **P.S.** `/*`는 전체 무효화라 편하지만, 파일 수가 많을수록 처음 1,000개 이후 비용이 붙습니다. 실제 배포에서는 파일명에 해시를 붙이는 방식(`main.a3f9c1.js`)으로 무효화 없이 새 버전을 배포하는 걸 더 많이 씁니다.

---

## 이렇게 됐으면 됩니다

- [ ] S3 버킷이 생성되고 `index.html`이 업로드됨 (퍼블릭 차단 상태)
- [ ] CloudFront 배포가 `사용 가능` 상태
- [ ] CloudFront URL로 접속 시 페이지가 정상 표시됨
- [ ] S3 직접 URL 접속 시 `AccessDenied` 반환됨
- [ ] 파일 수정 후 무효화 실행 → 새 버전 확인됨

[이미지: CloudFront 콘솔 — 배포 도메인과 OAC 설정 확인 화면]

---

## 다음

전국 배달망까지 완성됐습니다. 이제 서버에 붙이는 저장소 — EBS, EFS, S3의 차이를 알아봅시다.

→ [EBS/EFS/S3 — 어떤 창고를 어디에 붙이나](/fundamentals/aws-basics/storage/) _(작성 예정)_

---

## 부록 — 커스텀 도메인 + HTTPS 연결

`xxxxxxxxxxxx.cloudfront.net` 대신 `example.com`으로 접속하게 하려면 ACM 인증서와 Route 53을 연결합니다.

**ACM 인증서 발급 (반드시 us-east-1에서):**

콘솔 경로: `리전을 us-east-1로 변경 → 서비스 > ACM > 인증서 요청`

```
도메인 이름: example.com
검증 방법: DNS 검증
```

Route 53을 쓰고 있다면 `Route 53에서 레코드 생성` 버튼으로 자동 검증 가능합니다. 인증서 상태가 `발급됨`이 될 때까지 수 분 대기.

**CloudFront에 인증서 연결:**

콘솔 경로: `CloudFront > 배포 > 설정 편집`

```
대체 도메인 이름(CNAME): example.com
사용자 정의 SSL 인증서: ACM에서 발급한 인증서 선택
```

**Route 53 레코드 추가:**

```
A 레코드 (Alias)
  도메인: example.com
  대상: CloudFront 배포 선택
```

설정 완료 후 `https://example.com`으로 접속하면 CloudFront를 통해 HTTPS로 서빙됩니다.
