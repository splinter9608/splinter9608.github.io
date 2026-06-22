---
layout: page
title: "봉인된 편지가 진짜인지 직접 확인해봅시다"
permalink: /fundamentals/aws-basics/ssl-tls/
---

## 봉인된 편지가 진짜인지 직접 확인해봅시다

[앞선 글]({% post_url 2026-06-21-aws-ssl-tls %})에서 HTTPS를 봉인된 편지에 비유했습니다. 봉인(암호화)이 되어 있고, CA가 서명한 신분증(인증서)이 붙어 있는 편지. 이번엔 그 인증서를 직접 뜯어봅시다. 어떤 정보가 담겨 있는지, 누가 서명했는지 눈으로 확인합니다.

---

## 이 페이지에서 할 것

`openssl`과 `curl` 명령어로 실제 SSL/TLS 인증서를 조회하고, 인증서 안에 어떤 정보가 담겨 있는지 확인합니다.

> **openssl** — SSL/TLS 관련 작업을 할 수 있는 터미널 도구입니다. macOS/Linux에 기본 설치돼 있습니다.
>
> **curl** — 터미널에서 HTTP/HTTPS 요청을 보내는 도구입니다. macOS/Linux 기본 설치, Windows는 PowerShell에서 사용 가능합니다.

---

## 준비물

- 터미널 (macOS/Linux) 또는 PowerShell (Windows)
- 인터넷 연결

---

## Step 1 — 인증서 기본 정보 확인하기

google.com에 연결해서 SSL 인증서 정보를 가져옵니다.

```bash
$ echo | openssl s_client -connect google.com:443 2>/dev/null | openssl x509 -noout -subject -issuer -dates
```
```
subject=CN=*.google.com
issuer=C=US, O=Google Trust Services, CN=WR2
notBefore=Apr  7 08:19:01 2026 GMT
notAfter=Jun 30 08:19:00 2026 GMT
```

- `subject` — 이 인증서가 발급된 도메인. `*.google.com`은 google.com의 모든 서브도메인(mail.google.com, drive.google.com 등)에 유효하다는 뜻입니다.
- `issuer` — 이 인증서를 서명한 CA(인증 기관). Google Trust Services가 서명했습니다.
- `notBefore` / `notAfter` — 인증서 유효 기간. 이 기간이 지나면 브라우저가 "인증서 만료" 경고를 띄웁니다.

---

## Step 2 — 인증서 전체 내용 보기

인증서 안에 어떤 정보가 더 있는지 전체를 출력해봅니다.

```bash
$ echo | openssl s_client -connect google.com:443 2>/dev/null | openssl x509 -noout -text | head -40
```
```
Certificate:
    Data:
        Version: 3 (0x2)
        Serial Number: ...
        Signature Algorithm: ecdsa-with-SHA256
    Validity
        Not Before: Apr  7 08:19:01 2026 GMT
        Not After : Jun 30 08:19:00 2026 GMT
    Subject: CN=*.google.com
    Subject Public Key Info:
        Public Key Algorithm: id-ecPublicKey
        ...
    X509v3 Subject Alternative Name:
        DNS:*.google.com, DNS:google.com
```

- `Signature Algorithm` — 인증서 서명에 사용된 알고리즘입니다. 앞 글에서 다룬 비대칭 암호화가 여기 쓰입니다.
- `Subject Alternative Name` — 이 인증서가 유효한 도메인 목록입니다. `*.google.com`과 `google.com` 둘 다 커버합니다.

---

## Step 3 — HTTPS 연결이 어떤 TLS 버전을 쓰는지 확인하기

```bash
$ curl -v --silent https://google.com 2>&1 | grep -E "TLS|SSL|cipher"
```
```
* SSL connection using TLSv1.3 / TLS_AES_256_GCM_SHA384
```

- `TLSv1.3` — 현재 사용 중인 TLS 버전입니다. TLS 1.3이 가장 최신이고 안전합니다. TLS 1.0, 1.1은 오래되어 현재 대부분의 서버에서 비활성화돼 있습니다.
- `TLS_AES_256_GCM_SHA384` — 실제 데이터 암호화에 사용되는 알고리즘(cipher suite)입니다. 앞 글에서 말한 "핸드셰이크 후 대칭 암호화로 전환"이 이 알고리즘으로 이뤄집니다.

---

## Step 4 — 만료된 인증서를 가진 사이트 접속해보기

인증서가 만료되면 어떻게 되는지 직접 확인해봅시다. expired.badssl.com은 테스트용으로 만료된 인증서를 제공하는 사이트입니다.

```bash
$ curl https://expired.badssl.com/
```
```
curl: (60) SSL certificate problem: certificate has expired
More details here: https://curl.se/docs/sslcerts.html
```

브라우저에서도 `expired.badssl.com`에 접속해보면 "연결이 비공개로 설정되어 있지 않습니다" 경고가 뜹니다. 인증서 만료가 실제로 어떻게 보이는지 확인할 수 있습니다.

**P.S.** badssl.com은 SSL 관련 다양한 오류 상태를 테스트할 수 있는 공개 사이트입니다. wrong.host, self-signed, revoked 등도 있으니 구경해보세요.

---

## Step 5 — 내 도메인 인증서 확인하기 (선택)

본인이 운영하는 도메인이 있다면 인증서 상태를 확인해봅니다.

```bash
$ echo | openssl s_client -connect [내도메인]:443 2>/dev/null | openssl x509 -noout -dates
```
```
notBefore=Jan  1 00:00:00 2026 GMT
notAfter=Mar 31 23:59:59 2026 GMT
```

만료일이 얼마 남지 않았다면 갱신이 필요합니다. AWS ACM을 쓰고 있다면 자동 갱신되므로 별도로 신경 쓸 필요가 없습니다.

---

## 이렇게 됐으면 됩니다

아래 세 가지를 확인했으면 완료입니다.

- [ ] `openssl s_client`로 google.com 인증서 발급자(issuer)와 유효 기간 확인
- [ ] `curl -v`로 TLS 버전과 암호화 알고리즘 확인
- [ ] `expired.badssl.com`에서 만료된 인증서 오류 직접 확인

---

## 다음

인증서가 뭔지 눈으로 봤으니, AWS에서 어떻게 발급하고 연결하는지로 넘어갑니다. ACM은 인증서 발급부터 자동 갱신까지 처리해줍니다.

> **ACM(AWS Certificate Manager)** — AWS에서 SSL/TLS 인증서를 발급·관리·갱신해주는 서비스입니다. ALB, CloudFront 등 AWS 서비스에 바로 연결할 수 있습니다. 자세한 내용은 나중에 다룹니다.

→ ACM + ALB 편: HTTPS를 ALB에 연결하는 방법 _(작성 예정)_

→ Route 53 편: 도메인과 DNS 레코드를 AWS에서 설정하기 _(작성 예정)_
