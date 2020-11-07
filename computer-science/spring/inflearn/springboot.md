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
            * `@ConditionalOnBean`
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