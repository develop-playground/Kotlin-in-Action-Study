# 코틀린 타입 시스템

코틀린의 타입 시스템은 코드의 가독성을 향상시키는데 도움이 되는 몇 가지 특성을 제공합니다. 이 특성으로는 nullable 타입과 읽기 전용 컬렉션이 있습니다. 이번 장에서는 이런 내용들에 대해 자세히 살펴보겠습니다.



## Null이 될 수 있는 타입

먼저 다음 자바 함수를 살펴보겠습니다.

``` java
int strLen(String s) {
  return s.length();
}
```



이 함수에 `null`을 넘기면 NPE가 발생합니다. 위 함수를 코틀린으로 다시 작성해보겠습니다.

``` kotlin
fun strLen(s: String) = s.length
```



코틀린에서는 위와 같은 방식으로 파라미터 타입을 지정했을 때 null을 넘길 수 없습니다. 혹시라도 `null`을 넘길 경우 컴파일 에러가 발생합니다.

``` kotlin
strLen(null) // ERROR
```



이 함수가 null과 문자열을 인자로 받을 수 있게 하려면 타입 이름 뒤에 물음표(?)를 명시해야 합니다.

``` kotlin
fun strLenSafe(s: String?) = ...
```



이처럼 `String?`, `Int?` 등 어떤 타입이든 이름 뒤에 물음표를 붙이면 그 타입의 변수나 프로퍼티에 `null` 참조를 저장할 수 있다는 의미가 됩니다. `null`이 될 수 있는 타입의 변수가 있다면 그에 대해 수행할 수 있는 연산이 제한됩니다. 예를 들어 `null`이 될 수 있는 타입인 변수에 대해 **변수.메소드()** 처럼 메소드를 직접 호출할 수는 없습니다.

``` kotlin
fun strLenSafe(s: String?) = s.length() // ERROR
```



그렇기에 우리는 단순하게 `if` 검사를 통해 `null` 값을 다룰 수 있습니다.

``` kotlin
fun strLenSafe(s: String?): Int {
  return if (s != null) s.length else 0
}
```



하지만 nullable을 다루기 위해 사용할 수 있는 도구가 `if` 검사뿐이라면 코드가 매우 번잡해질 것입니다. 그렇기에 코틀린은 nullable을 다루기 위해 사용할 수 있는 여러 도구를 제공합니다.



## 안전한 호출 연산자 : ?.

코틀린이 제공하는 가장 유용한 도구 중 하나는 안전한 호출 연산자인 `?.` 입니다. `?.`은 `null` 검사와 메소드 호출을 한 번의 연산으로 수행합니다. 예를 들어 `s?.toUpperCase()`는 훨씬 복잡한 `if (s != null) s.toUpperCase() else null`가 같습니다. 하지만 안전한 호출의 결과 타입도 nullable 타입이라는 사실에 유의하여야 합니다. 

``` kotlin
fun printAllCaps(s: String): String? {
  return s?.toUpperCase() // nullable
}
```



## 엘비스 연산자 : ?:

코틀린은 `null` 대신 사용할 디폴트 값을 지정할 때 편리하게 사용할 수 있는 연산자를 제공합니다. 그 연산자를 엘비스 연산자라고 합니다. 다음은 엘비스 연산자를 사용하는 예시입니다.

``` kotlin
fun foo(s: String?): String {
  return s ?: "" // null일 경우 "" 리턴
}
```



물론 `throw` 키워드에도 함께 사용할 수 있습니다. 

``` kotlin
fun foo(s: String?): String {
  return s ?: throw NullPointerException() // null일 경우 NPE!
}
```



## 안전한 캐스트 : as?

2장에서 우리는 코틀린 타입 캐스트 연산자인 `as`에 대해 살펴봤습니다. 자바 타입 캐스트와 마찬가지로 대상 값을 형변환 할 수 없으면 `ClassCastException`이 발생합니다. 물론 `as` 를 사용할 때마다 `is` 를 통해 일일이 검사해볼수도 있습니다. 하지만 안전하면서 간결한 언어를 지향하는 코틀린은 더 나은 해법을 제공합니다.

``` kotlin
fun foo(o: Any?): String {
  return (o as? String) ?: "" // String으로 형변환을 시도했는데 실패하여 null이 떨어질 경우 "" 리턴
}
```



### Quiz

다음 요구사항을 만족하는 메소드를 작성해봅시다.

