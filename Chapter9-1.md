# Chapter 9 : 제네릭



## 9.1 제네릭 타입 파라미터

제네릭을 사용하면 타입 파라미터를 받는 타입을 정의할 수 있는데, 제네릭 타입의 인스턴스를 만드려면 타입 파라미터를 구체적인 타입 인자로 치환해야 합니다.  예를 들어 `List`라는 타입이 있다면 그 안에 들어가는 원소의 타입을 알아야 쓸모가 있을 것입니다. 이 때 타입 파라미터를 사용하면 "이 변수는 리스트다" 대신 "이 변수는 문자열을 담는 리스트다" 라고 말할 수 있습니다.

코틀린에서 제네릭은 자바의 제네릭과 매우 비슷합니다. 하지만 자바와 달리 코틀린에서는 제네렉 타입의 타입 인자를 프로그래머가 명시하거나 컴파일러가 추론할 수 있어야 합니다. 



### 9.1.1 제네릭 함수와 프로퍼티

어떤 특정 타입을 타겟하는 함수가 아닌 모든 타입에 대한 함수를 원할 때 제네릭 타입을 활용하여 함수를 작성할 수 있습니다. 제네릭 함수를 호출할 때는 반드시 구체적 타입으로 타입 인자를 넘겨야만 합니다.

```kotlin
fun <T> printValue(value: T): T {
  println(value.toString())
  return value
}
```

위 함수에서 `<T>` 부분은 타입 파라미터 선언에 해당합니다. 물론 이 타입 파라미터는 파라미터의 타입으로 사용할 수 도 있고, 리턴 타입에 사용할 수도 있습니다. 이제 함수를 사용해보겠습니다.



```kotlin
val str = "hi"
printValue(str) // hi

val num = 2
printValue(num) // 2
```

우리가 따로 타입 인자를 명시적으로 지정할 필요없이 컴파일러가 `T`를 추론하여 함수를 실행시키는 것을 알 수 있습니다.



이번엔 제네릭 고차 함수를 호출해보겠습니다. 람다 파라미터에 대해서 자동으로 만들어진 변수 `it` 의 타입은 `T` 라는 제네릭 타입입니다. 

```kotlin
val peoples = listOf("Hongbeom", "Dmitry", "Svetlana")

fun <T> List<T>.filter(predicate: (T) -> Boolean): List<T>

peoples.filter { it == "Hongbeom" } // [Hongbeom]
```



또한 클래스나 인터페이스 안에 정의된 메소드, 확장 함수 또는 최상위 함수에서 타입 파라미터를 선언할 수 있습니다. 제네릭 함수를 정의할 때와 마찬가지 방법으로 제네릭 확장 프로퍼티를 선언할 수 있습니다. 예를 들어 다음은 리스트의 마지막 원소 바로 앞에 있는 원소를 반환하는 확장 프로퍼티입니다.

```kotlin
val <T> List<T>.penultimate: T
  get() {
    check(size > 1) {
      "Size must be greater than one"
    }
    return this[size - 2]
  }

println(listOf(1, 2, 3, 4).penultimate)
// 3
```



### 9.1.2 제네릭 클래스 선언

자바와 마찬가지로 코틀린에서도 타입 파라미터를 넣은 꺾쇠 기호를 클래스(또는 인터페이스) 뒤에 붙이면 클래스를 제네릭하게 만들 수 있습니다. 타입 파라미터를 이름 뒤에 붙이면 클래스 본문 안에서 타입 파라미터를 다른 일반 타입처럼 사용할 수 있다. 표준 자바 인터페이스인 `List`를 코틀린으로 정의해봅시다. 설명을 위해 `get` 메소드 하나만 남기고 전부 생략하겠습니다.

```kotlin
interface List<T> {
  operator fun get(index: Int): T
}
```



이제 구체적인 타입을 하위 클래스에서 상위 클래스로 넘길 수도 있고 타입 파라미터로 받은 타입을 넘길 수도 있습니다.

```kotlin
class StringList: List<String> {
  override fun get(index: Int): String = // ... //
}

class ArrayList<AT> : List<AT> {
  override fun get(index: Int): T = // ... //
}
```



하위클래스에서 상위 클래스에 정의된 함수를 오버라이드하거나 사용하려면 타입 인자 `T`를 구체적 타입으로 치환해야 합니다. 따라서 `StringList`에서는 `fun get(index: Int): T` 가 아니라 `fun get(index: Int): String` 이라는 시그니처를 사용합니다.

