---
layout: default
title: Java vs Kotlin — 핵심 차이 정리
---

# Java vs Kotlin
### Java 개발자를 위한 Kotlin 핵심 비교 노트

이 문서는 **Java와 Kotlin의 차이점**을 중심으로 Kotlin의 언어적 특징을 정리한 스터디 노트이다.  
문법 나열이 아니라, *왜 Kotlin이 이렇게 설계되었는지*를 이해하는 데 초점을 둔다.

---

## 1. 변수와 초기화

### 모든 값은 객체

Kotlin에서는 모든 값이 객체처럼 취급된다.  
Java의 primitive 타입에 해당하는 `Int`, `Long`, `Double` 등도 **클래스 기반**이다.

- Int, Long, Short, Double, Float, Boolean, Byte
- 필요 시 자동으로 boxing / unboxing 처리

---

### var / val

- `var` : 변경 가능한 변수
- `val` : 변경 불가능한 상수 (재할당 불가)

```kotlin
var x = 10
val y = 20
```

---

### 늦은 초기화 (Late Initialization)

#### lateinit

- 초기화를 나중으로 미루는 non-null 변수
- **primitive 타입에는 사용 불가**

```kotlin
lateinit var name: String

fun main() {
    name = "Kotlin"
    println(name)
}
```

---

#### by lazy

- 최초 접근 시점에 초기화
- thread-safe 옵션 제공

```kotlin
val lazyData: Int by lazy {
    println("initialize")
    10
}

fun main() {
    println("main")
    println(lazyData)
}
```

---

## 2. 타입 시스템과 컬렉션

### 기본 컬렉션 특성

- 기본 컬렉션은 **Immutable**
- Mutable 사용 시 명시 필요

```kotlin
val list = listOf(1, 2, 3)
val mutableList = mutableListOf(1, 2, 3)
```

---

### List

```kotlin
fun main() {
    val list = listOf(10, 20, 30)
    for ((index, value) in list.withIndex()) {
        assert((index + 1) * 10 == value)
    }
}
```

---

### Map

- 순서 보장
- Iterable
- `Pair` 또는 `to` 연산자 사용

```kotlin
fun main() {
    val map = mapOf(
        Pair("one", "hello"),
        "two" to "world"
    )

    assert(map["one"] == "hello")
    assert(map["two"] == "world")
}
```

---

### Set

```kotlin
fun main() {
    val set = mutableSetOf(5, 4, 3, 2, 1)
    assert(set.size == 5)

    for ((index, value) in set.withIndex()) {
        assert(5 - index == value)
    }
}
```

---

## 3. Pair와 Range

### Pair

```kotlin
fun main() {
    val pair = Pair("hello", 3)
    val (a, b) = pair

    assert(a == "hello")
    assert(b == 3)
}
```

---

### Range

```kotlin
fun main() {
    for (i in 0..4) {
        assert(i in 0..4)
    }
}
```

---

## 4. Any, Unit, Nothing

### Any

- 모든 타입의 최상위 타입

```kotlin
fun main() {
    val data: Any = 10
    when (data) {
        is String -> println("String")
        in 1..10 -> println("1~10")
        else -> println("Other")
    }
}
```

---

### Unit

- Java의 void 대체
- 값으로 취급 가능

```kotlin
fun some(): Unit {
    println("hello")
}
```

---

### Nothing

- 정상 종료되지 않는 함수
- 예외 처리, 무한 루프 표현

```kotlin
fun fail(): Nothing {
    throw Exception("error")
}
```

---

## 5. Null Safety

### Nullable 타입

```kotlin
var a: Int = 10
var b: Int? = null
```

---

## 6. 객체지향(OOP)

### 생성자

```kotlin
class User(var name: String)
```

```kotlin
class User(name: String) {
    var name: String

    init {
        this.name = name
    }
}
```

---

### 상속

- 기본적으로 final
- 상속 허용 시 `open` 필요

```kotlin
open class Vehicle(open val name: String)

class Plane(override val name: String) : Vehicle(name)
```

---

### 접근 제한자

- public (기본)
- internal
- protected
- private

---

### Data Class

- equals / hashCode / toString 자동 생성
- 값 기반 비교

```kotlin
data class User(val name: String, val age: Int)
```

---

### Object / Companion Object

#### Object

```kotlin
val singleton = object {
    val data = 10
}
```

---

#### Companion Object

```kotlin
class MyClass {
    companion object {
        var data = 10
    }
}

fun main() {
    MyClass.data = 20
}
```

---

## 정리

- Kotlin은 **Null Safety**와 **불변성**을 기본 철학으로 설계됨
- Java 대비 보일러플레이트 코드 감소
- 함수형 + 객체지향 혼합 언어

Java 개발자에게 Kotlin은  
**더 안전한 Java이자, 더 현대적인 JVM 언어**다.
