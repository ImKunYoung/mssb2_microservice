# ⭐ 스프링부트로 간단한 마이크로서비스 생성


<br/>

## ✔ 의존성 추가


##### ✏️ license-service/pom.xml
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

<br/>
<br/>

## ✔ 스프링 부트 애플리케이션 부팅

스프링 부트로 간단한 마이크로서비스를 시작한 후 반복하며 기능을 전달하기 위해선 라이선싱 서비스의 마이크로서비스에 다음 2개의 클래스를 생성해야 한다.

- 스프링 부트가 애프리케이션을 시작하고 초기화하는 데 사용되는 Spring Bootstrap 클래스
- 마이크로서비스에서 호출할 수 있는 HTTP 엔드포인트를 노출하는 Spring Controller 클래스

#### ✏ Application.java
```java
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class Application {

    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }
}
```
| 키워드                                            | 설명                                                                  |
|:-----------------------------------------------|:--------------------------------------------------------------------|
| @SpringBootApplication                         | @SpringBootApplication는 스프링 부트 프레임워크에 이 클래스가 프로젝트의 부트스트랩 클래스라고 지시한다 |
| SpringApplication.run(Application.class, args) | 스프링 부트 서비스를 시작하기 위해 호출한다                                            |

@SpringBootApplication는 자바 클래스 경로에 다른 스프링 빈이 있는지 찾기 위해 모든 클래스를 스캔한다.

> ##### 📑 스프링 빈은 다음 애너테이션을 사용해 정의할 수 있다.
> 1. @Component, @Service, @Repository 애너테이션이 붙은 클래스를 정의한다
> 2. @Configuration 애너테이션이 붙은 클래스에 스프링 빈을 생성하기 위한 @Bean 생성자 메서드를 정의한다.
































