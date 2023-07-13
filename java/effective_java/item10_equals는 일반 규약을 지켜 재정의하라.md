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
#### 반사성 `A.equals(A) == true`
#### 대칭성 `A.equals(B) == B.equals(B)`
#### 추이성 `A.equals(B) == B.equals(C)== C.equals(A)`
#### 일관성 `A.equals(A) == A.equals(A)`
#### null 아님 `A.equals(null) == false`

### equal 구현 방법과 주의사항
 * ㅇㅇ

## 완벽 공략
 * ㅇㅇ
