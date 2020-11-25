# 스프링부트 개념과 활용 강의
 * 강사 : 백기선님
 * 강의자료
    * https://docs.google.com/document/d/1vT7UaL4OOG2uL0n2hod_-2HQnnwilfqC-lCZls_oG7s/edit?usp=sharing
 

## 1. 스프링부트 원리
### 1. 의존성 관리
 * dependency management
 * `spring-boot-dependencies`에 자동으로 매핑할 각 deepndency 최신 버전을 가지고 있다. 그래서 의존성 설정할때 버전은 필수가 아님.

### 2. 자동설정
 * 빈은 사실 두 단계로 나눠서 읽힘 (@SpringBootApplication 안에ㅠ   있음)
     * 1단계: @ComponentScan
     * 2단계: @EnableAutoConfiguration
 * @ComponentScan
     * @Component
     * @Configuration @Repository @Service @Controller @RestController
     * applicaiton.java 와 동일한 패키지보다 같거나 하위에 위치한 것만 빈으로 등록함.
 * @EnableAutoConfiguration
     * ComponentScan 이후에 빈을 등록함
     * *웹* 어플리케이션에 필요한 설정 파일들이 자동으로 설정됨
     * spring.factories
        * org.springframework.boot.autoconfigure.EnableAutoConfiguration
     * @Configuration
     * 조건에 따라 빈을 등록하거나 말거나 결정됨
        * @ConditionalOnXxxYyyZzz
            * `@ConditionalOnWebApplication(type = XXX)` 웹 어플리케이션 타입이 XXX인 경우 빈 등록
            * `@ConditionalOnBean` 특정 빈이 이미 빈으로 등록되어 있다면
            * `@ConditionalOnClass(YYY.class)` YYY.class가 클래스패스에 있으면 빈 등록
            * `@ConditionalOnMissingBean` 덮어쓰기 방지하기위해 해당 빈이 없는 경우만 빈 등록
        * @autoConfigureAfter

### 3. 내장 서블릿 컨테이너
 * 스프링부트는 서버가 아니고 툴에 더 가깝다고 할 수 있음. (아래 과정을 자동을 설정 해줌)
    * 톰캣 객체 생성
    * 포트 설정
    * 톰캣에 컨텍스트 추가
    * 서블릿 생성
    * 톰캣에 서블릿 추가
    * 컨텍스트에 서블릿 매핑
    * 톰캣 실행 대기

```java
Tomcat tomcat = new Tomcat();
tomcat.setPort(8080);

Context context = tomcat.addContext("/context-path", "/docBase");
HttpServlet servlet = new HttpServlet();

tomcat.addServlet("/context-path", "servletName", servlet);
context.addServletMappingDecoded("url-mapping", "servletName");
tomcat.start();

tomcat.getServer().await();
```

 * 스프링부트의 서블릿 관련 설정
    * `ServletWebServerFactoryAutoConfiguration` 서블릿 웹 서버 생성
    * `DispatcherServletAutoConfiguration` 서블릿 만들고 등록

### 4. 독립적으로 실행 가능한 JAR
 * (maven 인 경우) `mvn package` 실행하면 JAR 파일이 생성된다.
    * 생성된 JAR의 압축을 풀어보면, 작성해둔 java 파일이 클래스 파일로 들어있고, 의존성으로 받아온 라이브러리들이 jar 형태로 들어있다.
    * `org.springframework.boot.loader.jar.JarFile`가 내장 jar를 읽고 `org.springframework.boot.loader.Launcher`가 실행한다.
    * (참고) `mvn clean` 은 target 하위 모든 파일 삭제

