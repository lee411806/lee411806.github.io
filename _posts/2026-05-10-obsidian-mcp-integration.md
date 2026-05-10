---
layout: post
title: "Obsidian - MCP 연동 개념 정리"
categories: [AI Experience]
---

## Obsidian이란?

- 마크다운 기반 로컬 노트 앱
- 노트끼리 링크로 연결 → 그래프로 시각화
- 정보 저장보다 **생각 연결**에 초점 ("두 번째 뇌" 컨셉)
- 개인 사용 무료, 모든 파일이 로컬 .md 파일로 저장됨

## Notion vs Obsidian

- **Notion**: 팀 협업, 클라우드 저장에 강함
- **Obsidian**: 개인 지식 연결, 로컬 저장에 강함

## Claude Code MCP 연동

- Obsidian은 MCP 서버가 여러 종류 존재 (커뮤니티 제작)
- 처음엔 `obsidian-http-mcp` 시도 → HTTP 서버 방식이라 pm2로 백그라운드 실행 필요, 비효율적
- 더 나은 방법: `obsidian-claude-code-mcp` (Claude Code 전용 플러그인)
  - Obsidian 커뮤니티 플러그인으로 설치
  - 플러그인 자체가 MCP 서버 역할 → Claude Code가 WebSocket으로 자동 연결
  - pm2 같은 별도 프로세스 불필요

## 핵심 교훈

- MCP 선택 시 해당 도구 전용으로 만들어진 걸 쓰는 게 훨씬 효율적
- Obsidian Local REST API 플러그인은 HTTPS 27123 포트로 동작
