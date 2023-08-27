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

#### 와일드카드와 제네릭 타입의 차이는?
 * 와일드카드는 뭔가 넣을때 쓰는게 아님.
 * 받을때 쓰는거다
   * 제네릭타입이 아무리 부모 클래스라도 엄연히 다른 타입이므로 컴파일 에러가 발생한다. 그래서 와일드카드가 필요하다.
   * 참고로 `Box<? extends Object> box` 는 `Box<?> box` 와 같다.
   * (아래 코드 예시)
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
 * 로 타입이면, 컴파일 타임에 잡아낼 수 있는 오류를 잡아낼 수 없다.
#### ㅇㅇ
 * ㅇㅇ

## 완벽 공략
#### ㅇㅇ
 * ㅇㅇ
#### ㅇㅇ
 * ㅇㅇ
