## ⭐ 스프링부트로 간단한 마이크로서비스 생성


<br/>

#### ✔ 의존성 추가

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <groupId>com.thoughtmechanix</groupId>
    <artifactId>licensing-service</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <packaging>jar</packaging>

    <name>Eagle Eye Licensing Service</name>
    <description>Licensing Service</description>

    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.0.3.RELEASE</version>
        <relativePath/>
        <!-- lookup parent from repository -->
    </parent>
    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-actuator</artifactId>
        </dependency>
    </dependencies>

    <properties>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <java.version>1.8</java.version>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <start-class>com.thoughtmechanix.licenses.Application</start-class>
        <docker.image.name>johncarnell/tmx-licensing-service</docker.image.name>
        <docker.image.tag>chapter2</docker.image.tag>
    </properties>

    <build>
        <plugins>
            <!-- We use the Resources plugin to filer Dockerfile and run.sh, it inserts actual JAR filename -->
            <!-- The final Dockerfile will be created in target/dockerfile/Dockerfile -->
            <plugin>
                <artifactId>maven-resources-plugin</artifactId>
                <executions>
                    <execution>
                        <id>copy-resources</id>
                        <!-- here the phase you need -->
                        <phase>validate</phase>
                        <goals>
                            <goal>copy-resources</goal>
                        </goals>
                        <configuration>
                            <outputDirectory>${basedir}/target/dockerfile</outputDirectory>
                            <resources>
                                <resource>
                                    <directory>src/main/docker</directory>
                                    <filtering>true</filtering>
                                </resource>
                            </resources>
                        </configuration>
                    </execution>
                </executions>
            </plugin>
            <plugin>
                <groupId>com.spotify</groupId>
                <artifactId>docker-maven-plugin</artifactId>
                <version>1.1.1</version>
                <configuration>
                    <imageName>${docker.image.name}:${docker.image.tag}</imageName>
                    <dockerDirectory>${basedir}/target/dockerfile</dockerDirectory>
                    <resources>
                        <resource>
                            <targetPath>/</targetPath>
                            <directory>${project.build.directory}</directory>
                            <include>${project.build.finalName}.jar</include>
                        </resource>
                    </resources>
                </configuration>
            </plugin>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>
</project>
```

| 키워드 | 내용                                                                                                     |
|:----|:-------------------------------------------------------------------------------------------------------|
|spring-boot-starter-parent| 라이브러리 간 의존성 관리 및 버전 충돌에 대한 이슈를 예방해줌                                                                    |
|spring-boot-starter-actuator| 스프링 부트 애플리케이션에서 제공하는 여러가지 정보를 모니터링하기 할 수 있게 도와줌                                                        |
|spring-boot-starter-web| Spring MVC를 사용하여 RESTful을 포함한 웹 애플리케이션을 구축하기 위한 시작 장치로 Tomcat을 기본 내장 컨테이너로 사용할 수 있게 해줌                 |
|spring-boot-devtools| Spring Boot에서 개발 편의를 위해 제공하는 라이브러리로 클래스 로딩 문제 진단, 속성 기본값, 자동 재시작, 라이브 리로드, 전역 설정, 원격 애플리케이션 등의 기능을 제공함 |
|<artifactId>maven-resources-plugin</artifactId>| 메이븐에 스프링 부트 애플리케이션의 빌드와 배포를 위한 스프링 전용 메이븐 플러그인을 포함하도록 지시                                               |