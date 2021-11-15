# 05. 람다로 프로그래밍

[노션링크](https://wsminyoung.notion.site/05-f1cec23725a94c11810cb0e2b5e56d46)

### 5장에서 다루는 내용

• 람다 식과 멤버 참조
• 함수형 스타일로 컬렉션 다루기
• 시퀀스: 지연 컬랙션 연산
• 수신 객체 지정 람다 사용

람다 식(lambda expression) 또는 람다는 기본적으로 다른 함수에 넘길 수 있는 작은 코드 조각을 뜻한다.

람다를 사용하면 쉽게 공통 코드 구조를 라이브러리 함수로 뽑아낼 수 있다.

5장에서는 컬렉션을 처리하는 패턴을 표준 라이브러리 함수에 람다를 넘기는 방식으로 대치하는 예제를 다수 살펴본다.

마지막으로 수신 객체 지정 람다(lambda with receiver)에 대해 살펴본다. 수신 객체 지정 람다는 특별한 람다로, 람다 선언을 둘러싸고 있는 환경과는 다른 상황에서 람다 본문을 실행할 수 있다.

# 5.1 람다 식과 멤버 참조

이번 절에서는 람다의 유용성을 보여주고 코틀린 람다 식 구문이 어떻게 생겼는지 알아본다.

## 5.1.1 람다 소개: 코드 블록을 함수 인자로 넘기기

"이벤트가 발생하면 이 핸들러를 실행하자"나 "데이터 구조의 모든 원소에 이 연산을 적용하자"와 같은 생각을 코드로 표현하기 위해 일련의 동작을 변수에 저장하거나 다른 함수에 넘겨야 하는 경우가 자주 있다.

함수형 프로그래밍에서는 함수를 값처럼 다루는 접근 방법을 택함으로써 이 문제를 해결한다. 클래스를 선언하고 그 클래스의 인스턴스를 함수에 넘기는 대신 함수형 언어에서는 함수를 직접 다른 함수에 전달할 수 있다.

람다 식을 사용하면 코드가 더욱 더 간결해진다. 람다 식은 함수를 선언할 필요가 없고 코드 블록을 직접 함수의 인자로 전달 할 수 있다.

예를 들어 버튼 클릭에 따른 동작을 정의하고 싶다.

5.1 *무명 내부 클래스로 리스너 구현하기*

```java
/* 자바 */
button.setOnClickListener(new OnClickListener() {
  @Override
  public void onClick(View view) {
     /* 클릭 시 수행할 동작 */
  }
});
```

위 코드는 무명 내부 클래스를 선언하느라 코드가 번잡스럽다. 클릭 시 벌어질 동작을 간단히 기술할 수 있는 표기법이 있다면 이런 불필요한 코드를 제거할 수 있을 것이다.

5.2 *람다로 리스너 구현하기*

```kotlin
/* 코틀린 */
button.setOnClickListener { /* 클릭 시 수행할 동작 */ }
```

이 코틀린 코드는 앞에서 살펴본 자바 무명 내부 클래스와 같은 역할을 하지만 훨씬 더 간결하고 읽기 쉽다. 

## 5.1.2 람다와 컬렉션

코드에서 중복을 제거하는 것은 프로그래밍 스타일을 개선하는 중요한 방법 중 하나다.

컬렉션을 다룰 때 수행하는 대부분의 작업은 몇 가지 일반적인 패턴에 속한다. 따라서 그런 패턴은 라이브러리 안에 있어야 한다. 하지만 람다가 없다면 컬렉션을 편리하게 처리할 수 있는 좋은 라이브러리를 제공하기 힘들다.

그에 따라 자바에서 쓰기 편한 컬렉션 라이브러리가 적었으며, 그에 따라 자바 개발자들은 필요한 컬렉션 기능을 직접 작성하곤 했다.

코틀린에서는 이런 습관을 버려야 한다.

예제를 하나 살펴보자. 사람의 이름과 나이를 저장하는 Person 클래스를 사용한다.

```kotlin
data class Person(val name: String, val age: Int)
```

사람들로 이뤄진 리스트가 있고 그중에 가장 연장자를 찾고 싶다. 람다를 사용해본 경험이 없는 개발자라면 루프를 써서 직접 검색을 구현할 것이다.

*5.3 컬렉션을 직접 검색하기* 

```kotlin
fun findTheOldest(people: List<Person>) {
  var maxAge = 0  // 가장 많은 나이를 저장한다.
  var theOldest: Person? = null // 가장 연장자인 사람을 저장한다.
  for (person in people) {   
     if (person.age > maxAge) {  // 현재까지 발견한 최연장자보다 더 나이가 많은
        maxAge - person.age      // 사람을 찾으면 최댓값을 바꾼다.
        theOldest = person
  }
}

println(theOldest)

}

>>> val people = listOf(Person("Alice", 29), Person("Bob", 31))
>>> findTheOldest(people)
Person(name=Bob, age=31)
```

코틀린에서는 더 좋은 방법이 있다. 라이브러리 함수를 쓰면 된다.

*5.4 람다를 사용해 컬렉션 검색하기*

```kotlin
>>> val people = listOf(Person("Alice", 29), Person("Bob", 31))
>>> println(people.maxBy { it.age }) // 나이 프로퍼티를 비교해서 값이 가장 큰 원소 찾기
Person(name=Bob, age=31)
```

maxBy → 가장 큰 원소를 찾기 위해 비교에 사용할 값을 돌려주는 함수

{ it.age } → 바로 비교에 사용할 값을 돌려주는 함수 (it이 그 인자를 가리킨다.)

이 예제에서는 컬렉션의 원소가 Person 객체였으므로 이 함수가 반환하는 값은 Person 객체의 age 필드에 저장된 나이 정보다.

*5.5 멤버 참조를 사용해 컬렉션 검색하기*

```kotlin
people.maxBy(Person::age)
```

이 코드는 리스트 5.4와 같은일을 한다. 이에 대해 5.1.5절에서 더 자세히 다룬다.

## 5.1.3 람다 식의 문법

람다 식 문법

```kotlin
{ x: Int, y: Int -> x + y }
```

코틀린 람다 식은 항상 중괄호로 둘러싸여 있다. 화살표(→) 가 인자 목록과 람다 본문을 구분해준다.

람다 식은 변수에 저장할 수 있다. 람다가 저장된 변수를 다른 일반 함수와 마찬가지로 다룰 수 있다.

```kotlin
>>> val sum = { x: Int, y: Int -> x + y }
>>> println(sum(1, 2))
3
```

원한다면 람다 식을 직접 호출해도 된다.

```kotlin
>>> { println(42) } ()
42
```

하지만 이런 코드는 그다지 쓸모가 없다. 만약 이렇게 코드의 일부분을 블록으로 둘러싸 실행할 필요가 있다면 run을 사용한다. run은 인자로 받은 람다를 실행해주는 라이브러리 함수이다.

```kotlin
>>> run { println(42) } // 람다 본문에 있는 코드를 실행한다.
42
```

다시 사람 목록에서 가장 연장자를 찾은 예제인 5.4로 되돌아가자.

```kotlin
>>> val people = listOf(Person("Alice", 29), Person("Bob", 31))
>>> println(people.maxBy { it.age }) // 나이 프로퍼티를 비교해서 값이 가장 큰 원소 찾기
Person(name=Bob, age=31)
```

이 예제에서 코틀린이 코드를 줄여 쓸 수 있게 제공했던 기능을 제거하고 정식으로 람다를 작성하면 다음과 같다.

```kotlin
people.maxBy({p: Person -> p.age})
```

하지만 이 코드는 번잡하다. 우선 구분자가 너무 많이 쓰여서 가독성이 떨어진다. 그리고 컴파일러가 문맥으로부터 유추할 수 있는 인자 타입을 굳이 적을 필요는 없다. 마지막으로 인자가 단 하나뿐인 경우 인자에 이름을 붙이지 않아도 된다.

이런 개선을 적용해보자. 먼저 중괄호부터 시작하면 코틀린에는 함수 호출 시 맨 뒤에 있는 인자가 람다 식이라면 그 람다를 괄호 밖으로 빼낼 수 있다는 문법 관습이 있다. 이 예제에서는 람다가 유일한 인자이므로 마지막 인자이기도 하다. 따라서 괄호 뒤에 람다를 둘 수 있다.

```kotlin
people.maxBy() { p: Person -> p.age }
```

이 코드처럼 람다가 어떤 함수의 유일한 인자이고 괄호 뒤에 람다를 썼다면 호출 시 빈 괄호를 없애도 된다.

```kotlin
people.maxBy { p: Person -> p.age }
```

이제 구문을 더 간단하게 다듬고 파라미터 타입을 없애자.

```kotlin
people.maxBy { p: Person -> p.age } // 파라미터 타입을 명시
people.maxBy { p -> p.age } // 파라미터 타입을 생략(컴파일러가 추론)
```

로컬 변수처럼 컴파일러는 람다 파라미터의 타입도 추론할 수 있다. 따라서 파라미터 타입을 명시할 필요가 없다. maxBy 함수의 경우 파라미터의 타입은 항상 컬렉션 원소타입과 같다.

마지막으로 람다의 파라미터 이름을 디폴트 이름인 it으로 바꾸면 람다 식을 더 간단하게 만들 수 있다. 람다의 파라미터가 하나뿐이고 그 타입을 컴파일러가 추론할 수 있는 경우 it을 바로 쓸 수 있다.

*5.6 디폴트 파라미터 이름 it 사용하기*

```kotlin
people.maxBy { it.age } // "it"은 자동 생성된 파라미터 이름
```

### 노트

> it을 사용하는 관습은 코드를 아주 간단하게 만들어준다. 하지만 이를 남용하면 안된다. 특히 람다 안에 람다가 중첩되는 경우 각 람다의 파라미터를 명시하는 편이 낫다. 파라미터를 명시하지 않으면 각각의 it이 가리키는 파라미터가 어떤 람다에 속했는지 파악하기 어려울 수 있다
> 

람다를 변수에 저장할 때는 파라미터의 타입을 추론할 문맥이 존재하지 않는다. 따라서 파라미터 타입을 명시해야 한다.

```kotlin
>>> val getAge = { p: Person -> p.age }
>>> people.maxBy(getAge)
```

## 5.1.4 현재 영역에 있는 변수에 접근

자바 메소드 안에서 무명 내부 클래스를 정의할 때 메소드의 로컬 변수를 무명 내부 클래스에서 사용할 수 있다. 람다 안에서도 같은 일을 할 수 있다. 람다를 함수 안에 정의하면 함수의 파라미터뿐 아니라 람다 정의의 앞에 선언된 로컬 변수까지 람다에서 모두 사용할 수 있다.

*5.7 함수 파라미터를 람다 안에서 사용하기*

```kotlin
fun printMessageWithPrefix(messages: Collection<String>, prefix: String) {
    messages.forEach { // 각 원소에 대해 수행할 작업을 람다로 받는다.
        println("$prefix $it") // 람다 안에서 함수의 "prefix" 파라미터를 사용한다.
    }
}

>>> val errors = listOf("403 Forbidden", "404 Not Found")
>>> printMessagesWithPrefix(errors, "Error:")
Error: 403 Forbidden
Error: 404 Not Found
```

자바와 다른 점 중 중요한 한 가지는 코틀린 람다 안에서는 파이널 변수가 아닌 변수에 접근할 수 있다는 점이다. 또한 람다 안에서 바깥으 변수를 변경해도 된다. 다음 리스트는 전달받은 상태 코드 목록에 있는 클라이언트와 서버 오류의 횟수를 센다.

*5.8 람다 안에서 바깥 함수의 로컬 변수 변경하기*

```kotlin
fun printProblemCounts(responses: Collection<String>) {
    var clientError = 0
    var servererror = 0
    responses.forEach {
        if (it.startsWith("4")) {
            clientErrors++
        } else if (it.startsWith("5")) {
            serverErrors++
        }
    }

    println("$clientErrors client errors, $serverErrors server errors")
}

>>> val responses = listOf("200 OK", "418 I'm a teapot",
                           "500 Internal Server Error")
...
>>> printProblemCounts(response)
1 client errors, 1 server errors
```

코틀린에서는 자바와 달리 람다에서 람다 밖 함수에 있는 파이널이 아닌 변수에 접근할 수 있고, 그 변수를 변경할 수도 있다. 이 예제의 prefix, clientErrors, serverErrors와 같이 람다 안에서 사용하는 외부 변수를 '람다가 포획(capture)한 변수'라고 부른다.

포획한 변수가 있는 람다를 저장해서 함수가 끝난 뒤에 실행해도 람다의 본문 코드는 여전히 포획한 변수를 읽거나 쓸 수 있다. 

### **어떻게 가능할까?**

1. 파이널 변수를 포획한 경우에는 람다 코드를 변수 값과 함께 저장한다. 
2. 파이널이 아닌 변수를 포획한 경우에는 변수를 특별한 래퍼로 감싸서 나중에 변경하거나 읽을 수 있게 한 다음, 래퍼에 대한 참조를 코드와 함께 저장한다.

```kotlin
class Ref<T>(var value: T) // 변경 가능한 변수를 포획하는 방법을 보여주기 위한 클래스
>>> val counter = Ref(0)
>>> val inc = { counter.value++ } // 공식적으로는 변경 불가능한 변수를 포획했지만
                                  // 그 변수가 가리키는 객체의 필드 값을 바꿀 수 있다.

// 실제 코드에서는 이런 래퍼를 만들지 않아도 된다. 대신, 변수를 직접 바꾼다.
var counter = 0
val inc = { counnter ++ }

```

## 5.1.5 멤버 참조

람다를 사용해 코드 블록을 다른 함수에게 인자로 넘기는 방법을 살펴봤다. 하지만 넘기려는 코드가 이미 함수로 선언된 경우는 어떻게 할까?

코틀린에서는 자바 8과 마찬가지로 함수를 값으로 바꿀 수 있다. 이때 이중 콜론(::)을 사용한다.

```kotlin
val getAge = Person::age
```

::를 사용하는 식을 멤버 참조(member reference)라고 부른다. 멤버 참조는 프로퍼티나 메소드를 단 하나만 호출하는 함수 값을 만들어준다.

멤버 참조는 그 멤버를 호출하는 람다와 같은 타입이다. 따라서 다음 예처럼 그둘을 자유롭게 바꿔 쓸 수 있다.

```kotlin
people.maxBy(Person::age)
people.maxBy { p -> p.age }
people.maxBy { it.age }
```

최상위에 선언된(그리고 다른 클래스의 멤버가 아닌) 함수나 프로퍼티를 참조할 수도 있다.

```kotlin
fun salute() = println("Salute!")

>>> run(::salute) // 최상위 함수를 참조한다.
Salute!
```

람다가 인자가 여럿인 다른 함수한테 작업을 위임하는 경우 람다를 정의하지 않고 직접 위임 함수에 대한 참조를 제공하면 편리하다.

```kotlin
val action = { person: Person, message: String -> // 이 람다는 sendEmail 함수에게 작업을
    sendEmail(person, message)                    // 위임한다.
}

val nextAction = :: sendEmail // 람다 대신 멤버 참조를 쓸 수 있다.
```

생성자 참조(constructor reference)를 사용하면 클래스 생성 작업을 연기하거나 저장해둘 수 있다.

:: 뒤에 클래스 이름을 넣으면 생성자 참조를 만들 수 있다.

```kotlin
>>> val createPerson = ::Person
>>> val p = createPerson("Alice", 29)
>>> println(p)
Person(name=Alice, age=29)
```

*바운드 멤버참조*

바운드 멤버 참조를 사용하면 멤버 참조를 생성할 때 클래스 인스턴스를 함께 저장한 다음 나중에 그 인스턴스에 대해 멤버를 호출해준다. 따라서 호출 시 수신 대상 객체를 별도로 지정해 줄 필요가 없다,

```kotlin
>>> val p = Person("Dmitry", 34)
>>> val personAgeFunction = Person:age
>>> println(personsAgeFunction(p))
34

>>> val ditrysAgeFunction = p::age // 코틀린 1.1부터 사용할 수 있는 바운드 멤버 참조
>>> println(dmitrysAgeFunction())
34
```

# 5.2 컬렉션 함수 API

함수형 프로그래밍 스타일을 사용하면 컬렉션을 다룰 때 편리하다. 대부분의 작업에 라이브러리 함수를 활용할 수 있고 그로 인해 코드를 아주 간결하게 만들 수 있다.

이번절에서는 컬렉션을 다루는 코틀린 표준 라이브러리를 몇 가지 살펴본다.

## 5.2.1 필수적인 함수: filter와 map

filter와 map은 컬렉션을 활용할 때 기반이 되는 함수다. 대부분의 컬렉션 연산을 이 두개의 함수를 통해 표현할 수 있다.

숫자를 사용한 예제와 Person을 사용한 예제를 통해 이 두 함수를 자세히 살펴보자.

### filter 함수

filter 함수(필터 함수 또는 걸러내는 함수라고 부름)는 컬렉션을 이터레이션하면서 주어진 람다에 각 원소를 넘겨서 람다가 true를 반환하는 원소만 모은다.

```kotlin
>>> val list = listOf(1, 2, 3, 4)
>>> println(list.filter { it % 2 == 0 }) // 짝수만 남는다.
[2, 4]
```

결과는 입력 컬렉션의 원소 중에서 주어진 술어(참/거짓을 반환하는 함수를 술어(predicate)라고 한다)를 만족하는 원소만으로 이뤄진 새로운 컬렉션이다.

### map 함수

filter 함수는 컬렉션에서 원치 않는 원소를 제거한다. 하지만 filter는 원소를 변환할 수는 없다, 원소를 변환하려면 map 함수를 사용해야 한다.

map 함수는 주어진 람다를 컬렉션의 각 원소에 적용한 결과를 모아서 새 컬렉션을 만든다. 다음과 같이 하면 숫자로 이뤄진 리스트를 각 숫자의 제곱이 모인 리스트로 바꿀 수 있다.

```kotlin
>>> val list = listOf(1, 2, 3, 4)
>>> println(list.map { it * it })
[1, 4, 9, 16]
```

사람의 리스트가 아니라 이름의 리스트를 출력하고 싶다면 map으로 사람의 리스트를 이름의 리스트로 변환하면 된다.

```kotlin
>>> val people = listOf(Person("Alice", 29), Person("Bob", 31))
>>> println(people.map { it.name })
[Alice, Bob]
```

멤버 참조를 사용해보자.

```kotlin
people.map(Person::name)
```

이런 호출을 쉽게 연쇄시킬 수 있다. 예를 들어 30살 이상인 사람의 이름을 출력해보자.

```kotlin
>>> people.filter{ it.age > 30 }.map(Person::name)
[Bob]
```

필터와 변환 함수를 맵에 적용할 수도 있다.

```kotlin
>>> val numbers = mapOf(0 to "zero", 1 to "one")
>>> println(numbers.mapValues{ it.value.toUpperCase() })
{ 0=ZERO, 1=ONE }
```

## 5.2.2 all, any, count, find: 컬렉션에 술어 적용

컬렉션에 대해 자주 수행하는 연산으로 컬렉션의 모든 원소가 어떤 조건을 만족하는지 판단하는 연산이 있다. 

코틀린에서는 all과 any가 이런 연산이다.

이런 함수를 살펴보기 위해 어떤 사람의 나이가 27살 이하인지 판단하는 술어 함수 canBeInClub27를 만들자.

```kotlin
val canBeInClub27 - { p: Person -> p.age <= 27 }
```

모든 원소가 이 술어를 만족하는지 궁금하다면 all 함수를 쓴다.

```kotlin
>>> val people = listOf(Person("Alice", 27), Person("Bob", 31))
>>> println(people.all(canBeInClub27)
false
```

술어를 만족하는 원소가 하나라도 있는지 궁금하면 any를 쓴다.

```kotlin
>>> println(people.any(canBeInClub27)
true
```

가독성을 높이려면 any와 all 앞에 !를 붙이지 않는 편이 낫다.

```kotlin
>>> val list = listOf(1, 2, 3)
>>> println(!list.all { it == 3 }) // !를 눈치 못채는 경우가 자주 있다.
true
>>> println(list.any { it != 3 }) 
true
```

술어를 만족하는 원소의 개수를 구하려면 count를 사용한다.

```kotlin
>>> val people = listOf(Person("Alice", 27), Person("Bob", 31))
>>> println(people.count(canBeInClub27))
1
```

### 노트

> **함수를 적재적소에 사용하라: count와 size**
count가 있다는 사실을 잊어버리고, 컬렉션을 필터링한 결과의 크기를 가져오는 경우가 있다.

>println(people.filter(canBeInClub27).size)
>
>1
>
>하지만 이렇게 처리하면 조건을 만족하는 모든 원소가 들어가는 중간 컬렉션이 생긴다. 반면 count는 조건을 만족하는 원소의 개수만을 추적하지 조건을 만족하는 원소를 따로 저장하지 않는다.
>따라서 count가 훨씬 더 효울적이다.



술어를 만족하는 원소를 하나 찾고 싶으면 find 함수를 사용한다.

```kotlin
>>> val people = listOf(Person("Alice", 27), Person("Bob", 31))
>>> println(people.find(canBeInClub27))
Person(name=Alice, age=27)
```

이 식은 조건을 만족하는 원소가 하나라도 있는 경우 가장 먼저 조건을 만족한다고 확인된 원소를 반환하며, 만족하는 원소가 전혀 없는 경우 null을 반환한다. (find는 firstOrNull과 같다.)

## 5.2.3 groupBy: 리스트를 여러 그룹으로 이뤄진 맵으로 변경

컬렉션의 모든 원소를 어떤 특성에 따라 여러 그룹으로 나누고 싶다고 하자. 예를 들어 사람을 나이에 따라 분류해보자. 특성을 파라미터로 전달하면 컬렉션을 자동으로 구분해주는 함수가 있다면 편리할 것이다.

groupBy 함수가 그런 역할을 한다.

```kotlin
>>> val people = listOf(Person("Alice", 31),
     ... Person("Bob", 29), Person("Carol", 31))
>>> println(people.groupBy { it.age })

{29=[Person(name=Bob, age=29)],
 31=[Person(name=Alice, age=31), Person(name=Carol, age=31)]}
```

각 그룹은 리스트다. 따라서 groupBy의 결과 타입은 Map<Int, List<Person>>이다.

다른 예로 멤버 참조를 활용해 문자열을 첫 글자에 따라 분류하는 코드를 보자.

```kotlin
>>> val list = listOf("a", "ab", "b")
>>> println(list.groupBy(String::first))
{a=[a, ab], b=[b]}
```

## 5.2.4 flatMap과 flatten: 중첩된 컬렉션 안의 원소 처리

이제 사람에 대한 관심을 책으로 돌려보자. Book으로 표현한 책에 대한 정보를 저장하는 도서관이 있다고 가정하자.

```kotlin
class Book(val title: String, val authors: List<String>
```

책마다 저자가 한 명 또는 여러 명 있다. 도서관에 있는 책의 저자를 모두 모은 집합을 다음과 같이 가져올 수 있다.

```kotlin
books.flatMap { it.authors }.toSet() // books 컬렉션에 있는 책을 쓴 모든 저자의 집합
```

1. flatMap 함수는 인자로 주어진 람다를 컬렉션의 모든 객체에 적용(map)
2. 람다를 적용한 결과 얻어지는 여러 리스트를 한 리스트로 한데 모은다.

# 5.3 지연 계산(lazy) 컬렉션 연산

앞 절에서 map이나 filter 같은 몇가지 컬렉션은 컬렉션을 즉시(eaagerly) 생성한다. 이는 컬렉션 함수를 연쇄하면 매 단계마다 계산 중간 결과를 새로운 컬렉션에 임시로 담는다.

시퀀스(sequence)를 사용하면 중간 임시 컬렉션을 사용하지 않고도 컬렉션 연산을 연쇄할 수 있다.

```kotlin
people.map(Person::name).filter {it.startsWith("A") }
```

filter와  map을 호출할때 리스트를 반환한다.

결국 한 리스트는 filter의 결과를 담고 다른 하나는 map의 결과를 담는다.

이를 더 효율적으로 만들기 위해 시퀀스를 사용해보자.

```kotlin
people.asSequence() // 원본 컬렉션을 시퀀스로 변환한다.
    .map(Person::name) // 시퀀스도 컬렉션과 똑같은 API를 제공한다.
    .filter{ it.startsWith("A) }
    .toList() // 결과 시퀀스를 다시 리스트로 변환한다.
```

### 시퀀스를 사용할 때 toList()를 사용하는 이유

- 원소를 차례로 이터레이션해야 한다면 시퀀스를 직접 써도 된다.
- 하지만 시퀀스 원소를 인덱스를 사용해 접근하는 등의 다른 API 메소드가 필요하다면 시퀀스를 리스트로 변환해야한다.

 

## 5.3.1 시퀀스 연산 실행: 중간 연산과 최종 연산

시퀀스에 대한 연산은 중간(intermediate) 연산과 최종(terminal) 연산으로 나뉜다. 중간 연산은 다른 시퀀스를 반환한다. 그 시퀀스는 최초 시퀀스의 원소를 변환하는 방법을 안다.

중간 연산은 항상 지연 계산된다. 최종 연산이 없는 예제를 살펴보자.

```kotlin
>>> listOf(1, 2, 3, 4).asSequence()
...           .map { print("map($it) "); it * it}
...           .filter { print("filter($it) "); it % 2 == 0 }
```

이 코드를 실행하면 아무 내용도 출력되지 않는다. 이는 map과 filter변환이 늦춰져서 결과를 얻을 필요가 있을 때 (즉 최종 연산이 호출될 때 ) 적용된다는 뜻이다.

```kotlin
>>> listOf(1, 2, 3, 4).asSequence()
...           .map { print("map($it) "); it * it}
...           .filter { print("filter($it) "); it % 2 == 0 }
...           .toList()
map(1) filter(1) map(2) filter(4) map(3) filter(9) map(4) filter(16)
```

kotlin의 시퀀스는 자바 스트림과 동일하게 동작한다.

map으로 리스트의 각 숫자를 제곱하고 제곱한 숫자 중에서 find로 3보다 큰 첫 번째 원소를 찾아보자.

```kotlin
>>> println(listOf(1, 2, 3, 4).asSequence()
                              .map { it * it }.find { it > 3 })
4

map(1) filter(1) map(2) filter(4)
```

## 5.3.2 시퀀스 만들기

지금까지는 모두 컬렉션에 대해 assequence()를 호출해 시퀀스를 만들었다. 시퀀스를 만드는 다른 방법으로 generateSequence 함수를 사용할 수 있다.

```kotlin
>>> val naturalNumbers = generateSequnce(0) { it + 1 }
>>> val numbersTo100 = naturalNumbers.taskWhile { it <= 100 }
>>> println(numbersTo100.sum()) // 모든 지연 연산은 "sum"의 결과를 계산할 때 수행된다.
5050
```

# 5.4 수신 객체 지정 람다: with와 apply

이번 절은 자바의 람다에는 없는 코틀린 람다의 독특한 가능을 설명한다. 그 기능은 수신 객체를 명시하지 않고 람다의 본문 안에서 다른 객체의 메소드를 호출할 수 있게하는 것이다. 

그런 람다를 수신 객체 지정 람다(lambda with receiver)라고 부른다.

## 5.4.1 with 함수

with의 유용성 알기 위해 먼저 다음 예제를 살펴보고 이를 with를 사용해 리팩토링해보자.

```kotlin
fun alphabet (): String {
    val result = StringBuilder()
    for (letter in 'A'..'Z') {
        result.append(letter)
    }
    result.append("\nNoew I konw the aplhabet!")
    return result.toString()
}
>>> println(alphabet())
ABCDEFGHIJKLMNOPQRSTUVWXYZ
Now I know the alphabet!
```

이제 앞에 예제를 with로 다시 작성한 결과를 살펴보자.

```kotlin
fun alphabet (): String {
    val stringBuilder = StringBuilder()
    return with(stringBuilder) { // 메소드를 호출하려는 수신 객체를 지정한다.
    for (letter in 'A'..'Z') {
        this.append(letter) // "this"를 명시해서 앞에서 지정한 수신 객체의 메소드를 호출한다.
    }
    append("\nNoew I konw the aplhabet!") // "this"를 생략하고 메소드를 호출한다.
    return this.toString() //람다에서 값을 반환한다.
  }
}
```

with 함수는 첫 번째 인자로 받은 객체를 두 번째 인자로 받은 람다의 수신 객체로 만든다. 인자로 받은 람다 본문에서는 this를 사용해 그 수신 객체에 접근 할 수 있다.

앞에 alphabet 함수를 더 리팩터링해서 불필요한 stringBuilder 변수를 없앨 수도 있다.

```kotlin
fun alphabet () = with(StringBuilder()) { 
    for (letter in 'A'..'Z') {
        append(letter) 
    }
    append("\nNoew I konw the aplhabet!") 
    toString() 
  }
}
```

### 메소드 이름 충돌

with에게 인자로 넘긴 객체의 클래스와 with를 사용하는 코드가 들어있는 클래스 안에 이름이 같은 메소드가 있다면 무슨 일이 생길까? 그런 경우 this 참조 앞에 레이블을 붙이면 호출하고 싶은 메소드를 명확하게 정할 수 있다.

만약 alphabet 함수가 OuterClass의 메소드라고 하면 StringBuilder가 아닌 바깥쪽 클래스 toString을 호출하고 싶다면 다음과 같은 문구를 사용하면 된다.

```kotlin
this@OuterClass.toString()
```

때로는 람다의 결과 대신 수신 객체가 필요한 경우도 있다. 그럴 때는 apply 라이브러리 함수를 사용할 수 있다.

## 5.4.2 apply 함수

apply 함수는 with와 유일한 차이점은 apply는 항상 자신에게 전달된 객체를 반환한다는 점이다.

apply를 써서 alphabet 함수를 다시 리팩터링해보자.

```kotlin
fun alphabet() = StringBuilder().apply {
    for (letter in 'A'..'Z') {
        append(letter)
    {
    append("\nNow I know the alphabet!")
}.toString
```

apply는 확장 함수로 정의돼 있다. 이런 apply 함수는 객체의 인스턴스를 만들면서 즉시 프로퍼티 중 일부를 초기화해야 하는 경우 유용하다. ex) 자바의 Builder

