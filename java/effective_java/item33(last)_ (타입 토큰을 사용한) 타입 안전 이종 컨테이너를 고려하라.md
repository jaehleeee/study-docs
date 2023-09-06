# 33. (타입 토큰을 사용한) 타입 안전 이종 컨테이너를 고려하라

## 핵심 정리
#### 타입 안전 이종 컨테이너
 * : 한 타입의 객체만 담을 수 있는 컨테이너가 아니라 여러 다른 타입(이종)을 담을 수 있는 타입 안전한 컨테이너
 * 타입 토큰 : String.class 또는 Class<String>
 * 구현 방법 : 컨테이너가 아니라 "키"를 매개변수화 하라
```
private Map<Class<?>, Object> map = new HashMap<.();

public <T> void put(class<T> clazz, T value) {
  this.map.put(clazz, value);
}

```


## 완벽 공략
#### 수퍼 타입 토큰
 * ㅇㅇ

#### 한정적 타입 토큰
 * ㅇㅇ
