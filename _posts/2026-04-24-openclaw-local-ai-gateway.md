---
layout: post
title: "OpenClaw - 로컬 AI 비서 게이트웨이"
categories: [AI Experience]
---

## 핵심 5줄 요약

1. 내 서버에서 24시간 돌아가는 **오픈소스 로컬 AI 비서** 게이트웨이다
2. Ollama, LM Studio 등 로컬 LLM과 연결해서 외부 API 비용 없이 운영 가능하다
3. WhatsApp, Telegram, Slack, Discord, iMessage 등 **모든 채널에 연동**된다
4. 데이터가 외부로 나가지 않아 **완전한 프라이버시**가 보장된다
5. YAML/환경변수로 설정하며 데스크탑 앱, CLI, 웹 UI 세 가지 인터페이스를 지원한다

## YAML 설정이란?

OpenClaw한테 "어떤 LLM 써, 어떤 채널 연결해" 알려주는 설정 파일

```yaml
llm:
  provider: ollama
  model: qwen2.5

channels:
  - telegram
  - slack
```

한번 설정해두면 끝 — 이후엔 자동으로 24시간 돌아감

## 3가지 인터페이스

같은 AI 비서에 접근하는 방법이 3가지인 것 (똑같은 Ollama + 내 서버에 붙는 거)

| 인터페이스 | 어떻게 쓰냐 | 언제 쓰냐 |
|---|---|---|
| **데스크탑 앱** | 설치한 앱 열어서 대화 | 컴퓨터 앞에 있을 때 |
| **CLI** | 터미널에서 명령어로 대화 | 개발자 스타일 |
| **웹 UI** | 브라우저에서 대화 | 다른 기기에서 접속할 때 |

넷플릭스를 TV앱/폰/PC 브라우저로 보든 같은 계정·콘텐츠인 것처럼, 접근 방법만 다를 뿐 동일한 AI 비서!