- `Any?` 타입을 파라미터로 받아서 문자열 타입으로 형변환 후 길이를 리턴하는 함수. 형변환에 실패할 경우엔 0을 리턴합니다.



## Null 아님을 단언 : !!

느낌표를 이중으로 사용하면 어떤 값이든 널이 될 수 없는 타입으로 강제로 바꿀 수 있습니다. 하지만 실제로 `null`인 값에 대해 `!!`를 적용하면 NPE가 발생합니다. 



```kotlin
fun foo(s: String?): Int {
  val sNotNull: String = s!!
  return sNotNull.length
}
```

`null`이 아니라고 단언했으므로 `null`일 경우 처리를 따로 해주지 않아서 작성할 때는 편할 수 있지만 `null`이 인자로 들어올 경우 에러를 초래하므로 실제로는 엘비스 연산자 등을 사용하여 핸들링을 해주는 것이 좋습니다.



## 나중에 초기화할 프로퍼티

코틀린에서 클래스 안의 `null`이 될 수 없는 프로퍼티를 생성자 안에서 초기화하지 않고 특별한 메소드 안에서 초기화할 수 는 없습니다. 일반적으로 생성자에서 모든 프로퍼티를 초기화해야하고 프로퍼티 타입이 `null`이 될 수 없는 타입이라면 반드시 `null`이 아닌 값으로 그 프로퍼티를 초기화해야 합니다. 

이를 해결하기 위해 `lateinit` 변경자를 사용할 수 있습니다. (혹은 `Lazy`)

``` kotlin
class MyService {
  fun performAction(): String = "foo"
}

class MyTest {
  
  private lateinit var myService: MyService
  
  @Before fun setUp() {
    myService = MyService()
  }
  
  @Test fun testAction() {
    Assert.assertEquals("foo", myService.performAction())
  }
  
}
```



위 변경자를 사용하여 초기화하는 프로퍼티는 항상 `var` 이어야 합니다. `val` 프로퍼티는 `final` 필드로 컴파일되며, 생성자 안에서 반드시 초기화해야 합니다. 하지만 위 변경자를 사용하여 초기화하는 프로퍼티는 초기화되기 전에 프로퍼티에 접근하면 `"lateinit property ~ has not been initialized"`라는 예외가 발생합니다. 이런 예외를 방어하기 위해 다음과 같은 방법을 사용할 수 있습니다. (필요하다면)

``` kotlin
class Foo {
  
  private lateinit var str: String
  
  fun action() {
    if (this::str.isInitialized) {
      println(str)
    }
  }
  
}
```



## Null이 될 수 있는 타입 확장

`null`이 될 수 있는 타입에 대한 확장 함수를 정의하면 `nullable` 타입에 대한 도구로 활용할 수 있습니다. 어떤 메소드를 호출하기 전에 수신 객체 역할을 하는 변수가 `null`이 될 수 없다고 보장하는 대신, 직접 변수에 대해 메소드를 호출해도 확장 함수인 메소드가 알아서 `null`을 처리해줍니다. 

코틀린의 `isNullOrBlank()` 함수를 살펴봅시다.

```kotlin
fun verifyUserInput(input: String?) {
  if (input.isNullOrBlank()) { // ?. 호출을 하지 않아도 됨.
    println("success") 
  }
}
```



`isNullOrBlank` 함수는 `null`을 명시적으로 검사하여 `null`인 경우 `true`를 리턴하고 반대의 경우 `false`를 리턴합니다.

```kotlin
fun String?.isNullOrBlank(): Boolean = this == null || this.isBlank() // 뒤의 this는 스마트 캐스트가 적용됨.
```



## 타입 파라미터의 Null 가능성

`null` 가능성은 제네릭 타입에서도 유효합니다. 타입 파라미터 T를 클래스나 함수 안에서 타입 이름으로 사용하면 이름 끝에 물음표가 없더라도 T는 nullable한 타입입니다.

```kotlin
fun <T> printHashCode(t: T) {
  println(t?.hashCode()) // t가 null이 될 수 있으므로 ?.로 호출해야한다.
}
```



타입 파라미터가 `null`이 아님을 확실하게 하려면 타입 상한을 지정해야 합니다. 아래 예시를 보겠습니다.

```kotlin
fun <T: Any> printHashCode(t: T) {
  println(t.hashCode()) 
}
```



