# 8장 URL 단축기 설계

## 1단계 문제 이해 및 설계 범위 확정
 * 쓰기 연산 : 매일 1억개의 단축 url 생성
 * 초당 쓰기 연산 1억/24/3600 = 1160
 * 읽기 : 쓰기의 1/10으로 가정
 * 서비스를 10년간 운영한다고 가정하면 3650억개 레코드 보관해야 한다.
 * 축약전 URL 평균 길이 100 이라 가정하면
 * 3560억 * 100바이트 = 36.5TB

## 2단계 개략적 설계안

<img src="https://github.com/jaehleeee/study-docs/assets/48814463/664c7e7f-8ebb-4943-a6f9-44ebf1b614c0" width="500"/>

### API 엔드포인트
 * 단축 URL 생성 메서드 POST
 * URL 리디렉션용 엔드포인트 : 단축 url이 오면 원래 URL을 보내주기 위한 용도
### URL 리디렉션
 * URL을 받은 서버는 그 url을 원래 url로 바꿔서 301 응답에 Location 헤더에 넣어 반환한다.
 * 301과 302 차이
   * 301 Permanently Moved : 영구적으로 Location 헤더에 반환된 url로 이전되었다는 응답. 브라우저는 이 응답을 캐시한다.
   * 302 Found : 일시적으로 Location 헤더가 지정하는 Url에 의해 처리되어야 한다는 응답.
 * URL 리디렉션 구현하는 가장 직관적인 방법은 해시테이블을 사용하는 것.
   * 단축 URL 생성시 hashTable.put(단축url, 원래url)
   * 원래 URL = hashTable.get(단축url)
### URL 단축
 * 단축 url이 www.tinyurl.com/{hashValue} 형태라고 해보자.
 * f(긴URL) = 단축URL이 되는 f(x) 해시함수를 찾아야 한다.
 * 해시함수의 조건
   * 긴 url이 다르면 해시값도 달라야 한다.
   * 복원될 수 있어야 한다.
  
## 3단계 상세설계
### 데이터 모델







