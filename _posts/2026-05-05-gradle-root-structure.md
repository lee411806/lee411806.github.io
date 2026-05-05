---
layout: post
title: "🐘 Gradle Root Project 구조와 설정 (Plugin vs Dependency)"
categories: [멘토링 1주차]
---

멀티 모듈 프로젝트에서 루트의 `build.gradle`은 전체 프로젝트의 뼈대를 잡는 역할을 합니다.

### 1. Root build.gradle 구조

- **`plugins`**: 루트에서 플러그인을 선언하고 버전을 중앙 관리합니다.
- **`allprojects`**: 루트를 포함한 모든 모듈에 공통으로 적용할 설정을 정의합니다.
- **`subprojects`**: 하위 모듈에만 적용할 설정을 정의합니다.

### 2. Plugin vs Dependency

- **Plugin**: 빌드 과정과 컴파일 동작을 확장하는 **도구**입니다.
- **Dependency**: 애플리케이션 실행 시 필요한 **라이브러리(부품)**입니다.

### 3. Kotlin 실행 과정

결국 Gradle은 **Plugin**이라는 도구를 사용하여 우리가 작성한 코드와 **Dependency**라는 외부 라이브러리를 결합해, JVM에서 실행 가능한 결과물을 만드는 과정입니다.

---

### 💭 회고

복잡한 설정들 사이에서 **"무엇이 도구(Plugin)이고 무엇이 재료(Dependency)인가"**를 명확히 구분하는 것만으로도 빌드 구조를 이해하는 데 큰 도움이 되었습니다. 딱 필요한 핵심 위주로 정리하며 Gradle의 역할을 다시 한번 확인했습니다.
