# 16. 변경 가능성을 최소화 하라
## 핵심 정리 (불변 클래스)
#### 불변 클래스는 가변 클래스보다 설계 및 구현하기 쉽고, 오류 여지가 적어 훨씬 안전하다
#### 불변 클래스 만드는 5가지 규칙
 * 객체 상태 변경 메서드(setter)를 제공하지 않는다.
 * 확장하지 못하게 한다. (public final class로 선언)
 * 모든 필드를 final로 선언
 * 모든 필드를 private으로 선언
 * 자신 외에는 내부의 가변 컴포넌트에 접근할 수 없도록 한다

#### 불변 클래스 장단점
 * 함수형 프로그래밍에 적합
 * 불변 객체의 장점이 많다.
    * 단순하다
    * 스레드 safe
    * 안심하고 공유 가능
    * 불변 객체끼리는 내부 데이터 공유 가능
 * 실패 원자성을 제공한다 (item 76 ) : 연산에 사용되더라도 그 연산이 실패했을때 값이 바뀌지 않는다.
 * 단점 : 값이 다르다면 반드시 별도의 객체를 만들어야 한다.

#### 불변 클래스 사용 예시 - 복소수 클래스
```
public final class Complex {
  private final double re;
  private final double im;

  public static final Complex ZERO = new Complex(0, 0);
  public static final Complex ONE = new Complex(1, 0);
  public static final Complex I = new Complex(0, 1);

  ...
}
```

#### 불변 클래스 만들때 고려할 점
 * 상속 막기
    * final 키워드
    * private 생성자 + 정적 팩터리 (상속을 허용해주는 방법이다, 제한된 범위내에서)
 * 재정의가 가능한 클래스는 방어적 복사 사용해야 한다
 * 모든 외부 공개 필드는 final 이어야 한다. 

## 완벽 공략
#### ㅇㅇ
 * ㅇㅇ
#### ㅇㅇ
 * ㅇㅇ




