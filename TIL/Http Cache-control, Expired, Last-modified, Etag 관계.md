## Cache-control: max-age 와 Expires 의 관계
 * cache 가 가장 먼저 참고 하는 것은 max-age(seconds), 그 다음이 expires
 * 문법 `Cache-Control: max-age=<seconds>`, `Expires: <http-date>`
   
`응답 내에 "max-age" 혹은 "s-max-age" 디렉티브를 지닌 Cache-Control 헤더가 존재할 경우, Expires 헤더는 무시됩니다.`

## Etag
 * 특정 버전의 리소스를 식별하는 식별자
 * 특정 URL 의 리소스가 변경된다면, 새로운 ETag 가 생성
 * w/ 지시자가 붙으면 weak
### 유즈케이스
```
ETag의 용례는 변경되지 않은 리소스를 캐시하는 것.

사용자가 URL을 재방문 했을 때(ETag를 가지고 있는 상태에서)
보유한 ETag가 너무 오래되어 사용될 수 없다고 판단되면
클라이언트는 If-None-Match 헤더 필드에 ETag를 전송

If-None-Match: "33a64df551425fcc55e4d42a148795d9f25f89d4"

서버는 클라이언트의 ETag를 현재 버전 리소스의 ETag와 비교하고,
두 값이 일치하는 경우(리소스에 변경이 없는 경우) 304 Not Modified 상태를 반환.
(이 상태는 캐시된 버전이 여전히 유효하다는 것을 클라이언트에게 알려줌)
```

## Last-Modified (응답 헤더)
 * 본 서버가 리소스가 마지막으로 수정되었다고 생각하는 날짜와 시간이 포함
 * ETag 헤더보다 정확하진 않지만 이 태그는 대비책으로 사용
    * If-Modified-Since 또는 If-Unmodified-Since헤더를 포함하는 조건부 요청은 이 필드를 사용
 * `Last-Modified: <day-name>, <day> <month> <year> <hour>:<minute>:<second> GMT`

## If-Modified-Since
 * 요청 헤더
 * 조건부 요청으로 서버가 지정된 날짜 이후 수정된 경우엔 200과 함께 리소스를 리턴
 * 만약 수정되지 않았다면, 304 응답.
```
If-Modified-Since는 If-Unmodified-Since 와는 다르게 GET 또는 HEAD 에서만 쓸수 있습니다.
서버가 If-None-Match를 지원하지 않는 한 If-None-Match 를 함께 사용시 무시 됩니다.
```

### 참고
 * https://developer.mozilla.org/ko/docs/Web/HTTP/Headers
