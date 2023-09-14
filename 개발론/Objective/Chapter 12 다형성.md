# 01 다형성

- 하나의 추상 인터페이스에 대해 코드를 작성하고 서로 다른 구현을 연결할 수 있는 능력
- 분류
    - 오버로딩
    - 강제 다형성 (캐스팅)
    - 매개변수 다형성 (제네릭 등)
    - 포함 다형성 (수신 받는 객체에 따라 메시지 동작이 달라진다)

  

# 02 상속의 양면성

- 상속의 목적은 코드 재사용이 아니다.
- 상속은 프로그램을 구성하는 개념들을 기반으로 다형성을 가능하게 하는 타입 계층을 구축하기 위한 것이다.
- 관점에 따른 상속
    - 데이터 관점의 상속 : 부모 클래스에서 정의한 인스턴스 필드를 자식 클래스가 가짐
    - 행동 관점의 상속 : 자식 클래스의 인스턴스를 통해 부모 클래스에 정의된 메서드를 실행

#### 자식 클래스의 인스턴스를 통해 어떻게 부모 클래스에 정의된 메서드를 실행하는가

- 메시지를 수신한 객체는 class  포인터로 연결된 자신의 클래스에 적절한 메서드가 있는지 탐색
- 만약 존재하지 않으면?
    - 클래스의 parent 포인터를 따라 부모 클래스를 차레대로 훑어 가면서 적절한 메서드가 존재하는지 검색한다.

  

# 03 업캐스팅과 동적바인딩

- 업캐스팅 : 부모 클래스 타입으로 선언된 변수에 자식 클래스 인스턴스를 할당. 명시적 타입 변환 X
    - 업캐스팅 : Lecture lecture = new GradeLecture(...);
    - 다운캐스팅 : GradeLecture gradeLecture = (GradeLecture) lecture;
- 동적바인딩 : 런타임에 메시지를 수신하는 객체의 타입에 따라 실행되는 메서드가 결정됨.  
    - 정적바인딩 : 코드를 작성하는 시점에 호출될 코드가 결정됨.

  

# 04 동적 메서드 탐색과 다형성

- self 참조 (self reference)
    - java에서는 this
    - 객체가 메시지를 수신하면 컴파일러가 self 참조라는 임시 변수를 자동으로 생성한 뒤, 메시지를 수신받은 객체를 가르키도록 설정

  

#### 자동적인 메시지 위임

- 상속 계층에 따라 부모에게 자동 위임
- 오버라이딩과 오버로딩시 동작 차이

#### 동적인 문맥

- 메시지를 수신한 객체가 무엇이냐에 따라 메서드 탐색을 위한 문맥이 동적으로 바뀐다.
    - 동적인 문맥을 결정하는 것은 바로 메시지를 수신한 객체를 가리키는 self 참조다.

  

주요 예시.

|   |
|---|
|public class Lecture {<br>	public String stats() {<br>		return String.format("Title: %s, %s", title, getEvaluationMethod());<br>	}<br><br>	public String getEvaluationMethod() {<br>		return "Pass or Fail";<br>	}<br>}<br><br>---<br><br>public class GradeLecture {<br><br>	public String getEvaluationMethod() {<br>		return "Grade";<br>	}<br>}|

- getEvaluationMethod() 라는 구문은 현재 클래스의 메서드를 호출하는 것이 아니라, 현재 객체에게 getEvaluationMethod 메세지를 전송하는 것이다.
- 현재 객체란 무엇인가?
    - self 참조가 가리키는 객체다.
    - 이 객체는 처음에 stats 메시지를 수신했던 바로 그 객체다.
- stats 메서드를 실행하던 중 getEvaluationMethod를 발견하면 시스템은 self 참조가 가리키는 현재 객체에게 메시지를 전송해야 한다고 판단한다.

  

![](https://wiki.navercorp.com/download/attachments/1563249353/image-2023-8-18_15-55-51.png?version=1&modificationDate=1692341752000&api=v2 "LEE JAE HYEON[ 이재현 ] > Chapter 12 다형성 > image-2023-8-18_15-55-51.png")

#### self vs super

- super : 지금 이 클래스의 부모 클래스부터 동적 메서드 탐색을 시작
- self : 메시지를 수신하는 객체의 클래스에 따라 메서드 탐색 시작 위치를 동적으로 결정

# 05 상속 대 위임

- ㅇㅇ
