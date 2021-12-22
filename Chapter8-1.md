# 08. 고차 함수: 파라미터와 반환 값으로 람다 사용

## 고차 함수 정의

- 고차 함수로 코드를 간결하게 다듬고 코드 중복을 없애고 더 나은 추상화를 구축하는 방법
- 람다를 사용함에 따라 발생할 수 있는 성능상 부가 비용을 없애고 람다 안에서 더 유연하게 흐름을 제어할 수 있는 **인라인 함수**

### 함수 타입

- 고차 함수를 정의하려면 함수 타입에 대해 먼저 알아야 한다.
- 람다를 인자로 받는 함수를 정의하려면 먼저 람다 인자의 타입을 어떻게 선언할 수 있는지 알아야 한다.
    - 람다 인자의 타입 선언 방식 → 로컬 변수에 대입하는 경우
    
    ```kotlin
    // 컴파일러가 타입을 추론한다.
    val sum = {x: Int, y: Int -> x + y}
    val action = { println(42) }
    
    // 각 변수에 구체적인 타입 선언 추가
    val sum: (Int, Int) -> Int = { x, y -> x + y }
    val action: () -> Unit = { println(42) }
    ```
    
    ![스크린샷 2021-12-20 오후 6.53.39.png](08%20%E1%84%80%E1%85%A9%E1%84%8E%E1%85%A1%20%E1%84%92%E1%85%A1%E1%86%B7%E1%84%89%E1%85%AE%20%E1%84%91%E1%85%A1%E1%84%85%E1%85%A1%E1%84%86%E1%85%B5%E1%84%90%E1%85%A5%E1%84%8B%E1%85%AA%20%E1%84%87%E1%85%A1%E1%86%AB%E1%84%92%E1%85%AA%E1%86%AB%20%E1%84%80%E1%85%A1%E1%86%B9%E1%84%8B%E1%85%B3%E1%84%85%E1%85%A9%20%E1%84%85%E1%85%A1%E1%86%B7%E1%84%83%E1%85%A1%20%E1%84%89%E1%85%A1%E1%84%8B%E1%85%AD%203f1dbcee8de146e1bd8ec492a3818284/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA_2021-12-20_%E1%84%8B%E1%85%A9%E1%84%92%E1%85%AE_6.53.39.png)
    
- 함수 타입에도 반환 타입을 널이 될 수 있는 타입으로 지정할 수 있다.
    
    ```kotlin
    var canReturnNull: (Int, Int) -> Int? = {x,y -> null}
    ```
    
- 널이 될 수 있는 함수 타입 변수를 정의할 수도 있다.
    
    ```kotlin
    var funOrNull: ((Int, Int) -> Int)? = null
    ```
    
- 함수 타입에서 파라미터 이름을 지정할 수도 있다.
    
    ```kotlin
    fun performRequest(
    	url: Stirng,
    	callback: (code: Int, content: String) -> Unit
    ) {}
    ```
    

### 인자로 받은 함수 호출

**고차 함수 구현 방법**

`리스트 8.1 간단한 고차 함수 정의하기`

```kotlin
fun twoAndThree(operation: (Int, Int) -> Int) {
	val result = operation(2, 3)
	println("The result is $result")
}

>>> twoAndThree { a, b -> a + b }
The result is 5
>>> twoAndThree { a, b -> a * b }
The result is 6
```

`그림 8.2 고차 함수를 파라미터로 받는 filter 함수 정의`

![스크린샷 2021-12-20 오후 7.07.48.png](08%20%E1%84%80%E1%85%A9%E1%84%8E%E1%85%A1%20%E1%84%92%E1%85%A1%E1%86%B7%E1%84%89%E1%85%AE%20%E1%84%91%E1%85%A1%E1%84%85%E1%85%A1%E1%84%86%E1%85%B5%E1%84%90%E1%85%A5%E1%84%8B%E1%85%AA%20%E1%84%87%E1%85%A1%E1%86%AB%E1%84%92%E1%85%AA%E1%86%AB%20%E1%84%80%E1%85%A1%E1%86%B9%E1%84%8B%E1%85%B3%E1%84%85%E1%85%A9%20%E1%84%85%E1%85%A1%E1%86%B7%E1%84%83%E1%85%A1%20%E1%84%89%E1%85%A1%E1%84%8B%E1%85%AD%203f1dbcee8de146e1bd8ec492a3818284/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA_2021-12-20_%E1%84%8B%E1%85%A9%E1%84%92%E1%85%AE_7.07.48.png)

`리스트 8.2 filter 함수를 단순하게 만든 버전 구현하기`

