# 스프링 웹 MVC 강의
 * 강사 : 백기선님
 * 강의자료
    * https://drive.google.com/file/d/1DM0Hh_HRieS_e1X2TiUC1ZKX3VXNwg2J/view?usp=sharing
 

## 1. 스프링 MVC 동작원리
### 1. MVC 소개
 * MVC란?
     * M : 모델, 화면에 전달할 혹은 전달받은 데이터 객체(DTO)
     * V : 뷰, 데이터를 보여주는 화면
     * C : 컨트롤러, 모델 객체의 데이터를 변경하거나 뷰에 전달
 * MVC 장점
   * 뷰, 모델, 컨트롤러 각각 따로 개발 가능 (낮은 의존도)
   * 한 모델에 여러 형태 뷰 가질 수 있음.
 * MVC 단점
   * 높은 학습 곡선
   * 코드 일관성 유지 노력이 필요함 

### 2. 서블릿
#### 1) 서블릿이란?
   * 자바 웹 애플리케이션 스펙과 api 제공
   * 한 프로세스 내에 자원을 공유하는 쓰레드를 만들어서 요청을 처리
     * 서블릿 이전에 사용 기술인 CGI는 요청마다 프로세스를 만들어 사용함.
 * 서블릿 엔진 또는 컨테이너 (톰켓, 제티, 언더토우)
   * 세션관리, 네트워크 서비스, 서블릿 생명주기 관리
   * 서블릿 컨테이너가 서블릿을 실행할 수 있다.
 * 서블릿 생명주기
   1. 최소 요청을 받으면 서블릿 컨테이너가 서블릿 인스턴스의 `init()` 메서드를 호출하여 초기화 한다.
       * 최초 요청만 초기화 작업을 진행하고 그 다음 요청부터는 이 과정을 생략
   2. 각 요청은 별도의 쓰레드로 처리하고, 이때 서블릿 인스턴스의 `service()` 메서드를 호출
       * 이 안에서 http 요청을 받고 http 응답을 만든다.
       * `service()` 메서드는 보통 HTTP Method에 따라 `doGet()`, `doPost()` 등으로 처리를 위임한다.
   3. 서블릿 컨테이너 판단에(보통 서버 종료시점) 따라 해당 서블릿을 메모리에서 내려야할 시점에 `destory()` 호출한다.
#### 2) 서블릿 생성 방법
 * HttpServlet을 상속한 Servlet 클래스를 만든다.
 * web.xml에 `<servlet>` 태그에 생성한 서블릿의 네임을 `<servlet-name>`태그에 설정하고 클래스 위치와 클래스명을 `<servlet-class>` 태그에 설정한다.
 * 또한 web.xml에 `<servlet-mapping>` 태그에  `<servlet>` 태그로 정의해둔 서블릿을 `<url-pattern>`으로 어떤 요청과 매핑할지 설정한다.
 * 요청마다 전부 서블릿을 만드는 것은 너무 비효율적
    * 그래서 dispatcherServlet이 나옴. (아래서 자세히)

#### 3) 서블릿 리스너
 * 웹 앱에서 발생한 이벤트를 갑지하여 필요한 작업을 넣는 역할
 * 서블릿 컨텍스트 이벤트과 세션과 관련된 이벤트 크게 2가지로 나눌 수 있음.
 * 생성 방법
    * `ServletContextListener`를 implement하는 클래스 생성
    * `contextInitialized` 메서드를 오버라이드하면 이벤트가 발생하는 직후 해당 함수가 호출됨
        * dl 함수의 파라미터인 `servletContextEvent` 변수의 `getServletContext().setAttribute(...)` 를 통해 컨테스트에 어트리뷰트를 추가할 수 있다.
        * 해당 어트리뷰트는 서블릿 객체에서 `getServletContext()/getAttribute(...)` 메서드를 통해 사용가능 
    * 이렇게 생성한 리스너를 web.xml에 `<listener>` 태그에 설정 가능.

