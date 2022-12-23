# 포인트컷

## 포인트컷 지시자 (Pointcut Designator)

### 종류
![image](https://user-images.githubusercontent.com/48814463/209236029-14ebcb49-3d50-4ffe-8874-fcac74f79286.png)

### execution 문법
`execution(접근제어자? 반환타입 선언타입?메서드이름(파라미터) 예외?)`
 * ?는 생략 가능
 * 메서드 실행 조인 포인트를 매칭한다.
 * `*` 같은 패턴을 지정할 수 있다.

#### 가장 많은 생략 예시
`execution(* *(..))`