```kotlin
fun String.filter(predicate: (Char) -> Boolean): String {
	val sb = StringBuilder()
	for (index in 0 until length) {
		val element = get(index)
		if (predicate(element)) sb.append(element)
	}
	return sb.toString()
}

>>> println("ab1c".filter {it in 'a'..'z'})
```

### 자바에서 코틀린 함수 타입 사용

- 컴파일된 코드 안에서 함수 타입은 일반 인터페이스로 바뀐다. 즉 함수 타입의 변수는 FunctionN 인터페이스를 구현하는 객체를 저장한다.
- 함수 인자의 개수에 따라 `Function0<R>`(인자가 없는 함수), `Function1<P1, R>`(인자가 하나인 함수) 등의 인터페이스를 제공 → 각 인터페이스의 invoke를 호출하여 함수를 실행할 수 있다.
- 함수 타입인 변수는 인자 개수에 따라 적당한 FunctionN 인터페이스를 구현하는 클래스의 인스턴스를 저장하며, 그 클래스의 invoke 메소드 본문에는 람다의 본문이 들어간다.
- 자바 8 람다를 넘기면 자동으로 함수 타입의 값으로 반환된다.
    
    ```kotlin
    // 코틀린 선언
    fun processTheAnswer(f: (Int) -> Int) {
    	println(f(42))
    }
    
    // 자바 코드에서 호출
    >>> processTheAnswer(number -> number + 1);
    43
    ```
    
- 자바 8이전 에서는 필요한 FunctionN 인터페이스의 invoke 메소드를 구현하는 익명 클래스를 넘기면 된다.
    
    ```kotlin
    processTheAnswer(
    	new Function1<Integer, Integer>() {
    		@Override
    		public Integer invoke(Integer number) {
    			System.out.println(number);
    			return number + 1;
    		}
    	});
    ```
    

### 디폴트 값을 지정한 함수 타입 파라미터나 널이 될 수 있는 함수 타입 파라미터

- 함수 타입의 파라미터에 대한 디폴트 값을 지정할 수 있다.
    
    `리스트 8.4 함수 타입의 파라미터에 대한 디폴트 값 지정하기`
    
    ```kotlin
    fun <T> Collection<T>.joinToString(
        separator: String = ", ",
        prefix: String = "",
        postfix: String = "",
        // 함수 타입 파라미터를 선언하면서 람다를 디폴트 값으로 지정한다.
        transform: (T) -> String = { it.toString() }
    ): String {
        val result = StringBuilder(prefix)
        for ((index, element) in this.withIndex()) {
            if (index > 0) result.append(separator)
            result.append(transform(element))
        }
        result.append(postfix)
        return result.toString()
    }
    >>> println(letters.joinToString { it.toLowerCase() })
    alpha, beta
    >>> println(letters.joinToString(separator = "! ", postfix = "! ", transform = { it.toUpperCase() }))
    ALPHA! BETA!
    ```
    
- 널이 될 수 있는 함수 타입을 사용할 수도 있다.
    
    `리스트 8.5 널이 될 수 있는 함수 타입 파라미터를 사용하기`
    
    ```kotlin
    fun <T> Collection<T>.joinToString(
        separator: String = ", ",
        prefix: String = "",
        postfix: String = "",
        transform: ((T) -> String)? = null
    ): String {
        val result = StringBuilder(prefix)
        for ((index, element) in this.withIndex()) {
            if (index > 0) result.append(separator)
            val str = transform?.invoke(element) **?:** element.toString()
            result.append(str)
        }
        result.append(postfix)
        return result.toString()
    }
    ```
    

⇒ 지금까지의 내용은 함수를 인자로 받는 함수를 만드는 방법에 대한 설명

### 함수를 함수에서 반환

- 프로그램의 상태나 조건에 따라 달라질 수 있는 로직이 있을 경우
- 함수가 함수를 반환할 필요가 있는 경우보다는 함수가 함수를 인자로 받아야할 필요가 있는 경우가 훨씬 많으나 함수를 반환하는 함수도 유용하다.
    
    `리스트 8.6 함수를 반환하는 함수 정의하기`
    
    ```kotlin
    enum class Delivery { STANDARD, EXPEDITED }
    class Order(val itemCount: Int)
    
    // 반환 타입에 고차 함수 정의
    fun getShippingCostCalculator(
        delivery: Delivery
    ): (Order) -> Double {
        if (delivery == Delivery.EXPEDITED) {
            return { order -> 6 + 2.1 * order.itemCount }
        }
        return { order -> 1.2 * order.itemCount }
    }
    
    >>> calculator = getShippingCostCalculator(Delivery.EXPEDITED)
    >>> println("Shipping costs ${calculator(Order(3))}")
    Shipping costs 12.3
    
    >>> calculator = getShippingCostCalculator(Delivery.STANDARD)
    >>> println("Shipping costs ${calculator(Order(3))}")
    Shipping costs 3.5999999999999996
    ```
    
    - 함수를 반환하려면 return 식에 람다나 멤버 참조나 함수 타입의 값을 계산하는 식 등을 넣으면 된다.

