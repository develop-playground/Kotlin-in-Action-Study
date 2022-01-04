## 10.2.3 애노테이션을 활용한 직렬화 제어

이제 Jkid에서 정의한 `@JsonExclude`, `@JsonName`, `@CustomSerializer` 애노테이션들을 `serializeObject` 함수가 어떻게 처리하는지 알아보자.

먼저 `@JsonExclude`부터 보자.

어떤 프로퍼티를 직렬화에서 제외하고 싶을 때 이 애노테이션을 쓸 수 있다.

클래스의 모든 멤버 프로퍼티를 가져오기 위해 `KClass` 인스턴스의 `memberProperties` 프로퍼티를 사용했었다.

하지만 지금은 `@JsonExclude` 애노테이션이 붙은 프로퍼티를 제외해야 한다.

`KAnnotatedElement` 인터페이스에는 `annotations` 프로퍼티가 있다.

이는 소스코드상에서 해당 요소에 적용된 모든 애노테이션 인스턴스의 컬렉션이다.

`KProperty`는 `KAnnotatedElement`를 확장하므로 `propery.annotations`를 통해 프로퍼티의 모든 애노테이션을 얻을 수 있다.

여기서는 모든 애노테이션이 아닌 특정 애노테이션만 찾으면 되므로 `findAnnotation`라는 함수를 사용할 수 있다.

```kotlin
inline fun <reified T> KAnnotatedElement.findAnnotation(): T?
        = annotations.filterIsInstance<T>().firstOrNull()
```

`findAnnotation` 함수는 인자로 전달받은 타입에 해당하는 애노테이션이 있으면 그 애노테이션을 반환한다.

9장에서 배운 타입 실체화를 사용해 타입 파라미터를 `reified`로 만들어 애노테이션 클래스를 타입 인자로 전달한다.

이제 `findAnnotation`을 표준 라이브러리 함수인 `filter`와 함께 사용하면 `@JsonExclude` 애노테이션된 프로퍼티를 없앨 수 있다.

```kotlin
val properties = obj.javaClass.kotlin.memberProperties
        .filter { it.findAnnotation<JsonExclude>() == null }
```

다음은 `@JsonName`이다.

기억을 되살리기 위해 `@JsonName` 선언과 사용법을 다시 보자.

```kotlin
@Target(AnnotationTarget.PROPERTY)
annotation class JsonName(val name: String)

data class Person(
  @JsonName("alias") val firstName: String,
  val age: Int
)
```

이 경우 애노테이션의 존재 여부뿐 아니라 애노테이션에 전달한 인자도 알아야 한다.

`@JsonName`의 인자는 프로퍼티를 직렬화해서 JSON에 넣을 때 사용할 이름이다.

다행히 이 경우에도 `findAnnotation`을 사용할 수 있다.

```kotlin
val jsonNameAnn = prop.findAnnotation<JsonName>()  // @JsonName 애노테이션이 있으면 그 인스턴스를 얻는다.
val propName = jsonNameAnn?.name ?: prop.name  // 애노테이션에서 "name" 인자를 찾고 그런 인자가 없으면 prop.name을 사용한다.
```

프로퍼티에 `@JsonName` 애노테이션이 없다면 `jsonNameAnn`은 `null`일 것이다.

그런 경우 여전히 `prop.name`을 JSON의 프로퍼티 이름으로 사용할 수 있다.

프로퍼티에 `@JsonName` 애노테이션이 있다면 애노테이션이 지정하는 이름을 대신 사용한다.

앞에서 본 `Person` 클래스 인스턴스를 직렬화하는 과정을 보자.

`firtName` 프로퍼티를 직렬화하는 동안 `jsonNameAnn`에는 `JsonName` 애노테이션 클래스에 해당하는 인스턴스가 들어있다.

따라서 `jsonNameAnn?.name`의 값은 `null`이 아니고 `alias`이며, 직렬화 시 이 이름을 키로 사용한다.

