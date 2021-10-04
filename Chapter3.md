# 03. 함수 정의와 호출

이 장에서 다루는 내용

* 컬렉션, 문자열, 정규식을 다루기 위한 함수
* 이름 붙인 인자, 디폴트 파라미터 값, 중위 호출 문법
* 확장 함수와 확장 프로퍼티
* 최상위 및 로컬 함수와 프로퍼티를 사용해 코드 구조화

<br/>

# 1. 코틀린에서 컬렉션 만들기

### [ HashSet, ArrayList, HashMap 생성 ]

```kotlin
fun main() {
  // 집합 (Set)
  val numberSet = hashSetOf(1, 2, 3)

  // 리스트 (List)
  val numberArrayList = arrayListOf(4, 5, 6)

  // 맵 (Map)
  val numberMap = hashMapOf(
    1 to "one",
    2 to "two",
    3 to "three"
  )

  // [1, 2, 3]
  println(numberSet)

  // [4, 5, 6]
  println(numberArrayList)

  // {1=one, 2=two, 3=three}
  println(numberMap)
}
```

* 코틀린에서는 위와 같이 `Set` , `List` , `Map` 을 만들 수 있다.
* Map을 만들 때 사용한 `to` 는 특별한 키워드가 아니라 `일반 함수` 이다. - *to에 대해서는 나중에 다룬다.*

<br/>

컬렉션을 만들어봤으니, 컬렉션이 어떤 클래스로 되어있는지 확인한다.

### [ Collection 클래스 확인 ]

```kotlin
fun main() {
  // 집합 (Set)
  val numberSet = hashSetOf(1, 2, 3)

  // 리스트 (List)
  val numberArrayList = arrayListOf(4, 5, 6)

  // 맵 (Map)
  val numberMap = hashMapOf(
    1 to "one",
    2 to "two",
    3 to "three"
  )

  // class java.util.HashSet
  println(numberSet.javaClass)

  // class java.util.ArrayList
  println(numberArrayList.javaClass)

  // class java.util.HashMap
  println(numberMap.javaClass)
}
```

* `javaClass` : 호출한 객체의 Class 타입을 반환해주는 제네릭 확장 함수

  ```kotlin
  /**
   * Returns the runtime Java class of this object.
   */
  public inline val <T : Any> T.javaClass: Class<T>
      @Suppress("UsePropertyAccessSyntax")
      get() = (this as java.lang.Object).getClass() as Class<T>
  ```

* Set, List, Map이 자바 컬렉션인 것을 확인할 수 있다.

* 이처럼 코틀린은 표준 자바 컬렉션을 활용함으로써, **자바 코드와 상호작용하기 쉽도록** 만들어져 있다.

* 하지만 코틀린에서는 자바보다 더 많은 기능을 쓸 수 있다.

  * Java를 활용하여 List에서 max 값 구하기

    ```java
    public class CollectionJava {
    
      public static void main(String[] args) {
        final List<Integer> numberList = List.of(1, 2, 3);
        final Integer max = numberList.stream().max(Integer::compareTo).get();
        System.out.println(max); // 3
      }
    
    }
    ```

  * Kotlin을 활용하여 List에서 max 값 구하기

    ```kotlin
    fun main() {
        val numberArrayList = arrayListOf(1, 2, 3)
        println(numberArrayList.maxOrNull()) // 3
    }
    ```

<br/>

> 코틀린으로 컬렉션을 만들고, 최댓값을 구하는 함수에 대해 간단히 살펴봤다.
>
> 이제 함수를 만들고 호출하는 것에 대해 좀 더 자세히 살펴보자.

<br/>

# 2. 함수를 호출하기 쉽게 만들기

컬렉션의 원소들을 내가 원하는 형태로 출력시키는 함수를 작성해보자.

### [ 함수를 만들고 사용하는 예시 ]

```kotlin
fun <T> joinToString( ... ) { ... }

fun main() {
    val numberList = listOf(1, 2, 3)

    println(numberList)

    val joinToString = joinToString(numberList, " or ", "<", ">")
    println(joinToString)
}
```

