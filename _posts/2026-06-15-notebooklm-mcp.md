---
layout: single
title: "Claude Desktop에 NotebookLM MCP 연결하기"
categories: [dev-notes, tools]
tags: [Claude, MCP, NotebookLM, 개발환경]
---

---

# Claude Desktop에 NotebookLM MCP 연결하기

> NotebookLM은 Google이 만든 AI 기반 노트북 서비스로, 업로드한 문서를 기반으로 정확한 답변을 제공한다. 이를 Claude Desktop과 MCP(Model Context Protocol)로 연결하면, Claude가 NotebookLM의 자료를 직접 참조해 답변할 수 있게 된다.
>
> 이 글에서는 NotebookLM MCP를 Claude Desktop에 연결하고, 노트북을 등록해 실제로 질의하는 전 과정을 정리한다.

---

## 1. MCP 서버 등록

Claude Code(터미널)에서 아래 명령어로 MCP 서버를 등록한다.

```powershell
claude mcp add notebooklm npx notebooklm-mcp@latest
```

이 명령어는 로컬 프로젝트 config에 등록되므로, **Claude Desktop에서 사용하려면 별도로 `claude_desktop_config.json`에도 추가**해야 한다.

---

## 2. Claude Desktop config에 추가

`claude_desktop_config.json` 파일을 열어 `mcpServers` 항목에 아래 내용을 추가한다.

```
C:\Users\{사용자명}\AppData\Roaming\Claude\claude_desktop_config.json
```

```json
"notebooklm": {
  "command": "npx",
  "args": [
    "notebooklm-mcp@latest"
  ]
}
```

저장 후 **Claude Desktop을 재시작**한다.

---

## 3. Google 계정 인증

처음 연결 시 Google 계정 인증이 필요하다. PowerShell에서 아래 명령어를 실행한다.

```powershell
npx notebooklm-mcp@latest auth --show-browser
```

> `--show-browser` 플래그가 없으면 헤드리스 모드로 실행되어 브라우저가 열리지 않는다. 반드시 붙여줄 것.

브라우저가 열리면 Google 계정으로 로그인한다. 완료되면 세션이 로컬에 저장되어 이후 재인증 없이 사용 가능하다.

> 세션이 만료된 경우에는 동일한 명령어를 다시 실행하면 된다.

---

## 4. 노트북 등록

NotebookLM의 노트북은 Claude 대화에서 URL을 전달해 등록한다. PowerShell CLI로는 등록이 불가하다.

> **등록 방법**
>
> 1. NotebookLM(notebooklm.google.com)에서 등록할 노트북을 연다.
> 2. 주소창의 URL을 복사한다.
> 3. Claude Desktop 대화에서 아래와 같이 전달한다.

```
NotebookLM 노트북 등록해줘
URL: https://notebooklm.google.com/notebook/{uuid}
이름: 노트북 이름
내용: 어떤 자료가 들어있는지 간단히 설명
```

등록된 노트북 정보는 아래 경로에 저장되어 **새 대화에서도 유지**된다.

```
C:\Users\{사용자명}\AppData\Local\notebooklm-mcp\Data\library.json
```

---

## 5. 실제 질의

등록된 노트북에 Claude가 직접 질의할 수 있다.

```
CRA Annex I 내용 정리해줘
```

NotebookLM이 업로드된 원문을 기반으로 Gemini 2.5가 답변을 생성하고, Claude가 이를 정리해 전달한다.

---

## 활용 구조 정리

| 도구 | 역할 |
|------|------|
| NotebookLM | 법안 원문 등 정확도가 중요한 레퍼런스 자료 보관 및 질의 |
| Obsidian | 작업 맥락, 기억, 메모 등 개인 두 번째 뇌 |
| Claude | 두 곳을 오가며 실행하는 허브 |

세 도구를 역할에 따라 분리해두면, Claude가 정확한 자료는 NotebookLM에서, 개인 맥락은 Obsidian에서 참조하는 워크플로우가 완성된다.