#### 4) 필터
* 들어온 요청을 서블릿으로 보내거나 응답을 클라이언트로 보내기전에 특별한 처리나 제한이 필요한 경우 설정.
* 체인 형태이므로 반드시 체인 설정을 해야 각 필터를 거치는 필터를 만들 수 있다.
* 생성 방법
    * `Filter`를 implement하는 클래스 생성
    * override 한 함수 중 `doFilter` 함수에서 `chain.doFilter(request, response)` 해줘야 필터가 작동된다.
    * web.xml에서 `filter` 태그와 `filter-mapping` 태그를 설정해야 한다.


### 3. 스프링 IoC 컨테이너
#### 1) 서블릿 애플리케이션에 스프링이 제공하는 IoC 컨테이너 연동 방법
 * `spring web MVC` dependency 추가
 * `org.springframework.web.context.ContextLoaderListener` 를 리스너로 추가 가능.
    * application context를 서블릿들이 공통으로 사용하는 ServletContext에 추가한다.
       * ContextLoaderListener의 `initWebApplicationContext(ServletContext servletContext)` 함수 내부 코드 중
          * `servletContext.setAttribute(WebApplicationContext.ROOT_WEB_APPLICATION_CONTEXT_ATTRIBUTE, this.context);`
    * 서블릿들이 applicationContext를 사용할 수 있도록 생애주기에 맞춰서 생성하여 서블릿 컨텍스트에에 등록해주는 역할을 한다. + 서블릿이 종료될 때 applicationContext를 제거한다.
    * `web.xml` 스프링 설정이 필수
      * `<context-param>` 태그 하위에 여러 context 파라미터를 설정할 수 있다.
         * contxtClass : 어플리케이션 컨텍스트 타입 지정
         * contextConfiguration : contextConfiguration 의 위치
      ```xml
      <context-param>
        <context-name>contxtClass</context-name>
        <context-value>org.springframework.web.context.support.AnnotationConfigWebApplicationContext</context-value>
      </context-param>
      <context-param>
        <context-name>contextConfiguration</context-name>
        <context-value>{contextConfiguration 위치}</context-value>
      </context-param>
      ```
      `contextClass`(어떤 application context타입으로 설정할 것인가),`contextConfigLocation`(자바 설정 파일)
### 4. DispatcherServlet
https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-servlet
#### 1) DispatcherServlet 하는 일
 * Root WebApplicationConetext를 상속하는 Servlet WebApplicationConetext를 만든다.
    * Root WebApplicationConetext가 설정되어 있지 않으면 상속하지 않음.
 * 참고
    * Servlet WebApplicationConetext는 web과 관련된 빈을 등록한다. ex) Controller, HandlerMapping 등
    * 다른 DispatcherServlet도 공용으로 사용할 수 있는 Root WebApplicationConetext는 web과 관련되지 않은 빈을 등록한다. ex) Service, Repository 등
 * 스프링 부트 사용하지 않는 스프링 MVC 
    * 서블릿 컨네이너(ex, 톰캣)에 등록한 웹 애플리케이션(WAR)에 DispatcherServlet을 등록한다. 
       * web.xml에 서블릿 등록 
       * 또는 WebApplicationInitializer에 자바 코드로 서블릿 등록 (스프링 3.1+, 서블릿 3.0+) ● 세부 구성 요소는 빈 설정하기 나름. 
 * 스프링 부트를 사용하는 스프링 MVC 
    * 자바 애플리케이션에 내장 톰캣을 만들고 그 안에 DispatcherServlet을 등록한다. 
       * 스프링 부트 자동 설정이 자동으로 해줌. 
    * 스프링 부트의 주관에 따라 여러 인터페이스 구현체를 빈으로 등록한다.