**실행결과**

```kotlin
<1 or 2 or 3>
```

<br/>

함수의 요구사항

* 함수명 : **joinToString**

* 함수 시그니처

  ```kotlin
  fun <T> joinToString(
    collection: Collection<T>,  // 컬렉션
    separator: String,          // 구분자
    prefix: String,             // 접두사
    postfix: String,            // 접미사
  ): String
  ```

* 함수 기능

  * StringBuilder를 활용
  * 컬렉션을 출력
    * 맨 앞에 접두사( **prefix** ) 출력
    * 각 원소 사이에 구분자( **separator** ) 출력
    * 맨 뒤에 접미사( **postfix** ) 출력

<br/>

### [ `joinToString()` 함수 초기 구현 ]

```kotlin
// 컬렉션의 원소들을 내가 원하는 형태로 출력시키는 함수
fun <T> joinToString(
  collection: Collection<T>,  // 컬렉션
  separator: String,          // 구분자
  prefix: String,             // 접두사
  postfix: String,            // 접미사
): String {
  val result = StringBuilder(prefix)

  for ((index, element) in collection.withIndex()) {
    // 첫 원소의 앞에는 구분자를 붙이면 안 되기 때문에
    // 1번째 부터 추가
    if (index > 0) result.append(separator)
    result.append(element)
  }

  result.append(postfix)
  return result.toString()
}

fun main() {
  val numberList = listOf(1, 2, 3)

  // [1, 2, 3]
  println(numberList)

  val joinToString = joinToString(numberList, " or ", "<", ">")

  // <1 or 2 or 3>
  println(joinToString)
}
```

* 함수 시그니처를 살펴보면 제네릭 함수인 것을 알 수 있다. => *제네릭은 뒤쪽에서 자세히..*
* 즉, 모든 타입의 컬렉션을 처리할 수 있다.

<br/>

> 함수를 호출할 때, 모든 인자를 전달하지 않고 기본 값을 제공하는 방법에 대해 살펴보자.

<br/>

## 2.1 이름 붙인 인자

함수의 기본 값을 제공하는 방법을 살펴보기 전에, 함수 호출 부분의 가독성을 향상시켜보자.

<br/>

`joinToString` 함수를 활용해 컬렉션의 인자들을 이어 붙여서 출력하면 다음과 같이 호출하게 된다.

```kotlin
val joinToString = joinToString(numberList, "", "", "")
```

* 해당 함수의 시그니처를 모르는 개발자가 이 코드를 봤을 때, 두 번째와 세 번째, 네 번째가 파라미터가 어떤 의미를 갖는지 전혀 알지 못한다.

<br/>

이러한 혼동을 막기 위해 코틀린에서는 함수를 호출할 때, **인자에 이름을 명시할 수 있다.**

```kotlin
// 인자의 이름을 명시하지 않은 함수 호출 예시
val joinToString = joinToString(
  numberList, 
  "", 
  "", 
  ""
)

// 인자의 이름을 명시한 함수 호출 예시
val joinToString = joinToString(
  collection = numberList,
  separator = "",
  prefix = "",
  postfix = ""
)
```

<br/>

## 2.2. 디폴트 파라미터 값

함수 호출 부분의 가독성을 향상시키는 것에 대해 살펴봤으니, 함수의 디폴트 파라미터 값(기본값)을 제공하는 방법에 대해 알아보자.

<br/>

대부분의 경우 아무 접두사나 접미사 없이 콤마로 원소를 구분하기 때문에, 해당 값 들을 디폴트로 지정해보자.

### [ 디폴트 파라미터 값을 사용해 `joinToString()` 정의하기 ]

```kotlin
fun <T> joinToString(
  collection: Collection<T>,  
  separator: String = ", ",   // 디폴트 값이 지정된 파라미터
  prefix: String = "",        // 디폴트 값이 지정된 파라미터
  postfix: String = "",       // 디폴트 값이 지정된 파라미터
): String {
  val result = StringBuilder(prefix)

  for ((index, element) in collection.withIndex()) {
    if (index > 0) result.append(separator)
    result.append(element)
  }

  result.append(postfix)
  return result.toString()
}

fun main() {
  val numberList = listOf(1, 2, 3)
  val joinToString = joinToString(numberList)

  // 1, 2, 3
  println(joinToString)
}
```

