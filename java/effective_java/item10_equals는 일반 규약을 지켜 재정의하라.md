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
 * ㅇㅇ
