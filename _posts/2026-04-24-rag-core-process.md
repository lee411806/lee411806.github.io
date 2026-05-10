---
layout: post
title: "RAG - 핵심 과정부터 생태계까지"
categories: [AI Experience]
---

## RAG란?

AI가 답변하기 전에 **내 문서/DB에서 관련 내용을 찾아서** 그걸 바탕으로 답하는 방식

## 핵심 과정 3단계

**1. 저장** — 문서를 잘게 잘라서(청킹) 벡터DB에 저장

**2. 검색** — 질문이 들어오면 벡터DB에서 관련 문서 조각 찾기

**3. 생성** — 찾은 문서 + 질문을 LLM에 던져서 답변 생성

## 흐름

```
[저장]
문서 → 청킹 → 벡터 변환 → 벡터DB 저장

[질문]
질문 → 벡터 변환 → 유사한 조각 검색 → LLM → 답변
```

## LangChain이란?

RAG 구현의 각 과정을 편하게 묶어주는 **프레임워크**

**직접 구현 시** — 각각 따로 코드 짜야 함

```
문서 로딩 → 청킹 → 임베딩 → 벡터DB → 검색 → LLM 연결
```

**LangChain 사용 시** — 몇 줄로 끝남

```python
chain = RetrievalQA.from_chain_type(llm=llm, retriever=vectorstore.as_retriever())
chain.run("질문")
```

## RAG 생태계

| 역할 | 툴 |
|---|---|
| **프레임워크** | LangChain, LlamaIndex |
| **벡터DB** | Pinecone, ChromaDB, Weaviate |
| **임베딩 모델** | OpenAI, HuggingFace |
| **LLM** | Claude, GPT, Ollama(로컬) |

LangChain이 가장 유명하고, LlamaIndex는 문서 검색에 더 특화

## 가장 힘든 부분

**청킹 전략** — 문서를 어떻게 잘게 자르냐가 검색 품질을 좌우함
