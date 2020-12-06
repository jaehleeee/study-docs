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
#### 서블릿이란?
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
#### 서블릿 생성 방법
 * HttpServlet을 상속한 Servlet 클래스를 만든다.
 * web.xml에 `<servlet>` 태그에 생성한 서블릿의 네임을 `<servlet-name>`태그에 설정하고 클래스 위치와 클래스명을 `<servlet-class>` 태그에 설정한다.
 * 또한 web.xml에 `<servlet-mapping>` 태그에  `<servlet>` 태그로 정의해둔 서블릿을 `<url-pattern>`으로 어떤 요청과 매핑할지 설정한다.
 * 요청마다 전부 서블릿을 만드는 것은 너무 비효율적
    * 그래서 dispatcherServlet이 나옴. (아래서 자세히)

#### 서블릿 리스너
 * 웹 앱에서 발생한 이벤트를 갑지하여 필요한 작업을 넣는 역할
 * 서블릿 컨텍스트 이벤트과 세션과 관련된 이벤트 크게 2가지로 나눌 수 있음.
 * 생성 방법
    * `ServletContextListener`를 implement하는 클래스 생성
    * `contextInitialized` 메서드를 오버라이드하면 이벤트가 발생하는 직후 해당 함수가 호출됨
        * dl 함수의 파라미터인 `servletContextEvent` 변수의 `getServletContext().setAttribute(...)` 를 통해 컨테스트에 어트리뷰트를 추가할 수 있다.
        * 해당 어트리뷰트는 서블릿 객체에서 `getServletContext()/getAttribute(...)` 메서드를 통해 사용가능 
    * 이렇게 생성한 리스너를 web.xml에 `<listener>` 태그에 설정 가능.


#### 필터
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
