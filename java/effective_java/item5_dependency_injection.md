# 5. 자원을 직접 명시하지 말고 의존 객체 주입을 사용하라
#### 요약 : 클래스 내부적으로 하나 이상의 자원에 의존하고, 그 자원이 클래스 동작에 영향을 준다면 싱글턴이나 정적 유틸리티 클래스는 사용하지 않는 것이 좋다.

## 핵심 정리
 * 사용하는 자원에 따라 동작이 달라지는 클래스는 정적 유틸리티 클래스나 싱글턴 방식이 적합하지 않다.
    * 자원에 따라 새롭게 클래스를 생성해줘야하기 때문에

#### 정적 유틸리티를 잘못 사용한 예시 - 사전 종류에 따라 다른 dictionary를 사용해야할 경우 불편하다.
```
public class SpellChecker {
   private static final Dictionary dictionary = ...;

   private SpellChecker() {} // 객체 생성 방지

   public static boolean isValid(String word) { ... } // dictionary를 사용한 코드가 포함됨
   public static List<String> suggestions(String typo) { ... } // dictionary를 사용한 코드가 포함됨
}
```

 * 이러한 경우엔, 인스턴스를 생성할 때 생성자에 필요한 자원을 넘겨주는 방식인 의존 객체 주입 방식이 적절하다.
 * 의존 객체 주입이란?
    * 인스턴스를 생성할 때 필요한 자원을 넘겨주는 방식이다.

#### 의존 객체 주입을 통한 자원 주입
```
public class SpellChecker {
   private static final Dictionary dictionary = ...;

   public SpellChecker (Dictionary dictionary) {
      this.dictionary = Objects.requireNonNull(dictionary);
   }

   ...
}

```

 * 이 방식의 변형으로 생성자에 자원 팩터리를 넘겨줄 수 있다.
    * 팩터리란? 호출할 때마다 특정 타입의 인스턴스를 반복해서 만들어주는 개체를 말한다.
    * 자바8의 `Supplier<T>` 인터페이스가 팩터리를 표현한 완벽한 예시다.
    * 클라이언트는 이 방식을 통해 자신이 명시한 타입의 하위 타입이라면 무엇이든 생성할 수 있는 팩터리를 넘길 수 있다.
 * 의존 객체 주입을 사용하면 클래스의 유연성, 재사용성, 테스트 용이성을 개선할 수 있다.


## 완벽 공략

### 1. 팩터리를 넘겨주는 패턴
```
public class SpellChecker {
   private static final Dictionary dictionary = ...;

   public SpellChecker (Supplier<? extends Dictionary> supplier) {
      this.dictionary = supplier.get();
   }

   ...
}

```

### 2. 스프링 IoC
 * Inversion of Control
    * 자기 코드에 대한 제어권이 자기 자신이 아니라 외부에서 제어하는 경우
    * 여기서 제어권이란?
       * 인스턴스를 만들거나, 어떤 메서드를 실행하거나, 필요로 하는 의존성을 주입 받는 등의 작업을 의미한다.  
 * 스프링 IoC 장점
    * 자바 표준 스팩 지원
    * 손쉽게 싱글톤 scope 사용
    * 객체 생성(bean) 관련 라이프사이클 인터페이스 제공 
