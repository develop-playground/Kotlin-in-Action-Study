# 07. 연산자 오버로딩과 기타 관계

자바에는 표준 라이브러리와 연관된 언어 기능이 몇 가지 있다.

예로 for ... in 루프에 Iterable을 구현한 객체를 사용하거나, 

try-with-resource 문에 Autoclosable을 구현한 객체를 사용할 수 있다.

코틀린에서는 이런 언어 기능이 어떤 타입과 연괸되기 보다는 함수와 연결된다.

예를 들어 어떤 클래스 안에 `plus`라는 이름의 메서드를 정의하면 그 클래스의 인스턴스에 대해 `+` 연산자를 사용할 수 있다. 

이런 식으로 어떤 언어 기능과 미리 정해진 이름의 함수를 연결해주는 기법을 코틀린에서 관례(convention)라고 부른다. 

이러한 관례들에 대해 알아보도록 하자.

# 1. 산술 연산자 오버로딩

자바에서는 원시 타입과 String에 대해서만 산술 연산자를 사용할 수 있었다.

하지만 산술 연산자가 유용한 경우가 많다.

예를 들어 BigInteger 클래스를 `add`로 더하는 것보단 `+` 연산을 사용하는 것이 나을 것이다.

이런 산술 연산자를 어떻게 정의할 수 있는지 보자.  
  

## 1.1 이항 산술 연산 오버로딩

```kotlin
data class Point(val x: Int, val y: Int) {
  **operator** fun plus(other: Point): Point {
    return Point(x + other.x, y + other.y)
  }
}

>>> val p1 = Point(10, 20)
>>> val p2 = Point(30, 40)
>>> println(p1 + p2)
Point(x=40, y=60)
```

연산자를 오버로딩하는 함수 앞에는 꼭 `operator` 키워드를 붙여야 한다.

이를 통해 어떤 함수가 관례를 따르는 함수임을 명확히 할 수 있다.

내부적으론 다음과 같이 호출된다.