이와 같이 **디폴트 값을 지정해놓으면, 모든 인자를 쓸 수도 있고 일부를 생략할 수도 있다.**

```kotlin
// 1, 2, 3
println(joinToString(numberList))

// 1 2 3
println(joinToString(numberList, " "))

// [1 2 3]
println(joinToString(numberList, " ", "[", "]"))
```

<br/>

> 자바에서는 함수를 클래스 안에 선언해야만 사용할 수 있었으나, 지금까지 살펴본 코틀린의 함수들은 클래스를 선언하지 않고 
> 함수를 작성했다. 이와 같은 함수를 최상위 함수라고 하는데, 이에 대해 자세히 살펴보자.

<br/>

## 2.3. 정적인 유틸리티 클래스 없애기: 최상위 함수와 프로퍼티

정적인 유틸리티란 상태(필드)와 인스턴스 메서드를 갖지 않고 오로지 정적 메서드만을 갖는 클래스이다.

대표적인 예로 JDK의 *java.util.Collections* 클래스가 있다.

### [ `java.util.Collections` 클래스 ]

```java
public class Collections {
  private Collections() {
  }

  public static void reverse(List<?> list) { ... }
  public static void shuffle(List<?> list) { ... }
  ...
}
```

* 위와 같이 자바에서는 모든 코드를 클래스 안에 작성해야 하기 때문에, 정적인 유틸리티가 생겨난다.

<br/>

하지만, 코틀린에서는 이런 무의미한 클래스가 필요 없다.
함수를 클래스에 정의할 필요없이, **소스 파일의 최상위 수준(클래스의 밖)에 위치**시키면 된다.

<br/>

파일의 최상위에 구현된 함수는 맨 위에 정의된 패키지(ex. `package io.wisoft` )의 멤버 함수이므로,
다른 패키지에서 그 함수를 사용하고 싶을 때는 그 함수가 정의된 패키지를 임포트하면 된다.

<br/>

컴파일러가 코틀린의 최상위 함수를 어떻게 컴파일하는지 한 번 살펴보자.

먼저 `strings` 패키지를 만들고, 해당 패키지에 `Join.kt` 파일을 만든 다음 `joinToString` 함수를 정의해보자.

### [ `strings.Join.kt` ]

```kotlin
package strings // 패키지 위치

fun <T> joinToString(
    collection: Collection<T>,  // 컬렉션
    separator: String = ", ",   // 구분자
    prefix: String = "",        // 접두사
    postfix: String = "",       // 접미사
): String {
    val result = StringBuilder(prefix)

    for ((index, element) in collection.withIndex()) {
        if (index > 0) result.append(separator)
        result.append(element)
    }

    result.append(postfix)
    return result.toString()
}
```

위의 코드를 Java 코드로 변환하면 아래와 같이 변환된다.

<br/>

### [ Java로 변환된 코드 ]

```java
public final class JoinKt {
   @NotNull
   public static final String joinToString(...) {
      ...
   }
}

```

즉, JVM이 클래스 안에 들어 있는 코드만을 실행할 수 있기 때문에, 컴파일러는 `Join.kt` 파일을 컴파일 할 때 
새로운 클래스( `JoinKt` )를 정의해준다.

<br/>

### 최상위 프로퍼티

함수와 마찬가지로 프로퍼티도 파일의 최상위 수준에 위치시킬 수 있다.

```kotlin
const val PI: Double = 3.141592
var result: Double = 0.0

fun main() {
    result = PI * 2
    println(result)  // 6.283184
}
```

* 이런 프로퍼티 값은 정적 필드에 저장된다.

<br/>

어떻게 이렇게 구성될 수 있는지 궁금하기 때문에, 코드를 Java 코드로 변환해보자.

