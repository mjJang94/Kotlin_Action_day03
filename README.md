# 내부 클래스와 중첩된 클래스: 기본적으로 중첩 클래스
View 요소를 만든가소 생각했을 때, 그 View의 상태를 직렬화해야 한다(필요한 모든 데이터를 다른 도우미 클래스로 복사할 수 있다).   
이를 위해서는 State 인터페이스를 선언하고 Serializable을 구현하는데 View인터체이스 안에는 뷰의 상태를 가져와 저장할 때 사용할 getCurrentState와 restoreState 메소드 선언이 있다.

<pre>
<code>
interface State: Serializable

interface View {
  fun getCurrentState(): State
  fun restoreState(state: State) ()
}
</code>
</pre>

만약 Button 클래스가 있다고 생각하고, Button 클래스의 상태를 저장하는 클래스는 Button 클래스 내부에 선언하면 좀 더 편리하다.   
자바식으로 한번 보자.

<pre>
<code>
public class Button implements View{
  @Override
  public State getCurrentState() {
  
  return new ButtonState();
 }
 
 @Override
 public void restoreState(State state) { }
 
 public class ButtonState implements State { }
</code>
</pre>

State 인터페이스를 구현한 ButtonState 클래스를 정의해서 Button에 대한 구체적인 정보를 저장한다.   
근데 버튼의 상태를 직렬화 하면 **java.io.NotSerializableException: Button**이라는 오류가 발생한다.   
왜냐하면 자바에서는 다른 클래스 안에 정의한 클래스는 자동으로 내부 클래스가 되기 때문이다. 위 코드의 ButtonState 클래스는 바깥쪽 Button 클래스에 대한 참조를 묵시적으로 포함하기 때문에 ButtonState를 직렬화할 수 없는것이다.   
해결하기 위해서는 ButtonState를 static클래스로 선언하면 된다.   

하지만 코틀린에서는 중첩된 클래스가 기본적으로 동작하는 방식은 매우 다르다.

<pre>
<code>
class Button: View {
  override fun getCurrentState(): State = ButtonState()
  override fun restoreState(state: State) {}
  class ButtonState: State {}
 }
</pre>
</code>

코틀린에서는 중첩 클래스에 아무런 변경자가 붙지 않으면 자바 static 중첩 클래스와 같다고한다. 이를 내부 클래스로 변경해서 바깥쪽 클래스에 대한 참조를 포함하게 만들고 싶다면 inner 변경자를 붙여줘야 한다.   

            java                kotlin
중첩클래스   static class A  |   class A  
내부클래스   class A         |   inner class A


코틀린에서 내부클래스에서 바깥쪽 클래스의 인스턴스를 가리킬 때에도 조금 다른데, this@Outer라고 써야한다.
<pre>
<code>
class Outer{
  inner class Inner{
    fun getOuterReference() : Outer = this@Outer
  }
}
</pre>
</code>

