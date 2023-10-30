# 26. 로(raw) 타입은 사용하지 말라

#### 제네릭 용어 정리
```
List // 로 타입
List<E> // 제네릭 타입
List<String> // 매개변수화 타입
E // 타입 매개변수
String // 실제 타입 매개변수
List<E extends Number> // 한정적 타입 매개변수
Class<?> // 비한정적 와일드카드 타입
Class<? extends Annotation> // 한정적 와일드카드 타입
```

### 와일드카드와 제네릭 타입의 차이는?
#### 먼저 공변과 불공변을 알아야 한다.
 * 공변(covariant) : A가 B의 하위 타입일 때, `T<A>` 가 `T<B>`의 하위 타입이면 T는 공변 (ex: Array)
 * 불공변(invariant) : A가 B의 하위 타입일 때, `T<A>` 가 `T<B>`의 하위 타입이 아니면 T는 불공변 (ex: Collection)
 * 제네릭타입이 아무리 부모 클래스라도 엄연히 다른 타입이므로 컴파일 에러가 발생한다. 그래서 와일드카드가 필요하다.
 * 와일드카드는 any type 이 아니라 unknown type 이다.
 * 무슨말이냐면, `Collection<?> c = new ArrayList<String>();` 가 있을때 `c.add(new Object());` // 컴파일 에러 발생
 * 그래서 뭔가를 넣을때(ex: add) 쓰지 못한다. 뭔가를 넣을땐 정의된 제네릭과 같거나 하위 타입인지 검사하기 때문이다.
 * 반면에 꺼낼때(get) 사용 가능하다. 왜냐면 꺼낼때는 자식 확인을 하지 않으며, 적어도 Object 타입의 하위 타입임은 보장되기 때문이다.
    * 이 경우, 너무 제한이 없어져서 아무 타입이나 get 되면 안되는 경우가 있을 수 있다. 이때 한정적 키워드가 필요해진다. 
 * 참고로 `Box<? extends Object> box` 는 `Box<?> box` 와 같다.
#### (아래 코드 예시)
```
  public static void main(String[] args) {
    Box<Integer> box = new Box<>();
    box.add(10);
    printBox1(box); // 컴파일 에러 발생
    printBox2(box); // 정상 동작
  }
  
  private static void printBox1(Box<Object> box) {
    System.out.println(box.get());
  }

  private static void printBox2(Box<? extends Object> box) {
    System.out.println(box.get());
  }

```

## 핵심 정리
#### 로(raw) 타입은 사용하지 말라
 * 로 타입이면, 컴파일 타임에 잡아낼 수 있는 오류를 잡아낼 수 없다. (안정성)
 * 제네릭을 사용하면 어떤 타입인지 명확히 표현 가능하다 (표현력)
#### 자바는 왜 로 타입을 지원하는가?
 * (item28에서 자세히 소개)
 * 하위버젼 호환성을 위해
 * 실제로 컴파일된 코드를 보면, 로 타입으로 보인다.
#### `List`와 `List<Object>` 의 차이는?
 * 안정성과 표현력의 차이
#### `Set`과 `Set<?>`의 차이는?
 * 로타입은 안정성이 깨진다 (add 할때 아무거나 넣을 수 있다)
 * `Set<?>` Set을 캐스팅하는 측면에서는가장 범용적인 타입이다. 대신 add 할땐 아무것도 넣을 수 없다


## 완벽 공략
#### GenericRepository

```
public class GenericRepository<E extends Entity> {
   private Set<E> entities;
   private Optional<E> findById(Long id) {
      return entities.stream().filter(a -> a.getId().equals(id)).findAny();
   }
}

---

public class AccountRepository extends GenericRepository<Account> {
}
```