### [ 최상위 프로퍼티를 사용한 Kotlin 파일을 Java 코드로 변환한 예시 ]

```java
public final class SuperPropertyKt {
   public static final double PI = 3.141592D;
   private static double result;

   public static final double getResult() {
      return result;
   }

   public static final void setResult(double var0) {
      result = var0;
   }

   ...
}
```

* 보시다시피 `const val` 은 `public static final` 로 변환이 되었으며,
  `var` 는 `private static` 으로 변환이 되며 getter와 setter가 구현되었다.

<br/>

> 지금까지 디폴트 파라미터 값과 최상위 함수와 프로퍼티를 살펴보며 `joinToString` 함수를 개선했다.
> 이제 확장 함수와 확장 프로퍼티를 활용해, 함수를 좀 더 개선해보도록 하자.

<br/>

# 3. 확장 함수와 확장 프로퍼티

**확장 함수**는 기존의 클래스를 상속받거나 재작성하지 않고도, **해당 클래스의 멤버 메서드인 것처럼 호출할 수 있도록 한다.**

예시를 통해 자세히 살펴보도록 하자.

<br/>

어떤 문자열의 마지막 문자를 돌려주는 메소드를 확장 함수로 만들어보자.

### [ 문자열의 마지막 문자를 돌려주는 확장 함수 ]

```kotlin
// 확장 함수
fun String.lastChar(): Char = this[this.length - 1]

fun main() {
    println("abc".lastChar()) // c
}
```

* 확장 함수를 만들려면 추가하려는 함수 이름 앞에 그 함수가 확장할 클래스의 이름을 덧붙이기만 하면 된다.
* 확장할 클래스 이름을 **수신 객체 타입(receiver type)**이라 부른다. 
  -> `fun String.lastChar(): Char` 에서 **`String`**
* 확장 함수가 호출되는 대상이 되는 값(객체)을 **수신 객체(receiver object)**라고 부른다. 
  -> `this[this.length - 1]` 에서  **`this`**
  -> `"abc".lastChar()` 에서 **`"abc"`**

* 현재는 `this` 를 사용해 수신 객체 멤버에 접근하고 있지만, `this` 없이도 접근할 수 있다.     54ㄷ

  *ex) `this` 를 생략한 예시*

  ```kotlin
  //fun String.lastChar(): Char = this[this.length - 1]
  fun String.lastChar(): Char = get(length - 1)
  
  fun main() {
      println("abc".lastChar()) // c
  }
  ```

<br/>

위와 같이 확장 함수 내부에서는 수신 객체 타입의 메소드나 프로퍼티를 바로 사용할 수 있다.
하지만, 클래스 내부에서만 사용할 수 있는 `private` 멤버나 `protected` 멤버는 접근할 수 없다.

<br/>

> 확장 함수가 무엇이고, 어떻게 정의하며 어떻게 사용하는지 간단하게 알아봤고,
> 확장 함수를 임포트 하는 것과 자바에서의 확장 함수 호출, 확장 함수의 제약 사항 등을 살펴보며,
> 확장 함수에 대해 좀 더 자세히 살펴보자.

<br/>

## 3.1. 임포트와 확장 함수

확장 함수를 사용하기 위해서는 그 함수를 다른 클래스나 함수와 마찬가지로 임포트해야만 한다.

### [ `strings.Print.kt` ]

```kotlin
package strings

fun String.print() = println(this)
```

### [ `ImportAndExtension.kt` ]

```kotlin
// strings 패키지에 있는 print 확장 함수를 임포트하는 코드
import ch03.strings.print
// *로 모두 임포트 할 수 있다.
import ch03.strings.*
// as 키워드로 함수를 다른 이름으로 부를 수 있다.
import ch03.strings.print as println

fun main() {
    "abc".print()     // abc
    "abc".println()   // abc
}
```

* 한 파일 안에서 다른 여러 패키지에 속해있는 이름이 같은 함수를 가져와 사용해야 하는 경우, `as` 키워드로 이름을 바꿔서 
  임포트하면 **이름 충돌을 막을 수 있다.**