#### 2) DispatcherServlet 동작원리
 * 초기화
    * 다음의 특별한 타입의 빈들을 찾거나, 기본 전략에 해당하는 빈들을 등록한다.
    *   HandlerMapping: 핸들러를 찾아주는 인터페이스 
    * HandlerAdapter: 핸들러를 실행하는 인터페이스 
    * HandlerExceptionResolver 
    * ViewResolver 
 * 동작순서
    * 요청을 분석한다. (로케일, 테마, 멀티파트 등) 
    * (핸들러 맵핑에게 위임하여) 요청을 처리할 핸들러를 찾는다. `getHandler`
      * DispatcherServlet는 기본적으로 2가지 핸들러매핑을 만든다.
         * BeanNameUrlHandlerMapping : 요청 url과 빈 name이 일치하는 Controller를 매핑
         * RepquestMappingHandlerMapping (주로 사용됨, request에 맞는 RestController 라는 Handler를 찾아줌)
    * (등록되어 있는 핸들러 어댑터 중에) 해당 핸들러를 실행할 수 있는 “핸들러 어댑터”를 찾는다.
    * 찾아낸 “핸들러 어댑터”를 사용해서 핸들러의 응답을 처리한다. 
      * 핸들러의 리턴값을 보고 어떻게 처리할지 판단한다. 
        *  이름에 해당하는 뷰를 찾아서 모델 데이터를 랜더링한다. 
        * @ResponseEntity가 있다면 Converter를 사용해서 응답 본문을 만들고. 
    * (부가적으로) 예외가 발생했다면, 예외 처리 핸들러에 요청 처리를 위임한다. 
    * 최종적으로 응답을 보낸다. 
#### 3) DispatcherServlet 구성요소
 * 기본 전략
    * DispatcherServlet.properties
 * LocaleResolver
    * 클라이언트의 위치(Locale) 정보를 파악하는 인터페이스 
    * 기본 전략은 요청의 accept-language를 보고 판단. 
 * ThemeResolver 
    * 애플리케이션에 설정된 테마를 파악하고 변경할 수 있는 인터페이스 
    * 참고: https://memorynotfound.com/spring-mvc-theme-switcher-example/ 
 * HandlerAdapter 
    * HandlerMapping이 찾아낸 “핸들러”를 처리하는 인터페이스
    * 여러 형태의 Handler 처리 방식이 있어도 모두 처리 가능해주는 인터페이스
    * 스프링 MVC 확장력의 핵심 
 * RequestToViewNameTranslator 
    * 핸들러에서 뷰 이름을 명시적으로 리턴하지 않은 경우, 요청을 기반으로 뷰 이름을 판단하는 인터페이스 

## 2. 스프링 MVC 설정
### 1. 직접 빈 둥록
 * @Configuration 이 사용된 자바 설정 파일에 직접 @Bean 어노테이션을 사용해서 등록.
    * 설정하지 않은 빈들은 DispatcherServlet.properties 를 따름.
 * 하지만 이 방법은 low level 방법
### 2. @EnableWebMvc
 * 반드시, WebApplicationContext에 ServletContext를 추가해야 한다.
 * 어노테이션 기반 mvc 사용할 때, 필요한 Bean 설정들을 자동으로 해주는 어노테이션 (Dispa tcherServlet.properties 와는 requestMapping 종류가 다르거나, messageConverter 개수 등이 좀 다름)
    * EnableWebMvc가 제공하는 Bean을 커스터마이징 하려면 WebMvcConfigurer interface를 implements 해서 커스터마이징하고 싶은 메서드만 커스터마이징하면 된다.
       * WebMvcConfigurer는 스프링부트에서도 사용됨.
 * EnableWebMvc 안에 DelegatingWebMvcConfiguration 클래스를 import 하는데, 이 안에 여러 기본적인 빈이 설정되어 있고, 커스텀하게 추가로 설정할 수 있는 함수도 제공된다.
    * 더 구체적으로 빈을 등록해주는 부분은 DelegatingWebMvcConfiguration이 상속하는 WebMvcConfigurationSupport 이 제공한다.
