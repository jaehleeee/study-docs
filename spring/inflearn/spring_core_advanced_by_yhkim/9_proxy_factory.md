# 프록시 팩토리
 * JDK 동적 프록시는 인터페이스가 있을때, CGLIB는 구체 클래스가 있을때 사용할 수 있다. 그럼 상황 상황마다 바꿔가면서 써야하나? 불편하게?
   * 이러한 불편을 해결해주는 공통 로직이 있으면 좋겠다.
 * 스프링은 이러한 부분을 통합해서 편리하게 해주는 `프록시 팩토리` 라는 기능을 제공한다.
   * 인터페이스가 있으면 JDK 동적 프록시를 생성해주고, 구체 클래스만 있다면 CGLIB를 사용하며 이러한 설정도 커스텀하게 변경 가능하다.
 * 그럼 InvocationHander와 MethodInterceptor는 어떻게 해결할까? `Advice` 라는 개념을 만들었다. 
 * 개발자는 `Advice`에 공통 로직을 만들고, 결과적으로 InvocationHander와 MethodInterceptor는 `Advice`를 호출하도록 스프링이 세팅한다.

![image](https://user-images.githubusercontent.com/48814463/204924136-a0822caf-50d7-4928-afca-1dd4e4a5cd12.png)

![image](https://user-images.githubusercontent.com/48814463/204924601-e03c3dcb-d8e1-4402-b12f-f94c615e1f87.png)


#### 특정 조건에 맞을 때 프록시 로직을 적용하는 기능도 공통으로 제공되었으면?
 * 스프링은 `Pointcut` 이라는 개념을 도입해서 이 문제를 일관성있게 해결한다.

## ㅇㅇ
