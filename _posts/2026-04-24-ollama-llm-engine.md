---
layout: post
title: "Ollama - LLM 실행에 엔진이 필요한 이유"
categories: [AI Experience]
---

## LLM은 혼자 실행이 될까?

LLM 모델 파일은 그냥 **데이터 덩어리(가중치)**이다. 이걸 메모리에 올리고 API로 서빙해주는 엔진이 필요하다.

**비유:**
- LLM = 책 (내용물)
- Ollama 같은 엔진 = 책 읽어주는 사람 (실행기)

## LLM 엔진 종류

| 엔진 | 특징 |
|---|---|
| **Ollama** | 설치 쉽고 명령어 간단, 가장 대중적 |
| **LM Studio** | GUI 있어서 클릭으로 사용 가능 |
| **vLLM** | 서버용, 고성능 |
| **llama.cpp** | 가장 가볍고 저사양에서도 동작 |

## Ollama 사용법

```bash
ollama pull llama3      # 모델 다운로드
ollama run llama3       # 모델 실행
ollama run qwen2.5      # 다른 모델 실행
```

## 클라우드 vs 로컬 엔진 차이

- **Claude/ChatGPT** → Anthropic/OpenAI가 엔진 직접 운영, 우리는 API만 사용
- **Ollama** → 내 컴퓨터가 엔진 역할, 직접 모델 실행
