# 9. try-finally 보다 try-with-resources 를 사용하라
## 핵심 정리
 * try-finally 는 java7부터 더이상 최선의 방법이 아니다
    * 특히 관리,해제할 자원이 2개 이상이라서 try-finally을 2번 써야하면 코드가 매우 지저분해진다.
    * 자원이 2개인데 try-finally 한번만 사용하면, 둘 중 하나의 자원에서 에러가 발생했을때 다른 자원에서 leak 발생할 수 있다.
 * try-with-resources 사용하면 코드가 더 짧고 분명해진다.
    * 또한 try-finally 와 마찬가지로, catch와 finally 같이 사용 가능하기 때문에 단점은 없고 장점만 존재한다.
 * 더 중요한 장점은, Exception 씹힘이다.
    * try-finally 에서는, try 에서도 에러 발생하고 finally 에서 자원 해제 과정에서도 에러가 발생했다고 하면, try쪽 에러는 씹히고 finally 에러만 나타난다.
    * try-with-resources 에서 동일한 상황에서는, try 에서 발생한 에러가 먼저 보일 것이고, 이후 close 호출될때 자원 해제시 발생하는 에러도 나타나게 된다.

#### try-finally 와 try-with-resources 사용 예시 비교
```
BufferedReader br = new BufferedReader(new FileReader(path));
try {
  return br.readLine();
} catch() {

} finally {
  br.close();
}
```

 * BufferedReader가 Closeable을 구현하고 있기 때문에 자원 반납 코드가 필요 없다.
 * 특히 자원이 여러개일때 효과적이다.
```
try (BufferedReader br = new BufferedReader(new FileReader(path))) {
  return br.readLine();
}
```

## 완벽 공략
#### 자바 퍼즐러(자바 퀴즈를 내며 공부할 수 있는 책) 예외 처리 코드 실수
 * -
#### try-with-resources 바이트 코드
 * close()를 여러번 호출한다 - 멱등성 필요
 * suppressed로 예외를 모두 포함시키게 한다
```
BufferedReader br = new BufferedReader(new FileReader(path));
String var2;
try {
   var2 = br.readLive();
} catch(Throwable var5) {
   try {
      br.close();
   } catch(Throwable var6) {
      var5.addSuppressed(var6);
   }
   throw var5;
}
br.close();
```


