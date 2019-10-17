# Maven Modularity



프로젝트를 수행하면서 공통적으로 필요한 코드를 어떻게 처리해야 할까?

가장 쉬운 방식이 package 를 분리해 가져다 쓰는 것이다. 그러면 이렇게 package 로 분리개발된 코드를 다른 프로젝트에서 사용해야 한다면? 아무생각 없이 복사/붙여넣기 하는 경우, 동일한 코드를 유지보수 하는데 시간이 늘고 수정사항이 누락되는 경우가 발생한다. 그래서 프로젝트를 모듈화 하는 방식으로 이를 해결한다.‌

모듈화 하는 방법은 언어레벨에서 지원하는 방식\(Java 9~ Jigsaw\) 또는 빌드 툴\(Maven, Gradle, etc.\) 에서 지원하는 모듈을 이용한다.

## Maven Modularity

### 디렉토리 구조

아래 예시를 보면 최상위 pom.xml 파일과 각 프로젝트 내부에 pom.xml 을 각각 가지고 있다.

```text
$ tree -L 1 -d
.
├── creavorite
├── creavorite/pom.xml (sub module pom)
├── creavorite-common
├── creavorite-common/pom.xml
pom.xml (parent pom)
```

### pom 설정

parent pom.xml

```text
...
<modules>
  <module>creavorite</module>
  <module>creavorite-crowdsourcing</module>
  <module>creavorite-common</module>
</modules>
...
<dependencyManagement>
  <dependencies>
    <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-context</artifactId>
      <version>${spring.version}</version>
    </dependency>
  </dependencies>
</dependencyManagement>
```

submodule creavorite-service/pom.xml

```text
<parent>
  <groupId>com.ovncorp</groupId>
  <artifactId>creavorite-service</artifactId>
  <version>1.0</version>
  <relativePath>../</relativePath>
</parent>
...
<dependencies>
  ...
  <dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-context</artifactId>
    <!-- 버전 생략 -->
  </dependency>
  <dependency>
    <groupId>com.ovncorp</groupId>
    <artifactId>creavorite-common</artifactId>
    <version>1.0</version>
  </dependency>
  ...
</dependencies>
```

