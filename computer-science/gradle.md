# gradle 정리

## 1. gradle 기본 내용
### gradle 이란?
 * gradle 은 빌드툴이다.
 * 빌드툴이란? 
 * 소프트웨어는 개발하기 위해 몇가지 정형화된 작업들이 필요하다.
코드를 컴파일하고, 규약에 맞는 확인하고, 테스트하고, javadoc 같은 문서를 만들고, 클래스 파일과 리서스 파일을 패키징하여 압축 파일을 만들고 배포하는 등
이러한 정형화된 작업을 자동화하기 위한 소프트웨어가 빌드툴이며 가장 대표적으로 maven과 gradle 이 있다.
### maven 과의 차이점
 * gradle은 maven 처럼 규칙 기반 빌드, 의존관계 관리 기능을 제공하면서, maven보다 유연하게 기능을 변경하거나 확장할 수 있는 빌드 툴 
### gradle 특징
 * 그루비 문법을 사용한다
 * 스크립트 작성하지 않아도 사용가능한 내장 태스크가 존재한다.
    * `gradle tasks` 명령어로 확인 가능
 * `init` 명령어를 사용하면 프로젝트를 자동 생성할 수 있다.
    * `gradle init --type java-library` : 자바 라이브러리 프로젝트에 필요한 구조와 디렉터리들을 자동으로 생성
    * 자동 생성된 프로젝트의 build.gradle은 아래의 구조와 같다.
```
apply plugin: 'java' // java 플러그인 적용. 자바 프로젝트 빌드에 필요한 태스크들이 포함된다.

repositoryes {
    mavenCentral() // 의존관계 해결을 위한 저장소로 maven 사용. dependencies에 작성된 의존 라이브러리들을 메이븐 중앙 저장소에서 내려받는다.
}

dependencies {
    ...
}

```

### gradle wrapper
 * gradle을 따로 설치하지 않아도 gradlew 를 이용해서 빌드 등의 task들을 실행할 수 있다.
    * `gradle init --type java-library` 명령어 실행시 자동으로 gradle wrapper 포함된다.
 * wrapper 실행시, 자동으로 gradle 바이너리들이 설치되며 설치된 바이너리로 빌드 스크립트가 실행된다.


## 2. java plugin
### java plugin 이란?
 * 자바 빌드에 필요한 태스크, 규칙, sourceSet, 속성 등이 포함된다.

### sourceSet? 
컴파일이 필요한 자바 코드와 리소스를 논리적으로 그룹화한 개념. main/test 2가지 sourceSet이 있으며 sourceSet 단위로 전용 태스크와 속성을 제공한다.

### java 빌드에 필요한 과정과 태스크
#### 1) 과정
 1. 컴파일
 2. 테스트
 3. javadoc 생성
 4. jar 파일로 압축

### 2) 태스크
 * compileJava : sourceSet의 main 하위 중 확장자가 .java인 파일을 컴파일 한다.
 * procjessResources : sourceSet의 main 하위 중 확장자가 .java 아닌 파일들을 클래스패스에 복사한다.
 * jar : jar 파일을 생성한다. compileJava, procjessResources 태스크들의 결과물을 모아 압축한다.
 * assemble : 테스트를 실행하지 않고 결과물(jar)만 생성한다. 자바 플러그인에서는 jar 태스크와 기본적으로 동일
 * check : 프로젝트를 검사한다. 자바 플러그인에서는 test 태스크와 기본적으로 같다.
 * build : check와 assemble 태스크를 실행한다.


### 규칙
`프로덕션 자바 코드 파일은 src/main/java 디렉토리 안래에 둘 것.` 과 같이 빌드에 필요한 코드 파일을 어디에 둘지에 대한 규칙들이 포함되어 있다.

### 속성
전체 속성은 gradle userguide java_plugin을 참조하자.
 * libsDir : 라이브러리(jar) 파일 출력 디렉터리 지정, 디폴트 build/libs
 * sourceCompatibility : 자바 컴파일에 사용하는 자바 버젼, 디폴트 jvm 버젼
 * targetCompatibility : 컴파일할때 클래스 생성 대상에 사용하는 자바 버젼, 디폴트 sourceCompatibility
 * archivesBaseName : Jar 같은 압축파일의 기본 이름.
 * manifest : 모든 JAR파일에 포함되는 매니페스트.

### jar 파일명 지정
```
// JAR 파일명은 {baseName}-{appendix}-{version}-{classifier}.jar 이므로 아래처럼 상세 설정 가능.

jar {
    baseName = 'example'
    appendix = 'bin'
    version = '0.1'
    classifier = 'jdk17'
}  
```

혹은 직접 지정도 가능.

```
jar {
    archiveName = 'example.jar'
}
```

### sourceSet 수정 혹은 추가
```
sourceSets {
	main {
		java {
			srcDirs "src/main/java", 'src/main/generated'
		}

		resources {
			srcDirs = ["src/main/resources-${profile}", "src/main/resources", "src/main/java"]
		}
	}
}
```

### 의존관계 변경
의존관계에 따라 컴파일 단계에서만 필요하고 런타임에는 불필요한 것들도 있다. 의존성 옵션을 통해 불필요한 빌드를 생략한다.
 * `CompileOnly` : 컴파일 시점에만 사용
 * `runtimeOnly` : 컴파일 시점에는 사용되지 않고, 실행 시점에 사용
 * `compile`, `implementation` 차이
    * compile : A 모듈을 수정하면, A를 의존하는 다른 모든 모듈이 rebuild
    * implementation : A 모듈을 수정하면, A 모듈을 직접적으로 의존하는 모듈까지만 rebuild


### task 정의에서 `<<` 의미
 * task 실행 순서를 후 순위로 연기시킨다는 의미이다.
 * gradle 5.0부터 deprecated 되었고, task.dolast(Action) 형태를 대신 사용중이다.