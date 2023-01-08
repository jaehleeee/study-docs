# 3. private 생성자나 열거 타입으로 싱글턴임을 보증하라

## 내용 요약
 * 싱글턴 : 인스턴스를 오직 하나만 생성할 수 있는 클래스
 * 클래스를 싱글턴으로 만들면 이를 사용하는 클라이언트를 테스트하기가 어려워질 수 있다.
 * 싱글턴을 만드는 방식은 보통 2가지 (+1 가지 방법)
#### 첫번째 방법 : 유일한 인스턴스 접근 수단으로 public static final 멤버 제공
 * 생성자를 private으로 감춘 후, 유일한 인스턴스 접근 수단으로 public static 멤버를 하나 마련해준다.
 * private 생성자는 public static 멤버를 초기화할 때 한번만 호출
 * 장점1 : 해당 클래스가 싱글턴임이 API에 명백히 드러난다. (final 제어자 덕분)
 * 장점2 : 간결함
 * 싱글턴이 아닌 예외는 딱 1가지 - 리플렉션 API를 이용해 private 생성자를 호출할 수 있다. (이를 방어하려면 2번째 객체 생성시 예외를 던지게 하면 된다.)
```java
public class Elvis {
  public static final Elvis INSTANCE = new Elvis();
  pirvate Elvis() {};
}
```


#### 두번째 방법 : private static final 멤버를 public static 정적 팩터리 메서드로 제공
 * 생성자를 private으로 감춘 후, 정적 팩터리 메서드를 public static 멤버로 제공.
 * 장점1 : API를 바꾸지 않고도 싱글턴이 아니게 변경할 수 있다는 점
 * 장점2 : 원한다면 정적 팩터리를 제네릭 싱글턴 팩터리로 만들 수 있다는 점
 * 장점3 : 정적 팩터리의 메서드 참조를 공급자로 사용할 수 있다
 * 마찬가지로 리플렉션 API를 이용해 private 생성자를 호출하는 예외는 존재 (이를 방어하려면 2번째 객체 생성시 예외를 던지게 하면 된다.)

```java
public class Elvis {
  private static final Elvis INSTANCE = new Elvis();
  pirvate Elvis() {};
  public static Elvis getInstance() {return INSTANCE;}
}
```

### 세번째 방법 : 원소가 하나인 Enum 생성
 * public 필드 방식과 비슷하지만 더 간격하고, 추가 노력없이 직렬화 가능하고, 리플렉션 공격을 막아준다.
 * 단, 만들려는 클래스가 다른 클래스를 상속해야한다면 이 방법을 사용할 수 없다.
