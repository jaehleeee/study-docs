# 11. equals 재정의 하려거든 hashCode도 재정의하라
## 핵심 정리
#### hashCode 규약
 * equals 비교에 사용하는 정보가 변경되지 않았다면 hashCode도 매번 같은 값을 리턴해야 한다.
   * 즉, equals에 사용되는 정보들을 이용해 hashCode를 만들어야 한다.
 * 두 객체의 equals가 같다면, hashCode의 값도 같아야 한다.
 * 두 객체의 equals가 다르더라도, hashCode는 값이 같을 수 있지만 성능을 고려해 다른 값 리턴이 좋다.
    * 서로 다른 객체의 hashCode가 같다면(hash collision), Hash Table에서 같은 버킷에 들어가 링크드 리스트로 저장된다. 즉 서로 다른 hashCode보다 성능이 떨어진다.
#### hashCode의 쓰임새
 * Hash Table의 적절한 위치에 저장할때 사용된다. 즉 haspMap, hashSet, HashTable 등의 컬렉션 프레임워크 클래스 내부적으로 사용된다.
#### hashCode의 구현 방법
 * 간단한 방법 2가지
    * lombok의 @EqualsAndHashCode 어노테이션 활용
    * Objects 클래스에서 자동으로 만들어주는 방법이 있다. `Objects.hash(field1, field2, ..)`
 * 직접 만들어볼꺼라면?
   * 핵심 필드를 하나 고르고, 해당 필드가 primitive 타입이라면 wrapper 타입의 hashCode를 사용하면 된다.
   * primitive 타입이 아니라면, 해당 타입의 hashCode를 사용한다.
   * 필드가 여러개면? 31을 이전 필드 hashCode에 곱해서 다음 필드 hashCode를 더한다.
      * 왜 31? 홀수여야하고 + 어떤 두명이 개발자가 사전의 모든 단어를 hash할때 31이 가장 충돌이 적었다고 한다..
 
#### 전형적인 hashCode 구현 예시
```
@Override public int hashCode() {
  int result = Short.hashCode(areaCode);
  result = 31 * result + Short.hashCode(prefix);
  result = 31 * result + Short.hashCode(lineNum);

  return result;
}
```


## 완벽 공략
#### 해시맵 내부 연결리스트
 * ㅇ
#### 스레드 안전
 * ㅇ
