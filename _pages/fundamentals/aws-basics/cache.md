---
layout: page
title: "냉장고 앞에 어떤 음료가 채워지는지, 직접 들여다봅시다"
permalink: /fundamentals/aws-basics/cache/
---

## 냉장고 앞에 어떤 음료가 채워지는지, 직접 들여다봅시다

[앞선 글]({% post_url 2026-06-19-aws-cache %})에서 캐시를 편의점 냉장고에 비유했습니다. 자주 달라는 건 냉장고 앞에, 나머지는 창고에. 이번엔 Redis로 직접 데이터를 넣고 꺼내면서 Hit, Miss, Eviction, Invalidation이 실제로 어떻게 생기는지 확인해봅시다.

> **Redis** — 데이터를 RAM(메모리)에 저장하는 인메모리 저장소입니다. DB보다 훨씬 빠르지만 서버가 꺼지면 데이터가 사라집니다. AWS에서는 ElastiCache가 이 Redis를 관리형으로 제공합니다. 둘 다 나중에 자세히 다룰 예정이고, 여기서는 캐시 동작 원리를 손에 익히는 용도로 씁니다.

---

## 이 페이지에서 할 것

Redis를 로컬에 설치하고, 네 가지 캐시 동작 — Hit, Miss, Eviction, Invalidation — 을 직접 명령어로 확인합니다.

---

## 준비물

- macOS 또는 Linux 환경
  - Windows라면 WSL을 씁니다. WSL은 Windows 안에서 Linux를 실행할 수 있게 해주는 도구입니다. [Microsoft 공식 설치 가이드](https://learn.microsoft.com/ko-kr/windows/wsl/install)
- Homebrew (macOS 기준) — macOS용 패키지 관리자입니다. 없으면 [brew.sh](https://brew.sh)에서 설치하면 됩니다.

---

## Step 1 — Redis 설치하고 실행하기

**macOS:**
```bash
$ brew install redis
$ brew services start redis
```
```
==> Successfully started `redis` (label: homebrew.mxcl.redis)
```

**Linux (Ubuntu/Debian):**
```bash
$ sudo apt update && sudo apt install redis-server -y
$ sudo systemctl start redis
```

설치 후 접속 확인:
```bash
$ redis-cli ping
```
```
PONG
```

`redis-cli`는 Redis에 명령어를 보내는 터미널 도구입니다. `PONG`이 나오면 Redis가 정상 실행 중입니다.

---

## Step 2 — 데이터 저장하고 Cache Hit 확인하기

Redis에 데이터를 넣고 꺼내봅니다. 냉장고에 콜라를 채워두고 바로 꺼내는 과정입니다.

```bash
$ redis-cli
```
```
127.0.0.1:6379>
```

데이터 저장:
```
127.0.0.1:6379> SET drink "cola"
```
```
OK
```

저장한 데이터 꺼내기:
```
127.0.0.1:6379> GET drink
```
```
"cola"
```

냉장고에 콜라가 있으니 바로 나왔습니다. **이게 Cache Hit입니다.** DB까지 갔다올 필요가 없었습니다.

---

## Step 3 — 없는 데이터 요청으로 Cache Miss 만들기

냉장고에 없는 음료를 달라고 해봅시다.

```
127.0.0.1:6379> GET juice
```
```
(nil)
```

`(nil)`은 "없다"는 뜻입니다. **이게 Cache Miss입니다.** 실제 서비스라면 이 시점에 DB에서 데이터를 가져와 캐시에 저장하는 로직이 실행됩니다.

직접 채워봅시다 (Lazy Loading 흉내):
```
127.0.0.1:6379> SET juice "orange"
127.0.0.1:6379> GET juice
```
```
"orange"
```

다음 요청부터는 Cache Hit이 납니다.

---

## Step 4 — TTL로 Eviction 동작 확인하기

TTL(Time To Live)은 데이터 유효 기간입니다. 설정한 시간이 지나면 Redis가 해당 데이터를 자동으로 제거합니다.

10초짜리 TTL로 데이터 저장:
```
127.0.0.1:6379> SET temp_drink "lemonade" EX 10
```
```
OK
```

남은 시간 확인:
```
127.0.0.1:6379> TTL temp_drink
```
```
(integer) 8
```

10초 후 다시 꺼내보면:
```
127.0.0.1:6379> GET temp_drink
```
```
(nil)
```

유통기한이 지나면 냉장고에서 사라집니다. **이게 Cache Eviction입니다.**

---

## Step 5 — DEL로 Cache Invalidation 해보기

DB에서 데이터가 바뀌면 캐시의 예전 값을 수동으로 지워줘야 합니다.

현재 캐시 확인:
```
127.0.0.1:6379> GET drink
```
```
"cola"
```

DB에서 "콜라"가 "제로콜라"로 바뀌었다고 가정하고, 캐시 무효화:
```
127.0.0.1:6379> DEL drink
```
```
(integer) 1
```

```
127.0.0.1:6379> GET drink
```
```
(nil)
```

캐시가 비워졌습니다. 다음 요청에서 Cache Miss가 나면 DB에서 최신 값("제로콜라")을 가져와 다시 캐시에 올립니다. **이게 Cache Invalidation입니다.**

---

## 이렇게 됐으면 됩니다

아래 네 가지를 직접 확인했으면 완료입니다.

- [ ] `SET` / `GET` 으로 Cache Hit 확인
- [ ] 없는 키 요청 시 `(nil)` 응답 — Cache Miss 확인
- [ ] `EX` 옵션으로 TTL 설정, 만료 후 자동 제거 확인
- [ ] `DEL` 로 수동 무효화 후 `(nil)` 확인

**P.S.** 명령어가 다 비슷해 보여도 괜찮습니다. 중요한 건 "캐시에 있으면 빠르고, 없으면 DB까지 간다"는 흐름이 손에 잡히는 것입니다.

---

## 다음

Redis 동작 원리를 직접 봤으니, AWS에서 이 Redis를 어떻게 쓰는지로 넘어갑니다. ElastiCache는 Redis를 완전 관리형으로 제공하는 서비스입니다 — 설치, 패치, 클러스터 관리를 AWS가 대신 해줍니다.

→ ElastiCache 편: Redis를 AWS에서 쓰는 방법 _(작성 예정)_

개념을 더 이어가고 싶다면:

→ DNS 편: 도메인 이름이 IP로 바뀌는 과정 _(작성 예정)_
