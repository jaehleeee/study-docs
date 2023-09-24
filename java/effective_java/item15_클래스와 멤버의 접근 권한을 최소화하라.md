# 15. 클래스와 멤버의 접근 권한을 최소화하라
## 핵심 정리
#### 구현과 API를 분리하는 정보 은닉의 장점
 * 시스템 개발 속도 증가 - 인터페이스만 열어주면 서버/클라이언트 같이 개발 가능
 * 시스템 관리 비용 낮춘다 (캡슐화의 장점)
 * 성능 최적화에 도움 (캡슐화의 장점) 원인파악이 쉬워지기 때문
 * 소프트웨어 재사용성 증가 (캡슐화의 장점)
 * 개발 난이도 낮춘다 (캡슐화의 장점) - 모듈 단위 개발은 개발 난이도를 낮춘다

#### 클래스와 인터페이스의 접근 제한자 사용 원칙
 * 모든 접근성은 가능한 좁히는게 좋다.
 * 패키지 외부에서 쓰지 않을 인터페이스라면 package-private(default)
 * 한 클래스에서만 사용하는 인터페이스는 해당 클래스 내부에 private static으로 중첩 시키자 (item24에서 더 자세히)
   * *왜 static 이어야하는가?*
     * static이 아닌 내부 클래스는 감싸고 있는 외부 클래스에 의존하게 된다.
     * static은 감싸고 있는 외부 클래스와 의존이 아예없는 남남이다.
     * 저자가 중첩 클래스를 권장한 의도의 클래스는 서로 의존이 없던 클래스였으므로 static을 추천한 것이다.
     * (반대로 외부 클래스의 필드 의존성이 필요하다면 static 없이 중첩 내부 클래스를 만들어도 된다.)

#### 멤버의 접근 제한자 사용 원칙
 * private, default는 내부 구현용
 * public 클래스의 protected와 public은 공개 API
 * public static final은 상수로 사용한다
    * 배열은 이렇게 만들지 마라. 변경이 가능하므로


#### private 혹은 default 접근성의 클래스를 참조하는 클래스의 테스트할때는 mock을 사용하자
 * 자동으로 mock 객체를 만들어서 넣어준다
```
@ExtendWith(MockitoExtension.class)
class ItemServiceTest {

  @Mock
  MemberService memberService;

  @Test
  void itemService() {
    ItemService itemService = new ItemService(memberService);
    assertNotNull(itemService.getMemberService());
  }
}
```

## 완벽 공략
#### Java Platform Module System (JPMS)
 * java 9부터 도입된 기능으로, 모듈 경로라는 개념을 도입하여 모듈들을 로드하고 실행하는데 사용된다
 * 안정성 - 순환참조 허용하지 않는다.
 * 캡슐화 - public 인터페이스라도 공개된 패키지만 사용 할 수 있다.
 * 확장성 - 필요한 자바 플랫폼 모듈만 모아서 최적의 JRE 구성 가능