## 2. 스프링부트 활용
### 1. SpringApplication
 * [참고 문서](https://docs.spring.io/spring-boot/docs/current/reference/html/spring-boot-features.html#boot-features-spring-application)
 * main 함수에서 application을 `run` 하는데, 그때 여러가지 설정 가능.
```java
public static void main(String[] args) {
    SpringApplication app = new SpringApplication(MySpringConfiguration.class);
    app.setBannerMode(Banner.Mode.OFF);
    app.run(args);
}
```
#### FailureAnalyzers
    * application start에 실패한 경우, 실패한 이유에 대해 자세히(+이쁘게) 로깅해주는 기능
 * 배너 설정 가능 (application 부팅시 나오는 로그)
#### ApplicationEvent
    * `ApplicationListener<ApplicationStartingEvent>` 인터페이스를 구현한 이벤트 리스너 클래스 생성
    * Override 함수 중 `onApplicationEvent(ApplicationStartingEvent event)` 함수를 오버라이딩하면 어플리케이션 시작 이벤트 발생시 기능 추가 가능.
    * `ApplicationStartingEvent` 이벤트는 ApplicationContext 생성 전이기 때문에 어노테이션으로 빈을 등록할 수 없고, 직접 메인함수의 SpringApplication 객체에 리스너를 인스턴스로 등록해야 한다.
       * 어노테이션으로 빈을 등록하려면 `ApplicationStartedEvent` 이벤트를 이용
    * 이외에도 여러 Application event가 있다.
       * https://docs.spring.io/spring-boot/docs/current/reference/html/spring-boot-features.html#boot-features-application-events-and-listeners
#### WebApplicationType
 * SERVLET(webMVC로 동작, 1순위)
    * `AnnotationConfigServletWebServerApplicationContext` 사용
 * REACTIVE(webFlux로 동작)
    * `AnnotationConfigReactiveWebServerApplicationContext` 사용
 * NONE
    * `AnnotationConfigApplicationContext` 사용
#### ApplicationArguments
 * 신규 클래스 생성 후 파라미터로 ApplicationArguments로 두면 ApplicationArguments를 사용할 수 있다.
    * java -jar XXX.jar 실행시 `-D`는 JVM 옵션 `--` ApplicationArguments 이다.

### 2. 외부설정
#### 프로퍼티 우선순위
 * 프로퍼티 우선 순위
 * 유저 홈 디렉토리에 있는 spring-boot-dev-tools.properties
 * 테스트에 있는 @TestPropertySource 어트리뷰트
    * location 어트리뷰트로 프로퍼티 파일을 지정할 수도 있고, property 어트리뷰트로 직접 설정도 가능
 * @SpringBootTest 애노테이션의 properties 어트리뷰트
 * 커맨드 라인 아규먼트
 * SPRING_APPLICATION_JSON (환경 변수 또는 시스템 프로티) 에 들어있는 프로퍼티
 * ServletConfig 파라미터
 * ServletContext 파라미터
 * java:comp/env JNDI 애트리뷰트
 * System.getProperties() 자바 시스템 프로퍼티
 * OS 환경 변수
 * RandomValuePropertySource
 * JAR 밖에 있는 특정 프로파일용 application properties
 * JAR 안에 있는 특정 프로파일용 application properties
 * JAR 밖에 있는 application properties
 * JAR 안에 있는 application properties
 * @PropertySource
 * 기본 프로퍼티 (SpringApplication.setDefaultProperties)

#### application.properties 우선순위 (순위 높은게 낮은거를 덮어씀)
 * `file:./config/` (상대경로는 실행파일 위치 기준)
 * `file:./` (상대경로는 실행파일 위치 기준)
 * `classpath:/config/`
 * `classpath:/`

#### 타입-safe 프로퍼티 @ConfigurationProperties
 * @ConfigurationProperties 어노테이션을 붙인 클래스를 생성해 특정 프로퍼티를 타입 세이프하게 가져올 수 있다.
 * 해당 프로퍼티 클래스를 빈으로 등록해서 사용할 수 있는데, main 함수가 있는 Application 클래스에 `@EnableConfigurationProperties({XXXProperties.class, ...})` 를 추가하면 된다.
    * 이 작업없이, 빈으로 등록 방법(프로퍼티 설정에선 잘 사용되진 않음)
       * Properties 클래스에 @Component 어노테이션을 붙여도 된다.
       * @Configuration 어노테이션이 있는 클래스(설정 클래스)에서 @Bean 어노테이션을 붙이고 빈으로 등록하고자 하는 객체를 return하는 함수를 만들면 빈으로 등록
 * Relaxed Binding
    * properites 파일에 있는 설정 변수명이 케밥, 언더스코어, 카멜케이스 등등이라도 properties.java로 매핑될때 융통성있게 바인딩 해준다.
### 3. 프로파일
#### @Profile
 * `spring.profiles.active={profile}` 프로파일 설정
 * 프로파일 설정시, 설정된 프로파일에 따라 application-{profile}.properties 이 자동으로 매핑됨
    * 참고 https://docs.spring.io/spring-boot/docs/1.5.6.RELEASE/reference/html/boot-features-external-config.html#boot-features-external-config-profile-specific-properties
 * `spring.profiles.include={profile}` 프로파일 추가
 * (참고) classpath 란?
    * java source나 resource 파일들이 어딨는지 알 수 있도록 applicationContext에 지정하는 위치.
       * intellij에서 File -> project structure -> Modules 탭에서 간단히 지정 가능.
    * resource에 있는 파일을 java에서 불러오려면 ResourceLoader 인터페이스(혹은 이를 구현한 FileSystemResourceLoader 클래스 등)를 이용하면 된다. 

## 3. 스프링 테스트 & devtools
### @SpringBootTest
 * 통합 테스트시 사용되는 테스트로, main 함수가 있는 `@SpringBootApplication` 을 찾아서 applicationContext를 구동하고 그 안의 빈도 모두 등록한다.(필요시 mock 객체로 등록)
    * 통합 테스트 말고, 슬라이스 테스트를 하고 싶다면, `@WebMvcTest`, `@JsonTest`, `@DataJpaTest` 등을 이용해야 한다.
       * `@WebMvcTest` 는 web과 관련된 빈(Controller, ControllerAdvice, Converter. Filter 등등만 해당)만 등록된다. service 등은 직접 mock 빈으로 직접 등록해줘야한다.(아래 @MockBean 이용) 이 어노테이션으로 테스트를 작성할때 `MockMvc` 객체를 @Autowired만 이용해서 바로 빈 등록 가능하다.
 * `@RunWith(SpringRunner.class)` 와 함께 사용해야함
 * SpringBootTest의 어트리뷰트로 webEnvironment 설정 가능
    * `webEnvironment = SpringBootTest.webEnvironment.MOCK` 설정시, 내장 톰캣 구동하지 않고 mock 테스트 가능
    * `webEnvironment = SpringBootTest.webEnvironment.RANDOM_PORT`(혹은 `DEFINED_PORT`) 설정시, 내장 통캣 사용함. `TestRestTemplate` 필요

```java
@RunWith(SpringRunner.class)
@SpringBootTest 

```

### @MockBean
 * ApplicationContext에 있는 빈을 Mock 객체로 교체
    * 컨트롤러 테스트 중 실제 서비스가 아닌 mock 서비스 객체로 테스트하고 싶을때 사용
    * 아래 코드 처럼 테스트에서 사용 가능
   ```java
   @MockBean
   HelloService helloService
   ```
 * 모든 @Test 마다 자동으로 리셋
 * `@AutoConfigureMcokMvc`를 클래스명 위에 붙이고 MockMvc 빈 객체를 @Autowired로 사용 가능

#### Spring-boot-devtools
 * 백기선님 본인은 안쓴다고 함. ;;
 * classpath에 있는 파일이 변경되면 application을 restart 하는데, 직접 껐다켜는것보다 훨씬 빠르다.
 * 단순히 소스 파일을 변경한다고 restart 하는건 아니고, 빌드를 해야 한다.
 * 라이브 리로드 : 리스타트 헀을때 브라우저 자동 리스페시 하는 기능.
    * 브라우저에 확장앱 설치도 필요함.

## 4. 스프링 웹 MVC
### 1. 웹 MVC 소개
#### 웹 mvc 설정
 * MVC 설정 확장 : `@Configuration` + `WebMvcConfigurer` 인터페이스 구현
 * MVC 설정 재정의(이 케이스는 거의 없음.) : `@Configuration` + `@EnableWebMvc` 
### 2. HttpMessageConverters
 * https://docs.spring.io/spring/docs/5.0.7.RELEASE/spring-framework-reference/web.html#mvc-config-message-converters
 * http 요청 본문을 객체로 변경하거나, 객체를 http 응답 본문으로 변경시 사용됨.
### 3. ViewResolver
#### ViewResolver
 * 특별히 설정하지 않으면, DispatcherServlet에 등록된 InternalResourceViewResolver(기본적으로 jsp 지원)가 사용됨.
 * spring-boot-starter-web 에는 톰캣은 있지만 jsp 엔진은 없음.
    * 물론 추가적인 의존성 설정으로 간단히 설정 가능.
 * 폴더 구조
    * src > main > resource > templates : 뷰 관련 파일
#### viewResolver 설정
 * 설정 방법1 : java
 ```java
@Configuration 
@EnableWebMvc 
public class MvcConfiguration extends WebMvcConfigurerAdapter{ 
    @Bean public ViewResolver getViewResolver() { 
        InternalResourceViewResolver resolver = new InternalResourceViewResolver(); 
        resolver.setPrefix("/WEB-INF/"); 
        resolver.setSuffix(".jsp"); 
        return resolver; 
    } 
        
    @Override 
    public void configureDefaultServletHandling(DefaultServletHandlerConfigurer configurer) { 
            configurer.enable(); 
        } 
    }
}
 ```
 * 설정 방법2 : yml
```yaml
spring:
  mvc:
    view:
      prefix: /WEB-INF/views/stadmin/
      suffix: .jsp
```
### 4. freemarker 설정방법 (freemarker 의존성이 이미 있다면)
```properties
spring.freemarker.template-loader-path=classpath:/templates
spring.freemarker.prefix=/freemarker/
spring.freemarker.suffix=.ftl
```
#### 참고
 * 스프링 문서에서는 Thymeleaf, freeMarker와 같은 템플릿 엔진 사용을 권장하고, jsp 사용은 제한함.
    * https://docs.spring.io/spring-boot/docs/current/reference/html/spring-boot-features.html#boot-features-spring-mvc-template-engines
### ContentNegotiatingViewResolver
* view 매핑을 위해 요청 url 확장자와 AcceptHeader를 이용해 뷰를 매핑


### 5. 정적 리소스
 * ResourceHttpRequestHandler가 클래스패스에 있는 정적리소스 위치를 매핑시킴
 * 기본 정적 리소스 매핑 규칙 : `/**` : 루트로 요청을 하면 클래스패스로 매핑
    * 변경하고 싶다면, `spring.mvc.static-path-pattern=/temp/**` : 요청 시작을 `/temp/` 로 시작해야 리소스로 매핑이됨.
##### 기본 리소스 위치
 * `classpath:/static`
    * 예시) /hellow.html 리소스를 요청하면, static (혹은 다른 ) 하위에 hellow.html이 매핑됨.
 * `classpath:/public`
 * `classpath:/resources/`
 * `classpath:/META-INF/resources`