```
@Configuration
@EnableWebMvc
public class WebConfig implements WebMvcConfigurer {

  @Override
  public void configureViewResolvers(ViewResolverRegistry registry){
    ...
  }
}
```
### 3. WebMvcConfigurer 인터페이스
 * EnableWebMvc가 제공하는 Bean을 커스터마이징할 수 있게 도와줌.
 * ViewResolver를 커스터마이징하고 싶다? -> ConfigureViewResolver 를 override 해서 viewResolver 커스터마이징 가능

### 4. springBoot 없이, web.xml 없이 시작하기
``` java
public class WebApplication implements WebApplicationInitializer {
  @Override
  public void onStartup(ServletContext servletContext) throws ServletException {
    AnnotationConfigWebApplicationContext context = new AnnotationConfigWebApplicationContext();
    context.setServletContext(servletContext);
    context.register(WebConfig.class);
    context.refresh();

    DispatcherServlet dispatcherServlet = new DispatcherServlet(context);
    ServletRegistration.Dynamic app = servletContext.addServlet("app", ...);
    app.addMapping("/app/*")
  }
}
```
## 3. springboot의 MVC 설정
### 1. 설정
#### 1) 자동설정
 * 기본적으로 아무 설정을 하지 않으면 저장된 목록을 통해 자동으로 필수 빈을 설정한다.
 * 커스터마이징하고 싶다면?
    * application.properties
    * @Configureation + Implements WebMvcConfigurer : 스프링부트가 지원하는 스프링 MVC 자동설정 + 추가 설정
    * @Configureation + @EnableWebMvc + Implements WebMvcConfigurer : 스프링부트의 스프링 MVC 자동설정 사용하지 않음.
#### 2) Formatter 추가
 * converter, formatter는 WebMvcConfigurer를 이용한 addFormatter 필요없이, @Bean으로 등록해놓으면 springboot 가 알아서 빈으로 추가 등록한다.
#### 3) jsp 사용하려면?
 * JAR 프로젝트 만들 수 없고, WAR 프로젝트로 만들어야 함
    * war 프로젝트로 만들면,, ServletInitializer 가 자동으로 생성되며 이를 통해 tomcat에 배포할 수 있다.
 * java -jar로 실행은 가능함. 하지만 `실행 가능한 JAR 파일`은 지원하지 않음
### 2. WebMvcConfigurer 설정
#### 1) Formatter 추가
 * Formatter란?
    * Printer : 객체를 문자열로 출력
    * Parser : 문자열을 객체로 변환
 * Converter와의 차이점 : Converter는 좀 더 일반적인 변환체. 객체 -> 다른 객체로 변환시켜주는 역할 가능
 * 추가하는 방법
    * 추가하려는 커스터마이징 formatter를 @Conponent 추가되어 있어야 함.
    * 방법은 2가지
       1. WebMvcConfigurer의 addFormatters(FormatterRegistry formatterRegistry) 메서드 @Override 재정의
       2. (스프링부트에서만 가능) 추가하고자하는 포매터를 @Bean 으로 등록하면 springboot가 알아서 추가 등록해준다.
#### 2) 핸들러 인터셉터
 * 핸들러 인터셉터란?
    * 핸들러 맵핑에 설정할 수 있는 인터셉터, 핸들러 실행 전/후 시점에 부가 작업을 넣고 싶을때 사용
    * 로깅, 인증체크 Locale 변경 등에 사용됨
 * 구성
    * preHandle : 요청 처리 직전 부가작업 추가 가능, 어떤 핸들러(Controller)가 매핑되는지도 알 수 있음.
    * postHandler : 요청 처리 직후 부가작업 추가 가능, ModelAndView 객체에 뭔가를 추가할 수도 있다. 비동기 요청처리일 경우 호출되지 않음
    * afterCompletion : 뷰 렌더링까지 마친 이후 부가작업 추가 가능, 비동기 요청처리일 경우 호출되지 않음
 * 서블릿 필터와의 차이점
    * 서블릿보다 더 구체적인 처리(처리시점, 알 수 있는 정보가 다름) 가능
    * 서블릿은 좀 더 일반적인 용도의 기능 구현에 사용하는게 좋다.

