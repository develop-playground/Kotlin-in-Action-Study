# 10. 애노테이션과 리플렉션

- 지금까지는 함수나 클래스 이름을 소스코드에서 정확하게 알고 있어야만 사용할 수 있었다.
- 애노테이션과 리플렉션을 사용하면 그런 제약을 벗어나서 미리 알지 못하는 임의의 클래스를 다룰 수 있다.
- 애노테이션을 사용하면 실행 시점에 컴파일러 내부 구조를 분석할 수 있다.
- 애노테이션과 리플렉션의 사용법을 보여주는 예제로 이 장에서는 JSON 직렬화와 역질렬화 라이브러리인 제이키드를 구현한다.
- 이 라이브러리는 실행 시점에 코틀린 객체의 프로퍼티를 읽거나 JSON 파일에서 읽은 데이터를 코틀린 객체로 만들기 위해 리플렉션을 사용한다.
- 그리고 애노테이션을 통해 제이키드 라이브러리가 클래스와 프로퍼티를 직렬화하고 역직렬화하는 방식을 변경한다.

## 애노테이션 선언과 적용

- 코틀린 애노테이션도 자바 개념과 동일하게 메타데이터를 선언에 추가하면 애노테이션을 처리하는 도구가 컴파일 시점이나 실행 시점에 적절한 처리를 해준다.

### 애노테이션 적용

- 애노테이션은 @과 애노테이션 이름으로 이뤄진다. 함수나 클래스 등 여러 다른 코드 구성 요소에 애노테이션을 붙일 수 있다.
- 예를 들어 제이유닛(JUnit) 프레임워크를 사용한다면 테스트 메소드 앞에 `@Test` 애노테이션을 붙여야 한다.
    
    ```kotlin
    import org.junit.*
    
    class MyTest {
      // @Test 애노테이션을 사용해 제이유닛 프레임워크에게 이 메소드를 테스트로 호출하라고 지시
    	@Test fun testTrue() { 
    		Assert.assertTrue(true)
    }
    ```
    
- `@Deprecated` 을 코틀린에서는 replaceWith 파라미터를 통해 옛 버전을 대신할 수 있는 패턴을 제시할 수 있다.
    
    ```kotlin
    @Deprecated("Use removeAt(index) instead.", **ReplaceWith**("removeAt (index)"))
    fun remove(index: Int) {...}
    // ReplaceWith는 애노테이션의 인자로 사용하기 때문에 @를 붙이지 않는다.
    ```
    
- 애노테이션에 인자를 넘길 때는 일반 함수와 마찬가지로 괄호 안에 인자를 넣는다. 애노테이션의 인자로는 원시 타입의 값, 문자열, enum, 클래스 참조, 다른 애노테이션 클래스, 이 요소들의 배열이 들어갈 수 있다. 애노테이션 인자를 지정하는 문법은 자바와 약간 다르다.
    - 클래스를 애노테이션 인자로 지정할 때는 `@MyAnnotation(MyClass::class)` 처럼 ::class를 클래스 이름 뒤에 넣어야 한다.
    - 다른 애노테이션을 인자로 지정할 때는 인자로 들어가는 애노테이션의 이름 앞에 @를 넣지 않아야 한다.
    - 배열을 인자로 지정하려면 `@RequestMapping(path=arrayOf(”/foo”,”/bar”))` 처럼 arrayOf 함수를 사용한다.
    - 자바에서 선언한 애노테이션의 경우 value라는 이름의 파라미터가 필요에 따라 자동으로 가변 길이 인자로 변환된다. 따라서 그럼 경우 `@JavaAnootationWithArrayValue(”abc”,”foo”,”bar”)` 처럼 arrayOf 함수를 쓰지 않아도 된다.
    - 애노테이션 인자를 컴파일 시점에 알 수 있어야 한다. 따라서 임의의 프로퍼티를 인자로 지정할 수는 없다. 프로퍼티를 애노테이션 인자로 사용하려면 그 앞에 const 변경자를 붙여야 한다. (컴파일러는 const가 붙은 프로퍼티를 컴파일 시점 상수로 취급한다)
        
        `const val TEST_TIMEOUT = 100L`
        
        `@Test(timeout = TEST_TIMEOUT) fun testMethod() {...}`
        
    - const가 붙은 프로퍼티는 파일의 맨 위에 object 안에 선언해야 하며, 원시 타입이나 String으로 초기화해야 한다. 일반 프로퍼티를 애노테이션 인자로 사용하려하면 오류 발생 `(”Only const val can be used in constant expressions”)`