#### 리소스 위치를 추가하고 싶다면?
 * @Configuration 어노테이션이 있고 WebMvcConfigurer 인터페이스를 구현한 클래스에 addResourceHandlers를 @Override한다.
 * 파람으로 있는 registry에 특정 요청에 대해 특정 리소스 위치를 매핑 가능.
 * 예시 코드 : /tt1 으로 시작하는 요청이 들어오면, tt2 경로 리소스를 매핑한다.
 ```java
registry.addResourceHandler("/tt1/**")
        .addResourceLocation("/tt2/") // 꼭 경로를 닫아야함.
        .setCachePeriod(20);

 ```

#### 참고
 * `WebMvcConfigurer` 자동 구성된 스프링 MVC 구성에서 뭔가를 재정의할때 사용.
    * @Configuration, @EnableWebMvc 2가지 어노테이션과 WebMvcConfigurer 인터페이스를 구현하는 방식으로 사용됨.
    * 디폴트 메서드가 지원되지 않던 java7(스프링부트 1.5 이하) 이하 버전들에서는 편의를 위해 `WebMvcConfigurerAdapter`가 대신 사용되었는데, 지금은 deprecated
 * `WebMvcRegistrations` 마찬가지로 자동 구성된 구성에서 뭔가를 재정의할때 사용, Handler 관련 재정의


