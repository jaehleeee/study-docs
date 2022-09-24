# 컴포넌트 스캔과 의존관계 자동 주입
 * 스프링은 설정 정보가 없어도 자동으로 스프링 빈을 등록하는 컴포넌트 스캔이라는 기능을 제공한다.
 * 또, 의존관계도 자동으로 주입하는 `@Autowired` 기능도 제공한다.

### `@ComponentScan`
 * @Configuration 과 함께 설정 클래스에 붙여서 사용
    * 스프링부트에서는 Application 클래스 파일에 자동으로 붙는 `@SpringBootApplication`에 포함되어 있어서 수동으로 추가핦 필요없다.
 * `@Component` 어노테이션이 붙은 클래스를 모두 스프링 빈으로 등록한다.
 * 탐색위치는 `basePackages = "{package 위치}"` 파라미터를 통해 탐색 시작 위치를 설정할 수 있다. (설정하지 않으면, 디폴트는 ComponentScan 어노테이션이 붙은 클래스의 패키지부터 시작한다.)
 * 특정 component를 스캔에 포함시키려면, includeFilter = @ComponentScan.Filter(type = FilterType.ANNOTATION, classes = MyIncludeComponent.class)
 * 특정 component를 스캔에서 제외하려면, excludeFilter = @ComponentScan.Filter(type = FilterType.ANNOTATION, classes = MyExcludeComponent.class)
 * 탐색 대상
    * @Component
    * @Controller(스프링 MVC 컨트롤러로 인식하는 기능도 있음)
    * @Service
    * @Repository(스프링 데이터 계층으로 인식, 데이터 계층의 예외를 스프링 예외로 변환해준다.)
    * @Configuration (스프링 설정 정보로 인식)

###  `@Autowired`
 * 의존관계를 자동으로 주입해준다.
 * 생성자에서 여러 의존관계도 하번에 주입받을 수 있다.
