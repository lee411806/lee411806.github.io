---
layout: post
title: "Kotlin 핵심 특징 정리"
categories: [Kotlin]
---

> Java가 할 수 있는 건 다 되면서, 코드는 더 짧고 깔끔하게

---

### 1. Null Safety

- 컴파일 타임에 null 체크 강제
- `NullPointerException` 위험 감소

👉 **안정성 확보**

---

### 2. 간결한 문법 (Concise Syntax)

매번 반복적으로 써야 하는 틀에 박힌 코드(boilerplate)를 줄여줍니다.

**Java** (데이터 클래스 하나 만들려면):

```java
public class User {
    private String name;
    private int age;

    public User(String name, int age) {
        this.name = name;
        this.age = age;
    }

    public String getName() { return name; }
    public void setName(String name) { this.name = name; }
    public int getAge() { return age; }
    public void setAge(int age) { this.age = age; }

    @Override
    public String toString() { return "User(name=" + name + ", age=" + age + ")"; }

    @Override
    public boolean equals(Object o) { ... }
}
```

**Kotlin** (똑같은 클래스):

```kotlin
data class User(val name: String, val age: Int)
```

👉 생성자, getter/setter, toString, equals 전부 자동 생성 — Java 30줄 → Kotlin 1줄

---

### 3. Java Interoperability

- Java와 100% 호환
- 기존 라이브러리 그대로 사용 가능

👉 **기존 생태계 활용 가능**

---

### 4. 함수형 프로그래밍 지원

Java도 Java 8부터 되긴 하지만, Kotlin이 훨씬 간결하고 정갈합니다.

**함수형이란?**

단계를 하나하나 지시하는 게 아니라, 결과를 한 번에 선언하는 것입니다. 코드가 사람 말에 가까워집니다.

**명령형 (OOP 스타일)** — "어떻게 할지" 단계별로:

```kotlin
val result = mutableListOf<Int>()
for (n in numbers) {       // 1. 반복해
    if (n % 2 == 0) {      // 2. 짝수 확인해
        result.add(n * 2)  // 3. 2배해서 넣어
    }
}
```

**함수형 스타일** — "뭘 원하는지" 한 문장으로:

```kotlin
val result = numbers.filter { it % 2 == 0 }.map { it * 2 }
// "짝수인 것들 중에, 2배한 것들 줘"
```

| | Java | Kotlin |
|---|---|---|
| 시작 | `.stream()` 붙여야 함 | 바로 사용 |
| 람다 | `n -> n % 2 == 0` | `{ it % 2 == 0 }` |
| 마무리 | `.collect(Collectors.toList())` 필수 | 없어도 됨 |
| 불변 변수 | `final` 키워드 별도로 써야 함 | `val` 기본 제공 |

👉 **유연한 설계 및 가독성 향상**

---

### 5. 확장 함수 (Extension Function)

기존 클래스 코드를 수정하지 않고 기능을 붙일 수 있습니다.

**프로젝트 실제 예시** — `JwtHandlerAdapter.kt`:

JWT에서 꺼낸 `subject`(String?)가 null이 아닐 때만 `UsernameVO`로 변환.

```kotlin
// String 클래스를 건드리지 않고 .let {} 확장 함수로 null 안전하게 변환
claims.subject?.let { UsernameVO.from(it) }
```

Java였으면 직접 null 체크를 작성해야 합니다:

```java
String subject = claims.getSubject();
return subject != null ? UsernameVO.from(subject) : null;
```

**한 줄 해석:**

| 부분 | 의미 |
|---|---|
| `claims.subject` | subject 꺼냄 |
| `?.` | null이면 여기서 멈춰 null 반환 |
| `.let { }` | null이 아니면 블록 안 실행 |
| `it` | subject 값 그 자체 (자동으로 쓸 수 있는 이름) |
| `UsernameVO.from(it)` | subject를 UsernameVO로 변환 |

`.let`은 원래 `String` 클래스에 없는 기능인데, Kotlin이 확장 함수로 `String`에 붙여놓은 것입니다. `String` 소스코드는 건드리지 않았는데 `.let {}` 기능이 생겼습니다.

👉 **유지보수성과 확장성 증가**

---

### 차별화 포인트 — Pragmatic Language (실용적인 언어)

- Java OOP와 다른 점은 없음 (클래스, 상속, 인터페이스, 다형성 전부 동일)
- Kotlin은 거기에 FP를 자연스럽게 얹을 수 있음
- 클래스 없이 함수를 쓸 수 있는 것도 컴파일러가 파일명 기준으로 클래스를 자동 생성해주는 것 (사라진 게 아님)

> **"Java가 할 수 있는 건 다 되면서, 코드는 더 짧고 깔끔하게"**

새로운 패러다임을 강요하는 게 아니라, 기존 OOP 개발자가 자연스럽게 넘어오면서 필요할 때 FP도 쓸 수 있게 설계된 언어입니다.