apply를 객체 초기화에 활용하는 예로 안드로이드 TextView 컴포넌트를 만들면서 특성 중 일부를 설정해보자.

```kotlin
fun createViewWithCustomAttributes(context: Context) =
    TextView(context).apply {
        text = "sample Text"
        textSize = 20.0
        setPadding(10, 0, 0, 0)
```

with와 apply는 수신 객체 지정 람다를 사용하는 일반적인 예제 중 하나다. 더 구체적인 함수를 비슷한 패턴으로 활용할 수 있다. 예를 들어 표준 라이브러리의 buildString 함수를 사용하면 alphabet 함수를 더 단순화 할 수 있다.

```kotlin
fun alphabet() = buildString { // buildString은 StringBuilder 객체를 만드는 일과
    for (letter in 'A'..'Z') { // toString을 호출해주는 일을 알아서 해준다.
        append(letter)
    {
    append("\nNow I know the alphabet!")
}
```

# 5.5 요약

- 람다를 사용하면 코드 조각을 다른 함수에게 인자로 넘길 수 있다.
- 코틀린에서는 람다가 함수 인자인 경우 괄호 밖으로 람다를 빼낼 수 있고, 람다의 인자가 단 하나뿐인 경우 인자 이름을 지정하지 않고 it이라는 디폴트 이름으로 부를 수 있다.
- 람다 안에 있는 코드는 그 람다가 들어있는 바깥 함수의 변수를 읽거나 쓸 수 있다.
- 메소드, 생성자, 프로퍼티의 이름 앞에 ::을 붙이면 각각에 대한 참조를 만들 수 있다. 그런 참조를 람다 대신 다른 함수에게 넘길 수 있다.
- filter, map, all, any 등의 함수를 활용하면 컬렉션에 대한 대부분의 연산을 직접 원소를 이터레이션하지 않고 수행할 수 있다.
- 시퀀스를 사용하면 중간 결과를 담는 컬렉션을 생성하지 않고도 컬렉션에 대한 여러 연산을 조합할 수 있다.
- 함수형 인터페이스(추상 메소드가 단 하나뿐인 SAM 인터페이스)를 인자로 받는 자바 함수를 호출할 경우 람다를 함수형 인터페이스 인자 대신 넘길 수 있다.
- 수신 객체 지정 람다를 사용하면 람다 안에서 미리 정해둔 수신 객체의 메소드를 직접 호출할 수 있다.
- 표준 라이브러리의 with 함수를 사용하면 어떤 객체에 대한 참조를 반복해서 언급하지 않으면서 그 객체의 메소드를 호출할 수 있다. apply를 사용하면 어떤 객체라도 빌더 스타일의 API를 사용해 생성하고 초기화할 수 있다.