<br/>

## 3.2. 자바에서 확장 함수 호출

코틀린의 확장 함수는 수신 객체를 첫 번째 인자로 받는 정적 메소드다.

따라서 확장 함수를  `Print.kt` 파일에 정의했다면 다음과 같이 호출할 수 있다.

### [ 자바에서 확장 함수를 호출하는 예시 ]

```java
import strings.PrintKt;

public class PrintExam {
  public static void main(String[] args) {
    PrintKt.print("abc");  // abc
  }
}
```

<br/>

## 3.3. 확장 함수로 유틸리티 함수 정의

일반 함수 였던 `joinToString` 함수를 확장 함수로 바꿔보자.

### [ `joinToString` 함수를 확장 함수로 변환 ]

```kotlin
package ch03.strings

// 기존의 joinToString 함수
//fun <T> joinToString(
//    collection: Collection<T>,  // 컬렉션
//    separator: String = ", ",   // 구분자
//    prefix: String = "",        // 접두사
//    postfix: String = "",       // 접미사
//): String {
//    val result = StringBuilder(prefix)
//
//    for ((index, element) in collection.withIndex()) {
//        if (index > 0) result.append(separator)
//        result.append(element)
//    }
//
//    result.append(postfix)
//    return result.toString()
//}

// Collection<T> 에 대한 확장 함수 선언
fun <T> Collection<T>.joinToString(
  separator: String = ", ",   // 구분자
  prefix: String = "",        // 접두사
  postfix: String = "",       // 접미사
): String {
  val result = StringBuilder(prefix)

  // "this"는 수신 객체를 의미한다.
  for ((index, element) in this.withIndex()) {
    if (index > 0) result.append(separator)
    result.append(element)
  }

  result.append(postfix)
  return result.toString()
}

fun main() {
  val joinToString = listOf("a", "b", "c").joinToString()

  // a, b, c
  println(joinToString)
}
```

<br/>

현재는 수신 객체인 컬렉션의 타입을 제네릭 타입으로 선언했지만, 더 구체적인 타입으로 지정할 수도 있다.

```kotlin
// Collection<T> 에 대한 확장 함수 선언
//fun <T> Collection<T>.joinToString(
//    separator: String = ", ",   // 구분자
//    prefix: String = "",        // 접두사
//    postfix: String = "",       // 접미사
//): String {
//    val result = StringBuilder(prefix)
//
//    // "this"는 수신 객체를 의미한다.
//    for ((index, element) in this.withIndex()) {
//        if (index > 0) result.append(separator)
//        result.append(element)
//    }
//
//    result.append(postfix)
//    return result.toString()
//}

fun Collection<String>.join(
    separator: String = ", ",   // 구분자
    prefix: String = "",        // 접두사
    postfix: String = "",       // 접미사
): String {
    val result = StringBuilder(prefix)

    // "this"는 수신 객체를 의미한다.
    for ((index, element) in this.withIndex()) {
        if (index > 0) result.append(separator)
        result.append(element)
    }

    result.append(postfix)
    return result.toString()
}

fun main() {
    println(listOf("a", "b", "c").join())

    // 타입이 맞지 않기 때문에, 컴파일 에러 발생!
    println(listOf(1, 2, 3).join())
}
```

<br/>

## 3.4. 확장 함수는 오버라이드할 수 없다

먼저 멤버 함수를 오버라이드하여 사용하는 예시를 살펴보자.

### [ 멤버 함수 오버라이드하기 ]

```kotlin
open class View {
    open fun click() = println("View clicked")
}

// Button은 View를 확장한다.
class Button : View() {
    override fun click() = println("Button clicked")
}

fun main() {
    val view: View = Button()

    // Button clicked
    view.click()
}
```

* `View` 타입 변수에 대해  `click` 메소드를 호출했는데, `Button` 이 오버라이드한 `click` 이 호출되어
  **Button clicked** 라는 결과가 출력된다.

<br/>

