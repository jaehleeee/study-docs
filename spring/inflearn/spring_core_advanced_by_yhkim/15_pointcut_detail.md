# 포인트컷

## 포인트컷 지시자 (Pointcut Designator)

### 종류
![image](https://user-images.githubusercontent.com/48814463/209236029-14ebcb49-3d50-4ffe-8874-fcac74f79286.png)

### execution 문법
`execution(접근제어자? 반환타입 선언타입?메서드이름(파라미터) 예외?)`
 * ?는 생략 가능
 * 메서드 실행 조인 포인트를 매칭한다.
 * `*` 같은 패턴을 지정할 수 있다.
 * 부모 타입을 설정해도 자식 타입은 매칭된다. 단, 부모 타입에 있는 메서드 (override)만 매칭된다.

![image](https://user-images.githubusercontent.com/48814463/209296520-3f55c7fd-50ae-4ff8-930c-8e2b204d978b.png)


#### 가장 많은 생략 예시
`execution(* *(..))`

### execution 이외의 pointcut
 * within은 부모 타입으로 지정시, 자식 클래스는 매칭되지 않는다. 정확한 클래스에만 매칭된다.
 * args는 부모 타입을 허용한다. 실제 런타임에 넘어온 파라미터 객체 인스턴스를 보고 판단한다. (execution은 메서드 시그니처로 판단하므로 부모타입 허용하지 않는다.)
 * `@target` : 해당되는 인스턴스의 모든 메서드를 조인포인트로 적용
 * `@within` : 해당되는 인스턴스 내 해당되는 특정 타입만 조인 포인트로 적용
