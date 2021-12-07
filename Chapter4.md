# 04. 클래스, 객체, 인터페이스

[노션 링크(노션으로 보는 것이 더 깔끔합니다.)](https://www.notion.so/moochipark/04-dc2a5cff20d24b589f5724c1235fbc58)

**4장에서 다루는 내용**  
• 클래스와 인터페이스  
• 뻔하지 않은 생성자와 프로퍼티  
• 데이터 클래스  
• 클래스 위임  
• object 키워드 사용  

여기선 코틀린 클래스를 다루는 방법을 더 깊이 이해하는 장이다.

코틀린의 클래스와 인터페이스는 자바의 것과는 약간 다르다.
예를 들어, 인터페이스에 프로퍼티 선언이 들어갈 수 있다.

자바와 달리 코틀린의 선언은 기본적으로 final이며 public이다.
게다가 중첩 클래스는 기본적으로는 내부 클래스가 아니다. 즉, 코틀린 중첩 클래스에는 외부 클래스에 대한 참조가 없다.

짧은 주 생성자 구문으로도 거의 모든 경우를 잘 처리할 수 있지만, 복잡한 초기화 로직을 수행하는 경우를 대비해 완전한 문법도 있다.
프로퍼티도 마찬가지로 간결한 구문으로 충분히 제 기능을 하지만, 필요하면 접근자를 직접 정의할 수 있다.

코틀린 컴파일러는 번잡스러움을 피하기 위해 유용한 메서드를 자동으로 만들어준다.
클래스를 data로 선언하면 일부 표준 메서드를 생성해준다.

그리고 코틀린 언어가 제공하는 위임(delegation)을 사용하면 위임을 처리하기 위한 준비 메서드를 직접 작성할 필요가 없다.

또한 클래스와 인스턴스를 동시에 선언하면서 만들 때 쓰는 object 키워드에 대해 알아본다.
싱글턴 클래스, 동반 객체(companion object), 객체 식(object expression = java 익명 클래스)을 표현할 때 사용한다.

먼저 클래스와 인터페이스에 대해 알아보고 코틀린에서 클래스 계층을 정의할 때 주의해야 할 점을 알아보자.

# 1. 클래스 계층 정의


코틀린의 가시성/접근 변경자는 자바와 비슷하지만 기본 가시성이 다르다. 또한 클래스 상속을 제한하는 sealed 변경자도 추가되었다.

## 1.1 코틀린 인터페이스

코틀린 인터페이스는 자바8 인터페이스와 비슷하다.
코틀린 인터페이스 안에는 추상 메서드뿐 아니라 구현이 있는 메서드(자바8의 디폴트 메서드와 비슷하다)도 정의할 수 있다.

다만 인터페이스에는 아무런 상태(필드)도 저장될 수 없다. 

```kotlin
interface Clickable {
    fun click()
}
```

위 코드는 click이라는 추상 메서드가 있는 인터페이스를 정의한다.
이 인터페이스를 구현하는 모든 비추상 클래스(구현 클래스)는 click에 대한 구현을 제공해야 한다.

```kotlin
class Button : Clickable {
    override fun click() = println("I was clicked")
}

>>> Button().click()
I was clicked
```

자바에서는 extends와 implements 키워드를 사용하지만, 
코틀린에서는 클래스 이름 뒤에 콜론을 붙이는 것으로 확장과 구현을 모두 처리한다.

자바와 마찬가지로 클래스는 인터페이스를 원하는 만큼 마음대로 구현할 수 있지만, 클래스는 오직 하나만 확장할 수 있다.

자바의 @Override 애노테이션과 비슷한 override 변경자는 상위 클래스나 상위 인터페이스에 있는 프로퍼티나 메서드를 재정의 한다는 뜻이다. 
**하지만 자바와 달리 코틀린에서는 override 변경자를 반드시 사용해야 한다.**
이는 실수로 상위 클래스의 메서드를 오버라이드 하는경우를 방지해준다.

상위 클래스에 있는 메서드와 시그니처가 같은 메서드를 우연히 하위 클래스에서 선언하는 경우 컴파일이 안되기 때문에 
override를 붙이거나 메서드 이름을 바꿔야만 한다.

인터페이스 메서드도 디폴트 구현을 제공할 수 있다.
default를 붙여야하는 자바와 달리 그냥 메서드 본문을 추가하면 된다.

```kotlin
interface Clickable {
    fun click()
    fun showOff() = println("I'm clickable!")
}
```

이 인터페이스를 구현하는 클래스는 click에 대한 구현을 제공해야 하는 반면,
showOff 메서드의 경우 재정의할 수도 있고, 디폴트 구현을 사용할 수도 있다.

다음의 경우를 보자.

```kotlin
interface Focusable {
    fun setFocus(b: Boolean) =
        println("I ${if (b) "got" else "lost"} focus.")
    
    fun showOff() = println("I'm focusable!")
}
```

한 클래스에서 이 두 인터페이스를 함께 구현하면 어떻게 될까?
정답은 어느쪽도 선택되지 않는다.
클래스가 구현하는 두 상위 인터페이스에 정의된 구현을 대체할 오버라이딩 메서드를 직접 제공하지 않으면 컴파일러 오류가 발생한다.

```kotlin
Class 'Button' must override public open fun showOff(): Unit defined 
in ch04ClassObjectInterface.`interface`.Clickable because it inherits multiple interface methods of it
```

코틀린 컴파일러는 두 메서드를 아우르는 구현을 하위 클래스에 직접 구현하게 강제한다.

```kotlin
class Button : Clickable, Focusable {
    override fun click() = println("I was clicked")

    override fun showOff() {
        super<Clickable>.showOff()
        super<Focusable>.showOff()
    }
}
```

상위 타입의 구현을 호출할 때는 자바와 마찬가지로 super를 사용하지만, 구체적으로 타입을 지정하는 문법이 다르다.

자바에서는 Clickable.super.showOff() 처럼 기반 타입을 명시하지만,
코틀린에서는 <> 기호 안에 기반 타입의 이름을 지정한다.

이제 이 클래스의 인스턴스를 만들고 구현대로 상속한 모든 메서드를 호출하는지 검증해보자.

```kotlin
fun main() {
    val button = Button()
    button.showOff()
    button.setFocus(true)
    button.click()
}

>>>
I'm clickable!
I'm focusable!
I got focus.
I was clicked
```

Button 클래스는 Focusable 인터페이스 안에 선언된 setFocus의 구현을 자동으로 상속한다.

## 1.2 open, final, abstract 변경자: 기본적으로 final

자바처럼 기본적으로 상속이 가능하면 편리한 경우도 많지만 문제가 생기는 경우도 많다.

취약한 기반 클래스(fragile base class)라는 문제는 하위 클래스가 기반 클래스를 변경함으로써 깨져버린 경우를 말한다.
어떤 클래스가 자신을 상속하는 방법에 대한 정확한 규칙(어떤 메서드를 어떻게 오버라이드해야 하는지 등)을 제공하지 않는다면
그 클래스의 클라이언트는 기반 클래스를 작성한 사람의 의도와 다른 방식으로 오버라이드할 위험이 있다.

모든 하위 클래스를 분석하는 것은 불가능하므로 기반 클래스를 변경하는 경우 하위 클래스의 동작이 예기치 않게 바뀔 수 있다는 면에서 기반 클래스는 취약하다.

이 문제를 해결하기 위해 프로그래밍 기법에 대한 책 중 가장 유명한 조슈아 블로크(Joshua Block)가 쓴 이펙티브 자바에서는

> "상속을 위한 설계와 문서를 갖추거나, 그럴 수 없다면 상속을 금지하라" - Josua Block
> 

고 조언한다. 이는 특별히 하위 클래스에서 오버라이드하게 의도된 메서드가 아니라면 모두 final로 만들라는 뜻이다.

코틀린도 마찬가지의 철학을 따른다. 자바의 클래스와 메서드는 기본적으로 상속에 대해 열려있지만 코틀린은 기본적으로 final이다.

어떤 클래스의 상속을 허용하려면 클래스 앞에 open 변경자를 붙여야 한다.
그와 더불어 오버라이드를 허용하고 싶은 메서드나 프로퍼티의 앞에도 open 변경자를 붙여야 한다.

```kotlin
open class RichButton : Clickable {  // 이 클래스는 열려있다. 다른 클래스가 이 클래스를 상속할 수 있다.
    fun disable() {}  // 이 함수는 final이다. 하위 클래스가 이 메서드를 override할 수 없다.
    
    open fun animate() {}  // 이 함수는 열려있다. 하위 클래스에서 이 메서드를 override해도 된다.
    
    override fun click() {}  // 이 함수는 (상위 클래스에서 선언된) 열려있는 메서드를 override한다. override한 메서드는 기본적으로 열려있다.
}
```

기반 클래스나 인터페이스의 멤버를 오버라이드하는 경우 그 메서드는 기본적으로 열려있다.

```kotlin
open class RichButton : Clickable {
 
   final override fun click() {}  // 여기 있는 final은 쓸데 없이 붙은 중복이 아니다. 
								  // final이 없는 override 메서드나 프로퍼티는 기본적으로 열려있다.
}
```

**열린 클래스와 스마트 캐스트**
클래스의 기본적인 상속 가능 상태를 final로 함으로써 얻을 수 있는 큰 이익은 다양한 경우에 스마트 캐스트가 가능하다는 점이다.
클래스 프로퍼티의 경우 이는 val이면서 커스텀 접근자가 없는 경우에만 스마트 캐스트를 쓸 수 있다는 것이다.
이 요구 사항은 또한 프로퍼티가 final이어야만 한다는 뜻이기도 하다.
프로퍼티가 final이 아니라면 그 프로퍼티를 다른 클래스가 상속하면서 커스텀 접근자를 정의함으로써 스마트 캐스트의 요구 사항을 깰 수 있다.
코틀린의 경우 프로퍼티가 기본적으로 final이므로 고민할 필요 없이 대부분의 프로퍼티를 스마트 캐스트에 활용할 수 있다.

자바처럼 코틀린에서도 클래스를 abstract로 선언할 수 있다.

- abstract로 선언한 추상 클래스는 인스턴스화할 수 없다.
- 추상 클래스에는 구현이 없는 추상 멤버가 있기 때문에 하위 클래스에서 오버라이드해야만 한다.
- 추상 멤버는 항상 열려있다. 따라서 open 변경자를 명시할 필요가 없다.

```kotlin
abstract class Animated {  // 이 클래스는 추상클래스다. 이 클래스의 인스턴스를 만들 수 없다.
    abstract fun animate()  // 이 함수는 추상 함수다. 이 함수에는 구현이 없다. 하위 클래스에서는 이 함수를 반드시 오버라이드해야 한다.
    
    open fun stopAnimating() {}  // 추상 클래스에 속했더라도 비추상 함수는 기본적으로 final이지만 원한다면 open으로 오버라이드를 허용할 수 있다.
    
    fun animateTwice() {}
}
```

[코틀린 클래스 내에서의 상속 제어 변경자(access modifier)](https://www.notion.so/3f7be7bfbb0a4d70be759946146517b4)

인터페이스 멤버의 경우는 위의 final, open, abstract를 사용하지 않는다. 

## 1.3 가시성 변경자: 기본적으로 공개

기본적으로 코틀린 가시성 변경자는 자바와 비슷하다.

자바와 비슷한 public, protected, private 변경자가 있다.

하지만 코틀린의 기본 가시성은 자바와 다르게 public이다.

자바의 기본 가시성인 package-private(패키지 전용)은 코틀린에 없다.

코틀린은 패키지를 네임스페이스를 관리하기 위한 용도로만 사용하기 때문이다. 따라서 패키지를 가시성 제어에 사용하지 않는다.

이를 위한 대안으로 코틀린에는 internal이라는 새로운 가시성 변경자를 도입했다.

internal은 '모듈 내부에서만 볼 수 있음'이라는 의미다. 여기서의 모듈은 한 번에 같이 컴파일되는 코틀린 파일들을 의미한다.
(예: intelliJ 모듈, maven, gradle 프로젝트 등)

모듈 내부 가시성은 모듈의 구현에 대해 진정한 캡슐화를 제공한다는 장점이 있다.

자바에서는 패키지가 같은 클래스를 선언하기만 하면 어떤 프로젝트의 외부에 있는 코드라도 패키지 내부에 있는 패키지 전용 선언에 쉽게 접근할 수 있다.

그래서 모듈의 캡슐화가 쉽게 깨진다.

또 다른 차이는 코틀린에서는 최상위 선언(클래스, 함수, 프로퍼티)에 대해 private 가시성을 허용한다는 점이다.

비공개 가시성인 최상위 선언은 그 선언이 포함된 파일 내부에서만 사용할 수 있다.

이는 하위 시스템의 자세한 구현 사항을 외부에 감추고 싶을 때 유용한 방법이다.

[코틀린의 가시성 변경자(visibily modifier)](https://www.notion.so/592d6ef5277942749910158970c81a1b)

가시성 규칙을 위반하는 예시를 보자.

```kotlin
internal open class TalkativeButton : Focusable {
    private fun yell() = println("Hey!")
    protected fun whisper() = println("Let's talk!")
}

fun TalkativeButton.giveSpeech() {  // 오류: public 멤버가 자신의 internal 수신 타입인 TalkativeButton을 노출함.
    yell()                          // 오류: yell에 접근할 수 없음. yell은 TalktiveButton의 private 멤버임.
    whisper()                       // 오류: whisper에 접근할 수 없음. whisper는 TalktiveButton의 protected 멤버임.
}
```

코틀린은 public 함수인 giveSpeech 안에서 가시성이 더 낮은(이 경우 internal) 타입인 TalkativeButton을 참조하지 못하게 한다.

이는 어떤 클래스의 기반인 타입이거나 타입 파라미터에 들어있는 타입의 가시성보다 높아야 한다는 일반적인 규칙에 해당한다.

여기서 컴파일 오류를 없애려면 확장 함수의 가시성을 internal로 바꾸거나 기반 클래스의 가시성을 public으로 바꿔야 한다.

(private, protected인 멤버는 여전히 접근할 수 없다.)

자바의 경우 같은 패키지 안에서 protected 멤버에 접근할 수 있었지만, 코틀린에서는 그렇지 않다.

코틀린의 가시성 규칙이 더 단순한데, protected 멤버는 오직 어떤 클래스나 그 클래스를 상속한 클래스 안에서만 보인다.

또다른 규칙의 차이로 코틀린에서는 외부 클래스가 내부 클래스, 중첩 클래스의 private 멤버에 접근할 수 없다는 점인데, 다음 절에서 자세히 알아보자.  
  

## 1.4 내부 클래스와 중첩된 클래스: 기본적으로 중첩 클래스

자바처럼 코틀린에서도 클래스 안에 다른 클래스를 선언할 수 있다.

자바와의 차이는 코틀린의 중첩 클래스(nested class)는 명시하지 않는 한 외부 클래스 인스턴스에 대한 접근 권한이 없다는 점이다.

직렬화할 수 있는 View 요소를 예시로 들어보자.

```kotlin
interface State : Serializable

interface View {
    fun getCurrentState(): State
    fun restoreState(state: State) {}
}
```

View의 일종인 Button이 있다고 해보자. Button 클래스의 상태를 저장하는 클래스는 Button 클래스 내부에 선언하면 편리할 것이다.

자바에서 이를 어떻게 하는지 보자.

```java
public class Button implements View {
    @Override
    public State getCurrentState() {
        return new ButtonState();
    }

    @Override
    public void restoreState(final State state) { /* ... */ }
    
    public class ButtonState implements State { /* ... */ }
}
```

State 인터페이스를 구현한 ButtonState 클래스를 정의해서 Button에 대한 구체적인 정보를 저장한다.

이를 getCurrentState 메서드 안에서 인스턴스를 만들어서 정보를 관리한다.

하지만 이 코드는 문제가 있다. 선언한 버튼의 상태를 직렬화하면 NotSerializableException: Button이라는 오류가 발생한다.

직렬화하려는 변수는 ButtonState 타입의 인스턴스였는데, 왜 Button을 직렬화할 수 없다는 오류가 발생할까?

자바에서 다른 클래스 안에 정의한 클래스는 자동으로 내부 클래스(inner class)가 된다는 사실을 기억한다면 어디가 잘못된 건지 명확히 알 수 있다.

ButtonState 클래스는 외부의 Button 클래스에 대한 참조를 묵시적으로 포함한다.

그 참조로 인해 Button을 직렬화할 수 없으므로 문제가 발생하는 것이다.

자바의 경우는 이런 중첩 클래스를 static으로 선언하면 그 클래스를 둘러싼 바깥쪽 클래스에 대한 묵시적인 참조가 사라진다.

코틀린의 경우 중첩 클래스가 기본적으로 동작하는 방식이 자바와 정반대이다.

```kotlin
class Button : View {
    override fun getCurrentState(): State = ButtonState()

    override fun restoreState(state: State) { /*...*/ }
    
    class ButtonState : State { /*...*/ }  // 이 클래스는 자바의 정적 중첩 클래스와 대응한다.
}
```

코틀린 중첩 클래스에 아무런 변경자가 붙지 않으면 자바 static 중첩 클래스와 같다.

이를 내부 클래스로 변경해서 바깥쪽 클래스에 대한 참조를 포함하게 만들고 싶다면 inner 변경자를 붙여야 한다.

[자바와 코틀린의 중첩 클래스와 내부 클래스 관계](https://www.notion.so/3b76c67f28ef4dc4bfce4b34e1f5deb4)

![Untitled](https://s3.us-west-2.amazonaws.com/secure.notion-static.com/a2927a1c-f47a-4aac-a36d-89125882d3c4/Untitled.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=AKIAT73L2G45O3KS52Y5%2F20211025%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Date=20211025T065509Z&X-Amz-Expires=86400&X-Amz-Signature=f9ddc3127f4e6ef7842cf1c43b3423e5ae73677b972f0726ec78a158dcb27fe3&X-Amz-SignedHeaders=host&response-content-disposition=filename%20%3D%22Untitled.png%22)

중첩 클래스 안에는 바깥쪽 클래스에 대한 참조가 없지만 내부 클래스에는 있다.

코틀린에서 바깥쪽 클래스의 인스턴스를 가리키는 참조를 표기하는 방법도 자바와 다르다.

```kotlin
class Outer {
    inner class Inner {
        fun getOuterReference(): Outer = this@Outer
    }
}
```

```java
public class OuterJava {
    class InnerJava {
        public OuterJava getOuterReference() {
            return OuterJava.this;
        }
    }
}
```

이제는 코틀린 중첩 클래스를 유용하게 사용하는 예시를 보자.

클래스 계층을 만들되 그 계층에 속한 클래스의 수를 제한하고 싶은 경우 중첩 클래스를 쓰면 편리하다.

## 1.5 봉인된 클래스: 클래스 계층 정의 시 계층 확장 제한

상위 클래스 혹은 인터페이스 아래로 하위 클래스가 있을 때, 다양한 패턴 매칭으로 처리할 수가 있다.

```kotlin
interface Expr

class Num(val value: Int) : Expr
class Sum(val left: Expr, val right: Expr) : Expr

fun eval(e: Expr): Int =
    when (e) {
        is Num -> e.value
        is Sum -> eval(e.right) + eval(e.left)
        else -> throw IllegalArgumentException("Unknown expression")
    }
```

코틀린 컴파일러는 when을 사용해 open 클래스 타입 혹은 인터페이스 타입을 검사할 때 반드시 디폴트 분기인 else 분기를 덧붙이게 강제한다.

이 예제의 else 분기에서는 반환할 만한 의미 있는 값이 없어 예외를 던지게 했다.

항상 이처럼 디폴트 분기를 추가하는 것이 안전하겠지만 편하지는 않다.

디폴트 분기가 있을 경우 클래스 계층에 새로운 하위 클래스를 추가하더라도 컴파일러가 when이 모든 경우를 처리하는지 검사할 수가 없다.

혹시나 새로운 클래스 처리를 잊어버렸더라도 디폴트 분기가 선택되기 때문에 심각한 버그가 발생할 수도 있다.

코틀린은 이런 문제에 대한 해법으로 sealed 클래스를 내놓았다.

상위 클래스에 sealed 변경자를 붙이면 그 상위 클래스를 상속한 하위 클래스 정의를 제한할 수 있다.

sealed 클래스의 하위 클래스를 정의할 때는 상위 클래스 안에 중첩시켜야 한다 (코틀린 1.1부터는 같은 파일 내에서도 하위 클래스를 정의할 수 있다).

```kotlin
**sealed class Expr {
    class Num(val value: Int) : Expr()
    class Sum(val left: Expr, val right: Expr) : Expr()
}**

fun eval(e: Expr): Int =
    when (e) {
        is Expr.Num -> e.value
        is Expr.Sum -> eval(e.right) + eval(e.left)
    }
```

when 식에서 sealed 클래스의 모든 하위 클래스를 처리한다면 디폴트 분기가 필요 없다.

sealed로 표시된 클래스는 자동으로 open이므로 open 변경자를 붙일 필요가 없다.

내부적으로 sealed 클래스는 private 생성자를 가진다. 따라서 그 생성자는 같은 파일, 같은 클래스 내부에서만 호출할 수 있다.

즉, 같은 파일 혹은 sealed 클래스 내부에서만 상속을 받을 수 있다는 의미이다.

# 2. 뻔하지 않은 생성자와 프로퍼티를 갖는 클래스 선언


코틀린도 자바와 비슷하게 생성자를 하나 이상 선언할 수 있다.

하지만 코틀린은 주 생성자와 부 생성자를 구분한다.

또한 초기화 블록을 통해 초기화 로직을 추가할 수 있다.

## 2.1 클래스 초기화: 주 생성자와 초기화 블록

```kotlin
class User(val nickname: String)
```

위처럼 클래스 이름 뒤에 오는 괄호로 둘러싸인 코드를 **주 생성자**라고 부른다.

위 선언을 같은 목적을 달성하는 가장 명시적인 선언으로 풀어서 실제론 어떤 일이 벌어지는지 보자.

```kotlin
class User constructor(nickname: String) {
    val nickname: String

    init {
        this.nickname = nickname
    }
}
```

constructor 키워드는 주 생성자나 부 생성자 정의를 시작할 때 사용한다.

init 키워드는 초기화 블록을 시작한다.

초기화 블록에는 클래스의 객체가 만들어질 때 실행될 초기화 코드가 들어간다.

초기화 블록은 주 생성자와 함께 사용될 수 있는데, 주 생성자는 별도의 코드를 포함할 수 없으므로 초기화 블록이 필요한 경우가 있다.

주 생성자에 별도의 애노테이션이나 가시성 변경자가 없다면 constructor를 생략할 수 있다. 위 코드를 간략화하면 다음과 같다.

```kotlin
class User(nickname: String) {
    val nickname: String = nickname
}
```

위 세 가지 User 선언은 모두 같은 동작을 수행한다.

함수 파라미터와 마찬가지로 생성자 파라미터에도 디폴트 값을 정의할 수 있다.

```kotlin
class User(
    val nickname: String,
    val isSubscribed: Boolean = true,
)
```

클래스에 기반 클래스가 있다면 주 생성자에서 기반 클래스의 생성자를 호출해야 할 필요가 있다.

기반 클래스를 초기화하려면 기반 클래스 이름 뒤에 생성자 인자를 넘기면 된다.

```kotlin
open class User(val nickname: String) { }

class TwitterUser(nickname: String) : User(nickname) {
```

코틀린에서는 기반 클래스를 상속을 받는 하위 클래스는 기반 클래스의 생성자를 호출해야 한다.

```kotlin
class RadioButton : Button()
```

이 규칙으로 인해 기반 클래스의 이름 뒤에는 꼭 빈 괄호가 들어간다. 물론 생성자 인자가 있다면 괄호 안에 인자가 들어간다.

반면 인터페이스는 생성자가 없기 때문에 어떤 클래스가 인터페이스를 구현하는 경우 인터페이스 이름 뒤에 아무 괄호도 없다.

이를 통해 기반 클래스와 인터페이스를 쉽게 구별할 수 있다.

어떤 클래스를 외부에서 인스턴스화하지 못하게 막고 싶다면 모든 생성자를 private으로 만들면 된다.

```kotlin
class Secretive private constructor() { }
```

실제로 대부분의 경우 클래스의 생성자는 코틀린의 주 생성자 구문만으로 충분할 만큼 단순하다.

하지만 복잡한 생성자가 생기는 경우를 대비해 코틀린은 다양한 생성자를 정의할 수 있게 해준다.

## 2.2 부 생성자: 상위 클래스를 다른 방식으로 초기화

일반적으로 코틀린에서 생성자가 여럿 있는 경우는 자바보다 훨씬 적다.

자바에서 오버로드한 생성자가 필요한 상황 중 상당수는 코틀린의 디폴트 파라미터 값과 이름 붙인 인자 문법을 사용해 해결할 수 있다.

그래도 생성자가 여럿 필요한 경우가 가끔 있다.

예를 들어 자바에서 생성자가 2개인 View 클래스를 코틀린으로 정의해보자.

```kotlin
open class View {
    constructor(ctx: Context) { }
    constructor(ctx: Context, attr: AttributeSet) { }
}
```

이 클래스는 주 생성자를 선언하지 않고, 부 생성자만 2가지 선언한다.

필요에 따라 얼마든지 부 생성자를 많이 선언해도 된다.

이 클래스를 확장하면서 똑같이 부 생성자를 정의해보자.

```kotlin
class MyButton : View {
    constructor(ctx: Context) : super(ctx) { }
    constructor(ctx: Context, attr: AttributeSet) : super(ctx, attr) { }
}
```

여기서 두 부 생성자는 super() 키워드를 통해 자신에 대응하는 상위 클래스 생성자를 호출한다.

![Untitled](https://s3.us-west-2.amazonaws.com/secure.notion-static.com/d692d8a2-aad4-4cde-b6cf-f3c418094002/Untitled.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=AKIAT73L2G45O3KS52Y5%2F20211025%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Date=20211025T065545Z&X-Amz-Expires=86400&X-Amz-Signature=3a7ef9eb8de62ebb5599af3ebd089d9334de7a0955c38b8c3f5bb1321c4b1195&X-Amz-SignedHeaders=host&response-content-disposition=filename%20%3D%22Untitled.png%22)

상위 클래스의 여러 생성자에게 객체 생성 위임하기

그림에서 생성자가 상위 클래스의 생성자에게 객체 생성을 위임한다는 것을 나타낸다.

자바와 마찬가지로 생성자에서 this()를 통해 클래스 자신의 다른 생성자를 호출할 수 있다.

```kotlin
class MyButton : View {
    constructor(ctx: Context) : this(ctx, MY_STYLE) { }
    constructor(ctx: Context, attr: AttributeSet) : super(ctx, attr) { }
}
```

생성자 중 하나가 파라미터의 디폴트 값을 넘겨서 같은 생성자에게 생성을 위임한다.

![Untitled](https://s3.us-west-2.amazonaws.com/secure.notion-static.com/b1a2e1b3-73b0-4ea2-a9af-c574d2ab5df2/Untitled.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=AKIAT73L2G45O3KS52Y5%2F20211025%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Date=20211025T065626Z&X-Amz-Expires=86400&X-Amz-Signature=0a8654c9f3dfaaaf979ba2134f03cf097a29ad9cdffdca7381450fcec353d71f&X-Amz-SignedHeaders=host&response-content-disposition=filename%20%3D%22Untitled.png%22)

두 번째 생성자는 여전히 super()를 호출하여 상위 클래스에게 생성을 위임한다.

결국 각 부 생성자들에서 객체 생성을 따라가다보면 그 끝에는 상위 클래스 생성자를 호출해야 한다는 의미다.

부 생성자가 필요한 주된 이유는 주로 자바와의 상호 운용성이다. 

물론 인스턴스를 생성할 때 파라미터 목록이 다른 생성 방법이 여럿 존재하는 경우 부 생성자를 사용할 수밖에 없다. 

이는 뒤의 동반 객체에서 다시 알아보자.

## 2.3 인터페이스에 선언된 프로퍼티 구현

코틀린에서는 인터페이스에 추상 프로퍼티 선언을 넣을 수 있다.

```kotlin
interface User {
  val nickname: String
}
```

이는 User 인터페이스의 구현 클래스가 nickname의 값을 얻을 수 있는 방법을 제공해야 한다는 것이다.

인터페이스에 있는 프로퍼티 선언에는 뒷받침하는 필드(백킹 필드)나 게터 등의 정보가 들어있지 않다.

사실 인터페이스는 아무 상태도 포함할 수 없으므로 상태를 저장할 필요가 있다면 인터페이스를 구현한 하위 클래스에서 상태 저장을 위한 프로퍼티를 만들어야 한다.

위 인터페이스를 구현 가능한 몇 가지 방법을 알아보자.

```kotlin
class PrivateUser(override val nickname: String) : User  // 주 생성자에 있는 프로퍼티

class SubscribingUser(val email: String) : User {
  override val nickname: String
    get() = email.substringBefore('@')  // 커스텀 게터

class FacebookUser(val accountId: Int) : User {
  override val nickname = getFacebookName(accountId)  // 프로퍼티 초기화 식
```

PrivateUser는 주 생성자 안에 프로퍼티를 직접 선언하는 간결한 구문을 사용했다.

이 프로퍼티는 User의 추상 프로퍼티를 구현하고 있으므로 override를 표시해야 한다.

SubscribingUser는 커스텀 게터로 nickname 프로퍼티를 설정한다.

이 프로퍼티는 뒷받침하는 필드에 값을 저장하지 않고 매번 이메일 주소에서 별명을 계산해 반환한다.

FacebookUser에서는 초기화 식으로 nickname 값을 초기화한다.

getFacebookName이라는 함수가 비용이 많이 들 수 있어서 객체를 초기화하는 단계에 한 번만 호출하도록 되어있다.

두 번째와 세 번째 방식은 비슷해보이지만 커스텀 게터는 매번 호출될 때마다 값을 계산하고,

객체 초기화 식은 객체 초기화 시에 계산한 데이터를 뒷받침하는 필드에 저장했다가 불러오는 방식을 사용한다.

인터페이스에 뒷받침하는 필드가 없다고 했지만, 이를 통해 알 수 있는 것은 커스텀 게터, 세터는 선언이 가능하다는 것이다.

당연하게도 이런 게터와 세터는 뒷받침하는 필드를 참조할 수 없다.

뒷받침하는 필드가 있다면 인터페이스에 상태를 추가하는 셈이다. 인터페이스는 상태를 저장할 수 없다.

```kotlin
interface User {
  val email: String,
  val nickname: String
    get() = email.substringBefore('@')  // 프로퍼티에 뒷받침하는 필드가 없다. 대신 매번 결과를 계산해 돌려준다.
}
```

이 인터페이스에는 추상 프로퍼티인 email과 커스텀 게터가 있는 nickname 프로퍼티가 함께 있다.

하위 클래스는 추상 프로퍼티인 email을 반드시 오버라이드해야 하지만, nickname은 오버라이드하지 않고 상속할 수 있다.

인터페이스에 선언된 프로퍼티와 달리 클래스에 구현된 프로퍼티는 뒷받침하는 필드에 원하는 대로 접근할 수 있다.

접근자에서 뒷받침하는 필드를 가리키는 방법을 보자.

## 2.4 게터와 세터에서 뒷받침하는 필드에 접근

값을 저장하는 동시에 특정 로직이 수행되어야 한다면 접근자 안에서 백킹 필드에 접근할 수 있어야 한다.

```kotlin
class User(val name: String) {
  var address: String = "unspecified"
  set(value: String) {
    println("""
      Address was changed for $name:
      "$field" -> "$value".""".trimIndent())  // 백킹 필드 값 읽기
      field = value
  }
}
```

코틀린에서 프로퍼티의 값을 바꿀 때는 user.address = "new value"처럼 필드 설정 구문을 사용한다.

이 구문은 내부적으로 address의 세터를 호출한다.

접근자의 본문에서는 field라는 특별한 식별자를 통해 백킹 필드에 접근할 수 있다.

게터에서는 field 값을 읽을 수만 있고, 세터에서는 읽거나 쓸 수 있다.

변경 가능 프로퍼티의 게터와 세터 중 한쪽만 직접 정의할 수 있다는 점을 기억하자.

클래스의 프러퍼티를 사용하는 쪽에서 프로퍼티를 읽는 방법이나 쓰는 방법은 백킹 필드의 유무와는 관계가 없다.

컴파일러는 디폴트 접근자 구현을 사용하건, 커스텀 접근자를 사용하건 관계없이  게터나 세터에서 field를 사용하는 프로퍼티에 대해 백킹 필드를 생성해준다.

다만 field를 사용하지 않는 커스텀 접근자 구현을 정의한다면 백킹 필드는 존재하지 않는다.
(프로퍼티가 val인 경우 게터에 field가 없으면 되지만, var인 경우에는 게터나 세터 모두에 field가 없어야 한다.)

때로 접근자의 기본 구현을 바꿀 필요는 없지만 가시성을 바꿀 때가 있다. 이를 어떻게 하는지 보자.

## 2.5 접근자의 가시성 변경

접근자의 가시성은 기본적으로 프로퍼티의 가시성과 같다.

하지만 원한다면 가시성 변경자를 추가하여 접근자의 가시성을 변경할 수 있다.

```kotlin
class LengthCounter {
  var counter: Int = 0
    private set               // 이 클래스 밖에서 이 프로퍼티의 값을 바꿀 수 없다.

  fun addWord(word: String) {
    counter += word.length
  }
}
```

### 프로퍼티에 대해 나중에 다룰 내용

- lateinit 변경자를 null이 될 수 없는 프로퍼티에 지정하면 프로퍼티를 생성자가 호출된 다음에 초기화한다는 의미다. (6장)
- 요청이 들어올 때 초기화되는 지연 초기화 프로퍼티는 위임 프로퍼티(delegated property)의 일종이다. (7장)
- 자바와의 호환성을 위해 자바의 특징을 코틀린에서 에뮬레이션하는 애노테이션을 활용할 수 있다. (10장)