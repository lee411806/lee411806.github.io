---
layout: post
title: "Claude API, OpenClaw - 사용 방식 차이"
categories: [AI Experience]
---

## 핵심 차이

**누구 서버를 쓰냐**의 차이

- **Claude Code** → Anthropic(Claude 만든 회사) 서버에 API로 요청
- **OpenClaw + Ollama** → 내 컴퓨터가 서버 역할

## 비교표

| 항목 | Claude Code | OpenClaw + Ollama |
|---|---|---|
| AI 두뇌 | Anthropic 서버 | 내 컴퓨터 |
| 방식 | API 호출 | 로컬 실행 |
| 비용 | 토큰당 비용 발생 | 무료 |
| 인터넷 | 필요 | 불필요 |
| 성능 | Claude 풀스펙 | 내 GPU 성능에 달림 |
| 프라이버시 | Anthropic 서버로 전송 | 데이터 외부 유출 없음 |

## 흐름 비교

**Claude Code:**

```
나 → API 요청 → Anthropic 서버 → 응답
```

**OpenClaw + Ollama:**

```
나 (Telegram/Slack 등) → OpenClaw → Ollama → 내 컴퓨터 LLM → 응답
```

## 참고

- Anthropic = Claude 만든 회사 (구글이 Gemini, OpenAI가 ChatGPT 만든 것과 동일)
- Ollama = 로컬에서 LLM 모델을 실행시켜주는 엔진
- OpenClaw = Ollama 위에서 각종 채널(Telegram, Slack 등)과 연결해주는 게이트웨이
