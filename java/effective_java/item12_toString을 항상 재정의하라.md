# 12. toString을 항상 재정의하라
## 핵심 정리
#### 좋은 toString 
 * toString은 간결하면서 사람이 읽기 쉬운 형태의 유익한 정보를 반환해야 한다.
 * Objects의 toString은 `클래스이름@16진수해시코드`
 * 객제가 가진 모든 정보를 보여주는 것이 좋다. (물론 노출하지 않아야하는 데이터는 빼야겠지만)
#### (우리가 원하는 포맷이 있는 경우엔) 경우에 따라 lombok 등의 라이브러리를 사용하지 않는게 더 적절할 수 있다