`age` 프로퍼티를 직렬화할 때는 `@JsonName` 애노테이션이 없으므로 `age`를 키로 사용한다.

다음은 지금까지 설명한 내용을 반영한 직렬화 로직이다.

```kotlin
private fun StringBuilder.serializeObject(obj: Any) {
    obj.javaClass.kotlin.memberProperties
        .filter { it.findAnnotation<JsonExclude>() == null }
        .joinToStringBuilder(this, prefix = "{", postfix = "}") {
            serializeProperty(it, obj)
        }
}
```

위 코드는 `@JsonExclude`로 애노테이션한 프로퍼티를 제외시킨다.

또한 프로퍼티 직렬화와 관련된 로직을 `serializeProperty`라는 확장 함수로 분리해 호출한다.

```kotlin
private fun StringBuilder.serializeProperty(
    prop: KProperty1<Any, *>, obj: Any,
) {
    val jsonNameAnn = prop.findAnnotation<JsonName>()
    val propName = jsonNameAnn?.name ?: prop.name
    serializeString(propName)
    append(": ")

    val value = prop.get(obj)
    val jsonValue = prop.getSerializer()?.toJsonValue(value) ?: value
    serializePropertyValue(jsonValue)
}
```

앞에서 설명한 것처럼 `@JsonName`에 따라 프로퍼티 이름을 처리한다.

`out`: 생산할 수 있지만 소비할 수 없음. 공변성

`in`: 소비할 수 있지만 생산할 수 없음. 반공변성

다음으로 나머지 애노테이션인 `@CustomSerializer`를 구현해보자.

이 구현은 `getSerializer`라는 함수에 기초한다.

`getSerializer`는 `@CustomSerializer`를 통해 등록한 `ValueSerializer` 인스턴스를 반환한다.

예를 들어 `Person` 클래스를 다음과 같이 정의하고 `birthDate` 프로퍼티를 직렬화하면서 `getSerializer()`를 호출하면 `DateSerializer` 인스턴스를 얻을 수 있다.

```kotlin
data class Person(
    val name: String,
    @CustomSerializer(DateSerializer::class) val birthDate: Date
)
```

`getSerializer` 구현을 이해하기 위해 `@CustomSerializer` 선언을 살펴보자.

```kotlin
annotation class CustomSerializer(
    val serializerClass: KClass<out ValueSerializer<*>>
)
```

`getSerializer` 구현은 다음과 같다.

```kotlin
fun KProperty<*>.getSerializer(): ValueSerializer<Any?>? {
val customSerializerAnn = findAnnotation<CustomSerializer>() ?: return null 
val serializerClass = customSerializerAnn.serializerClass
val valueSerializer = serializerClass.objectInstance
        ?: serializerClass.createInstance()
@Suppress("UNCHECKED_CAST")
return valueSerializer as ValueSerializer<Any?>
}
```

`getSerializer`가 주로 다루는 객체가 `KProperty` 인스턴스이기 때문에 `KProperty`의 확장 함수로 정의한다.

`getSerializer`는 `findAnnotation` 함수를 호출해서 `@CustomSerializer` 애노테이션이 있는지 찾는다.

`@CustomSerializer` 애노테이션이 있다면 그 애노테이션의 `serializerClass`가 직렬화기 인스턴스를 얻기 위해 사용해야할 클래스다.

여기서 `@CustomSerializer`의 값으로 클래스와 객체(`object`)를 처리하는 방식을 보자.

클래스와 객체는 모두 `KClass` 클래스로 표현된다.

다만 객체에는 `object` 선언에 의해 생성된 싱글턴을 가리키는 `objectInstance`라는 프로퍼티가 있다는 것이 클래스와 다른 점이다.

따라서 그 싱글턴 인스턴스를 사용해 모든 객체를 직렬화하면 되므로 객체를 생성할 필요가 없다.