### 애노테이션 대상

- 애노테이션을 붙일 때 어떤 요소에 애노테이션을 붙일지 표시할 필요가 있다.
- 사용 지점 대상(use-site target) 선언으로 애노테이션을 붙일 요소를 정할 수 있다. 사용 지점 대상은 @기호화 애노테이션 이름 사이에 붙으며, 애노테이션 이름과는 콜론(:)으로 분리된다. 그림 10.1의 get은 `@Rule` 애노테이션을 프로퍼티 게터에 적용하라는 뜻이다.
    
    ![스크린샷 2022-01-01 오후 8.41.45.png](10%20%E1%84%8B%E1%85%A2%E1%84%82%E1%85%A9%E1%84%90%E1%85%A6%E1%84%8B%E1%85%B5%E1%84%89%E1%85%A7%E1%86%AB%E1%84%80%E1%85%AA%20%E1%84%85%E1%85%B5%E1%84%91%E1%85%B3%E1%86%AF%E1%84%85%E1%85%A6%E1%86%A8%E1%84%89%E1%85%A7%E1%86%AB%200a0abf2cb16e4b23a184620f3d920a10/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA_2022-01-01_%E1%84%8B%E1%85%A9%E1%84%92%E1%85%AE_8.41.45.png)
    
- Rule 애노테이션 사용 예 → 제이유닛에서 각 테스트 메소드 앞에 해당 메소드를 실행하기 위한 규칙 지정
    - 예를 들어 TemporaryFolder라는 규칙을 사용하면 메소드가 끝나면 삭제될 임시 파일과 폴더를 만들 수 있다.
    - 규칙을 지정하려면 공개 필드(public(나 메소드 앞에 `@Rule` 를 붙여야 하지만 코틀린 필드는 기본적으로 비공개 이기 때문에 예외가 발생한다 따라서 정확한 대상에 적용하려면 `@get:Rule`을 사용해야 한다.
        
        ![스크린샷 2022-01-01 오후 8.45.24.png](10%20%E1%84%8B%E1%85%A2%E1%84%82%E1%85%A9%E1%84%90%E1%85%A6%E1%84%8B%E1%85%B5%E1%84%89%E1%85%A7%E1%86%AB%E1%84%80%E1%85%AA%20%E1%84%85%E1%85%B5%E1%84%91%E1%85%B3%E1%86%AF%E1%84%85%E1%85%A6%E1%86%A8%E1%84%89%E1%85%A7%E1%86%AB%200a0abf2cb16e4b23a184620f3d920a10/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA_2022-01-01_%E1%84%8B%E1%85%A9%E1%84%92%E1%85%AE_8.45.24.png)
        
- 자바에 선언된 애노테이션을 사용해 프로퍼티에 애노테이션을 붙이는 경우 기본적으로 프로퍼티의 필드에 그 애노테이션이 붙는다. 하지만 코틀린은 프로퍼티 대상을 직접 적용해서 애노테이션을 만들 수 있다.
- 사용 지점 대상을 지정할 때 지원하는 대상 목록
    - property → 프로퍼티 전체, 자바에서 선언된 애노테이션에는 이 사용 지점 대상을 사용할 수 없다.
    - field → 프로퍼티에 의해 생성되는 필드
    - get → 프로퍼티 게터
    - set → 프로퍼티 세터
    - receiver → 확장 함수나 프로퍼티의 수신 객체 파라미터
    - param → 생성자 파라미터
    - setparam → 세터 파라미터
    - delegate → 위임 프로퍼티의 위임 인스턴스를 담아둔 필드
    - file → 파일 안에 선언된 최상위 함수와 프로퍼티를 담아두는 클래스
        - file 대상을 사용하는 애노테이션은 package 선언 앞에서 파일의 최상위 수준에만 적용할 수 있다.
        - example `@file:JvmName("StringFunctions")` → 파일에 있는 최상위 선언을 담는 클래스의 이름을 바꿔주는 기능
- 자바와 달리 코틀린에서는 애노테이션 인자로 클래스나 함수 선언이나 타입 외에 임의의 식을 허용한다. (컴파일러 경고를 무시한기 위한 `@Supress` 애노테이션)
    
    ```kotlin
    fun test(list: List<*>( {
    	@Supress("UNCHECKED_CAST")
    	val strings = list as List<String>
    	// ...
    }
    ```
    

### 애노테이션을 활용한 JSON 직렬화 제어

- 애노테이션을 사용하는 고전적인 예제로 객체 직렬화 제어를 들 수 있다.
- 잭슨(jackson), 지슨(GSON), **제이키드**
- **직렬화(serialization)**는  객체를 저장장치에 저장하거나 네트워크를 통해 전송하기 위해 텍스트나 이진 형식으로 변환하는 것이다.
- 반대 과정인 **역직렬화(deserialization)**는 텍스트나 이진 형식으로 저장된 데이터로부터 원래의 객체를 만들어낸다.
    
    ```kotlin
    fun main() {
        val person = Person("Alice", 29)
        println(serialize(person))
    
        val json = """{"name":"Alice", "age": 29}"""
        println(deserialize<Person>(json))
    }
    
    >>> {"age": 29, "name": "Alice"}
    >>> Person(name=Alice, age=29)
    ```
    
    ![스크린샷 2022-01-01 오후 9.02.20.png](10%20%E1%84%8B%E1%85%A2%E1%84%82%E1%85%A9%E1%84%90%E1%85%A6%E1%84%8B%E1%85%B5%E1%84%89%E1%85%A7%E1%86%AB%E1%84%80%E1%85%AA%20%E1%84%85%E1%85%B5%E1%84%91%E1%85%B3%E1%86%AF%E1%84%85%E1%85%A6%E1%86%A8%E1%84%89%E1%85%A7%E1%86%AB%200a0abf2cb16e4b23a184620f3d920a10/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA_2022-01-01_%E1%84%8B%E1%85%A9%E1%84%92%E1%85%AE_9.02.20.png)
    
- 애노테이션을 활용해 객체를 직렬화하거나 역직렬화하는 방법을 제어할 수 있다.
    - `@JsonExclude` 애노테이션을 사용하면 직렬화나 역직렬화 시 그 프로퍼티를 무시할 수 있다.
    - `@JsonName` 애노테이션을 사용하면 프로퍼티를 표현하는 키/값 쌍의 키로 프로퍼티 이름 대신 애노테이션이 지정한 이름을 쓰게 할 수 있다.
        
        ```kotlin
        data class Person(
            @JsonName("alias") val firstName: String,
            @JsonExclude val age: Int
        )
        
        fun main() {
            val person = Person("test", 20)
            println(serialize(person))
        
            val json = """{"alias": "test", "age":20}"""
            println(deserialize<Person>(json))
        }
        
        >>> {"alias": "test"}
        >>> Person(firstName=test, age=20)
        ```
        

### 애노테이션 선언

- 애노테이션 선언의 예: `@annotation class JsonExclude` → 아무 파라미터도 없는 가장 단순한 애노테이션, 일반 클래스와의 차이는 class 앞에 annotation이 붙는다.
- 하지만 애노테이션 클래스는 오직 선언이나 식과 관련 있는 메타데이터의 구조를 정의하기 때문에 내부에 코드가 존재할 수 없다. (컴파일러에서 본문 정의못하도록 막음)
- 파라미터가 있는 애노테이션을 정의하려면 주 생성자에 파라미터를 선언해야 한다. → `annotation class JsonName(val name: String)` (모든 파라미터 앞에 val만 사용할 수 있다.)
- 자바에는 value라는 메소드가 있다. value는 특별하다. 어떤 애노테이션을 적용할 때 value를 제외한 모든 애트리뷰트에는 이름을 명시해야 한다. 반면 코틀린의 애노테이션 적용 문법은 일반적인 생성자 호출과 같다. → `@JsonName(name = “first_name”) = @JsonName(”first_name")`

### 메타애노테이션: 애노테이션을 처리하는 방법 제어

- 자바와 마찬가지로 코틀린 애노테이션에도 애노테이션을 붙일 수 있다. 애노테이션 클래스에 적용할 수 있는 애노테이션을 **메타애노테이션(meta-annotation)**이라고 부른다.
- 메타애노테이션들은 컴파일러가 애노테이션을 처리하는 방법을 제어한다.
    - `@Target` → 애노테이션을 적용할 수 있는 요소의 유형을 정의한다. (지정하지 않으면 모든 선언에 적용가능)
        
        ```kotlin
        // @Target(AnnotationTarget.PROPERTY, AnnotationTarget.CLASS)
        @Target(AnnotationTarget.PROPERTY) // 대상 지정
        annotation class JsonExclude
        ```
        
    - `@Retention` → 정의 중인 애노테이션 클래스를 소스 수준에서만 유지할지, .class 파일에 저장할지, 실행 시점에 리플렉션을 사용해 접근할 수 있게 할지를 지정하는 메타애노테이션이다. 자바 컴파일러는 기본적으로 애노테이션을 .class 파일에는 저장하지만 런타임에는 사용할 수 없게 한다. 하지만 대부분의 애노테이션은 런타임에도 사용할 수 있어야 하므로 코틀린에서는 RUNTIME으로 지정한다.

### 애노테이션 파라미터로 클래스 사용

- 어떤 클래스를 선언 메타데이터로 참조할 수 있는 기능이 필요할 때도 있다.
- 클래스 참조를 파라미터로 하는 애노테이션 클래스를 선언하면 그런 기능을 사용할 수 있다.
    
    ```kotlin
    @Target(AnnotationTarget.PROPERTY)
    annotation class DeserializeInterface(val targetClass: KClass<out Any>)
    
    data class Person(
        val name: String,
        @DeserializeInterface(CompanyImpl::class) val company: Company
    		// 일반적으로 클래스를 가리키려면 클래승 이름 뒤에 ::class 키워드를 붙여야 한다.
    )
    
    interface Company {
        val name: String
    }
    
    data class CompanyImpl(override val name: String) : Company
    ```
    
    - KClass는 자바 java.lang.Class 타입과 같은 역할을 하는 코틀린 타입이다. 코틀린 클래스에 대한 참조를 저장할 때 KClass 타입을 사용한다.
    - CompanyImpl::classd의 타입은 KClass<CompanyImpl>이며 이는 KClass<out Any>의 하위 타입이다. out 키워드가 있으면 모든 코틀린 타입 T에 대해 KClass<T>가 KClass<out Any>의 하위 타입이 된다(공변성)
        
        ![스크린샷 2022-01-01 오후 9.33.18.png](10%20%E1%84%8B%E1%85%A2%E1%84%82%E1%85%A9%E1%84%90%E1%85%A6%E1%84%8B%E1%85%B5%E1%84%89%E1%85%A7%E1%86%AB%E1%84%80%E1%85%AA%20%E1%84%85%E1%85%B5%E1%84%91%E1%85%B3%E1%86%AF%E1%84%85%E1%85%A6%E1%86%A8%E1%84%89%E1%85%A7%E1%86%AB%200a0abf2cb16e4b23a184620f3d920a10/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA_2022-01-01_%E1%84%8B%E1%85%A9%E1%84%92%E1%85%AE_9.33.18.png)
        

### 애노테이션 파라미터로 제네릭 클래스 받기

- 기본적으로 제이키드는 원시 타입이 아닌 프로퍼티를 중첩된 객체로 직렬화 한다. 이런 기본 동작을 변경하고 싶으면 값을 직렬화하는 로직을 직접 제공하면 된다.
    
    ```kotlin
    @Target(AnnotationTarget.PROPERTY)
    // ValueSerializer 타입을 참조하려면 항상 타입 인자를 제공해야 한다. 하지만 이 애노테이션이 
    // 어떤 타입에 대해 쓰일지 전혀 알 수 없으므로 스타 프로젝을 사용한다.
    annotation class CustomSerializer(val serializerClass: KClass<out ValueSerializer<*>>)
    
    data class Person(
        val name: String,
        @CustomSerializer(DateSerializer::class) val birthData: Date
    )
    
    // ValueSerializer 클래스는 제네릭 클래스라서 타입 파라미터가 있다.
    interface ValueSerializer<T> {
        fun toJsonValue(value: T): Any?
        fun fromJsonValue(jsonValue: Any?): T
    }
    
    object DateSerializer : ValueSerializer<Date> {
        private val dateFormat = SimpleDateFormat("dd-mm-yyyy")
    
        override fun toJsonValue(value: Date): Any? {
            return dateFormat.format(value)
        }
    
        override fun fromJsonValue(jsonValue: Any?): Date {
            return dateFormat.parse(jsonValue as String)
        }
    }
    
    fun main() {
        val person = Person("test", SimpleDateFormat("dd-mm-yyyy").parse("13-02-1987"))
        println(serialize(person))
    
        val json = """{"birthData": "13-02-1987", "name": "test"}"""
        println(deserialize<Person>(json))
    }
    
    >>> {"birthData": "13-02-1987", "name": "test"}
    >>> Person(name=test, birthData=Tue Jan 13 00:02:00 KST 1987)
    ```
    
- 클래스를 인자로 받아야 하면 애노테이션 파라미터 타입에 KClass<out 허용할 클래스 이름>을 쓴다. 제네릭 클래스를 인자로 받아야 하면 KClass<out 허용할 클래스 이름<*>> 처럼 허용할 클래스의 이름 뒤에 스타 프로젝션을 덧붙인다.