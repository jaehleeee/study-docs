# 27. 비검사 경고를 제거하라

## 핵심 정리
#### 비검사 경고란?
 * 컴파일러가 타입 안정성을 확인하는 필요 정보가 충분치 않을 때 발생
 * 발생 예시 : `Set names = new HashSet();` 이렇게 로 타입으로 객체를 생성할때 발생한다. 타입 안정성이 충분하지 않으므로
#### 비검사 경고를 제거하라
 * 경고 제거할 수 없지만 안전을 확신한다면 `@SuppressWarings("unchecked")` 애노테이션을 달아 경고를 숨기자
 * `@SuppressWarings`는 가능한 좁은 범위로 사용하라
 * `@SuppressWarings("unchecked")` 애너테이션을 사용할 땐 그 경고를 무시해도 안전한 이유를 항상 주석으로 남겨야 한다.  

## 완벽 공략
#### 애노테이션 정의
 * @Retention 애노테이션 정보를 얼마나 오래 유지할 것인가
    * RetentionPolicy.RUNTIME : 디폴트, 런타임중에도 참고 가능
    * RetentionPolicy.CLASS : 클래스 파일까지만 참고 가능
    * RetentionPolicy.SOURCE : 가장 좁은 범위, 바이트 코드에서도 참조할 수 없다.
 * @Target 애노테이션을 어디에 붙일 것인가
    * 배열로 넣을 수 있으니 여러 타겟을 설정할 수 있다.
    * ElementType.TYPE : 타입 정의
    * ElementType.METHOD, PARAMETER, CONSTRUCTOR, FIELD, LOCAL_VARIABLE 등
 * @Document java doc에 포함된다
