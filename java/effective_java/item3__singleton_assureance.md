# 3. private 생성자나 열거 타입으로 싱글턴임을 보증하라

## 내용 요약
 * 싱글턴 : 인스턴스를 오직 하나만 생성할 수 있는 클래스
 * 클래스를 싱글턴으로 만들면 이를 사용하는 클라이언트를 테스트하기가 어려워질 수 있다. (인터페이스 없이 정의한 경우)
    * 클라이언트 코드를 테스트할때마다 싱글턴을 쓰는건, 리소스적으로 비싸다.
    * 대신 인터페이스가 있다면, 이 인터페이스로 구현한 Mock 클래스를 따로 만들어두고 이 Mock 객체로 테스트가 가능하다.
 * 싱글턴을 만드는 방식은 보통 2가지 (+1)


#### 첫번째 방법 : private 생성자 & public static final 멤버로 제공
 * 생성자를 private으로 감춘 후, 유일한 인스턴스 접근 수단으로 public static 멤버를 하나 마련해준다.
 * private 생성자는 public static 멤버를 초기화할 때 한번만 호출
 * 장점1 : 해당 클래스가 싱글턴임이 API에 명백히 드러난다. (final 제어자 덕분)
 * 장점2 : 간결함
 * 단점1 : 리플렉션 API를 이용해 private 생성자를 호출 가능하므로 싱글턴이 깨질 수 있다.
    * 이를 방어하려면 2번째 객체 생성시 예외를 던지게 하면 된다. - 객체내 객체 생성여부를 체크하는 boolean 필드를 하나두고 이를 통해 확인 가능.
    * (간결함이라는 장점은 깨지는..)
 * 단점2 : 클래스를 직렬화/역직렬화하는 경우가 있다. 이때 생성자가 호출된다. 그래서 싱글턴이 깨질 수 있다.
    * 이를 방어하려면, readResolve() 라는 메서드르 클래스 내에 선언해줘야 한다. (override는 아닌데, 직렬화 과정에서 사용되는 하는 ;; 이상한 메서드인듯)
    * `private Object readResolve() {return INSTANCE}`
    * (간결함이라는 장점은 깨지는..)
```java
public class Elvis {
  public static final Elvis INSTANCE = new Elvis();
  pirvate Elvis() {};
}
```

#### 두번째 방법 : private 생성자 & private static final 멤버 & public static 팩터리 메서드로 제공
 * 생성자를 private으로 감춘 후, 정적 팩터리 메서드를 통해 private static 멤버를 제공.
 * 장점1 : API를 바꾸지 않고도 싱글턴이 아니게 변경할 수 있다는 점
    * static 팩터리 메서드에 private 멤버가 아닌, 신규 객체를 생성함으로써 클라이언트 코드 변경없이 싱글턴에서 싱글턴에서 아니게 변경 가능.
 * 장점2 : 정적 팩터리를 제네릭 싱글턴 팩터리로 만들 수 있다는 점
    * 제네릭한 타입으로 싱글턴 팩터리를 만들 수 있다.

```java
public class Meta<T> {
  private static final Meta<Object> INSTANCE = new Elvis();
  pirvate Meta() {};
  public static <T> Meta<T> getInstance() {return (Meta<T>) INSTANCE;}
}

---

// `Meta<String> meta1 = Meta.getInstance();`
// `Meta<Integer> meta2 = Meta.getInstance();`
// meta1과 meta2는 같은 해시코드를 가지지만, == 비교는 타입이 달라서 불가능하다.

```

 * 장점3 : 정적 팩터리의 메서드 참조를 공급자로 사용할 수 있다
 * 단점은 첫번째 방법과 동일하며 해결책도 동일하다.

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



## 완벽 공략
### 메서드 참조


### 함수형 인터페이스


### 객체 직렬화
