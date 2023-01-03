# Checked Exception vs Unchecked Exception

## Checked Exception
 * 반드시 처리해야 한다.
 * 처리하지 않으면 컴파일 단계에서 걸린다.
 * try, catch로 감싸든, throw 하든.
 * 트랜잭션에서 롤백하지 않는다.
    * * 롤백하고 싶다면 스프링의 @Transaction 어노테이션에서 `rollbackforClassName` 설정을 통해 롤백 할 수 있다.
 * 처리가 불필요한 Exception이라면, catch하여 Unchecked Exception 으로 변환해주고 좀 더 명확한 Exception name 으로 변경해주면 좋다.
 * 

## Unchecked Exception
 * Runtime Exception 하위 클래스이다.
 * 반드시 처리하지 않아도 된다.
 * 보통 개발자의 부주의로 발생한다.
 * 트랜잭션에서 롤백한다.
    * Error 클래스의 하위 클래스가 발생해도 롤백한다. 
    * 롤백하고 싶지 않다면 스프링의 @Transaction 어노테이션에서 `noRollbackforClassName` 설정을 통해 제외할 수 있다.

![image](https://user-images.githubusercontent.com/48814463/210290049-e1b4dafb-2c44-4485-9b34-3612f80c095e.png)

![image](https://user-images.githubusercontent.com/48814463/210290073-f5713883-55c1-45e3-8f89-da3f43284130.png)


### 참고
 * https://www.nextree.co.kr/p3239/
