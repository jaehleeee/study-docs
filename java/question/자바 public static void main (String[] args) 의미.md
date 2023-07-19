#  자바 public static void main (String[] args) 의미

https://docs.oracle.com/javase/specs/jls/se12/html/jls-12.html#jls-12.1.4

```
The method main must be declared public, static, and void. It must specify a formal parameter (§8.4.1) whose declared type is array of String.
Therefore, either of the following declarations is acceptable:

public static void main(String[] args)
,
public static void main(String... args)

```

 * java의 진입점을 의미한다.
 * jvm은 main 메서드를 가장 먼저 찾아서 실행한다.
    * 이 시점엔 다른 클래스는 로드되어 있지 않았기 때문에, 무조건 public으로 선언해야 한다.
    * 그리고 인스턴스들이 생성되지 않은 시점이기 때문에, static 으로 선언해야 한다.
    * 메인 매서드의 종료는 프로그램의 종료이므로 리턴 타입이 필요없다. 그러니 void 리턴
 * public, static, void, args 각 의미는 이미 알고 있으니 생략
