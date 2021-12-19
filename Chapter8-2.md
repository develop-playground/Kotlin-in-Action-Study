# 고차 함수 : 파라미터와 반환 값으로 람다 사용

## 8.3 고차 함수 안에서 흐름 제어

루프 같은 명령형 코드를 람다로 바꾸게 되면 우리는 곧 return 문제에 부딫힐 것입니다. 루프의 중간에 있는 `return` 문의 의미를 이해하기는 쉬운데, 이 루프를 `filter`와 같이 람다를 호출하는 함수로 바꾸고 인자로 전달하는 람다 안에서 `return`을 사용한다면 어떤 일이 벌어질까요?



### 8.3.1 람다 안의 return 문: 람다를 둘러싼 함수로부터 리턴

아래 예시를 살펴봅시다. 

```kotlin
fun lookForAlice(people: List<Person>) {
  people.forEach {
    if (it.name == "Alice") {
      println("Found!")
      return
    }
  }
  println("Alice is not found")
}
```



`forEach`에 넘긴 람다 안에서 `return`을 사용하면 람다로부터만 반환되는게 아니라 그 람다를 호출하는 함수가 실행을 끝내고 리턴됩니다. 이렇게 자신을 둘러싸고 있는 블록보다 더 바깥에 있는 다른 블록을 리턴하게 만드는 `return` 문을 non-local `return`이라고 부릅니다.

이렇게 `return`이 바깥쪽 함수를 리턴시킬 수 있는 때는 람다를 인자로 받는 함수가 인라인 함수인 경우뿐입니다. 따라서 `return`식이 바깥쪽 함수(여기선 `lookForAlice`)를 리턴시키도록 컴파일 됩니다. 하지만 인라이닝되지 않는 함수에 전달되는 람다 안에서 `return`을 사용할 수는 없습니다. 



### 8.3.2 람다로부터 반환 : 레이블을 사용한 return

람다 식에서도 로컬 `return`을 할 수 있습니다. 로컬 `return`은 람다의 실행을 끝내고 람다를 호출했던 코드의 실행을 계속 이어갑니다. 이 때 로컬 `return`과 non-local `return`을 구분하기 위해 레이블을 사용해야 합니다. 아래 예시를 살펴보겠습니다.

```kotlin
fun lookForAlice(people: List<Person>) {
  people.forEach label@ {
    if (it.name == "Alice") return@label
  }
  println("Alice might be somewhere")
}
```



물론 람다에 레이블을 붙여서 사용하는 대신 람다를 인자로 받는 인라인 함수의 이름을 `return` 뒤에 레이블로 사용해도 됩니다.

```kotlin
fun lookForAlice(people: List<Person>) {
  people.forEach {
    if (it.name == "Alice") return@forEach
  }
  println("Alice might be somewhere")
}
```



람다 식의 레이블을 명시하면 함수 이름을 레이블로 사용할 수 없다는 점에 유의하여야 합니다. 

하지만 non-local 리턴문은 장황하고, 람다 안의 여러 위치에 `return` 식이 들어가야 하는 경우 사용하기 불편합니다. 이런 경우, 무명 함수를 통해 이 불편함을 해소할 수 있습니다.



### 8.3.3 무명 함수 : 기본적으로 로컬 return

무명 함수는 일반 함수와 비슷해 보이지만 함수 이름이나 파라미터 타입을 생략할 수 있습니다.

```kotlin
fun lookForAlice(people: List<Person>) {
  people.forEach(fun (person) {
    if (it.name == "Alice") return
    println("${person.name} is not Alice")
  }) 
}
```



무명 함수 안에서 레이블이 붙지 않은 `return` 식은 무명 함수 자체를 리턴시킬 뿐 무명 함수를 둘러싼 다른 함수를 리턴시키지 않습니다. 사실 `return`에 적용되는 규칙은 단순히 `return`은 `fun` 키워드를 이용해 정의된 가장 안쪽 함수를 리턴시킨다는 것입니다. 



### Quiz

아래 클래스를 참고합시다.

```kotlin
data class Person(
  val name : String,
  val age: Int
)
```

- 결과가 참/거짓인 조건을 파라미터로 받아서 해당 조건으로 필터링된 `List<Person>`을 리턴하는 `List<Person>`에 대한 확장 함수를 만들어봅시다.

- `Person`을 인자로 사용하는 람다 파라미터를 받아서 이름이 "john"이고 나이가 20살인 `Person`을 만들어 해당 파라미터의 인자로 넘겨 실행시키는 함수를 만들어봅시다.

- `Person`을 리턴하는 람다 파라미터를 받아서 파라미터에서 리턴하는 `Person`을 출력하는 함수를 만들어봅시다.

- 수신 객체가 `Person`인 스코프를 가지는 `person` 함수를 만들어봅시다.

- 아래 함수를 요구사항대로 실행시키는 코드를 작성해봅시다. 

  요구사항 : 우리는 `name`을 첫 번째 람다식의 인자로 넘기고 `age`를 두 번째 람다식의 인자로 넘겨서 `name`과 `age`를 출력하려고 합니다.

  ```kotlin
  fun testMethod(action: (String) -> (Int) -> Unit): (String) -> (Int) -> Unit =
  { s -> { i -> action(s)(i) } }
  ```

  