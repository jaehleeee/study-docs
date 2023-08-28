# 10. equals는 일반 규약을 지켜 재정의하라
## 핵심 정리 : 재정의할 필요 없다면 하지 않는 것이 최선
#### 다음 경우 equals를 재정의할 필요 없다.
 * 각 인스턴스가 본질적으로 고유한 경우 (싱글턴, ENUM)
 * 인스턴스의 '논리적 동치성' 검사 필요 없는 경우
    * Money 라는 클래스가 있고, 100달러 지폐 인스턴스가 있다. 굳이 100달러 지폐 인스턴스들의 동치성을 따질 필요 없다
 * 상위 클래스 재정의한 equals가 하위 클래스에도 적절한 경우
    * Collections 클래스들이 보통 상위 클래스에서 잘 정의되어 있다
 * 클래스가 private이거나 package-private이곡 equals 메서드 호출할 일이 없다

### equal 규약
 * 반사성 `A.equals(A) == true`
 * 대칭성 `A.equals(B) == B.equals(A)`
 * 추이성 `A.equals(B) == B.equals(C)== C.equals(A)`
 * 일관성 `A.equals(B) == A.equals(B)`
 * null 아님 `A.equals(null) == false`

### equal 구현 방법과 주의사항
#### 기본적인 방법
 * 본인(this)과 == 맞는지 체크
 * 타입 (instanceOf) 체크
 * 타입 맞다면, 해당 타입으로 캐스팅 후 핵심 필드값들이 같은지 체크

#### x,y 좌표계 클래스 Point 예시
```
public class Point {
   private int x;
   private int y;

   ...

   @Override
   public boolean equals(Object o) {
      if (this == o) return true;

      if (! (o instanceOf Point)) return false;

      point p = (Point) o;

      return p.x == x && p.y == y;      
   }
```
#### 직접 만들면 빼먹는게 생길 수도 있다, tool을 써라.
 * auto-value 어노테이션 라이브러리 활용
    * 구글이 만듬
    * 쓰는 방법이 좀 생소함... abstract class를 만들고 거기에 어노테이션 붙이면, 구현 클래스를 만들어줌
 * (java 11 이상이라면) lombok 어노테이션 활용
    * @EqualsAndHashCode 어노테이션 붙이면, 컴파일 타임에 해당되는 메서드들을 만들어준다
 * (java 17 이상이라면) java record 라는 타입을 사용하면 된다
 * intellij 자동 완성으로 만들기
    * 필드가 추가되거나 수정되면 자동 완성도 다시 해야한다

#### equals를 재정의했다면, hashCode도 반드시 함께 재정의해야한다.

## 완벽 공략
#### Value 기반 클래스
 * 클래스지만, 값을 표현하기 위해 만든 클래스다.
    * Point(좌표), 주소, 돈 등등
 * 식별자 없고 불변이다.
 * id가 있으면 안된다. 값 자체가 그 클래스를 대변한다.
 * 내부 필드를 final로 만들고, equals, hashcode, toString을 override 해서 특성에 맞게 만들어주면 된다.
 * Value 기반 클래스만드는 가장 좋은 방법은, java 17의 `record` 를 활용하면 가장 편하다. (아래 예시가 전부다.)
    * `record`로 만들면 equals 등을 java가 컴파일할때 자동으로 생성해준다.
```
public record Point(int x, int y) {

}
```

#### StackOverflowError
 * 스택과 힙
   * 힙 : 객체 인스턴스들이 존재, gc가 작용되는 공간
   * 스택 : 메서드 호출시 스택에 스택 프레임이 쌓인다.
 * 메서드가 무한루프를 돌거나 재귀적으로 호출되면, stack 영역이 꽉차면서 StackOverflowError 발생한다.

#### 리스코프 치환 원칙 : 객체지향 5대 원칙 SOLID의 L
 * '하위 클래스의 객체가 상위 클래스의 객체를 대체하더라도 소프트웨어의 기능을 깨트리지 않아야 한다.'
 * 상위 타입일때의 동작과 하위 타입의 동작이 같다면 결과도 같아야 한다.