하지만, 확장 함수는 이처럼 작동하지 않는다. 확장 함수는 클래스의 일부가 아니기 때문이다.

확장 함수는 호출될 때, 수신 객체로 지정한 변수의 정적 타입에 의해 어떤 확장 함수가 호출될지 결정된다.

### [ 기반 클래스와 하위 클래스에 확장 함수를 정의하고 호출하는 예시 ]

```kotlin
open class View {
    open fun click() = println("View clicked")
}

class Button : View() {
    override fun click() = println("Button clicked")
}

fun View.showOff() = println("I'm a view!")
fun Button.showOff() = println("I'm a button!")

fun main() {
    val view: View = Button()

    // I'm a view!
    view.showOff()
}
```

* 결과를 보시다시피, `view` 가 가리키는 객체의 실제 타입이 `Button` 이지만, 이 경우 `view` 객체의 타입이 `View` 이기 때문에
  무조거 `View` 의 확장 함수가 호출된다.

<br/>

어떤 클래스를 확장한 함수와 그 클래스의 멤버 함수의 이름과 시그니처가 같다면 확장 함수가 아닌 멤버 함수가 호출된다.
즉, 멤버 함수가 확장 함수보다 우선순위가 더 높다.

```kotlin
class User() {
    fun hello() = "hello"
}

fun User.hello() = "안녕하세요"

fun main() {
    val user = User()
    println(user.hello()) 
}
```

**실행결과**

```
hello
```

<br/>

기존 클래스에 새로운 메소드를 추가하는 확장 함수에 대해 살펴봤으니, 이제는 새로운 속성을 추가하는 
**확장 프로퍼티**에 대해 살펴보자.

<br/>

## 3.5. 확장 프로퍼티

### [ 확장 프로퍼티 선언하기 ]

```kotlin
val String.lastChar: Char
    get() = get(length - 1)

fun main() {
    println("abc".lastChar) // c
}
```

* 이처럼 확장 프로퍼티는 기존 클래스 객체에 대한 프로퍼티를 추가할 수 있다.
* 하지만 확장 프로퍼티는 뒷받침하는 필드가 없어서 **상태를 저장할 수는 없다.**
  그러므로 기본 게터가 제공되지 않아, **커스텀 게터를 꼭 정의해야 한다.**

* 마찬가지로 초기화 코드도 쓸 수 없다.

  ```kotlin
  // 컴파일 에러 발생 !!
  // Extension property cannot be initialized because it has no backing field
  // 백킹 필드가 없기 때문에 확장 속성을 초기화할 수 없습니다.
  val String.defaultChar: Char = "a"
  ```

<br/>

### [ 변경 가능한 확장 프로퍼티 선언하기 ]

```kotlin
var StringBuilder.lastChar: Char
    get() = get(length - 1)                     // 프로퍼티 게터
    set(value: Char) {
        this.setCharAt(length - 1, value)       // 프로퍼티 세터
    }

fun main() {
    val stringBuilder = StringBuilder("Kotlin?")
    println(stringBuilder)       // Kotlin?
    
    stringBuilder.lastChar = '!'
    println(stringBuilder)       // Kotlin!
}
```

* 확장 프로퍼티를 선언할 때, 커스텀 게터 뿐만 아니라 커스텀 세터 또한 정의할 수 있다.
* 위의 코드와 같이 `var` 로 확장 프로퍼티를 선언한 뒤, 커스텀 세터를 구현하면 된다.

<br/>

지금까지 확장에 대해 알아봤다.
이번에는 컬렉션을 처리할 때 유용한 라이브러리 함수들에 대해 살펴보자.

<br/>

# 4. 가변 길이 인자, 중위 함수 호출, 라이브러리 지원

코틀린 표준 라이브러리에는 수많은 확장 함수가 존재하는데, IDE의 코드 완성 기능을 통해 원하는 함수를 선택하면 되기 때문에 굳이 다 알 필요는 없다.

<br/>

## 4.1. 가변 인자 함수: 인자의 개수가 달라질 수 있는 함수 정의