![Untitled](https://s3.us-west-2.amazonaws.com/secure.notion-static.com/1ab84459-eb37-4f42-8311-07ee60c5dbca/Untitled.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Content-Sha256=UNSIGNED-PAYLOAD&X-Amz-Credential=AKIAT73L2G45EIPT3X45%2F20211213%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Date=20211213T044715Z&X-Amz-Expires=86400&X-Amz-Signature=4de934521e06d494ee1b6bc981192f5813a0e3b92d46f1de2647dc0e0c28e734&X-Amz-SignedHeaders=host&response-content-disposition=filename%20%3D%22Untitled.png%22&x-id=GetObject)

연산자를 확장 함수로 정의할 수도 있다.

```kotlin
operator fun Point.plus(other: Point): Point {
  return Point(x + other.x, y + other.y)
}
```

다음은 코틀린에서 정의할 수 있는 이항 연산자와 연산자의 함수 이름이다. 

![Untitled](https://s3.us-west-2.amazonaws.com/secure.notion-static.com/f1f9de1e-710c-466e-b400-b97660b79c39/Untitled.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Content-Sha256=UNSIGNED-PAYLOAD&X-Amz-Credential=AKIAT73L2G45EIPT3X45%2F20211213%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Date=20211213T044822Z&X-Amz-Expires=86400&X-Amz-Signature=428bf0a2dac0798c858d4ea87a9bc39f5bfb6b294de18e38f589b9853224437a&X-Amz-SignedHeaders=host&response-content-disposition=filename%20%3D%22Untitled.png%22&x-id=GetObject)

연산자를 직접 정의한 함수를 통해 구현하더라도 연산자 우선순위는 언제나 숫자 타입에 대한 연산자 우선순위와 같다.

예를 들어 a + b * c라는 식에선 언제나 곱셈이 덧셈보다 먼저 수행된다.

연산자를 정의할 때 두 피연산자가 같은 타입일 필요는 없다.

```kotlin
operator fun Point.times(**scale: Double**): Point {
  return Point((x * scale).toInt(), (y * scale).toInt())
}

>>> val p = Point(10, 20)
>>> println(p * 1.5)
Point(x=15, y=30)
```

두 피연산자의 타입이 다를 때, 코틀린 연산자가 자동으로 교환 법칙을 지원하지 않는다는 것을 명심해야 한다.

예를 들어 p * 1.5가 아닌 1.5 * p라고도 쓸 수 있어야 한다면

```kotlin
operator fun Double.times(p: Point): Point
```

와 같은 연산자 함수를 정의해주어야 한다.

또한 연산자 함수의 반환 타입이 꼭 두 피연산자 중 하나와 일치해야만 하는 것도 아니다.

```kotlin
operator fun Char.times(count: Int): String {
  return toString().repeat(count)
}

>>> print('a' * 3)
aaa
```

## 1.2 복합 대입 연산자 오버로딩

`plus`와 같은 연산자를 오버로딩하면 코틀린은 `+` 연산자뿐 아니라 그와 관련 있는 연산자인 `+=`도 자동으로 함께 지원한다.

`+=`, `-=` 등의 연산자는 복합 대입(compound assignment) 연산자라 불린다.

```kotlin
>>> var point = Point(1, 2)
>>> point += Point(3, 4)
>>> println(point)
Point(x=4, y=6)
```

`point += Point(3, 4)`는 `point = point + Point(3, 4)`라고 쓴 식과 같다. 

물론 변수가 변경 가능한 경우에만 복합 대입 연산자를 사용할 수 있다.

경우에 따라 `+=` 연산이 객체에 대한 참조를 다른 참조로 바꾸기 보다 원래 객체의 내부 상태를 변경해야 하는 경우도 있다.

이런 경우 반환 타입이 `Unit`인 `plusAssign` 함수를 정의하면 코틀린은 `+=` 연산자에 그 함수를 사용한다.

다른 복압 대입 연산자 함수도 비슷하게 `minusAssign`, `timesAssign` 등의 이름을 사용한다.

코틀린 표준 라이브러리는 변경 가능한 컬렉션에 대해 이 함수들을 정의하고 있다.

```kotlin
operator fun <T> MutableCollection<T>.plusAssign(element: T) {
  this.add(element)
}
```

하지만 어떤 클래스가 `plus`와 `plusAssign` 함수 모드를 정의하고 `+=`를 사용하는 경우 컴파일러는 오류를 발생시킨다.

물론 복합 대입 연산자 대신 일반 연산자를 사용하면 되지만, 클래스 설계의 일관성이 떨어진다.

예를 들어, Point처럼 변경이 불가능하다면 새로운 값을 반환하는 연산만을 추가해야 한다.

따라서 둘 중 하나의 방법으로만 설계를 하는 것이 좋다.

지금까지 이항(binary) 연산자에 대해 설명했으니, `-a`와 같은 단항(unary) 연산자를 알아보자. 

## 1.3 단항 연산자 오버로딩

단항 연산자를 오버로딩하는 절차도 이항 연산자와 같다.

```kotlin
operator fun Point.unaryMinus(): Point {
  return Point(-x, -y)
}

>>> val p = Point(10, 20)
>>> println(-p)
Point(-10, -20)
```

![Untitled](https://s3.us-west-2.amazonaws.com/secure.notion-static.com/75626178-7b8a-4603-bb3f-400c09b2a301/Untitled.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Content-Sha256=UNSIGNED-PAYLOAD&X-Amz-Credential=AKIAT73L2G45EIPT3X45%2F20211213%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Date=20211213T044925Z&X-Amz-Expires=86400&X-Amz-Signature=8e12aaff1da823141905f521ec83cf258e47cb43a3cb61bb4f5b75d969249ffd&X-Amz-SignedHeaders=host&response-content-disposition=filename%20%3D%22Untitled.png%22&x-id=GetObject)

코틀린에서 오버로딩할 수 있는 모든 단항 연산자를 보자.

![Untitled](
https://s3.us-west-2.amazonaws.com/secure.notion-static.com/a264ace5-e322-40ad-8110-9ddc3f61864a/Untitled.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Content-Sha256=UNSIGNED-PAYLOAD&X-Amz-Credential=AKIAT73L2G45EIPT3X45%2F20211213%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Date=20211213T044942Z&X-Amz-Expires=86400&X-Amz-Signature=b5848859f1408e832e6393ceb084b2a8b5dac7e613a0c5cf3e14aba5c41046f3&X-Amz-SignedHeaders=host&response-content-disposition=filename%20%3D%22Untitled.png%22&x-id=GetObject)

inc나 dec 함수를 정의해 증감 연산자를 오버로딩하는 경우 컴파일러는 일반적인 전위와 후위 증감 연산자와 같은 의미를 제공한다.

```kotlin
operator fun BigDecimal.inc() = this + BigDecimal.ONE

>>> var bd = BigDecimal.ZERO

>>> println(bd++)
0
>>> println(++bd)
2
```

후위 증가 연산자는 println이 실행된 다음에 값을 증가시킨다.

전위 증가 연산자는 println이 실행되기 전에 값을 증가시킨다.

이처럼 별다른 처리를 해주지 않아도 제대로 증감 연산자가 작동한다.

# 2. 비교 연산자 오버로딩

`equals`나 `compareTo`를 호출해야 하는 자바와 달리 코틀린에서는 `==` 비교 연산자를 직접 사용할 수 있어서 코드가 더 간결하며 이해하기 쉽다. 

## 2.1 동등성 연산자: equals

코틀린은 `==` 연산잘 호출을 `equals` 메서드 호출로 컴파일한다는 것을 이미 배웠다.

이것도 사실은 이전에 살펴봤던 경우와 동일하다.

`==`와 `≠`는 내부에서 인자가 null인지 검사하므로 다른 연산과 달리 null이 될 수 있는 값에도 사용할 수 있다.

![Untitled](https://s3.us-west-2.amazonaws.com/secure.notion-static.com/1341beff-3ab2-448c-b49d-797f7300b15a/Untitled.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Content-Sha256=UNSIGNED-PAYLOAD&X-Amz-Credential=AKIAT73L2G45EIPT3X45%2F20211213%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Date=20211213T045019Z&X-Amz-Expires=86400&X-Amz-Signature=8922a0d2bd130efc5b64828b04f2cfff607b83cf0ba0d22e27f1d04ab0bf3a16&X-Amz-SignedHeaders=host&response-content-disposition=filename%20%3D%22Untitled.png%22&x-id=GetObject)

`a == b`라는 비교를 처리할 때 `a`가 null인지 판단해서 null이 아닌 경우에만 `equals`를 호출한다.

`a`가 null이라면 `b`도 null인 경우에만 결과가 `true`이다.

`equals`는 다른 연산자 오버로딩 관례와 달리 `Any`에 정의된 메서드이므로 `operator` 대신 `override`가 필요하다. 

`Any`의 `equals`에 `operator`가 붙어있다.

이처럼 상위 클래스의 메서드에 `operator`가 붙어있다면 하위 클래스의 메서드에서 붙이지 않아도 적용된다.

## 2.2 순서비교 연산자: compareTo

자바에서 정렬이나 최댓값, 최솟값 등 값을 비교해야 하는 알고리즘에 사용할 클래스는 Comparable 인터페이스를 구현해야 한다.

Comparable 인터페이스에 들어있는 `compareTo` 메서드는 한 객체와 다른 객체의 크기를 비교해 정수로 나타내준다.

자바에서는 이 메서드를 짧게 호출할 수 있는 방법이 없지만 코틀린은 Comparable 인터페이스 안에 있는 `compareTo` 메서드를 호출하는 관례를 제공한다.

![두 객체를 비교하는 식은 compareTo의 결과를 0과 비교하는 코드로 컴파일된다.](https://s3.us-west-2.amazonaws.com/secure.notion-static.com/fe9d7629-f7bf-4dd1-92b1-986a807150ae/Untitled.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Content-Sha256=UNSIGNED-PAYLOAD&X-Amz-Credential=AKIAT73L2G45EIPT3X45%2F20211213%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Date=20211213T045043Z&X-Amz-Expires=86400&X-Amz-Signature=6390ea1be36d47bf93459024fc7219b140240b94402077e5615023eefa18412f&X-Amz-SignedHeaders=host&response-content-disposition=filename%20%3D%22Untitled.png%22&x-id=GetObject)

두 객체를 비교하는 식은 compareTo의 결과를 0과 비교하는 코드로 컴파일된다.

따라서 위처럼 비교 연산자(`<`, `>`, `≤`, `≥`)는 `compareTo` 호출로 컴파일 된다.

예시를 통해 알아보자.

```kotlin
class Person {
  val firstName: Strimg, val lastName: String
) : Comparable<Person> {
  override fun compareTo(other: Person): Int {
		// 인자로 받은 함수를 차례로 호출하면서 값을 비교한다. (코틀린 표준 라이브러리임)
    return compareValueBy(this, other, Person::lastName, Person::firstName)  				
  }
}

>>> val p1 = Person("Alice", "Smith")
>>> val p2 = Person("Bob", "Johnson")
>>> println(p1 < p2)
false
```

이렇게 하면 코드는 간결해지지만 필드를 직접 비교하는 것이 훨씬 더 빠르다는 것을 명심하자.

언제나 처음에는 이해하기 쉽고 간결한 코드를 작성하고, 나중에 그 코드가 자주 호출됨에 따라 성능이 문제가 된다면 개선하자.

# 3. 컬렉션과 범위에 대해 쓸 수 있는 관례

컬렉션을 다룰 때 가장 많이 쓰는 인덱스 연산과 속해있는지 검사하는 연산을 연산자 구문으로 사용할 수 있다.

이런 연산을 지원하기 위해 어떤 관례를 사용하는지 알아보자.

## 3.1 인덱스로 원소에 접근: get과 set

코틀린에서 맵의 원소에 접근할 때나 자바에서 배열 원소에 접근할 때 일반적으로 `[]`를 사용한다. 

```kotlin
val value = map[key]
mutableMap[key] = newValue
```

코틀린에서는 인덱스 연산자도 관례를 따른다.

인덱스 연산자를 사용해 원소를 읽을 때는 `get` 연산자 메서드로 변환되고, 쓰는 연산은 `set` 연산자 메서드로로 변환된다.

Map과 MutableMap 인터페이스에는 그 두 메서드가 이미 들어있다.

예제를 통해 알아보자.

```kotlin
operator fun Point.get(index: Int): Int {
  return when(index) {
    0 -> x
    1 -> y
    else -> throw IndexOutOfBoundsException("Invalid coordinate $index")
  }
}

val p = Point(10, 20)
>>> println(p[1])
20
```

`get`이라는 메서드를 만들고 `operator` 변경자를 붙이기만 하면 된다.

그러면 다음처럼 변환된다.

![Untitled](https://s3.us-west-2.amazonaws.com/secure.notion-static.com/fa365b9f-78f3-415d-a104-fbf7af3ef749/Untitled.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Content-Sha256=UNSIGNED-PAYLOAD&X-Amz-Credential=AKIAT73L2G45EIPT3X45%2F20211213%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Date=20211213T045059Z&X-Amz-Expires=86400&X-Amz-Signature=f9796654478b0f504483ac955d1d463d5157ea82bff0ec9959de3dcf7598ab68&X-Amz-SignedHeaders=host&response-content-disposition=filename%20%3D%22Untitled.png%22&x-id=GetObject)

`get` 메서드의 파라미터로 Int가 아닌 타입도 사용할 수 있다.

예를 들어 맵 인덱스 연산의 경우 파라미터 타입이 맵의 키 타입과 같은 임의의 타입이 될 수 있다.

또한 여러 파라미터를 사용하는 get을 정의할 수도 있는데, `operator fun get(rowIndex: Int, colIndex: Int)`를 `matrix[row, col]`로 호출할 수 있다.

컬렉션 클래스가 다양한 키 타입을 지원해야 한다면 오버로딩한 `get` 메서드를 여럿 정의할 수도 있다.

이제 `set`을 알아보자. (Point는 불변 클래스이므로 set이 의미가 없다.)

```kotlin
data class MutablePoint(var x: Int, var y: Int)

operator fun MutablePoint.set(index: Int, value: Int) {
  when(index) {
    0 -> x = value
	  1 -> y = value
    else -> throw IndexOutOfBoundsException("Invalid coordinate $index")
  }
}

>>> val p = MutablePoint(10, 20)
>>> p[1] = 42
>>> println(p)
MutablePoint(x=10, y=42)
```

![각괄호([])를 사용한 대입문은 set 함수 호출로 컴파일된다.](https://s3.us-west-2.amazonaws.com/secure.notion-static.com/e6f42450-fe88-4c9a-b927-31b536b2b1b9/Untitled.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Content-Sha256=UNSIGNED-PAYLOAD&X-Amz-Credential=AKIAT73L2G45EIPT3X45%2F20211213%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Date=20211213T045112Z&X-Amz-Expires=86400&X-Amz-Signature=88f7cbb0b1e488a4ff09db8695de16b1201043d9fe0996623efa9c939c20aa97&X-Amz-SignedHeaders=host&response-content-disposition=filename%20%3D%22Untitled.png%22&x-id=GetObject)

각괄호([])를 사용한 대입문은 set 함수 호출로 컴파일된다.

## 3.2 in 관례

컬렉션이 지원하는 다른 연산자로 객체가 컬렉션에 들어가 있는지 검사(멤버십 검사)하는  `in`이 있다.

이런 경우 `in` 연산자와 대응하는 함수는 `contains`다.

예제를 통해 알아보자.

```kotlin
data class Rectangle(val upperLeft: Point, val lowerRight: Point)

operator fun Rectangle.contains(p: Point): Boolean {
  return p.x in upperLeft.x until lowerRight.x &&   // 범위를 만들고 x 좌표가 그 범위 안에 있는지 검사한다.
    p.y in upperLeft.y until lowerRight.y
}

>>> val rect = Rectangle(Point(10, 20), Point(50, 50))
>>> println(Point(20, 30) in rect)
true

>>> println(Point(5, 5) in rect)
false
```

`in`의 우항에 있는 객체는 `contains` 메서드의 수신 객체가 되고, in의 좌항에 있는 객체는 메서드의 인자로 전달된다.

![Untitled](https://s3.us-west-2.amazonaws.com/secure.notion-static.com/bdd740a2-f9e1-4a54-bc02-a5f9e45c44cc/Untitled.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Content-Sha256=UNSIGNED-PAYLOAD&X-Amz-Credential=AKIAT73L2G45EIPT3X45%2F20211213%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Date=20211213T045129Z&X-Amz-Expires=86400&X-Amz-Signature=cfbc1bd0a70752bea1ea39094b66c2051c8b6ea657ec52621793c6af164942e4&X-Amz-SignedHeaders=host&response-content-disposition=filename%20%3D%22Untitled.png%22&x-id=GetObject)

## 3.3 rangeTo 관례

범위를 만들 때 `..` 구문을 사용할 수 있다. (`1..10`은 1부터 10까지 모든 수가 들어있는 범위이다.)

`..` 연산자는 `rangeTo` 함수를 간략하게 표현하는 방법이다.

![Untitled](https://s3.us-west-2.amazonaws.com/secure.notion-static.com/a2324fe8-315a-4705-b70d-402f3444d641/Untitled.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Content-Sha256=UNSIGNED-PAYLOAD&X-Amz-Credential=AKIAT73L2G45EIPT3X45%2F20211213%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Date=20211213T045139Z&X-Amz-Expires=86400&X-Amz-Signature=cfac18a93ea75ed481ccb1b904245fb6266dd4f37b6da2ae57d5bcb1b9f524cf&X-Amz-SignedHeaders=host&response-content-disposition=filename%20%3D%22Untitled.png%22&x-id=GetObject)

이 연산자를 아무 클래스에나 정의할 수 있지만 어떤 클래스가 Comparable 인터페이스를 구현하면 `rangeTo`를 정의할 필요가 없다.

코틀린 표준 라이브러리에는 모든 Comparable 객체에 대해 적용 가능한 `rangeTo` 함수가 들어있다.

```kotlin
operator fun <T: Comparable<T>> T.rangeTo(that: T): ClosedRange<T>
```

예를 들어 LocalDate 클래스를 사용해 날짜의 범위를 만들어보자.

```kotlin
>>> val now = LocalDate.now()
>>> val vacation = now..now.plusDays(10)
>>> println(now.plusWeeks(1) in vacation)
true
```

`rangeTo` 함수는 `LocalDate`의 멤버는 아니고, 앞에서 본대로 `Comparable`에 대한 확장 함수다.

`rangeTo` 연산자는 다른 산술 연산자보다 우선순위가 낮지만, 혼동을 피하기 위해 괄호로 인자를 감싸주는 것이 좋다.

```kotlin
>>> val n = 9
>>> println(0..(n + 1))  // == 0..n + 1
0..10

>>> (0..n).forEach { print(it) }  // 0..n.forEach 는 컴파일 불가
0123456789
```

## 3.4 for 루프를 위한 iterator 관례

코틀린의 for 루프는 범위 검사와 똑같이 `in` 연산자를 사용한다.

하지만 이 경우 in의 의미는 다르다. 

`for (x in list) { ... }` 와 같은 문장은 `list.iterator()`를 호출해서 이터레이터를 얻은 다음, 

자바와 마찬가지로 그 이터레이터에 대해 `hasNext`와 `next` 호출을 반복하는 식으로 변환된다. 

하지만 코틀린에선 이 또한 관례이므로 `iterator` 메서드를 확장 함수로 정의할 수 있다.

이런 성질로 인해 자바 문자열에 대한 `for` 루프가 가능하다.

코틀린 표준 라이브러리는 `String`의 상위 클래스인 `CharSequence`에 대한 `iterator` 확장 함수를 제공한다.

```kotlin
operator fun CharSequence.iterator(): CharIterator

>>> for (c in "abc") { ... }
```

클래스 안에 직접 iterator 메서드를 구현할 수도 있다.

```kotlin
operator fun ClosedRange<LocalDate>.iterator(): Iterator<LocalDate> =
  object : Iterator<LocalDate> {
    var current = start
    
    override fun hasNext() = current <= endInclusive
   
    override fun next() = current.apply { current = plusDays(1) }
```

코드에서 `ClosedRange<LocalDate>`에 대한 확장 함수 `iterator`를 정의했기 때문에 `LocalDate`의 범위 객체를 `for` 루프에 사용할 수 있다.
