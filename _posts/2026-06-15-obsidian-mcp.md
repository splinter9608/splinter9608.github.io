---
layout: single
title: "Claude Desktop에 Obsidian MCP 연결하기 — 삽질 기록"
categories: [dev-notes, tools]
tags: [Claude, MCP, Obsidian, 개발환경, 트러블슈팅]
---

---

# Claude Desktop에 Obsidian MCP 연결하기 — 삽질 기록

> Obsidian을 Claude Desktop과 MCP로 연결하면, Claude가 직접 vault에 노트를 생성하고 관리할 수 있다. 설정 자체는 간단해 보이지만, 패키지 선택 실수와 config 문법 오류 등 실제로 여러 문제가 발생했다. 이 글은 어떻게 설정하는지보다, 어디서 막혔고 어떻게 해결했는지에 더 집중한다.

---

## 1. 구성 방법

### Vault 생성

Obsidian에서 Claude 대화 아카이브용 vault를 별도로 생성한다. 기존 vault와 섞이면 관리가 복잡해지므로 분리하는 것이 낫다.

- Vault 경로: `D:\claude-archive\Claude-Archive`
- 폴더 구조: 프로젝트별로 폴더를 나누고 `Template` 폴더 추가

### MCP 패키지 선택

Obsidian MCP 패키지는 여러 종류가 있다. 최종적으로 아래 패키지를 사용한다.

```json
"mcpServers": {
  "obsidian": {
    "command": "npx",
    "args": [
      "-y",
      "@bitbonsai/mcpvault",
      "D:\\claude-archive\\Claude-Archive"
    ]
  }
}
```

### config 파일 위치

Claude Desktop을 **Microsoft Store**로 설치한 경우 config 경로가 다르다.

| 설치 방식 | config 경로 |
|-----------|------------|
| 일반 설치 | `%APPDATA%\Claude\claude_desktop_config.json` |
| Microsoft Store | `%LOCALAPPDATA%\Packages\Claude_pzs8sxrjxfjjc\LocalCache\Roaming\Claude\claude_desktop_config.json` |

설정 저장 후 Claude Desktop을 완전히 종료하고 재시작한다.

---

## 2. 발생했던 문제들

### 문제 1. `mcp-obsidian`은 읽기만 된다

처음에 `mcp-obsidian` 패키지를 설치했는데, Claude가 노트를 읽을 수는 있었지만 **생성/쓰기가 전혀 안 됐다**.

원인은 패키지 자체의 한계였다. `mcp-obsidian`은 `read_notes`, `search_notes` 두 가지 tool만 지원하고, 파일 쓰기 기능이 없었다. 공식 문서를 꼼꼼히 읽지 않고 이름만 보고 설치한 것이 실수였다.

### 문제 2. `create-note`가 작동하지 않고 tool 목록 자체가 불러와지지 않음

패키지를 교체한 이후에도 `create-note` tool 호출이 실패했고, 심지어 사용 가능한 tool 목록 자체가 Claude 대화에 로드되지 않는 현상이 발생했다.

```
Failed to call tool "create-note"
```

원인은 두 가지였다. 첫째, 패키지마다 지원하는 tool 이름이 다르다. `create-note`라는 tool이 없는 패키지에 해당 이름으로 호출하면 당연히 실패한다. 둘째, MCP 서버가 running 상태여도 tool 목록은 **새 대화를 열어야** 로드된다. 설정한 바로 그 대화에서는 tool이 보이지 않는다.

아래는 주요 패키지별 지원 tool 비교다.

| 패키지 | 쓰기 | 폴더 생성 | 기타 |
|--------|------|-----------|------|
| `mcp-obsidian` | ❌ | ❌ | 읽기 전용 |
| `obsidian-mcp` | ✅ | ❌ | `create-directory` 미지원 |
| `@bitbonsai/mcpvault` | ✅ | ✅ (자동) | `move_file` 등 지원 |

### 문제 3. `create-directory` tool 호출 실패

폴더를 먼저 만들려고 하면 아래 오류가 계속 발생했다.

```
Failed to call tool "create-directory"
```

`obsidian-mcp`(StevenStavrakis)는 `create-directory` tool 자체를 지원하지 않는다. Obsidian MCP 패키지 대부분이 폴더 생성을 별도 tool로 제공하지 않는다.

### 문제 4. filesystem MCP 서버 이름 대소문자 문제

Claude Desktop config에 `Filesystem`(대문자)과 `filesystem`(소문자) 두 개의 서버가 등록되면서 충돌이 발생했다. 대소문자를 통일하려고 소문자를 대문자로 바꿨더니 오히려 tool이 아예 잡히지 않았다.

결론적으로 **소문자 `filesystem`이 정상 작동하는 이름**이었다. MCP 서버 이름은 임의로 바꾸지 말고, 처음 등록 시 소문자로 통일해두는 것이 안전하다.

### 문제 5. config 수정 중 JSON 문법 오류

config 파일을 직접 편집하다가 쉼표 누락으로 JSON이 깨졌고, 저장하자 `mcpServers` 블록 전체가 날아갔다.

```json
// 잘못된 예 — 쉼표 누락
{
  "mcpServers": {
    "obsidian": { ... }   // ← 여기서 쉼표 빠지면 아래 항목 파싱 실패
    "filesystem": { ... }
  }
}
```

---

## 3. 해결 방법

### 해결 1, 2, 3. 패키지를 `@bitbonsai/mcpvault`로 교체

읽기 전용 문제, `create-note` 미작동, `create-directory` 실패를 동시에 해결하려면 패키지 자체를 교체해야 한다. `@bitbonsai/mcpvault`는 파일 쓰기와 폴더 처리가 모두 지원되며, 폴더를 먼저 만들 필요도 없다.extprotocol/server-filesystem",
    "D:\\claude-archive\\Claude-Archive"
  ]
}
```

### 해결 5. config 편집 시 JSON 검증

config 파일은 메모장 대신 **VSCode**로 열어서 편집하면 문법 오류를 실시간으로 잡아준다. 수정 후에는 아래 사이트에서 붙여넣어 검증하는 습관을 들이자.

```
https://jsonlint.com
```

---

## 최종 작동 구성 요약

| 항목 | 값 |
|------|-----|
| 패키지 | `@bitbonsai/mcpvault` |
| vault 경로 | `D:\claude-archive\Claude-Archive` |
| API 키 | 불필요 (직접 파일시스템 접근) |
| 폴더 생성 | `write_note` path에 경로 포함 시 자동 생성 |
| MCP 서버 이름 | 소문자로 통일 |
| tool 확인 | 새 대화 열고 입력창 `+` 버튼 |

처음부터 `@bitbonsai/mcpvault`를 쓰고, MCP 서버 이름은 소문자로 통일하면 대부분의 문제를 피할 수 있다.
