# SourceSets

## 나는 왜 SourceSets을 알아보게 되었는가?

기존 Gradle 기반으로 진행하던 단일 모듈 프로젝트를 멀티 모듈 프로젝트로 변경해야 할 만한 이유가 생겨서 변경을 진행하였다. 그 과정에서 생긴 작은 문제점과 해결을 이야기해보려 한다.

단일 모듈을 사용했던 이유는 각 MVC 패턴의 레이어를 분리할 이유도 없었고, 여러 개의 모듈러 나누어 관리하면 빌드 구조가 더 복잡해질 것이라고 판단했기 때문이다.

초기에 제공받은 스켈레톤 구조는 멀티 모듈이었다. 하지만 함께하는 팀원들 모두가 단일 모듈에 익숙했고, Gradle 보단 Maven에 익숙했었다. 경험 유무도 어느정도 영향이 있었지만 그보단 단일 애플리케이션을 구동하는데, 멀티 모듈을 가져갈 이유는 더더욱이나 없었다.

하지만, 이번에 Batch 모듈을 추가하면서, Model 이라던지, Mapper 등을 필요로 하게 되었는데, 이미 Model 역할을 수행하는 녀석이 기존 단일 모듈 내 존재하다보니 중복 소스코드가 발생하고, 변경사항에 따라 중복 관리해야하는 문제점이 발생할 것으로 우려되었다.

그래서 기존 스켈레톤 구조와 동일한 멀티 모듈로 다시 분리하는 작업을 시작하였다.

그림은 꽤나 그럴싸했다. 모듈도 분리된게 보였고, gradle도 여러 개의 모듈을 읽고있는 상태였고 애플리케이션 구동만 잘 되면 문제가 없겠다고 생각했다.

하지만 예기치 못한 문제가 발생했다.

애플리케이션을 구동시켰더니, resources 디렉터리 내 존재하는 특정 파일이 not found랜다. 분명 있는데 말이다.

그래서 build된 디렉터리에 대상 리소스가 존재하는지 확인을 했더니 없다.

아.. 이게 뭔가 같이 말려야되는데 그러지 못했구나.. 기존 스켈레톤 구조와 현재 구조를 비교해놓고 보니, resources 디렉터리를 말아 올리는 역할을 수행할 것만 같은 SourceSets라는 구문을 보게되었다. 프로파일별로 분리하여 대상 resources 디렉터리를 각 모듈별로 빌드할 수 있도록 동작하는 것 처럼 보였다.

(아.. 이놈이네 딱봐도)

## 그래서 이 SourceSets의 용도는 무엇이고, 어떻게 쓰는 것일까?

![Untitled](SourceSets/Untitled.png)

Gradle 공식문서를 살펴보면 “SourceSet은 자바 소스와 리소스파일의 논리적인 그룹을 나타낸다.” 라고 설명한다. 결국 Gradle이 이 논리적인 그룹을 의미하는 리소스 파일이나 자바파일을 가지고 무언가 액션을 수행할 수 있도록 도와주는 녀석인 셈이다.

좀더 알아보자.

### Properties

![Untitled](SourceSets/Untitled%201.png)

제공되는 프로퍼티로는 어떤 대상을 sourceSet으로 지정할지, 그리고 SourceSet의 Output을 어느 디렉터리로 잡을지, 리소스파일을 어느 output 디렉터리로 복사할지 등을 프로퍼티로 제공하고있다.

이점에서 우리는 아.. SourceSet은 컴파일될 때 결과물(Java source든 resource든)을 어떻게 관리할지를 다룰 때 쓰일 수 있겠구나 유추할 수 있다.

그러면 어떠한 기능을 제공하는지도 알아봐야겠다.

### Methods

![Untitled](SourceSets/Untitled%202.png)

컴파일 테스크명을 바환해주거나, SourceSet 을 클래스 디렉터리로 컴파일하거나, java source, resouce를 SourceSet으로 구성할 수 있는 기능들을 제공한다.

```groovy
plugins {
    id 'java'
}

sourceSets {
  main {
    java {
      exclude 'some/unwanted/package/**'
    }
  }
}
```

Gradle 공식문서에서 제공되는 예시는 한눈에 쉽게 들어온다. 소스셋으로 감아지는 녀석들 중에 som/unwated/package 내 존재하는 모든 녀석들을 exclude 하겠단다. 이 SourceSet을 사용하면, 실제 컴파일되었을때 필요로 하지 않는 녀석들도 위와 같이 제거할 수 있다.

## 다시 문제점으로 돌아가서, 그러면 왜 애플리케이션을 구동할 때 다른 모듈에 존재하는 resources 디렉터리가 안말려 올라갔겠는가?

답은 생각보다 간단하다.해당 모듈이 정상적으로 빌드 되지 않아서 이다.

SourceSet을 정상적으로 명시했음에도 불구하고, 일반 Intellij에서 애플리케이션 구동시 자동 수행되는(default) build는 gradle build와는 다른 부분이 있다.

Intellij 빌드(상단 메뉴바의 Build - Build Project)는 Java 기반 프로젝트를 컴파일하기 위해 IDE의 자체적으로 [jps 프로젝트 모델](https://github.com/JetBrains/intellij-community/blob/master/jps/model-api/src/org/jetbrains/jps/model/JpsModel.java)과 [빌더](https://www.jetbrains.com/help/idea/compiler-and-builder.html)를 사용한다고 한다. 그러다보니 좀더 진보된 빌드툴인 gradle 또는 maven에서 제공하는 build와는 별개이다보니 애플리케이션 모듈 외 다른 모듈들의 resources 디렉터리가 정상적으로 말리지 않은 문제였던 것이다.

Run Configurations의 before launch에 있는 build를 과감히 지우고, gradle build를 추가하였다.

세상에나 너무 잘된다.

SourceSet을 사실상 작성할 이유도 없었던 것이다.

그래도 이 SourceSet을 활용한다면 이러한 용도로 쓸 수 있을 것 같다.

서로다른 Spring Profile에 배포 환경마다 사용되어져야 할 특정 리소스 파일이 제각각인 경우라면, resources 디렉터리를 프로파일 별로 분리하여 관리하고 이를 sourceSet에 명시하거나, 테스트용으로 사용되는 파일을 실제 빌드할때는 제외시킨다던지의 액션을 수행할 수 있을 것 같다.