타입 파라미터는 nullable한 타입을 표시하려면 반드시 물음표를 타입 이름 뒤에 붙여야 한다는 규칙의 예외입니다. 이는 9장(제네릭)에서 더 자세히 살펴보겠습니다.



## 코틀린의 원시 타입

코틀린은 자바와 달리 원시 타입과 래퍼 타입을 구분하지 않습니다. 코틀린은 내부에서 원시 타입을 래핑하여 사용하는데, 어떻게 작동하는지, `Object`, `Void` 등의 자바 타입을 어떻게 대응하는지에 대해서 살펴보겠습니다.



### 원시 타입

코틀린은 원시 타입과 래퍼 타입을 구분하지 않으므로 항상 같은 타입을 사용합니다.

``` kotlin
val i : Int = 1
val list: List<Int> = listOf(1, 2, 3)
```

원시 타입과 참조(래퍼) 타입이 같다면 코틀린은 이것들을 항상 객체로 표현하는 건가라는 의문이 들 수 있습니다. 항상 객체로 표현한다면 이는 상당히 비효율적이겠지만 코틀린은 그렇지 않습니다. 

대부분의 경우 (변수, 파라미터, 리턴 타입 등등) 코틀린의 `Int` 타입은 자바 `int` 타입으로 컴파일 됩니다. 하지만 컬렉션 같은 제네릭 클래스를 사용하는 경우엔 `Int` 의 래퍼 타입에 해당하는 `java.lang.Integer` 객체가 들어가게 됩니다.

코틀린의 원시 타입에는 널 참조가 들어갈 수 없기 때문에 그에 상응하는 자바 원시 타입으로 컴파일 할 수 있습니다. 마찬가지로 반대로 자바 원시 타입도 코틀린에서 사용할 경우 널이 될 수 없는 타입으로 취급할 수 있습니다.



### 널이 될 수 있는 원시 타입 : Int?, Boolean? 등

`null` 참조를 자바의 참조 타입 변수에만 대입할 수 있기 때문에 널이 될 수 있는 코틀린 타입은 자바 원시 타입으로 표현할 수 없습니다. 따라서 nullable한 원시 타입을 코틀린에서 사용하면 그 타입은 자바의 래퍼 타입으로 컴파일 됩니다.

먼저 nullable한 타입을 사용하는 예시를 살펴보겠습니다. 

```kotlin
data class Person(
  val name: String, 
  val age: Int? = null
) {
  
  fun isOlderThan(other: Person) : Boolean? {
    if (age == null || other.age == null) {
      return null
    }
    return age > other.age
  }
  
}

println(Person("Sam, 35").isOlderThan("Amy, 32")) // false
println(Person("Sam, 35").isOlderThan(Person("Jane"))) // null

```



여기서 나이를 비교하는 함수를 살펴보면 `age` 는 nullable한 타입이기 때문에 먼저 두 값이 널이 아닌지 검사해야만 합니다.  컴파일러는 널 검사를 마친 다음에야 두 값을 일반적인 값처럼 다루게 허용합니다. 

`Person` 클래스에 선언된 `age` 프로퍼티의 값은 `java.lang.Integer` 로 저장됩니다. 하지만 이런 자세한 사항들은 자바에서 가져온 클래스를 다룰 때만 문제가 됩니다.

앞에서 이야기한 대로 제네릭 클래스의 경우 래퍼 타입을 사용하는데, 어떤 클래스의 타입 인자로 원시 타입을 넘기면 코틀린은 그 타입에 대한 박스 타입을 허용하게 됩니다. 예를 들어 아래 코드에서는 `null` 값이나 널이 될 수 있는 타입을 전혀 사용하지 않았지만 만들어지는 리스트는 래퍼인 `Integer` 타입으로 이루어진 리스트입니다.

```kotlin
val listOfInts = listOf(1, 2, 3)
```



이렇게 컴파일하는 이유는 JVM에서 제네릭을 구현하는 방법 때문입니다. JVM은 타입 인자로 원시 타입을 허용하지 않기에, 자바나 코틀린 모두 제네릭 클래스는 항상 박스 타입을 사용해야 합니다. 



### Any, Any? : 최상위 타입

자바에서 `Object` 가 최상위 타입이듯 코틀린에서는 `Any` 타입이 모든 널이 될 수 없는 타입의 조상 타입입니다. 자바와는 다르게 코틀린에서는 `Any`가 `Int` 등의 원시 타입을 포함한 모든 타입의 조상 타입이 됩니다.

