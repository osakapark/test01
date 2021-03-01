# eclipse 에서 환경 설정

## 0. IDE (Integrated development environment)  다운로드
  JDK : version 11
  [STS](https://spring.io/tools)


## 1. start.spring.io  에서 import 받기
  ![image description](/image/start_spring_io.png)

  1. 선택사항 : Java, Gradle Project, War
  2. Depedencies : Spring Boot DevTools, Lombok, spring Web, Thymeoleaf, Spring DAta Jpa
  3. 다운로드 : Generate 버튼 클릭

## 2. eclipse 에서 import   
 1. workspace 설정 : sts (spring tool suite)   folder 와 다른 곳에
 2. (1)에서 download 한 file 을 압축 풀어서 workspace 아래로 복사
 3. import 
   File > Import > Gradle > Existing Gradle Project 
   Project root directory 설정 : (예 : D:\00.study\my05\workspace\board)   
 4. 프로젝트 생성 확인
   ![image description](/image/sts_after_import.png)

## 3. UTF-8 설정
  1. Windows > Preference > General > Workapace
   ![image description](/image/sts_utf8.png)
  2. Windows > Preference > Web > (CSS, HTML)

## 4. gradle
### 4.1 build.gradle 
```gradle
plugins {
	id 'org.springframework.boot' version '2.4.3'
	id 'io.spring.dependency-management' version '1.0.11.RELEASE'
	id 'java'
	id 'war'
	id 'com.ewerk.gradle.plugins.querydsl' version '1.0.10'  //added
}
```

```gradle
 dependencies {
	implementation 'org.springframework.boot:spring-boot-starter-data-jpa'
	implementation 'org.springframework.boot:spring-boot-starter-thymeleaf'
	implementation 'org.springframework.boot:spring-boot-starter-web'
	compileOnly 'org.projectlombok:lombok'
	developmentOnly 'org.springframework.boot:spring-boot-devtools'
	annotationProcessor 'org.projectlombok:lombok'
	providedRuntime 'org.springframework.boot:spring-boot-starter-tomcat'
	testImplementation 'org.springframework.boot:spring-boot-starter-test'
	compile group: 'org.mariadb.jdbc', name: 'mariadb-java-client'	//added
	implementation 'com.querydsl:querydsl-jpa'                      //added
}
```

```gradle
test {
	useJUnitPlatform()
}

//이하 added
def querydslDir = "$buildDir/generated/querydsl"
querydsl {
    jpa = true
    querydslSourcesDir = querydslDir
}
sourceSets {
    main.java.srcDir querydslDir
}
configurations {
    querydsl.extendsFrom compileClasspath
}
compileQuerydsl {
    options.annotationProcessorPath = configurations.querydsl
}
```
추가된 부분
  - db 설정
  - querydsl 설정

### 4.2 Gradle Refresh
   build.gradle > (우측 마우스) > Gradle > Refresh Gradle Project

### 4.3 compileQuerydsl 실행
Gradle Tasks > View Menu (우측 아래로 점 3개) > Show All Tasks > other (folder) > compileQuerydsl > Run Gradle Ttasks
- Q도메인 Class 생성 (build/generated/querydsl/)

https://stackoverflow.com/questions/28444128/how-can-i-add-custom-task-to-the-eclipse-gradle-task-tab




 ## 5. application.properties
 ``` properties
 spring.datasource.driver-class-name=org.mariadb.jdbc.Driver
spring.datasource.url=jdbc:mariadb://localhost:3306/bootex
spring.datasource.username=bootuser
spring.datasource.password=bootuser

spring.jpa.hibernate.ddl-auto=update
spring.jpa.properties.hibernate.format_sql=true
spring.jpa.show-sql=true

spring.thymeleaf.cache=false
 ```
 -  spring.thymeleaf.cache=false 
   (교재 83page)


 ## 6. @EnableJpaAuditing 추가
 ```java
@SpringBootApplication
@EnableJpaAuditing  //추가
public class BoardApplication {
	public static void main(String[] args) {
		SpringApplication.run(BoardApplication.class, args);
	}
}
 ```

 ## 7. lombok 설치
 ```java
  Member member = Member.builder() //builder() 에 오류발생
 ```
 1. Project and External Dependencies > lombok-1.18.18.jar > Run As Java Application > 경고창 무시 (Proceed)
2. Project Lombok v1.18.18-Installer > Specify locatoin
3. STS 실행 파일 경로 설정 (예:D:\00.study\my05\sts-4.9.0.RELEASE)
4. Install/Update 
5. Quit Installer
6. STS 재실행
7. Clean


## 8.  Test 시 오류 발생
### 8.1 Use @Param for query method parameters, or when on Java 8+ use the javac flag -parameters
Preferences > Java > Compiler > Store information about method parameters (usable via reflection) [체크]
