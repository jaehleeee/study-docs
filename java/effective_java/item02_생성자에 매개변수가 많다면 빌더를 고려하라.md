# 2. 생성자에 매개변수가 많다면 빌더를 고려하라.

## 내용 요약
 * 정적 팩터리와 생성자에는 똑같은 제약이 있다. 선택적 매개변수가 많을 때 적절히 대응하기 어렵다는 점이다.

### 첫번째 대안 : 점층적 생성자 패턴
 * 점층적 생성자 패턴 : 선택 매개변수 1개 ~ 전부, 개수별로 생성자를 만들어놓는 방법
 * 생성자를 이용한다면, 사용자가 설정을 원치 않는 매개변수까지 포함시키기 쉽다. (보통 0 으로 세팅)
 * 클라이언트 코드를 읽기도 어렵고 헤깔린다. 타입이 같은 매개변수가 연달아 있으면 찾기 어려운 버그로 이어질 수 있다.

### 두번째 대안 : 자바빈즈 패턴
 * 매개변수가 없는 생성자로 객체를 만든 후, 세터로 메서드를 호출해 원하는 매개변수의 값을 설정하는 방식
 * 심각한 단점 : 객체 하나를 만들려면 메세드를 여러 개 호출해야 하고, 객체가 완전히 생성되기 전까지 일관성이 무너진 상태에 놓이게 된다.

### 세번째 대안 : 빌더 패턴
 * 필수 매개변수만으로 생성자(혹은 정적 팩터리)를 호출해 빌더 객체를 얻는다. 
    * 그런 다음 `빌더 세터 메서드(메서드 체이닝)`를 이용해 선택 매개변수들을 추가로 설정하고, 
    * `build()` 호출하여 객체의 일관성이 무너지지 않게 객체를 생성한다.
 * 요즘은 lombok 으로 쉽게 만들 수 있지만, 직접 builder 패턴을 구현하려면 코드가 많이 필요하다.
    * 빌더 패턴을 적용하려는 class 내에, `public static class Builder` 를 작성하고 그 내부에 기존 객체 필드들을 다시 선언해야 하므로 중복 코드이기도하다.
 * 빌더 패턴은 계층적으로 설계된 클래스와 함께 쓰기에 좋다. 추상 클래스는 추상 빌더를, 구체 클래스는 구체 빌더를 갖게 한다.

### 계층형 빌더 (책에서 Pizza 추상 클래스 예시)
 * 추상 클래스에선 하위 클래스를 위해 추상 빌더를 사용한다. (`abstract static class Builder<T extends Builders<T>>`)



## 완벽 공략
### 1. JavaBean
 * 자바로 작성한 자바 클래스 중에 자바 빈즈 컨벤션(Java Beans Convention) 에 맞게 작성된 클래스
 * JSP 기반 웹 어플리케이션에서 정보를 표현하기 위한 객체로 사용된다.
 * spring에서는 pojo 라고 불리기도 한다.
    * 다른 클래스나 인터페이스를 상속/implements 받아 메서드가 추가된 클래스가 아닌, 일반적으로 우리가 알고 있는 getter, setter 같이 기본적인 기능만 가진 자바 객체
    * 진정한 POJO란 객체지향적인 원리에 충실하면서, 환경과 기술에 종속되지 않고 필요에 따라 재활용될 수 있는 방식으로 설계된 오브젝트  from 토비의 스프링
    * 특정 환경, 특정 규약에 종속받지 않아야 한다.
#### JavaBean Convention
 * 자바빈은 기본(default)패키지 이외의 특정 패키지에 속해 있어야 한다.
 * 클래스는 인자(Argument)가 없는 기본 생성자(Default constructor)를 갖는다. <- 왜 기본 생성자? 객체를 만들기 가장 편하기 때문.
 * Serializable 인터페이스를 구현해야 한다.
 * 클래스의 멤버 변수는 프로퍼티(Properties)라고 하며 private 접근 제한자를 가져야 한다.
 * 클래스의 프로퍼티들은 Getter/Setter를 통해 접근할 수 있어야 한다.
 * Getter/setter의 접근 제한자는 public이어야 한다.
 * Getter의 이름은 get{프로퍼티 이름} 이며, Setter의 이름은 set{프로퍼티 이름}이다
 * Read Only인 경우 Setter는 없을 수 있다.
 * Getter의 경우 파라미터가 존재하지 않아야 하며, setter의 경우 하나 이상의 파라미터가 존재한다.
 * 프로퍼티의 타입이 Boolean인 경우 is로 시작할 수 있다.

### 2. 객체 프리징 (자바스크립트에서 나온 기능)
 * final로 객체를 선언하면, 해당 객체에 새로운 객체를 참조시킬 수 없다. 그러나 객체 하위 필드는 수정/삭제/신규생성 가능하다.
 * 이러한 객체 하위 필드 수정/삭제/신규생성 기능을 제한하고 싶다면, 조건에 따라 프리징 할 수 있다.
    * 자바에서는 실제로  구현해야하고, 자바스크립트에선 freeze 라는 함수가 있다.
 * 언제 프리징되는지 알 수 있으니, 실제 프로젝트에선 자주 사용되지 않을 것으로 보인다.

### 3. IllegalArgumentException
 * 잘못된 인자를 넘겨 받았을 때 사용할 수 있는 런타임 예외 (unchecked exception)
 * 예외가 발생했을때 클라이언트가 복구 가능한 상황이라면 checked exception, 복구가 힘들다면 unchecked exception
 * [RuntimeException Class JAVA Docs](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/RuntimeException.html)
 * [Unchecked Exceptions — The Controversy JAVA Docs](https://docs.oracle.com/javase/tutorial/essential/exceptions/runtime.html)
    * unchecked exception 은 catch or specify 할 필요가 없으므로 Runtime exception 으로 모든 예외를 만드는 경향이 생긴다.
    * checked exception 은 throw를 강제한 이유는 메소드를 호출한 이들에게 알리기 위함이다. (ex: IOException, SQLException 등)
    * runtime exception(unchecked exception)은 API 클라이언트 코드가 복구되거나 어떤 방식으로든 처리될 것을 합리적으로 기대할 수 없음을 의미한다.
    * runtime exception(unchecked exception)은 또한 어떤 상황이든 발생할 수 있다. 발생할 수 있는 경우가 numerous 하다. 모든 메소드 선언에 런타임 예외를 추가해야 하는 것은 프로그램의 명확성을 떨어뜨릴 것이다.
    * runtime exception을 던지는 유일한 경우는 일반 유저가 함수를 잘못 호출한 경우.
    * Generally speaking, 귀찮다는 이유로 runtime exception(unchecked exception) 을 던지지마라.
    * Here's the bottom line guideline: 
       * `If a client can reasonably be expected to recover from an exception, make it a checked exception. If a client cannot do anything to recover from the exception, make it an unchecked exception.`