### 람다를 활용한 중복 제거

- 함수 타입과 람다 식은 재활용하기 좋은 코드를 만들 때 쓸 수 있는 훌륭한 도구다.
- 람다를 사용할 수 없는 환경에서는 아주 복잡한 구조를 만들어야만 피할 수 있는 코드 중복도 람다를 활용하면 간결하고 쉽게 제거할 수 있다.

`리스트 8.8 사이트 방문 데이터 정의`

`리스트 8.9 사이트 방문 데이터를 하드 코딩한 필터를 사용해 분석하기`

```kotlin
data class SiteVisit(
    val path: String,
    val duration: Double,
    val os: OS
)

enum class OS { WINDOWS, LINUX, MAC, IOS, ANDROID }

val log = listOf(
    SiteVisit("/", 34.0, OS.WINDOWS),
    SiteVisit("/", 22.0, OS.MAC),
    SiteVisit("/login", 12.0, OS.WINDOWS),
    SiteVisit("/signup", 8.0, OS.IOS),
    SiteVisit("/", 16.3, OS.ANDROID)
)

fun main() {
    val averageWindowsDuration = log
        .filter { it.os == OS.WINDOWS }
        .map(SiteVisit::duration)
        .average()
    println(averageWindowsDuration)

    val averageWindowsDuration = log
        .filter { it.os == OS.MAC }
        .map(SiteVisit::duration)
        .average()
    println(averageWindowsDuration)
}

23.0

// 만약 MAC의 평균 방문 시간을 출력하고 싶다면 -> 동일한 로직 중복
```

`리스트 8.10 일반 함수를 통해 중복 제거하기`

```kotlin
fun List<SiteVisit>.averageDurationFor(os: OS) =
    filter {it.os == os}.map(SiteVisit::duration).average()

>>> println(log.averageDurationFor(OS.WINDOWS))
23.0
>>> println(log.averageDurationFor(OS.MAC))
22.0
```

- 모바일 사용자의 평균 방문 시간 구하기

`리스트 8.11 복잡하게 하드코딩한 필터를 사용해 방문 데이터 분석하기`

```kotlin
val averageMobileDuration = log
        .filter { it.os in setOf(OS.ANDROID, OS.IOS) }
        .map(SiteVisit::duration)
        .average()

println(averageMobileDuration)
```

- 함수 타입을 사용하면 필요한 조건을 파라미터로 뽑아낼 수 있다.

`리스트 8.12 고차 함수를 사용해 중복 제거하기`

```kotlin
// 특정 조건을 통과하는 고차함수를 매개변수로 받음
fun List<SiteVisit>.averageDurationFor(predicate: (SiteVisit) -> Boolean) =
    filter(predicate).map(SiteVisit::duration).average()

println(log.averageDurationFor { it.os in setOf(OS.ANDROID, OS.IOS) })
```

## 인라인 함수: 람다의 부가 비용 없애기

- 코틀린에서는 보통 람다를 익명 클래스로 컴파일하지만 그렇다고 람다 식을 사용할 때마다 새로운 클래스가 만들어지지는 않는다.
- 하지만 람다가 변수를 포획하면 람다가 생성되는 시점마다 새로운 익명 클래스 객체가 생긴다. 이 경우 실행 시점에 익명 클래스 생성에 따른 부가 비용이 든다.
- 따라서 람다를 사용하는 구현은 똑같은 작업을 수행하는 일반 함수를 사용한 구현보다 덜 효율적이다.
- inline 변경자를 함수에 붙이면 컴파일러는 그 함수를 호출하는 모든 문장을 함수 본문에 해당하는 바이트코드로 바꿔치기 해준다.

### 인라이닝이 작동하는 방식

- inline으로 선언하면 그 함수의 본문이 인라인된다. 다른 말로 하면 함수를 호출하는 코드를 함수를 호출하는 바이트코드 대신에 함수 본문을 번역한 바이트 코드로 컴파일한다.

