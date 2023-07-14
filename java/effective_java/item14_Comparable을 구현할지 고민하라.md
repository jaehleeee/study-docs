# 14. Comparable을 구현할지 고민하라
## 핵심 정리
#### compareTo 규약
 * 자기자신(this)이 compareTo 메서드 전달되는 파라미터보다 크면 양수, 같으면 0, 작으면 음수 (오름차순 기준)
 * 반사성(a==a), 대칭성(a<b -> b>a), 추이성(a<b & b<c -> a<c) 만족해야한다.
 * (반드시까진 아니지만 되도록) a.compareTo(b) == 0 이면 a.equals(b) == true 여야 한다.
#### Comparable 구현방법1 - Comparable 인터페이스 구현
 * compareTo 메서드 override
 * 비교에 사용될 필드들의 클래스에 compareTo 메서드가 구현되어 있다면 이를 활용한다.
#### Comparable 구현방법2 - Comparator 이용
 * Comparator 인터페이스의 static 메서드를 이용해 생성
 * 추가 비교 필드가 있다면 체이닝 형식으로 제공된 메서드(thenCompareingInt) 사용 가능
```
// 예시1
Comparator<Person> cmp = Comparator.comparing(Persion::getLastName, String.CASE_INSENSITIVE_ORDER);

// 예시2
Comparator<PhoneNumber> cmp = comparingInt((PhoneNumber ph) -> pn.areacode)
    .thenCompareingInt(PhoneNumber::getPrefix)
    .thenCompareingInt(PhoneNumber::getLineNum);

```

## 완벽 공략
#### 제네릭 인터페이스는 Comparator가 컴파일 타임에 정해진다.
 * 컴파일 시점 : 빌드 시점
 * 런타임 시점 : 실행중 시점