### 6. WebJar
 * 웹 관련 기능들(ex jQuery)들을 의존성으로 가져와서 /webjars/** 로 간단히 추가 가능.
내 메모 2020.11.12 00:27:07중요

### 7. ExceptionHandler
 * api 예외별로 처리를 다르게 할 경우 사용됨.
#### @ExceptionHandler
 * 특정 컨트롤러로 들어오는 요청 중 해당 Exception이 발생하는 경우 핸들링됨.
```java
@Controller
public class Sample Contriller {

   @ExceptionHandler(SampleException.class)
   public @ResponseBody ModelAndView sampleError(SampleExcepton e) {
      ModelAndView mav = new ModelAndView("/common/alertBack");
		mav.addObject("msg", e.getMessage());

		return mav;
   }
}
```

#### @ControllerAdvice
 * 전역적으로 에러 핸들링 가능하게 설정시 사용함.
```java
@ControllerAdvice
public class ExceptionController {

	@ExceptionHandler(ApiException.class)
	public ModelAndView handleCustomException(ApiException e) {
		ModelAndView mav = new ModelAndView("/common/alertBack");
		mav.addObject("msg", e.getMessage());

		return mav;
	}
}
```


## 5. 스프링 데이터
### 1. 인메모리 데이터베이스
 * 스프링부트는 dataSource 설정이 없으면, 자동으로 인메모리 데이터베이스를 설정한다.
 * 스프링부트는 H2(추천), HSQL, Derby 3가지를 지원한다.
#### 1. H2
 * 자동으로 dataSource와 jdbcTemplate을 빈으로 등록해주기 때문에 `@Autowired`로 가져와 사용 가능
 * 자동으로 설정되는 내용
   * db : `testdb`
   * username : `sa`
   * password : `""`
 * H2 콘솔 사용 방법
   * `spring.h2.console.enabled=true` 추가
      * `/h2-console` 로 접근 가능
### 2. DBCP와 MySQL
#### 1. 지원하는 DBCP
 * HikariCP (기본)
   * `spring.datasource.hikari.*XXX*` 로 프로퍼티 설정 가능
   * 설정
     * `autoCommit` : default true -> 커밋 해주지 않아도 자동으로 커밋된다.
     * `maximumPoolSize` : default: 10
        * 많다고 무조건 좋은 건 아님, 어차피 cpu 코어 개수만큼 밖에 일하지 못함.
 * Tomcat CP
   * `spring.datasource.tomcat.*` 로 프로퍼티 설정 가능
 * Commons DBCP2
   * `spring.datasource.dbcp2.*` 로 프로퍼티 설정 가능
#### 2. MySQL
 * 도커 이용한 설치
   * 도커 설치
      * `docker ps` 로 떠있는 인스턴스 확인
   * `docker run -p 3306:3306 --name mysql_boot -e MYSQL_ROOT_PASSWORD=1 -e MYSQL_DATABASE=springboot -e MYSQL_USER=keesun -e MYSQL_PASSWORD=pass -d mysql`
   * `docker exec -i -t mysql_boot bash` : 인스턴스를 bash로 접속
       * `docker stop mysql_boot` : 인스턴스 멈춤
       * `docker rm mysql_boot` : 인스턴스 삭제
   * `mysql -u root -p` mysql 접속
       * `show databases;`
       * `use [db_name];`
       * `show tables;`

 * datasource 설정
   * spring.datasource.url=jdbc:mysql://localhost:3306/springboot?useSSL=false
   * spring.datasource.username=keesun
   * spring.datasource.password=pass
#### 3. spring-data-jpa
 * `spring-boot-starter-data-jpa` 의존성 필요
    * Repository 빈 자동 생성
    * 쿼리 메소드 자동 구현
    * @EnableJpaRepositories (스프링부트가 자동 설정해줌)
 * `@DataJpaTest` 로 슬라이싱 테스트 가능
    * 인메모리 데이터베이스 필요
 * 데이터베이스 초기화
    * 데이터베이스 생성 `spring.jpa.hibernate.ddl-auto=update` 이외 create, none, validate
    * `spring.jpa.generate-dll=true` 이 값을 true로 해줘야 ddl-auto 설정 중 entity를 찾아 DDL 쿼리를 생성한다.
    * Spring 기본 설정 중, classpath에 `schema.sql` 파일이 있다면 이 스크립트를 서버 시작시 실행한다. DDL 스크립트.
       * DML 스크립트는 `data.sql`

### 3. redis, mongoDB
 * 둘 모두 springboot에서 의존성만 추가하면 자동 설정되므로 로 주입받아 사용 가능.
 * docker를 통해 실행시키고
 * template 객체를 autowired로 주입 받아 사용하거나 CrudRepository를 implements한 repository 인터페이스를 생성해서 쉽게 사용할 수 있다.

### 4. spring security
 * `spring-boot-starter-security` 의존성으로 설정
   * 자동설정시 자동으로 id,pw를 생성해줌. 
      * id : `user`, pw : 실행시 콘솔에 나옴.
      * 커스터마이징하려면 `UserDetailsService`를 implements한 @Service 클래스에서 가능.
      
 * 추가 설정없이 두면 모든 request를 가로채 security 체크 한다.
 * `WebSecurityConfigurerAdapter` 를 상속하는 @Configuration 클래스를 만들어서 커스텀한 설정을 할 수 있다.
    * `configure` 메서드를 오버라이드해서 커스터마이징 가능
    * `passwordEncoder` @Bean 생성 및 커스터마이징해서 `UserDetailsService`에서 사용 가능.
```
@Configuration
public class WebSecurityConfig extends WebSecurityConfigurerAdapter {
   @Override
   protected void configure(HttpSecurity http) throws Exception {
       http.authorizeRequests()
               .antMatchers("/", "/hello").permitAll()
               .anyRequest().authenticated()
               .and()
           .formLogin()
               .and()
           .httpBasic();
   }
}
```

### 5. RestTemplate & WebClient
#### 1. RestTemplate
 * 기본으로 java.net.HttpURLConnection 사용.
    * apache Connection을 사용하고 싶다면, 커스터마이저에서 RequestFactory 세팅을 변경해야 한다.
 * 커스터마이징
    * 로컬 커스터마이징
    * 글로벌 커스터마이징
       * RestTemplateCustomizer
       * 빈 재정의
#### 2. WebClient
 * 기본으로 Reactor Netty의 HTTP 클라이언트 사용.
 * 커스터마이징
    * 로컬 커스터마이징
    * 글로벌 커스터마이징
       * WebClientCustomizer
       * 빈 재정의

## 6. 스프링 운영
### 1. Spring actuator
 * [공식문서](https://docs.spring.io/spring-boot/docs/current/reference/htmlsingle/#production-ready-endpoints)
 * 의존성 `spring-boot-starter-actuator`
 * 다양한 Endpoints 제공.
    * JMX 또는 HTTP를 통해 접근 가능 함.
    * shutdown을 제외한 모든 Endpoint는 기본적으로 활성화 상태.
    * 활성화 옵션 조정
       * `management.endpoints.enabled-by-default=false`
       * `management.endpoint.info.enabled=true`
#### 1. JMX(Java Management Extensions)
 * 어플리케이션을 관리하고 모니터링하기 위한 표준
 * 터미널에서 `jconsole` 엔터로 jconsole 사용가능.
    * 힙 사용량, 스레드 개수, 프로세스 개수 등을 모니터링 가능.
#### 2. http 사용
 * `/actuator`
 * health와 info를 제외한 대부분의 Endpoint가 기본적으로 비공개 상태
 * 공개 옵션 조정
    * `management.endpoints.web.exposure.include=*`
    * `management.endpoints.web.exposure.exclude=env,beans`
### 2. spring boot admin
 * spring boot 제공이 아닌, 제 3자가 제공하는 오픈소스로 스프링부트 actuator UI 제공
#### 어드민 서버 설정
```
<dependency>
    <groupId>de.codecentric</groupId>
    <artifactId>spring-boot-admin-starter-server</artifactId>
    <version>2.0.1</version>
</dependency>
```
 * main 클래스에 이 어노테이션 추가 `@EnableAdminServer`

#### 클라이언트 설정
```
<dependency>
    <groupId>de.codecentric</groupId>
    <artifactId>spring-boot-admin-starter-client</artifactId>
    <version>2.0.1</version>
</dependency>
```
 * `spring.boot.admin.client.url=http://localhost:8080`
 * `management.endpoints.web.exposure.include=*`