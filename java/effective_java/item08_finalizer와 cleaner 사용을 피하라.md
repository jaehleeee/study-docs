# 8. finalizer와 cleaner 사용을 피하라
## 핵심 정리
#### finalizer와 cleaner(java 9 도입)는 문제가 많다.
 * 즉시 수행 보장이 없고 실행되지 않을 수도 있다. 
 * 동작 중 예외 발생시 정리 작업이 처리되지 않을 수 있다.
 * 성능 문제가 있다.
 * 보안 문제가 있다.


#### finalizer란? from ChatGPT
```
finalize() 메서드는 Object 클래스에 정의되어 있으며, 기본적으로 아무런 작업도 수행하지 않습니다.
하지만 개발자는 이 메서드를 재정의하여 객체의 소멸 전에 수행해야 하는 정리 작업을 구현할 수 있습니다.
예를 들어, 파일 핸들이나 네트워크 연결과 같은 시스템 리소스를 해제하는 작업을 수행할 수 있습니다.

finalize() 메서드는 객체가 가비지 컬렉션되기 전에 호출될 수 있으며, 호출 시점은 정확히 예측할 수 없습니다.
Java의 가비지 컬렉션은 객체가 더 이상 참조되지 않을 때 동작하므로, 객체가 가비지 컬렉션되기 전에 finalize() 메서드가 호출되는 것은 보장되지 않습니다.
실제로 finalize() 메서드의 호출은 보장되지 않으며, 호출되지 않을 수도 있습니다.

Java 9부터는 finalize() 메서드의 사용을 비권장(deprecated)으로 표시했습니다.
대신, 자원 관리와 관련된 작업은 try-with-resources나 close() 메서드 등을 사용하여 명시적으로 처리하는 것이 권장됩니다.
이러한 명시적인 자원 해제 방식을 사용하면 finalize() 메서드의 호출을 기다리지 않고도 자원을 안전하게 해제할 수 있습니다.

따라서 일반적으로는 finalize() 메서드를 사용하기보다는 명시적인 자원 관리 방식을 선호하는 것이 좋습니다.
```
#### 그럼 반납할 때 어떤 방법을?
 * AutoCloseable을 구현하고 close 메서드를 override하고, try-with-resource를 통해 자원을 해제하자.
    * try 블록을 벗어나기 전에 close 메서드가 호출되어 리소스가 해제된다.
 * 혹시나 AutoCloseable을 구현하고도 try-with-resource 형태를 사용하지 않는다면, 자원이 해제되지 않을수도 있다. 그럴땐 안정망으로 cleaner 클래스를 이용할 수 있다.

#### 안전망과 AutoCloseable 모두 구현된 클래스 예시
 * try-with-resource 사용되었다면, `close()` 메서드가 자동으로 실행되어 자원을 해제한다.
 * try-with-resource 사용되지 않았다면, gc가 발생할때 cleaner queue에 객체가 들어가게 되고, Runnable로 정의된 state 클래스 작업(run)이 실행된다.
```
public class Room implements AutoCloseable {
    private static final Cleaner cleaner = Cleaner.create();

    public Room(int numJunkPiles) {
        state = new State(numJunkPiles);
        cleanable = cleaner.register(this, state);
    }

    // try-with-resource 사용되었다면, 이 메서드가 자동으로 실행된다.
    @Override public void close() {
        cleanable.clean();
    }

    // 방의 상태. cleanable과 공유한다.
    private final State state;

    // cleanable 객체. 수거 대상이 되면 방을 청소한다.
    private final Cleaner.Cleanable cleanable;

    // 청소가 필요한 자원. 절대 Room을 참조해서는 안 된다!
    private static class State implements Runnable {
        int numJunkPiles; // 방 안의 쓰레기 수

        State(int numJunkPiles) {
            this.numJunkPiles = numJunkPiles;
        }

        // close 메서드나 cleaner가 호출한다.
        @Override public void run() {
            System.out.println("Cleaning room");
            numJunkPiles = 0;
        }
    }
}
```

## 완벽 공략
#### 중첩 클래스에서 정적이 아닌 클래스는 바깥 객체의 참조를 갖는다.
 * 아럐 예시 코드처럼, 정적이 아닌 (non-static) 중첩 클래스는 바깥 객체의 참조를 가질 수 있다.
   * 바깥 클래스의 참조를 가질 수 있다면, 이후 자원을 해제할때 해제가 잘 안될 수 있고 복잡성을 가진다.
   * 그러니 중첩 클래스는 정적(static) 클래스로 만드는 것이 좋다.
 * 람다도 마찬가지다. 클래스 내부의 람다는 외부 클래스에 대한 필드에 접근하는 경우 해당 필드에 대한 참조를 가지게 된다.
    * 그래서 람다 안에서 바깥 클래스 필드 중static 필드만 접근해야 한다. (안하는게 가장 좋다.) 
```
public class OuterClass {
   private void hi() {}

   class InnerClass {
      public void hello() {
         OuterClass.this.hi();
      }
   }
}
```

#### Finalizer 공격
 * Finalizer 공격이란?
```
악의적인 공격자가 finalize() 메서드를 악용하여 시스템을 공격하는 것입니다.
악의적인 객체를 생성하여 finalize() 메서드를 오버라이드하고, 해당 객체의 finalize() 메서드를 호출하도록 하여 악성 코드를 실행시킬 수 있습니다
```
 * 이를 막으려면?
    * finalize를 미리 오버라이드 후, 더 이상 상속해서 override 해서 쓰지 못하도록 final 키워드를 붙여둔다.
    * 혹은 클래스 자체에 final 키워드를 붙여 더 이상 상속을 못하게 막는 방법도 있다. (하지만 이 방법은 상속이 필요한 경우엔 안좋은 방법이 되니까 윗 방법이 더 좋다.)

#### `AutoClosable` interface
 * 자원반납을 자동으로 해준다.
 * override 해줘야할 close 메서드는 idempotent (멱등성)한게 좋다.
    * idempotent : 여러번 호출해도 매번 같은 결과가 나오는 특성    

 

