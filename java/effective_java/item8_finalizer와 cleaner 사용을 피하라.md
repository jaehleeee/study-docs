# 8. finalizer와 cleaner 사용을 피하라
## 핵심 정리
#### finalizer와 cleaner는 문제가 많다.
 * 즉시 수행 보장이 없고 실행되지 않을 수도 있다. 
 * 동작 중 예외 발생시 정리 작업이 처리되지 않을 수 있다.
 * 성능 문제가 있다.
 * 보안 문제가 있다.
#### 반납 자원이 있다면
 * AutoCloseable을 구현하고 close를 호출하거나, try-with-resource를 사용해야 한다.

## 완벽 공략
 * ㅇㅇ