#### 3) 리소스 핸들러
 * 리소스 핸들러 : 이미지, css, 자바스크립트, html 등 정적 리소스 요청을 처리하는 핸들러
 * 톰캣에는 이러한 정적 리소스를 처리하는 디폴트 서블릿이 기본적으로 내장되어 있다.
 * 보통 가장 우선순위가 낮은 핸들러로 설정되어 있다.
 * 스프링부트
    * 기본 정적 리소스 핸들러 제공 및 캐핑 제공
    * `resources > static`, `resources > public` 경로 아래 파일은 자동 매핑
    * 
 * 설정을 변경하려면? WebMvcConfigurer 이용
    * addResourceHandler 메서드를 오버라이드하여 어떤 패턴, 어디서 리소스를 찾을지, 캐싱 설정을 변경 및 추가할 수 있다.
       * ResourceResolver : 요청에 해당하는 리소스를 찾는 전략 설정
       * ResourceTransformer : 응답으로 보낼 리소스를 수정하는 전략 설정

#### 4) HTTP 메세지 컨버터
 * `@RequestBody` 와 `@ResponseBody` 에서 사용한다.
 * 기본 컨버터에 새로운 컨버터 추가하려면 : extendMessageConverters (W/ WebMvcConfigurer)
 * 기본 컨버터 무시하고 새롭게 설정하려면 : configureMessageConverters (W/ WebMvcConfigurer)
 * (추천) 의존성 추가로 컨버터 추가하기 (WebMvcConfigurationSupport <- 스프링부트 아닌 스프링MVC 기능)

 #### 5) 기타
  * 뷰 컨트롤러 : 단순 요청 url을 특정 뷰로 연결하고 싶을때 사용 가능
  * 뷰 리졸버 설정 : 핸들러에서 리턴하는 뷰 이름 문자열을 view 인스턴스로 변경해주는 뷰 리졸버 설정


## 4. 스프링 MVC 활용
### 1. 요청 매핑
#### 1) URL 매핑 패턴
 * `?`:한글자, `*`:여러 글자, `**`:여러 경로
 * 정규표현식 매핑
    * /{받을필드명:정규식} 형태로 받을 수 있음. 정규식에 해당되는 내용이 받을필드명으로 매핑됨 (pathvariable과 동일)
    * `@RequestMapping("/{name:[a-z]+}}")`
 * 패턴이 중복되는 경우? 가장 구체적으로 매핑되는 핸들러 선택
 * URI 확장자 매핑 지원
    * 스프링부트에서는 이 지원 기능이 기본적으로 막혀있음.
#### 2) 컨텐츠 타입 매핑
 * `@RequestMapping(consumes="application/json")` : 특정 타입의 컨텐츠 요청만 매핑, 요청헤더의 `contentType` 헤더와 연관됨
 * `@RequestMapping(produces="application/json")` : 특정한 타입의 응답만 만드는 핸들러, 요청헤더의 `accept` 헤더와 연관됨
#### 3) 메타 애노테이션
 * `@Retention` 애노테이션
    * Source : 소스코드까지만 유지, 컴파일 하면 사라짐
    * Class : 컴파일한 .class 파일에도 남아있음. 클래스 파일을 메모리로 읽어오면(런타임) 사라짐
    * Runtime : 클래스를 메모리로 읽어온 뒤에도 유지됨.
 * `@Documented` : 해당 애노테이션을 사용한 코드의 문서에 그 애노테이션 정보가 표기될지 결정
 * `@Target` : 해당 애노테이션을 어디에 사용할 수 있는지 결정