자바와 마찬가지로 코틀린에서도 원시 타입 값은 `Any` 타입의 변수에 대입하면 자동으로 값을 객체로 감싸게 됩니다.

```kotlin
val answer: Any = 42
```



`Any` 는 널이 될 수 없는 타입이며, 위 변수에는 `null` 이 들어갈 수 가 없습니다. 널을 포함하는 모든 값을 대입할 변수를 선언하려면 `Any?` 타입을 사용해야 합니다. 내부에서 `Any` 타입은 `java.lang.Object` 에 대응하며, 자바 메소드에서 `Object` 를 인자로 받거나 반환하면 코틀린에서는 `Any`로 그 타입을 취급합니다.



### Unit 타입 : 코틀린의 Void

코틀린 `Unit` 타입은 자바 `void` 와 같은 기능을 가지고 있습니다. 아무것도 반환하지 않는 함수의 리턴 타입으로 `Unit` 을 사용할 수 있습니다.

```kotlin
fun foo(): Unit { }
```



이는 리턴 타입 없이 정의한 블록이 본문인 함수와 같습니다.

```kotlin
fun foo() { }
```



대부분의 경우 `void`와 `Unit` 의 차이를 알기는 어렵습니다. 코틀린의 `Unit` 과 자바의 `void`와의 차이점은 무엇일까요? `Unit`은 모든 기능을 가지는 일반적인 타입이며, `void`와 달리 `Unit`을 타입 인자로 사용할 수 있습니다. 

```kotlin
interface Processor<T> {
  fun process(): T
}

class NoResultProcessor: Processor<Unit> {
  override fun process(): { 
    // process logic
  }
}
```











인터페이스의 시그니처는 `process`가 어떤 값을 리턴하라고 요구합니다.  `Unit` 타입도 `Unit` 값을 제공하기 때문에 메소드에서 `Unit` 값을 리턴하는 데는 아무 문제가 없습니다. 하지만 `NoResultProcessor` 에서 명시적으로 `Unit` 을 리턴할 필요 없이 컴파일러가 묵시적으로 `return Unit`을 넣어줍니다.



### Nothing 타입 : 이 함수는 결코 정상적으로 끝나지 않는다.

코틀린에는 결코 성공적으로 값을 돌려주는 일이 없어서 리턴 값이라는 개념 자체가 의미 없는 함수가 일부 존재합니다. 예를 들어 테스트 라이브러리들은 `fail` 이라는 함수를 제공하는 경우가 많은데, `fail`은 특별한 메시지가 들어있는 예외를 던져서 현재 테스트를 실패시킵니다. 다른 예시로 무한 루프를 도는 함수도 결코 값을 리턴하며 정상적으로 끝나지 않습니다.  이런 경우를 표현하기 위해 코틀린에는 `Nothing` 이라는 특별한 리턴 타입이 존재합니다.

```kotlin
fun fail(message: String): Nothing {
  throw IllegalStateException(message)
}

fun fail("Error occurred")
// java.lang.IllegalStateException: Error occurred
```



`Nothing` 타입은 아무 값도 포함하지 않으며 함수의 리턴타입이나 리턴 타입으로 쓰일 타입 파라미터로만 사용할 수 있습니다. 그 외의 다른 용도로 사용하는 경우 `Nothing` 타입의 변수를 선언하더라도 그 변수에 아무 값도 저장할 수 없으므로 아무 의미도 없습니다.

`Nothing` 을 리턴하는 함수를 엘비스 연산자의 우항에 사용해서 전제 조건을 검사할 수 있습니다.

```kotlin
val address = company.address ?: fail("No address")
println(address.city)
```



## 컬렉션과 배열

우리는 코틀린 컬렉션이 자바 라이브러리를 바탕으로 만들어졌고 확장 함수를 통해 기능을 추가할 수 있다는 사실을 배웠습니다. 이제 코틀린의 컬렉션 지원과 자바와 코틀린 컬렉션 간에 관계에 대해 살펴보겠습니다.



### 널 가능성과 컬렉션

변수 타입뒤에 ?를 붙이며 그 변수에 널을 저장할 수 있다는 뜻인 것처럼 타입 인자로 쓰인 타입에도 같은 표시를 사용할 수 있습니다. 아래 예시를 보겠습니다.