`ArrayList` 클래스는 자신만의 타입 파라미터 `AT`를 정의하면서 그 `AT`를 상위 클래스의 타입 인자로 사용합니다. 여기서의 `AT` 타입과 위에서 본 `List<T>` 의 `T`는 같지 않습니다. 

심지어 클래스가 자기 자신을 타입 인자로 참조할 수도 있습니다. `Comparable` 인터페이스를 구현하는 클래스가 이런 패턴의 예시입니다. 비교 가능한 모든 값은 자신을 같은 타입의 다른 값과 비교하는 방법을 제공해야만 합니다.

```kotlin
interface Comparable<T> {
  fun compareTo(other: T): Int
}

class People: Comparable<People> {
  override fun compareTo(other : People) : Int = /* ... */
}
```



지금까지 살펴본 코틀린의 제네릭은 자바의 제네릭과 비슷했습니다. 이제 자바와 비슷한 다른 개념을 살펴보겠습니다. 이 개념을 사용하면 비교 가능한 원소를 다룰 때 쓸모 있는 함수를 작성할 수 있습니다.



### 9.1.3 타입 파라미터 제약

타입 파라미터 제약은 클래스나 함수에 사용할 수 있는 타입 인자를 제한하는 기능입니다. 어떤 타입을 제네릭 타입의 타입 파라미터에 대한 상한으로 지정하면 그 제네릭 타입을 인스턴스화할 때 사용하는 타입 인자는 반드시 그 상한 타입이거나 그 상한 타입의 하위 타입이어야 합니다.

제약을 가하려면 타입 파라미터 이름 뒤에 콜론(:)을 표시하고 그 뒤에 상한 타입을 적으면 됩니다.

```kotlin
fun <T: Number> List<T>.sum(): T  // kotlin
```

```java
<T extends Number> T sum(List<T> list) // java
```



타입 파라미터 `T`에 대한 상한을 정하고 나면 `T` 타입의 값을 그 상한 타입의 값으로 취급할 수 있습니다.

```kotlin
fun <T: Number> oneHalf(value : T): Double {
  return value.toDouble() / 2.0 // Number 클래스에 정의된 메소드 호출
}

println(oneHalf(3)) // 1.5
```



이제 두 파라미터 사이에서 더 큰 값을 찾는 제네릭 함수를 작성해보겠습니다. 서로를 비교할 수 있어야 최댓값을 찾을 수 있으므로 시그니처에도 두 인자를 서로 비교할 수 있어야 한다는 사실을 지정해야 합니다. 

```kotlin
fun <T: Comparable<T>> max (first: T, second: T): T {
  return if (first > second) first else second
}

println(max("kotlin", "java")) 
// kotlin 
// 문자열은 알파벳순으로 비교
```



아주 드물지만 타입 파라미터에 대해 둘 이상의 제약을 가해야 하는 경우도 있습니다. 그런 경우에는 약간 다른 구문을 사용합니다. 예를 들어 아래 리스트는 `CharSequence`의 맨 끝에 마침표(.)가 있는지 검사하는 제네릭 함수입니다. 

```kotlin
fun <T> ensureTrailingPeriod(seq: T) where T: CharSequence, T: Appendable {
  if (!seq.endsWith('.')) {
    seq.append('.')
  }
}

val helloWorld = StringBuilder("Hello World")
ensureTrailingPeriod(helloWorld)
println(helloWorld)
// Hello World.
```



### 9.1.4 타입 파라미터를 널이 될 수 없는 타입으로 한정

제네릭 클래스나 함수를 정의하고 그 타입을 인스턴스화할 때는 널이 될 수 있는 타입을 포함하는 어떤 타입으로 타입 인자를 지정해도 타입 파라미터를 치활할 수 있습니다. 아무런 상한을 정하지 않은 타입 파라미터는 결과적으로  `Any?` 를 상한으로 정한 파라미터와 같습니다.

그러므로 항상 널이 될 수 없는 타입만 타입 인자로 받게 만드려면 타입 파리미터에 제약을 가해야합니다. 

```kotlin
class Processor<T: Any> {
  fun process(value: T) {
    value.hashCode()
  }
}
```



타입 파라미터를 널이 될 수 없는 타입으로 제약을 걸면 타입 인자로 널이 될 수 있는 타입이 들어오는 일을 막을 수 있기에, `Any`를 사용하지 않고 다른 널이 될 수 없는 타입을 사용하여 상한을 정해도 됩니다.