```kotlin
inline fun calculator(a: Int, b: Int, op: (Int, Int) -> Int): Int {
    println("calculator body")
    return op(a, b)
}

fun main() {
    val result = calculator(1, 2) { a, b -> a + b }
    println(result)
}
```

### 인라인 함수의 한계

- 인라이닝을 하는 방식으로 인해 람다를 사용하는 모든 함수를 인라이닝할 수는 없다.
- 함수가 인라이닝될 때 그 함수에 인자로 전달된 람다 식의 본문은 결과 코드에 직접 들어갈 수 있다. 하지만 이렇게 람다가 본문에 직접 펼쳐지기 때문에 함수가 파라미터로 전달받은 람다를 본문에 사용하는 방식이 한정될 수밖에 없다.
- 함수 본문에서 파라미터로 받은 람다를 호출한다면 쉽게 람다 본문으로 바꿀 수 있다. 하지만 파라미터로 받은 람다를 다른 변수에 저장하고 나중에 그 변수를 사용한다면 람다를 표현하는 객체가 어딘가는 존재해야 하기 때문에 람다를 인라이닝할 수 없다.
- 이 경우 컴파일러는 `“Illegal usage of inline-parameter”` 라는 메시지와 함께 인라이닝을 금지시킨다.

`example`

```kotlin
fun <T, R> Sequence<T>.map(transform: (T) -> R): Sequence<R> {
    return TransformingSequence(this, transform)
}
```

### 컬렉션 연산 인라이닝

- kotlin의 filter 함수는 inline 함수다. 따라서 filter 함수의 바이트코드는 그 함수에 전달된 람다 본문의 바이트코드와 함께 filter를 호출한 위치에 들어간다.

```kotlin
data class Person(val name: String, val age: Int)

fun main() {
    val people = listOf(Person("Alice", 29), Person("Bob", 31))
    println(people.filter { it.age < 30 })

    val result = mutableListOf<Person>()
    for (person in people) {
        if (person.age < 30) result.add(person)
    }
    println(result)
}
```

- filter와 map은 인라인 함수다. 따라서 추가 객체나 클래스 생성은 없다.
- 하지만 리스트를 걸러낸 결과를 저장하는 중간 리스트를 만든다. filter 함수에서 만들어진 코드는 원소를 중간 리스트에 저장하고 map 함수에서 만들어진 코드는 그 중간 리스트를 읽어서 사용한다.
- asSequence를 사용하면 중간 리스트로 인한 부가 비용은 줄어든다. 중간 시퀀스는 람다를 필드에 저장하는 객체로 표현되며, 최종 연산은 중간 시퀀스에 있는 여러 람다를 연쇄 호출한다. 시퀀스 연산에는 람다가 인라이닝되지 않기 때문에 크기가 작은 컬렉션은 오히려 일반 컬렉션보다 성능이 안좋을 수 있다. 시퀀스를 통해 성능을 향상시킬 수 있는 경우는 컬렉션 크기가 큰 경우뿐이다.

```kotlin
println(people.filter { it.age > 30 }.map(Person::name))
```

### 함수를 인라인으로 선언해야 하는 이유

- 람다를 인자로 받는 함수만 성능이 좋아질 가능성이 높기 때문에 다른 경우에는 주의 깊에 성능을 측정하고 조사해봐야 한다.

**일반 함수 호출**

- 일반 함수 호출의 경우 JVM은 이미 강력하게 인라이닝을 지원한다. JVM은 코드 실행을 분석해서 가장 이익이 되는 방향으로 호출을 인라이닝한다. 이런 과정은 바이트코드를 실제 기계어 코드로 번역하는 과정(JIT)에서 일어난다.
- JVM 최적화를 활용한다면 바이트코드에서는 각 함수 구현이 한 번만 있으면 되고 함수 호출 부분에서 따로 중복 코드가 필요 없다. 반면 인라인 함수는 바이트 코드에서 각 함수 호출 지점을 함수 본문으로 대치하기 때문에 코드 중복이 생긴다.

**람다를 인자로 받는 함수 호출**

- 함수 호출 비용을 줄일 수 있을 뿐 아니라 클래스와 객체를 만들 필요가 없어진다.
- 현재의 JVM은 함수 호출과 람다를 인라이닝해 줄 정도로 똑똑하지 못하다.

inline 변경자를 붙일 때는 코드 크기에 주의를 기울여야 한다. 인라이닝하는 함수가 큰 경우 본문에 해당하는 바이트코드를 모든 호출 지점에 복사해 바이트코드가 전체적으로 아주 커질 수 있다. 그런 경우 람다 인자와 무관한 코드는 비인라인 함수로 빼는 것이 좋다.