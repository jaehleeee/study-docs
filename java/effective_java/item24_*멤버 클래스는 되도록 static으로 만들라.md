# 24. 멤버 클래스는 되도록 static(정적 멤버 클래스)로 만들라
 * 중첩 클래스 : 클래스 안에 어딘가 정의되어 있는 클래스
 * 멤버 : scope이 클래스 전체이면 멤버다.
 * 멤버 클래스는 모두 중첩 클래스에 포함된다.

## 핵심 정리1 : 중첩 클래스 4가지
#### 정적 멤버 클래스
 * 바깥 클래스의 정적 멤버 필드에 접근 가능하다.
 * 바깥 클래스와는 독립적
 * 바깥 클래스와 함께 쓰일 때만 유요한 public 도우미로 자주 사용된다 (Calculator.Operation.PLUS)
#### 비정적 멤버 클래스
 * 바깥 클래스의 인스턴스와 암묵적 연결 상태
   * 바깥 클래스의 참조를 가진다
 * 클래스 외부에선 잘 사용되지 않는다, 생성이 묘하다 (new OuterClass.new InnerClass())
 * 어댑터 정의할때 자주 사용된다.
 * 바깥 인스턴스를 참조할 필요가 없다면 의미 없으니, **무조건 정적 멤버 클래스** 로 만들자.
    * 비정적 멤버 클래스는 생성하려면 바깥 클래스의 인스턴스도 항상 같이 생성해야 하는 제한이 있으므로 비추 (메모리 누수 가능성)
#### 익명 클래스
 * 쓰이는 시점과 동시에 인스턴스가 만들어진다.
 * 비정적인 문맥에서 사용될때만 바깥 클래스의 인스턴스를 참조할 수 있다.
 * 자바 람다 지원되기 전에 자주 사용되었다.
 * 정적 팩터리 메서드를 만들 때 사용할 수도 있다.
#### 지역 클래스
 * 가장 드물게 사용된다. (기선님도 써본적 없다고 함)
 * 가독성을 위해 짧게 사용해야 한다

## 완벽 공략
#### 어댑터 패턴
 * 기존 코드를 클라이언트가 사용하는 인터페이스의 구현체와 호환되도록 바꿔주는 패턴
    * 말그대로 어댑터
 * 실사용 예시
    * InputStream를 바로 사용할 수 없는 경우, InputStreamReader 라는 어댑터를 이용해서 사용할 수 있다
```
try (InputStream is = new FileInputStream("name.txt");
     InputStreamReader isr = new InputStreamReader(is);
     BufferedReader reader = new BufferedReader(isr);) {
    ...
  }
```

![image](https://github.com/jaehleeee/study-docs/assets/48814463/b69050ed-7090-48f4-8d7e-c68749c87416)