하지만 `KClass`가 일반 클래스를 표현한다면 `createInstance`를 호출해서 새 인스턴스를 만들어야 한다.

```kotlin
private fun StringBuilder.serializeProperty(
    prop: KProperty1<Any, *>, obj: Any,
) {
    val jsonNameAnn = prop.findAnnotation<JsonName>()
    val propName = jsonNameAnn?.name ?: prop.name
    serializeString(propName)
    append(": ")

    val value = prop.get(obj)
    val jsonValue = prop.getSerializer()?.toJsonValue(value) ?: value
                // 프로퍼티에 대해 정의된 커스텀 직렬화기가 있으면 그 커스텀 직렬화기를 사용한다.
       // 커스텀 직렬화기가 없으면 일반적인 방법을 따라 프로퍼티를 직렬화한다.
    serializePropertyValue(jsonValue)
}
```

이제 역직렬화 부분을 보자.

역직렬화 부분은 코드가 더 길기 때문에 전체 구조와 리플렉션을 어떻게 사용하는지를 주로 설명한다.

*4~5절의 역직렬화 부분은 애노테이션과 리플렉션보다도 구현에 관한 내용이 많아 생략한다.*

*사실 이 책에서 나온 내용만으로 애노테이션과 리플렉션을 완벽하게 사용할 수는 없으므로 사용해야 할 일이 생긴다면 다음의 문서를 참고하도록 하자.*

[Annotations | Kotlin](https://kotlinlang.org/docs/annotations.html)

[Reflection | Kotlin](https://kotlinlang.org/docs/reflection.html)



# 10.3 요약

- 코틀린에서 애노테이션을 적용할 때 사용하는 문법은 자바와 거의 같지만 더 넓은 대상에 애노테이션을 적용할 수 있다.
- 애노테이션 인자로 원시 타입, 문자열, 이넘, 클래스 참조, 다른 애노테이션 클래스의 인스턴스, 배열을 사용할 수 있다.
- @get:Rule와 같은 문법 사용해 애노테이션의 사용 대상을 명시하면 한 코틀린 선언이 여러 가지 바이트코드 요소를 만들어내는 경우 정확히 어떤 부분에 애노테이션을 적용할지 지정할 수 있다.
- 애노테이션 클래스를 정의할 때는 본문이 없고 주 생성자의 모든 파라미터를 val 프로퍼티로 표시한 코틀린 클래스를 사용한다.
- 메타애노테이션을 사용해 대상, 애노테이션 유지 방식 등 여러 애노테이션 특성을 지정할 수 있다.
- 리플렉션 API를 통해 실행 시점에 객체의 메서드와 프로퍼티를 열거하고 접근할 수 있다.
  리플렉션 API에는 클래스(`KClass`), 함수(`KFunction`) 등 여러 종류의 선언을 표현하는 인터페이스가 들어있다.
- 클래스를 컴파일 시점에 알고 있다면 `KClass` 인스턴스를 얻기 위해 `ClassName::class`를 사용한다.
  하지만 실행 시점에 `KClass` 인스턴스를 얻기 위해서는 `obj.javaClass.kotlin`을 사용한다.
- `KFunction`과 `KProperty` 인터페이스는 모두 `KCallable`을 확장한다. `KCallable`은 제네릭 `call` 메서드를 제공한다.
- `KFunction0`, `KFunction1` 등의 인터페이스는 모두 파라미터 수가 다른 함수를 표현하며, `invoke` 메서드를 사용해 함수를 호출할 수 있다.
- `KProperty0`은 최상위 프로퍼티나 변수, `KProperty1`은 수신 객체가 있는 프로퍼티에 접근할 때 쓰는 인터페이스다. 두 인터페이스 모두 `get` 메서드를 사용해 프로퍼티 값을 가져올 수 있다.
- `KMutablePropertyN`(`var`)은 `KPropertyN`을 확장하며, `set` 메서드를 통해 프로퍼티 값을 변경할 수 있게 해준다.