```kotlin
val list1: ArrayList<Int?> = ArrayList<Int?>() // 리스트 원소의 타입은 Int 타입이다.
val list2: ArrayList<Int>? = ArrayList<Int>() // 리스트 원소의 타입은 Int 타입이고 리스트 자체가 null일 수도 있다.
val list3: ArrayList<Int?>? = ArrayList<Int?>() // 리스트 원소의 타입은 Int? 타입이고 리스트 자체도 null일 수도 있다.
```



널이 될 수 있는 값으로 이뤄진 컬렉션으로 널 값을 걸러내는 경우가 자주 있어서 코틀린 표준 API에선 `filterNotNull` 이라는 함수를 제공합니다.

``` kotlin
val nullableList: List<Int?> = listOf(1, 2, null)
val notNullableList: List<Int> = nullableList.filterNotNull()
```



### 읽기 전용과 변경 가능한 컬렉션

코틀린 컬렉션과 자바 컬렉션을 나누는 가장 중요한 특성 중 하나는 코틀린에선 컬렉션 안의 데이터에 접근하는 인터페이스와 컬렉션 안의 데이터를 변경하는 인터페이스를 분리했다는 점입니다. 코틀린의 컬렉션을 다룰 때 사용하는 가장 기초적인 인터페이스인 `kotlin.collections.Collection` 에는 원소에 대한 이터레이션, 컬렉션의 크기, 어떤 값이 들어있는지 검사, 데이터를 읽는 등의 연산을 수행하는 메소드들이 존재합니다. 하지만 추가하거나 제거하는 메소드는 존재하지 않습니다. 

컬렉션의 데이터를 수정하려면 `kotlin.collection.MutableCollection` 인터페이스를 사용해야 합니다. 해당 인터페이스는 `kotlin.collection.Collection` 을 확장하며 원소를 추가하거나, 삭제하거나, 모두 지우는 등의 메소드를 더 제공합니다. 

`val`과 `var` 의 구별과 마찬가지로 컬렉션의 읽기 전용 인터페이스와 변경 가능 인터페이스를 구별한 이유는 프로그램에서 데이터에 어떤 일이 벌어지는지를 더 쉽게 이해하기 위함입니다. 어떤 메소드의 인자로 `MutableCollection` 이 타입으로 주어진다면 해당 메소드 내부에서 해당 인자의 원소를 변경할 수도 있습니다. 그러므로 `MutableCollection`을 사용할 경우 원소가 어디서든 변경될 수 있다는 점에 유의해야 합니다.

물론 읽기 전용 컬렉션을 사용한다고 무조건 안전하진 않습니다. 아래와 같은 코드 처럼 같은 객체를 읽기 전용 / 변경 가능 컬렉션에서 참조하고, 읽기 전용 컬렉션에서 해당 객체를 사용할 때 병렬적으로 변경 가능 컬렉션에서 객체에 대한 수정이 이루어진다면 `ConcurrentModificationException` 이나 다른 예외가 발생할 수 있습니다. 

```kotlin
val source = listOf(1, 2, 3)
val target1: List<Int> = source 
val target2: MutableList<Int> = source
```



### 배열

코틀린에서는 Array 타입으로 배열을 정의합니다.

```kotlin
 fun main(args: Array<String>) { ... }
```



배열의 생성 방법은 아래 코드와 같습니다.

```kotlin
val num: Array = arrayOf﴾1, 2, 3, 4﴿
val nulls = arrayOfNulls﴾10﴿ // Array<String?> 
val nulls2 = Array﴾20﴿ { i ‐> ﴾i + 1﴿.toString﴾﴿ }
```

그렇다면 원시 타입 배열은 어떻게 정의할까요? 아래 절에서 확인해보겠습니다.



### 원시 타입 배열

코틀린에선 원시 타입은 `IntArray` , `CharArray`, 등 원시 타입 전용 배열을 사용하여 배열로 나타낼 수 있으며, 자바 원시 타입 배열로 컴파일 되어 성능에서 이점을 취합니다. 배열 생성 방법은 다양하며 다음과 같은 방법들이 존재합니다.

```kotlin
IntArray(5) // ?
intArrayOf(1, 2, 3, 4) // ?
IntArray(5) { i -> i * i } // ?
```

또 `Array`와 같은 타입은 `toIntArray` 등의 함수를 사용하여 `IntArray`로 변환이 가능합니다.











