# 어노테이션

## 메타 어노테이션
### @Target
 * 어노테이션이 사용되는 위치

### @Retention
 * 어노테이션을 어느 범위까지 유효성을 유지할 것인가
#### 종류
 * SOURCE : 컴파일 전까지만 유효
 * CLASS : 컴파일러가 클래스 참조할때까지 유효
 * RUNTIME : 컴파일 이후에도 jvm에 의해 유효

<img width="789" alt="image" src="https://github.com/user-attachments/assets/c0603cc3-d339-49f0-a8d8-0f97e933af88">


## 커스텀 어노테이션 조건
 * @interface 는 자동으로 Annotation 클래스를 상속
 * 내부 메서드는 자동으로 abstract 키워드가 붙는다.


## 스프링에서 어노테이션 인식 과정
 * 리플렉션을 통해 특정 어노테이션을 인식한다.
#### 예시
```
Set<Class<?>> preInitiatedControllers = r efelections.getTypesAnnotationWIth(Controller.class);

```
