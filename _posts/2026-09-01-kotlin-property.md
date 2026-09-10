---
title: "코틀린 클래스 프로퍼티 공부 기록"
date: 2026-09-01 09:00:00 +0900
categories: [Language, Kotlin]
tags: [kotlin, property, getter, setter]
---

## 필드가 아니라 프로퍼티

자바에서는 필드를 선언하고 게터/세터를 따로 만들어야 했다.
코틀린은 프로퍼티 하나를 선언하면 접근자가 같이 만들어진다.

```kotlin
class Person(val name: String, var age: Int)
```

- `val name` → 게터만 생성 (읽기 전용)
- `var age` → 게터 + 세터 생성

즉 **`val`로 프로퍼티를 선언하면 게터를 무조건 포함한다.** 값을 읽는 통로가 없는 프로퍼티는 의미가 없기 때문이다.
반대로 세터는 `var`일 때만 생긴다.

자바로 치면 이 정도 코드가 대체된다.

```java
public class Person {
    private final String name;
    private int age;

    public Person(String name, int age) { this.name = name; this.age = age; }

    public String getName() { return name; }
    public int getAge() { return age; }
    public void setAge(int age) { this.age = age; }
}
```

## 자바에서 부를 때

코틀린 프로퍼티는 실제로 게터/세터 메서드로 컴파일된다. 그래서 자바에서는 메서드로 보인다.

```java
Person p = new Person("david", 20);
p.getName();   // val → 게터만 있음
p.setAge(21);  // var 라서 세터도 있음
```

코틀린 안에서는 `p.name`처럼 필드를 쓰듯 접근하지만, 실제로 호출되는 건 게터다.

## 커스텀 접근자

접근자를 직접 쓸 수도 있다. 이때 프로퍼티 값을 담는 저장 공간을 `field`(backing field)로 참조한다.

```kotlin
class Rectangle(val width: Int, val height: Int) {
    val isSquare: Boolean
        get() = width == height          // 저장 공간 없이 계산만
}

class User {
    var name: String = ""
        set(value) {
            field = value.trim()          // field = 뒷받침하는 필드
        }
}
```

`isSquare`처럼 게터가 계산만 하고 `field`를 쓰지 않으면 뒷받침하는 필드는 아예 만들어지지 않는다.
값을 저장하는 게 아니라 매번 계산해서 돌려주는 프로퍼티다.

## 정리

| 선언 | 게터 | 세터 |
| --- | --- | --- |
| `val` | O | X |
| `var` | O | O |

한 가지 예외는 `private` 프로퍼티다. 기본 접근자를 쓰는 `private` 프로퍼티는 클래스 안에서 필드를 바로 읽으면 되기 때문에, 컴파일러가 게터 메서드를 따로 만들지 않기도 한다.
"게터를 무조건 포함한다"는 건 문법적으로 게터를 통해 접근한다는 의미로 이해하는 게 맞겠다.
