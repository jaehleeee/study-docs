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
### 1. 