가변 길이 인자는 메소드를 호출할 때 원하는 개수만큼 값을 인자로 넘기면 컴파일러가 배열에 그 값들을 넣어주는 기능이다.

<br/>

### [ 자바에서의 가변 인자 사용 예시 ]

```java
public static <T> List<T> asList(T... a) {
  return new ArrayList<>(a);
}

public static void main(String[] args) {
  final List<Integer> integers = Arrays.asList(1, 2, 3);
}
```

* 자바에서는 `...` 문법을 활용해 가변인자를 사용한다.

<br/>

코틀린은 자바에서의 `...` 대신 `vararg` 변경자를 사용한다.

### [ 코틀린에서의 가변 인자 사용 예시 ]

```kotlin
val list: List<Int> = listOf(1, 2, 3)
```

<br/>

`listOf` 확장 함수 시그니처를 살펴보자.

### [ `kotlin.collections.Collections.kt` ]

```kotlin
package kotlin.collections

...

public fun <T> listOf(vararg elements: T): List<T> = ...
```

* `vararg` 변경자를 사용해 가변 인자를 전달하는 것을 확인할 수 있다.

<br/>

배열에 들어있는 원소를 가변 길이 인자로 넘길 때도 코틀린과 자바 구문이 다르다.

자바에서는 배열을 그냥 넘기면 되지만 **코틀린에서는 배열을 명시적으로 풀어서 배열의 각 원소가 인자로 전달되게 해야 한다.**

이때, **스프레드(spread) 연산자**를 사용한다.

<br/>

### [ 스프레드 연산자(*) 사용 예시 ]

```kotlin
fun printIntList(vararg elements: Int) = elements.forEach(System.out::println)

fun main() {
    val intArray = intArrayOf(1, 2, 3)
    printIntList(*intArray)
}
```

실행예시

```
1
2
3
```

<br/>

이번에는 함수 호출의 가독성을 향상시킬 수 있는 **중위 호출에** 대해 살펴보자.

<br/>

## 4.2. 값의 쌍 다루기: 중위 호출과 구조 분해 선언

코틀린에서는 Map을 만들 때, `mapOf` 함수를 사용한다.

### [ Map 생성 예시 ]

```kotlin
fun main() {
    val map = mapOf(1 to "one", 2 to "two", 3 to "three")
    println(map) // {1=one, 2=two, 3=three}
}
```

* 보시다시피 `to` 라는 키워드를 사용하는데, 이는 **중위 호출(infix call)** 이라는 특별한 방식으로 `to` 라는 일반 메소드를 
  호출한 것이다.

<br>

중위 호출 시에는 수신 객체와 유일한 메소드 인자 사이에 메소드 이름을 넣는 것이다.

### [ 일반 호출 방식과 중위 호출 방식 예시 ]

```kotlin
println(5.to("five"))   // (5, five) - 일반 호출
println(6 to "six")     // (6, six)  - 중위 호출
```

* 이처럼 인자가 하나뿐인 일반 메소드나 인자가 하나뿐인 확장 함수에 중위 호출을 사용할 수 있다.

<br>

`to` 확장 함수의 시그니처를 살펴보자.

```kotlin
public infix fun <A, B> A.to(that: B): Pair<A, B> = Pair(this, that)
```

* 함수를 중위 호출이 가능하도록 허용하고 싶으면 `infix` 변경자를 함수 선언 앞에 추가하면 된다.

<br>

`to` 함수는 `Pair` 인스턴스를 반환한다. 이는 두 원소로 이뤄진 순서쌍을 의미한다.

이 `Pair` 인스턴스를 활용해 두 변수를 즉시 초기화할 수 있다.

```kotlin
val (number, name) = 1 to "one"
println(number) // 1
println(name)   // one
```

* 이런 기능을 **구조 분해 선언(destructuring declaration)** 이라고 부른다.

<br/>

루프에서도 구조 분해 선언을 활용할 수 있다.

```kotlin
fun main() {
  val list = listOf("a", "b", "c")
  for ((index, element) in list.withIndex()) {
    println("$index : $element")
  }
}
```

