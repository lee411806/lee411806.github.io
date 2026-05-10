---
layout: post
title: "Superpowers - 설치 및 브레인스토밍 스킬 사용법"
categories: [AI Experience]
---

## 1. 마켓플레이스 추가

> **핵심**: Claude Code Light 버전은 `/plugin` UI에서 마켓플레이스 검색이 안 됨. 아래 명령어로 직접 추가해야 함 (이전 버전은 UI에서 검색 가능했음)

```
/plugin marketplace add obra/superpowers-marketplace
```

> SSH 오류 나면 HTTPS URL 방식(`obra/...`)으로 해야 함. `git@github.com` 방식은 known_hosts 문제로 실패함

## 2. 플러그인 설치

```
/plugin install superpowers
```

## 3. 리로드

```
/reload-plugins
```

## 4. 사용 방법

- 슬래시 명령어: `/superpowers:brainstorming`
- 자연어: "브레인스토밍 해줘" → Claude가 자동으로 스킬 실행

## 설치되는 주요 스킬 목록

- `brainstorming` — 기능 구현 전 요구사항 탐색
- `writing-plans` — 구현 계획 작성
- `systematic-debugging` — 버그 디버깅
- `requesting-code-review` — 코드 리뷰 요청
- `test-driven-development` — TDD 방식 구현
- 외 다수
