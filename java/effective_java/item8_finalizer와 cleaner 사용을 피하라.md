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
```
public class Room implements AutoCloseable {
    private static final Cleaner cleaner = Cleaner.create();

    public Room(int numJunkPiles) {
        state = new State(numJunkPiles);
        cleanable = cleaner.register(this, state);
    }

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

    // 방의 상태. cleanable과 공유한다.
    private final State state;

    // cleanable 객체. 수거 대상이 되면 방을 청소한다.
    private final Cleaner.Cleanable cleanable;

    @Override public void close() {
        cleanable.clean();
    }
}
```

## 완벽 공략
 * ㅇㅇ