실행결과

```
0 : a
1 : b
2 : c
```

<br/>

이번에는 확장 함수를 통해 문자열과 정규식을 더 편리하게 다루는 방법에 대해 살펴보자.

<br/>

# 5. 문자열과 정규식 다루기

자바와 코틀린 문자열 처리 API의 차이에 대해 살펴보자.

<br/>

## 5.1. 문자열 나누기

자바에서의 `split` 메소드를 활용해 `.` 을 기준으로 문자열을 분리하려다 실수하는 개발자들이 많다.

<br/>

예시를 통해 자세히 살펴보자.

### [ 자바에서의 `split` 메서드로 `.` 을 기준으로 문자열을 분리하는 예시 ]

```java
public static void main(String[] args) {
  final String[] stringArray = "2021.10.03".split(".");

  System.out.println("원소 출력");
  for (String element: stringArray) {
    System.out.println(element);
  }
}
```

실행결과

```
원소 출력

```

* 결과를 보면 빈 문자열("")이 출력된 것을 확인할 수 있다.

* 이와 같이 결과가 나오는 이유는 자바의 `split` 메서드는 파라미터로 정규식을 전달받기 때문이다.

  * *정규식으로 `.` 은 모든 문자와 대응된다.*

  * 꿀팁 : Intellij 에서 입력 커서를 정규식에 두고 `option + enter (Mac 기준)` 를 누르면, 
    `Check RegExp` 라는 기능을 사용할 수 있다.

    ![image](https://user-images.githubusercontent.com/43431081/135742774-6a56e678-266c-4289-9156-1e500e2bc1cc.png)

    ![image](https://user-images.githubusercontent.com/43431081/135742839-2eaa3a9e-c08b-40d9-956a-c1355f18fd75.png)

    ![image](https://user-images.githubusercontent.com/43431081/135742843-527cb115-01b9-4905-ba1a-10b58c5c94a8.png)

<br/>

하지만, 코틀린에서는 자바의 `split` 대신에 여러 가지 다른 조합의 파라미터를 받는 `split` 확장 함수를 제공한다.

### [ `kotlin.text.Strings` API ]

```kotlin
public inline fun CharSequence.split(
  regex: Regex, 
  limit: Int = 0
): List<String> { 
  ... 
}

public fun CharSequence.split(
  vararg delimiters: String, 
  ignoreCase: Boolean = false, 
  limit: Int = 0
): List<String> {
  ...
}
```

* 첫 번째 함수는 정규식을 파라미터로 받는 함수이고, 두 번째 함수는 나눌 기준의 문자열을 받는 함수이다.
* 따라서 정규식이나 일반 텍스트 중 어느 것으로 문자열을 분리하는지 쉽게 알 수 있다.

<br/>

대시(-)로 문자열을 분리하는 예시를 통해 살펴보자.

### [ 문자열을 대시(-)로 분리하는 예시 ]

```kotlin
fun main() {
  val strings: List<String> = "010-1234-1234".split("[\\-]".toRegex())
  println(strings) // [010, 1234, 1234]
}
```

* [\\\\-] : 대시(-)를 찾는 정규식 (대시는 특수문자이기 때문에 백슬래시로 찾을 문자로 사용할 것이라고 명시해야 함)

* 자바에서는 `split` 의 기본 파라미터가 정규식이지만, 코틀린은 문자열과 정규식을 각각 받는 함수가 따로 있기 때문에 
  편리하게 사용할 수 있다.

<br/>

코틀린에서는 `split` 확장 함수를 오버로딩한 버전 중에서 구분 문자열을 하나 이상 인자로 받는 함수가 있다.

```kotlin
fun main() {
  val telephone: List<String> = "+82 10-1234-1234".split(" ", "-")
  println(telephone) // [+82, 10, 1234, 1234]
}
```

* 공백과 대시로 전화번호 문자열을 나눠 문자열 배열로 변환한 예시이다.

<br/>

## 5.2. 정규식과 3중 따옴표로 묶은 문자열





---

