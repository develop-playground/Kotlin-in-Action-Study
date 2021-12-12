# 연산자 오버로딩과 기타 관례

### 구조 분해 선언과 component 함수

구조 분해를 사용하면 복합적인 값을 분해해서 여러 다른 변수를 한꺼번에 초기화할 수 있습니다. 아래 코드를 살펴보겠습니다.

```kotlin
val p = Point(10, 20)
val (x, y) = p
println(x) // 10
println(y) // 20
```



구조 분해 선언은 일반 변수 선언과 비슷해 보이지만 =의 좌변에 여러 변수를 괄호로 묶었다는 점이 다릅니다. 내부에서 구조 분해 선언은 다시 관례를 사용하는데, 구조 분해 선언의 각 변수를 초기화하기 위해 `componentN`이라는 함수를 호출합니다. 여기서 N은 구조 분해 선언에 있는 변수의 위치에 따라 붙는 번호를 의미합니다. 

```kotlin
val (x, y) = p
val x = p.component1()
val y = p.component2()
```



데이터 클래스의 주 생성자에 들어있는 프로퍼티에 대해서는 컴파일러가 자동으로 `componentN` 함수를 만들어줍니다. 구조 분해 선언은 함수에서 여러 값을 반환할 때 유용합니다. 물론 무한하게 컴포넌트 함수를 선언할 수는 없으므로 이런 구문을 무한정 사용할 수는 없습니다. 코틀린 표준 라이브러리에서는 맨 앞의 다섯 원소에 대한 `componentN` 함수를 제공합니다.

또한 표준 라이브러리의 `Pair`나 `Tripple` 클래스를 사용하면 함수에서 여러 값을 더 간단하게 반환할 수 있습니다.

```kotlin
val pair: Pair<Int, String> = 1 to "one"
val (index, str) = pair
println(index) // 1
println(str) // one
```



위 특징으로 인해 컬렉션의 크기가 5보다 크거나 컬렉션의 크기를 벗어나는 위치의 원소에 대한 구조 분해 선언을 사용하면 에러가 발생합니다.

```kotlin
val x = listOf(1, 2)
val (a, b, c) = x // 런타임 에러, ArrayIndexOutOfBoundsException 발생!
val (a, b, c, d, e, f) = x // 컴파일 에러, component6() 함수를 가질 수 없음!
```



### 구조 분해 선언과 루프

함수 본문 내의 선언뿐 아니라 변수 선언이 들어갈 수 있는 장소라면 어디든 구조 분해 선언을 사용할 수 있습니다. 예를 들어 루프 안에서도 구조 분해 선언을 사용할 수 있습니다.

```kotlin
fun printEntries(map: Map<String, String>) {
  for ((key, value) in map) {
    println("$key -> $value")
  }
}
```



### 프로퍼티 접근자 로직 재활용 : 위임 프로퍼티

코틀린의 강력한 기능중 하나인 위임 프로퍼티를 사용하면 값을 backing 필드에 단순히 저장하는 것보다 더 복잡한 방식으로 작동하는 프로퍼티를 쉽게 구현할 수 있습니다. 또한 이 과정에서 접근자 로직을 매번 재구현할 필요도 없습니다. 

위임(delegate)은 객체가 직접 작업을 수행하지 않고 다른 도우미 객체가 그 작업을 처리하게 맡기는 디자인 패턴을 말합니다. 이때 작업을 처리하는 도우미 객체를 위임 객체라고 부릅니다.



### by lazy를 사용한 프로퍼티 초기화 지연

지연 초기화는 객체의 일부분을 초기화하지 않고 남겨뒀다가 실제로 그 부분의 값이 필요한 경우 초기화할 때 흔히 쓰이는 패턴이다. 초기화 과정에 자원을 많이 사용하거나 객체를 사용할 때마다 꼭 초기화하지 않아도 되는 프로퍼티에 대해 지연 초기화 패턴을 사용할 수 있다.

에를 들어 `person` 클래스가 자신이 작성한 이메일의 목록을 제공한다고 가정해봅시다. 이메일은 DB에 들어있고 불러오려면 시간이 오래 걸립니다. 그래서 이메일 프로퍼티의 값을 최초로 사용할 때 단 한번만 이메일을 DB에서 가져오고 싶습니다. 이제 DB에서 이메일을 가져오는 `loadEmails` 함수가 있다고 해봅시다.

```kotlin
class Email {
  // ...
}

fun loadEmails(person: Person): List<Email> {
  println("${person.name}의 이메일을 가져옵니다.")
  return listOf(
    //...
  )
}
```



다음은 이메일을 불러오기 전에는 `null`을 저장하고 불러온 다음에는 이메일 리스트를 저장하는 `_emails` 프로퍼티를 추가해서 지연 초기화를 구현한 클래스를 보여줍니다.

```kotlin
class Person(val name: String) {
  
  private var _emails: List<Email>? = null
  val emails: List<Email>
    get() {
      if (_emails == null) {
        _emails = loadEmails(this)
      }
      return _emails!!
    }
  
}
```



여기선 backing 프로퍼티라는 기법을 사용했습니다. 하지만 이런 코드를 작성하는 일은 굉장히 성가십니다. 지연 초기화해야하는 프로퍼티가 많아지면 코드가 굉장히 지저분해 질 것입니다. 게다가 이 방식은 thread-safe 하지 않아서 언제나 제대로 작동한다고 말할 수도 없습니다.

위임 프로퍼티를 사용하면 이 코드는 훨씬 더 간단해집니다. 

```kotlin
class Person(val name : String) {
  val emails by lazy { loadEmails(this) }
}
```



`lazy` 함수는 코틀린 관례에 맞는 시그니처의 `getValue` 메소드가 들어있는 객체를 반환합니다. 또 `lazy` 함수는 기본적으로 thread-safe합니다. 하지만 필요에 따라 동기화에 사용할 락을 `lazy` 함수에 전달할 수도 있고, 다중 스레드 환경에서 사용하지 않을 프로퍼티를 위해 `lazy` 함수가 동기화를 하지 못하게 사용할 수도 있습니다.

그렇다면 과연 어떻게 이런 동작이 가능할까요? 그것은 아래 블로그에서 확인해보겠습니다.

https://medium.com/hongbeomi-dev/%EB%B2%88%EC%97%AD-kotlin%EC%9D%98-delegates%EB%A1%9C-%EC%9C%84%EC%9E%84%ED%95%98%EA%B8%B0-1%ED%8E%B8-87391a2f0645

https://medium.com/hongbeomi-dev/%EB%B2%88%EC%97%AD-%EB%82%B4%EC%9E%A5%EB%90%9C-delegates-2%ED%8E%B8-bc4a23cb6f10