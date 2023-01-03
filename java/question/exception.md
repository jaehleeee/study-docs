# Checked Exception vs Unchecked Exception

## Checked Exception
 * 반드시 처리해야 한다.
 * 처리하지 않으면 컴파일 단계에서 걸린다.
 * try, catch로 감싸든, throw 하든.
 * 트랜잭션에서 롤백하지 않는다.

## Unchecked Exception
 * 반드시 처리하지 않아도 된다.
 * 보통 개발자의 부주의로 발생한다.
 * 트랜잭션에서 롤백한다.

![image](https://user-images.githubusercontent.com/48814463/210290049-e1b4dafb-2c44-4485-9b34-3612f80c095e.png)

![image](https://user-images.githubusercontent.com/48814463/210290073-f5713883-55c1-45e3-8f89-da3f43284130.png)
