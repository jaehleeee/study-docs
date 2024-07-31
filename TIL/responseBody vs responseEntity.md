# REST API 응답으로 responseBody vs responseEntity 중 어떤 형식을 써야할까

## responseBody

## responseEntity
 * HttpEntity를 extends 하여 만들어짐.
 * header와 body로 구성

## 차이점: Spring 내부 ReturnValueHandler 인터페이스에서 차이 발생
 * ReturnValueHandler 인터페이스에서 서로 다른 구현체가 사용됨.
   * responseBody : RequestResponseBodyMethodProcessor
   * responseEntity : HttpEntityMethodProcessor
 * 동일한 매커니즘이기 때문에 성능 차이는 없다고 봐도 무방.

## 어떤걸 사용할까
 * responseBody는 상태코드와 헤더를 유연하게 변경하지 못함. 코드는 깔끔함
 * responseEntity는 responseBody 와 반대 장단점 가진다.

## responseBody를 사용 추천? 왠냐면,
 * @ResponseStatus 어노테이션과 HttpServletResponse 으로 위에서 언급한 단점 커버 가능.
 * 대신, ResponseStatus는 정적인 특성이 있음. 즉 동적으로 상태코드가 변경되지 않음. (204를 줘야하는데 200만 응답된다던지)

결론: 팀 by 팀 별로 알아서 사용하라.
