---
layout: post
title: "claude-hud - Windows 트러블슈팅"
categories: [AI Experience]
---

## 문제

claude-hud가 Windows에서 재시작 후 statusLine에 표시되지 않음

## 근본 원인

- **PowerShell이 자신의 stdin을 자식 프로세스(node)에 forwarding하지 않음**
- bash는 자식 프로세스가 stdin을 그대로 상속받지만, PowerShell은 내부에서 native 명령을 실행할 때 stdin을 새로 만들어 연결함

→ Claude Code가 보낸 JSON 데이터가 PowerShell에서 끊겨서 node까지 도달하지 못함

→ timeout을 아무리 늘려도 node는 데이터를 받을 수 없음

## 해결

`powershell -Command "..."` 래퍼 제거 → `node.exe` 직접 실행

```
"C:\Program Files\nodejs\node.exe" "C:\Users\admin\.claude\plugins\cache\claude-hud\claude-hud\0.1.0\dist\index.js"
```

Claude Code의 stdin이 node에 바로 연결되어 정상 동작.

## 추가 발견

- PowerShell의 `Get-ChildItem`이 `.claude/plugins/cache` 디렉토리를 열거하지 못함 → `cmd /c dir`로 우회

## 핵심 교훈

- Windows에서 statusLine 명령은 PowerShell 래퍼 없이 실행 런타임을 직접 호출해야 함
- 디버깅 시 정적 텍스트(`Write-Host 'TEST'`)로 statusLine 기능 자체가 동작하는지 먼저 확인

> **요약:** PowerShell을 중간에 끼워서 stdin이 끊긴 것 → node 다이렉트 실행하면 됨
