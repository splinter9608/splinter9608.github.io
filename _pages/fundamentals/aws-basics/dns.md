---
layout: page
title: "전화번호부를 직접 뒤져봅시다"
permalink: /fundamentals/aws-basics/dns/
---

## 전화번호부를 직접 뒤져봅시다

[앞선 글]({% post_url 2026-06-20-aws-dns-network %})에서 DNS를 전화번호부에 비유했습니다. 이름(도메인)으로 번호(IP)를 찾는 시스템. 이번엔 그 전화번호부를 직접 뒤져봅시다. 브라우저 뒤에서 일어나는 일을 명령어로 눈으로 확인합니다.

---

## 이 페이지에서 할 것

`dig` 명령어로 DNS 조회를 직접 실행하면서 도메인 → IP 변환 과정, 레코드 종류, TTL 동작을 확인합니다.

> **dig** — DNS 조회를 직접 실행해볼 수 있는 터미널 도구입니다. macOS/Linux에 기본 설치돼 있고, Windows는 nslookup을 대신 쓸 수 있습니다.
>
> **nslookup** — Windows에서 dig 대신 쓰는 DNS 조회 도구입니다. 동작은 비슷합니다.

---

## 준비물

- 터미널 (macOS/Linux) 또는 명령 프롬프트/PowerShell (Windows)
- 인터넷 연결

---

## Step 1 — 도메인 → IP 조회하기 (A 레코드)

가장 기본적인 조회입니다. google.com의 IP 주소가 뭔지 물어봅시다.

```bash
$ dig google.com A
```
```
;; ANSWER SECTION:
google.com.     287     IN      A       172.217.31.174
```

- `google.com.` — 조회한 도메인
- `287` — **TTL**. 이 응답을 캐시에 보관할 시간(초)입니다. 287초 안에 같은 요청이 오면 캐시에서 바로 줍니다.
- `A` — 레코드 타입. A 레코드는 도메인을 IPv4 주소와 연결합니다.
- `172.217.31.174` — 실제 IP 주소

**Windows (nslookup):**
```
> nslookup google.com
```
```
Name:    google.com
Address: 172.217.31.174
```

---

## Step 2 — DNS 조회가 어디서 오는지 확인하기 (NS 레코드)

google.com의 "공식 전화번호부"가 어느 서버인지 확인합니다. 앞선 글에서 TLD DNS → 네임 서버 순으로 탐색한다고 했는데, 그 네임 서버가 여기 나옵니다.

```bash
$ dig google.com NS
```
```
;; ANSWER SECTION:
google.com.     21600   IN      NS      ns1.google.com.
google.com.     21600   IN      NS      ns2.google.com.
google.com.     21600   IN      NS      ns3.google.com.
google.com.     21600   IN      NS      ns4.google.com.
```

> **NS 레코드(Name Server)** — 이 도메인의 DNS 정보를 갖고 있는 공식 서버 목록입니다. Route 53에서 도메인을 설정하면 여기에 Route 53 네임 서버 주소가 들어갑니다.

NS 레코드의 TTL이 21600(6시간)인 게 보입니다. 네임 서버 정보는 자주 바뀌지 않으니 길게 캐싱합니다.

---

## Step 3 — 별칭 주소 확인하기 (CNAME 레코드)

`www.google.com`과 `google.com`은 같은 곳으로 갑니다. 이렇게 다른 도메인 이름을 연결할 때 CNAME을 씁니다.

```bash
$ dig www.github.com CNAME
```
```
;; ANSWER SECTION:
www.github.com.   3600    IN      CNAME   github.com.
```

> **CNAME 레코드** — 도메인을 다른 도메인 이름에 별칭으로 연결합니다. IP 대신 다른 도메인을 가리킵니다.

실제로 `www.github.com`은 `github.com`의 별칭이라는 뜻입니다.

---

## Step 4 — DNS 조회 전체 경로 따라가기

브라우저가 어떤 경로로 DNS를 조회하는지 직접 추적합니다. `+trace` 옵션을 쓰면 Root → TLD → 네임 서버 순서를 단계별로 볼 수 있습니다.

```bash
$ dig google.com +trace
```
```
.                       518400  IN      NS      a.root-servers.net.   ← Root DNS
...
com.                    172800  IN      NS      a.gtld-servers.net.   ← .com TLD DNS
...
google.com.             172800  IN      NS      ns1.google.com.       ← 구글 네임 서버
...
google.com.             300     IN      A       172.217.31.174        ← 최종 IP
```

앞선 글에서 설명한 순서 그대로입니다. Root → TLD(.com) → 구글 네임 서버 → IP.

**P.S.** 출력이 길게 나와도 당황하지 마세요. 마지막 줄의 A 레코드만 봐도 됩니다.

---

## Step 5 — TTL이 줄어드는 걸 확인하기

캐시에 저장된 TTL이 실제로 줄어드는지 봅시다.

같은 명령을 30초 간격으로 두 번 실행합니다:

```bash
$ dig google.com A | grep -A1 "ANSWER SECTION"
```
```
;; ANSWER SECTION:
google.com.     287     IN      A       172.217.31.174
```

30초 후 다시:
```bash
$ dig google.com A | grep -A1 "ANSWER SECTION"
```
```
;; ANSWER SECTION:
google.com.     257     IN      A       172.217.31.174
```

287 → 257로 줄었습니다. 캐시 편에서 설명한 TTL이 실제로 카운트다운되고 있습니다. 0이 되면 다시 DNS 서버에 물어봅니다.

---

## 이렇게 됐으면 됩니다

아래 네 가지를 직접 확인했으면 완료입니다.

- [ ] `dig google.com A` 로 IP 주소 확인
- [ ] `dig google.com NS` 로 네임 서버 목록 확인
- [ ] `dig [도메인] CNAME` 으로 별칭 레코드 확인
- [ ] `dig google.com +trace` 로 Root → TLD → 네임 서버 경로 추적

---

## 다음

DNS 레코드를 눈으로 봤으니, 이제 AWS에서 직접 설정해봅시다. Route 53은 AWS의 DNS 서비스입니다 — 도메인 구입부터 A/CNAME/NS 레코드 설정까지 한 곳에서 됩니다.

→ Route 53 편: AWS에서 도메인과 DNS 레코드 설정하기 _(작성 예정)_

네트워크 개념을 더 이어가고 싶다면:

→ SSL/TLS 편: HTTPS는 어떻게 암호화되는가 _(작성 예정)_
