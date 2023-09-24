# 13. clone 재정의는 주의해서 진행하라
## 핵심 정리 (clone 안써서 skip)

## 완벽 공략
#### 깊은 복사와 얇은 복사
```
얇은 복사(Shallow Copy)는 원본 객체의 참조(reference)를 복사하는 것을 의미합니다.
즉, 복사된 객체는 원본 객체와 동일한 메모리 주소를 참조하게 됩니다.
따라서 한 객체의 상태 변경이 다른 객체에 영향을 줄 수 있습니다.
얇은 복사는 객체를 복사하는 데에는 빠르지만, 객체 간의 독립성을 보장하지 않습니다.

깊은 복사(Deep Copy)는 원본 객체의 내용을 완전히 복사하여 새로운 독립적인 객체를 생성하는 것을 의미합니다.
복사된 객체는 원본 객체와 다른 메모리 주소를 참조하게 됩니다.
따라서 한 객체의 상태 변경이 다른 객체에 영향을 주지 않습니다.
깊은 복사는 객체 간의 독립성을 보장하지만, 복사하는 데에는 더 많은 시간과 메모리를 필요로 할 수 있습니다.
```

#### UnCheckedException (RuntimeException)
 * 컴파일시 체크되지도 않고, try-catch로 잡지 않아도 된다. 롤백된다
 * UnCheckedException 예시: `NullPointerException`, `ArrayIndexOutOfBoundsException`
 * CheckedException 예시: `FileNotFoundException`, `DataAccessException`
 * 그럼 CheckedException는 언제 사용할까?
    * 클라이언트에게 알려줘야하는 경우
    * ex: 특정 조건의 파라미터를 호출하면 exception이 발생한다.
 * UnCheckedException는 언제 사용하는가? (왜 강제하는가)
    * 클라이언트가 해결할 방법이 없는 경우이기 때문이다.
    * 서버에서 수정이 필요한 경우다.

#### TreeSet
 * Abstract Set을 확장했고, 정렬된 Set이다. (추가된 순서는 상관없다)
 * 내부적으로 이진서치트리를 사용한다
    * 넣고 빼고 O(logN)
    * 조회할때 O(N) 
 * 기본적으로 자연적인 순서(natural order)이며 오름차순
 * 스레드 safe하지 않다.
   * 동기화가 필요하다면, Collections 에서 제공하는 static synchronizedSet 메서드를 감싸서 사용할 수 있다. (성능은 느려진다)
   * `Set<PhoneNumber> phoneNumbers = Collections.synchronizedSet(new TreeSet<>())`
 * Comparator를 즉시 구현함으로써, 정렬 기준을 커스터마이징할 수 있다
```
TreeSet<PhoneNumber> numbers = TreeSet<>(Comparator.comparingInt(PhoneNumber::hashCode));
```