### 2. 핸들러 메소드
#### 1) 메소드 아규먼트 & 리턴 타입
 * 요청 또는 응답 자체에 접근 가능한 API
   * WebRequest 
   * NativeWebRequest 
   * ServletRequest(Response)
   * HttpServletRequest(Response)
 * 요청 본문을 읽어오거나, 응답 본문을 쓸 때 사용할 수 있는 API
   * InputStream : requestBody를 읽음.
   * Reader : requestBody를 읽음.
   * OutputStream : responseBody를 읽음.
   * Writer : responseBody를 읽음
 * PushBuilder : 스프링 5, HTTP/2 리소스 푸쉬에 사용
 * HttpMethod : GET, POST, ... 등에 대한 정보
 * LocaleResolver가 분석한 요청의 Locale 정보
   * Locale 
   * TimeZone 
   * ZoneId
 * @PathVariable : URI 템플릿 변수 읽을 때 사용, 타입변환 지원
 * @MatrixVariable : URI 경로 중에 키/값 쌍을 읽어 올 때 사용
 * @RequestParam : 서블릿 요청 매개변수 값을 선언한 메소드 아규먼트 타입으로 변환해준다. 단순 타입인 경우에 이 애노테이션을 생략할 수 있다. Map 타입이나 MultiValueMap 타입으로 변경 가능
 * @RequestHeader : 요청 헤더 값을 선언한 메소드 아규먼트 타입으로 변환해준다.
 * @ModelAttribute : 여러 곳에 있는 단순 타입 데이터를 복합 타입 객체로 받아오거나 객체를 새로 만들때 사용함. 생략 가능.
    * 바인딩 에러를 잡으려면 BindingResult 와 @Valid, @Validated 를 통해 에러 정보 받을 수 있다.
    * 컨트롤러에서 @RequestMapping이 없는 메서드 위에서 사용하면 ->  Model 객체를 초기화하고 특정 속성을 해당 컨트롤러의 모든 Model에 넣어주고 싶을때 사용 가능
    * 메서드의 아규먼트가 아닌, @RequestMapping과 같이 메서드 위에 애노테이션으로 사용하면 -> 메서드에서 리턴하는 객체를 모델에 넣어준다. (생략가능, 이 경우 view명은 요청경로와 동일해야함)
 * @SessionAttributes : HttpSession을 직접사용하는 방법도 있지만, 이 애노테이션을 설정한 컨트롤러는 해당 필드명과 매핑된는 정보를 자동으로 세션에 추가해준다.
    * 세션 비우고 싶다면, 메서드에 SessionStatus 를 아규먼트로 받아서, `sessionStatus.setComplete()` 호출해주면 된다.
    * @SessionAttribute 는 컨트롤러 밖(인터셉터, 필터 등)에서 만들어준 세션 데이터에 접근할 때 사용
 * RedirectAttributes : 스프링mvc 에서는 redirect 할때 Model 객체를 그대로 전달하지만, 스프링부트는 이 기능이 디폴트로 꺼져있음. 따라서 리다이렉트에 데이터를 넘기고 싶을때 RedirectAttributes 사용
    * FlashAttribute를 이용하면 1회성으로 데이터를 전달해준다. -> 리다이렉트된 url에서 Model 객체에서 받아 사용할 수 있다.
 * HttpEntity<requestBody 타입> : reqeust 바디와 헤더 모두 접근하고 싶을 때 사용

#### 2) MultipartFile 업로드
 * 스프링부트에서는 MultipartAutoConfiguration을 통해 디폴트로 설정되므로, 추가적인 설정없이 MultipartFile를 이용하여 파일 업로드 가능
 * application.properties 에서 관련 부가설정 가능
#### 3) 파일 다운로드
 * 스프링 ResourceLoader 사용
`Resource resource = resourceLoader.getResource("classpath:"+filename);`
 * 파일 다운로드 헤더 `Content-Disposition` : 사용자가 파일 다운로드 후 사용할 파일명