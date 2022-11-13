# 4. 템플릿 콜백 패턴

#### 콜백이란?
 * 다른 코드의 인수로서 넘겨주는 실행 가능한 코드.
 * 콜백을 넘겨받는 코드는 이 콜백을 필요에 따라 즉시 실행할수도, 아니면 나중에 실행할 수도 있다.

>A callback function is a function which is: <br>
passed as an argument to another function, and, <br>
is invoked after some kind of event. <br>
(From StackOverflow)

#### 템플릿 콜백 패턴
 * GOF에 정의된 패턴은 아니고, 스프링 내부에서 자주 사용하는 패턴 (JdbcTemplate, RestTemplate, RedisTemplate 등)
 * 이전에 배운 전략 패턴 중 Strategy를 필드 주입이 아니라, 파라미터 주입을 통해서 사용하는 방식이 템플릿 콜백 패턴이라고 볼 수 있다.
    * Context를 Template으로, Strategy를 Callback 으로 부르면 동일. 

![image](https://user-images.githubusercontent.com/48814463/201499627-0148bcce-f45c-4459-a5e0-204c1118b1ba.png)

#### Trace 로깅 예시 코드
```java
public interface TraceCallback<T> {
    T call();
}

---

public class TraceTemplate {

    private final LogTrace trace;

    public TraceTemplate(LogTrace trace) {
        this.trace = trace;
    }

    public <T> T execute(String message, TraceCallback<T> callback) {
        TraceStatus status = null;
        try {
            status = trace.begin(message);
            
            //로직 호출
            T result = callback.call();

            trace.end(status);
            return result;
        } catch (Exception e) {
            trace.exception(status, e);
            throw e;
        }
    }
}
```

### RestTemplate 예시
 * RestTemplate 클래스는 RestOperations 인터페이스를 구현했으며, RestOperations 에는 Rest api에 사용되는 `getForObject, postForObject, exchange, execute` 메서드 등이 정의되어 있다.
 * RestTemplate 코드를 보면, `doExecute` 라는 메서드르 만들어 이 메서드를 이용하여 RestOperations의 `getForObject, postForObject, exchange` override에서 사용되고 있다.
 * 즉, 변하지 않는 부분은 doExecute에 모으고, 변하는 부분인 여러가지 전략들(getForObject, postForObject 등)들만 바꿔 끼워가며 RestTemplate를 제공하고 있다.

#### excute 메서드 예시
```java
@Override
@Nullable
public <T> T execute(String url, HttpMethod method, @Nullable RequestCallback requestCallback,
    @Nullable ResponseExtractor<T> responseExtractor, Object... uriVariables) throws RestClientException {

  URI expanded = getUriTemplateHandler().expand(url, uriVariables);
  return doExecute(expanded, method, requestCallback, responseExtractor);
}

---

@Nullable
protected <T> T doExecute(URI url, @Nullable HttpMethod method, @Nullable RequestCallback requestCallback,
    @Nullable ResponseExtractor<T> responseExtractor) throws RestClientException {

  Assert.notNull(url, "URI is required");
  Assert.notNull(method, "HttpMethod is required");
  ClientHttpResponse response = null;
  try {
    ClientHttpRequest request = createRequest(url, method);
    if (requestCallback != null) {
      requestCallback.doWithRequest(request);
    }
    response = request.execute();
    handleResponse(url, method, response);
    return (responseExtractor != null ? responseExtractor.extractData(response) : null);
  }
  catch (IOException ex) {
    String resource = url.toString();
    String query = url.getRawQuery();
    resource = (query != null ? resource.substring(0, resource.indexOf('?')) : resource);
    throw new ResourceAccessException("I/O error on " + method.name() +
        " request for \"" + resource + "\": " + ex.getMessage(), ex);
  }
  finally {
    if (response != null) {
      response.close();
    }
  }
}
```
