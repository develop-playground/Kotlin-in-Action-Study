# Chapter 10 : 애노테이션과 리플렉션

## 10. 2 : 리플렉션 : 실행 시점에 코틀린 객체 내부 관찰

리플렉션은 실행 시점에 동적으로 객체의 프로퍼티와 메소드에 접근할 수 있게 해주는 방법입니다. 타입과 관계없이 객체를 다뤄야 하거나 객체가 제공하는 메소드나 프로퍼티 이름을 오직 실행 시점에만 알 수 있는 경우가 있습니다. JSON 직렬화 라이브러리를 사용하는 경우입니다.

코틀린에서 리플렉션을 사용하려면 두 가지 서로 다른 리플렉션 API를 다뤄야 합니다.

- `java.lang.reflect`
- `kotlin.reflect` : 자바에는 없는 프로퍼티나 널이 될 수 있는 타입과 같은 코틀린 고유 개념에 대한 리플렉션 제공



### 10.2.1 코틀린 리플렉션 API : KClass, KCallable, KFunction, KProperty

`java.lang.Class`에 해당하는 `KClass`를 사용하면 클래스 안에 존재하는 모든 선언을 열거하고 접근하거나 상위 클래스를 얻는 등의 작업이 가능해집니다. `MyClass::class`라는 식을 써서 인스턴스를 얻을 수 있으며, 실행 시점에 객체의 클래스를 얻으려면 javaClass 프로퍼티를 사용하여 객체의 자바 클래스를 얻어야 합니다. 이 클래스를 얻었으면, `.kotlin` 확장 프로퍼티를 통해 자바에서 코틀린 리플렉션 API로 옮겨올 수 있습니다.

```kotlin
class Person(val name: String, val age: Int)

val person = Person("Hongbeom", 28)
val kClass = person.javaClass.kotlin
println(kClass.simpleName) // Person
kClass.memberProperties.forEach { println(it.name) } // age, name
```



`KClass` 인터페이스를 살펴보면, 클래스의 모든 멤버 목록이 `KCallable` 인스턴스의 컬렉션이라는 것을 알  수 있습니다. `KCallable`은 함수와 프로퍼티를 아우르는 공통 상위 인터페이스이며, 내부에 `call`이라는 메소드가 들어있어서 `call`을 사용하면 함수나 프로퍼티의 게터를 호출할 수 있습니다.

```kotlin
interface KClass<T : Any> {
  val simpleName: String?
  val qualifiedName: String?
  val members: Collection<KCallable<*>>
  val constructors: Collection<KCallable<*>>
  val nestedClasses: Collection<KCallable<*>>
}
```

```kotlin
interface KCallable<out R> {
  fun call(vararg args: Any?): R
}
```



다음 코드는 리플렉션이 제공하는 `call`을 사용하여 함수를 호출할 수 있음을 보여줍니다.

```kotlin
fun foo(x: Int) = println(x)
val kFunction = ::foo
kFunction.call(42) // 42
```



하지만 여기서 함수를 호출하기 위해 더 구체적인 메소드를 사용할 수도 있습니다. (call은 인자가 `vararg`라서 인자의 개수를 올바르게 맞춰주지 않아도 컴파일 에러가 안남) `::foo`의 타입 `KFunction1<Int, Unit>`에는 파라미터와 리턴값 정보가 들어있습니다. 1은 파라미터가 1개라는 뜻입니다. 우리는 `kFunction`의 함수를 직접 호출할 수 있습니다.

```kotlin
fun sum(x: Int, y: Int) = x + y
val kFunction: KFunction2<Int, Int, Int> = ::sum
println(kFunction.invoke(1, 2) + kFunction(3, 4)) // 10

kFunction(1) // ERROR : No value passed for parameter p2
```

![image](https://t1.daumcdn.net/cfile/tistory/99178A33599A8FFB37)

- https://t1.daumcdn.net/cfile/tistory/99178A33599A8FFB37 참조 이미지



## Quiz 

`var`로 선언한 프로퍼티를 `KProperty`로 접근하여 값을 `set`하고 `get`을 통해 해당 프로퍼티를 출력해봅시다

`name`과 `age`를 가지는 `Person` 클래스를 정의하고 `KProperty` 리플렉션을 통해 `age`에 접근한 후 접근한 `KProperty`를 사용하여 `age`를 출력해봅시다. 



### 10.2.2 리플렉션을 사용한 객체 직렬화 구현

직렬화 함수의 기능을 알아봅시다. 기본적으로 직렬화 함수는 객체의 모든 프로퍼티를 직렬화합니다. 아래 코드를 살펴봅시다.

```kotlin
private fun StrintBuilder.serializeObject(obj: Any) {
  val kClass = obj.javaClass.kotlin
  val properties = kClass.memberProperties
  
  properties.joinToStringBuilder(this, prefix = "{", postfix = "}") { prop ->
    serializeString(prop.name) // 프로퍼티 이름 얻기
    append(": ")
    serializePropertyValue(prop,get(obj)) // 프로퍼티 값 얻기
  }
}

// 결과 json 예시 : { prop1: value1, prop2: value2 }
```



`joinToStringBuilder` 함수는 프로퍼티를 콤마(,)로 분리해주며, `serializeString` 함수는 `JSON` 명세에 따라 특수 문자를 이스케이프 해줍니다. `serializePropertyValue` 함수는 어떤 값이 원시 타입, 문자열, 컬렉션, 중첩된 객체 중 어떤 것인지 판단하고 그에 따라 값을 적절히 직렬화 합니다.

하지만 이 예제 코드에서는 어떤 객체의 클래스에 정의된 모든 프로퍼티를 열거하기 때문에 정확히 각 프로퍼티가 어떤 타입인지 알 수 없는데, 따라서 `prop` 변수의 타입은 `KProperty<Any, *>`이며, `prop.get(obj)` 메소드 호출은 `Any` 타입의 값을 반환합니다. 이 경우 수신 객체 타입을 컴파일 시점에 검사할 방법이 없으나, 어떤 프로퍼티의 `get`에 넘기는 객체가 바로 그 프로퍼티를 가져온 객체(`obj`)이기 때문에 항상 프로퍼티 값이 제대로 리턴됩니다.



