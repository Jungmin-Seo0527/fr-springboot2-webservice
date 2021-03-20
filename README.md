# 스프링 부트와 AWS로 혼자 구현하는 웹 서비스

## 목차

## 1장. 인텔리제이로 스프링 부트 시작하기

### 1.4 그레이들 프로젝트를 스프링 부트 프로젝트로 변경하기

* 아래의 코드를 build.gradle에 작성 (기존에 있던 코드들은 지워버림)

```build.gradle
buildscript {
    ext {
        springBootVersion = '2.1.7.RELEASE'
    }
    repositories {
        mavenCentral()
        jcenter()
    }
    dependencies {
        classpath("org.springframework.boot:spring-boot-gradle-plugin:${springBootVersion}")
    }
}

apply plugin: 'java'
apply plugin: 'eclipse'
apply plugin: 'org.springframework.boot'
apply plugin: 'io.spring.dependency-management'

group 'com.jungmin'
version '1.0-SNAPSHOT'
sourceCompatibility = 1.8


repositories {
    mavenCentral()
}

dependencies {
    compile('org.springframework.boot:spring-boot-starter-web')
    compile('org.projectlombok:lombok')
    testCompile('org.springframework.boot:spring-boot-starter-test')
}

```

* [스프링 이니셜라이져](http://start.spring.io/) 를 이용하지 않고 직접 build.gradle에 코드를 추가해서 진행
* `buildscript` : 프로젝트의 플러그인 의존성 관리를 위한 설정
    * `ext` : 전역변수 설정
    * `springBootVersion` 이름의 전역 변수 생성
    * `2.1.7.RELEASE`값을 `springBootVersion`에 저장
      <br><br>
* `apply plugin:` : 플러그인 의존성들을 적용할 것인지 결정
    * `io.spring.dependency-management` : 스프링 부트의 의존성들을 관리해 주는 플러그인
      <br><br>
* `repositories` : 각종 의존성(라이브러리)들을 어떤 원격 저장소에서 받을지를 결정
    * 기본적으로는 mavenCentral
    * 라이브러리 업로드 난이도 때문에 jcenter도 많이 사용
    * jcenter에 라이브러리를 업로드하면 mavenCentral에도 자동적으로 업로드
* `dependencies` : 프로젝트 개발에 필요한 의존성들을 선언
    * `org.springframework.boot:spring-boot-starter-web`
    * `org.springframework.boot:spring-boot-starter-test`
    * 특정 버전을 명시하지 말아야 함 -> 아래의 코드에 명시한 버전을 따라가도록 하기 위함
    ```
      dependencies {
        classpath("org.springframework.boot:spring-boot-gradle-plugin:${springBootVersion}")
    }
  ```

### 1.5 인텔리제이에서 깃과 깃허브 사용하기

* 커밋과정은 내가 알고 있으므로 생략
* **.idea 디렉토리는 커밋하지 않는다.** 이 폴더의 파일들은 **프로젝트 실행시 자동으로 생성되는 파일들이다.**
* .idea 폴더를 앞으로의 모든 커밋 대상에서 제외되도록 처리
* .gitignore 파일 : 파일 안에 기입된 내용들은 모두 깃에서 관리하지 않겠다는 것을 의미
* 인텔리제이에서는 .gitignore 파일에 대한 기본적인 지원이 없음
* 대신에 플러그인에서 .gitignore 지원
  <br><br>

##### .ignore 플러그인 기능

* 파일 위치 자동완성
* 이그노어 처리 여부 확인
* 다양한 이그노어 파일 지원(.gitignore, .npmignore, .dockergnore)

#### 설정 방법

1. .ignore 플러그인 설치(프로그램 재부팅 필요)
2. 최상단에 .ignore file(Git) 생성
   ![](./readmeImages/gitignore파일생성.jpg)
3. gitignore에 코드 등록<br><br>
   ![](./readmeImages/gitignore파일등록.jpg)
4. gitignore 파일 commit

## 2장. 스프링 부트에서 테스트 코드를 작성하자

* TDD 혹은 최소한의 테스트 코드는 필수 조건이다.
* 코테에서도 단위 테스트가 나오는 경우도 있음

### 2.1 테스트 코드 소개

* TDD : 테스트가 주도하는 개발 (**테스트 코드를 먼저 작성하는것부터 시작**)
* 단위 테스트 : TDD의 첫 번째 단계인 **기능 단위의 테스트 코드를 작성** 하는것
    * 테스트 코드를 먼저 작성할 필요는 없음
    * 리팩토링도 포함되지 않음
    * 순수하게 테스트 코드만 작성하는 것     
      <br>

* 테스트 코드의 장점
    * 단위 테스트는 개발단계 초기에 문제를 발견하게 도와준다.
    * 단위 테스트는 개발자가 나중에 코드를 리팩토링하거나 라이브러리 업그레이드 등에서 기존 기능이 올바르게 작동하는지 확인할 수 있다.(회귀 테스트 ??)
    * 단위 테스트는 기능에 대한 불확실성을 감소시킬 수 있다.
    * 단위 테스트는 시스템에 대한 실제 문서를 제공한다. 즉 단위 테스트 자체가 문서로 사용될 수 있다.
    * 그 외...(작가님의 경험담...)
        * 빠른 피드백
        * 테스트 할때마다 톰캣을 내렸다가 다시 실행하는 일 불필요
        * 눈으로 직접 검증할 필요 없음 (System.out.println()) - 자동 검증
        * 개발자가 만든 기능을 안전하게 보호
          <br>
* 테스트를 도와주는 프레임워크 **xUnit**
    * JUnit - Java
    * DBUnit - DB
    * CppUnit - C++
    * NUnit - .net

### 2.2 Hello Controller 테스트 코드 작성하기

* 일반적으로 패키지명은 웹 사이트 주소의 역순
    * 예) www.admin.jungmin.com -> com.jungmin.admin

* com.jungmin.book.springboot 패키지 생성
* `Application` 클래스 생성

    ```java
    package com.jungmin.book.springboot;
    
    import org.springframework.boot.SpringApplication;
    import org.springframework.boot.autoconfigure.SpringBootApplication;
    
    @SpringBootApplication
    public class Application {
        public static void main(String[] args) {
            SpringApplication.run(Application.class, args);
        }
    }
    ```

    * `Application`클래스: 앞으로의 **메인 클래스**
    * `@SpringBootApplication`: 스프링 부트의 자동 설정, 스프링 Bean 읽기와 생성을 모두 자동으로 설정
        * **`@SpringBootApplication`이 있는 위치부터 설정을 읽어감**
        * **프로젝트의 최상단에 위치**
    * main 함수의 `SpringApplication.run`: 내장 WAS(Web Application Server)를 실행
        * 내장 WAS: 별도로 외부에 WAS를 두지 않고 애플리케이션을 실행할 때 내부에서 WAS를 실행하는 것
        * **항상 서버에 톰캣을 설치할 필요가 없음**
        * 스프링 부트로 만들어진 `Jar`파일(실행 가능한 Java 패키징 파일)로 실행하면 된다
        * 스프링 부트에서도 **내장 WAS를 사용하는 것을 권장**
        * 외장 WAS를 사용하면 모든 서버는 WAS의 종류와 버전, 설정을 일치시켜야 한다는 단점이 존재
          <br><br>

* 테스트를 위한 Controller 생성
    * `web`패키지 생성 (`com.jungmin.book.springboot.web`): **컨트롤러와 관련된 모든 클래스 위치**
    * `HelloController`클래스 생성

        ```java
        package com.jungmin.book.springboot.web;
        
        import com.jungmin.book.springboot.web.dto.HelloResponseDto;
        import org.springframework.web.bind.annotation.GetMapping;
        import org.springframework.web.bind.annotation.RequestParam;
        import org.springframework.web.bind.annotation.RestController;
        
        @RestController
        public class HelloController {
        
            @GetMapping("/hello")
            public String hello() {
                return "hello";
            }
        
            @GetMapping("/hello/dto")
            public HelloResponseDto helloDto(@RequestParam("name") String name, @RequestParam("amount") int amount) {
                return new HelloResponseDto(name, amount);
            }
        }
        ```
    * `@RestController`
        * 컨트롤러를 JSON을 반환하는 컨트롤러로 만들어 준다. (뭔말이지???)
        * 예전에는 `@ResponseBody`를 각 메소드마다 선언했던 것을 한번에 사용할 수 있게 해준하고 생각하면 된다.
    * `@GetMapping`
        * HTTP Method인 Get의 요청을 받을 수 있는 API를 만들어 준다.
        * /hello로 요청이 오면 문자열 hello를 반환하는 기능을 가지게 되었다.
        * 예전에는 @RequestMapping(method = RequestMethod.GET)으로 사용되었다.


* 테스트 패키지에서 동일한 경로로 패키지들을 생성
* 사실 따로 생성할 필요 없이 클래스에 `Alt+ins`에서 `test...`를 선택하면 자동으로 패키지와 클래스까지 생성해준다.
* `HelloControllerTest`클래스 생성 (**테스트 대상 클래스 이름에 Test를 붙인다.**)
* 다음 단계에 필요한 static method도 import 되어 있어서 불필요한 것들이 있음
* **_아직 그 기능이 이해가 가지 않는 메소드, 애노테이션이 존재함 ㅠㅠ_**
    ```java
    package com.jungmin.book.springboot.web;

    import org.junit.Test;
    import org.junit.runner.RunWith;
    import org.springframework.beans.factory.annotation.Autowired;
    import org.springframework.boot.test.autoconfigure.web.servlet.WebMvcTest;
    import org.springframework.test.context.junit4.SpringRunner;
    import org.springframework.test.web.servlet.MockMvc;
    
    import static org.hamcrest.Matchers.is;
    import static org.springframework.test.web.servlet.request.MockMvcRequestBuilders.get;
    import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.content;
    import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.jsonPath;
    import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.status;
    
    @RunWith(SpringRunner.class)
    @WebMvcTest
    public class HelloControllerTest {
    
        @Autowired
        private MockMvc mvc;
    
        @Test
        public void hello가_리턴된다() throws Exception {
    
            String hello = "hello";
    
            mvc.perform(get("/hello"))
                    .andExpect(status().isOk())
                    .andExpect(content().string(hello));
        }
    }
    ```
    * `@RunWith(SpringRunner.class)`
        * 테스트를 진행할 때 JUnit에 내장된 실행자 외에 다른 실행자를 실행 시킨다.
        * 여기서는 SpringRunner 라는 스프링 실행자를 사용
        * 즉, 스프링 부트 테스트와 JUnit 사이에 연결자 역할을 한다.
    * `@WebMvcTest`
        * 여러 스프링 테스트 어노테이션 중, Web(Spring MVC)에 집중할 수 있는 어노테이션이다.
        * 선언할 경우 `@Controller`, `@ControllerAdvice`등을 사용할 수 있다.
        * 단, `@Service`, `@Component`, `@Repository`등은 사용할 수 없다.
        * 여기서는 컨트롤러만 사용하기 때문에 선언한다.
    * `@Autowired`
        * 스프링이 관리하는 빈(Bean)을 주입 받는다.
    * `private MockMvc mvc`
        * 웹 API를 테스트할 때 사용한다.
        * 스프링 MVC 테스트의 시작점이다.
        * 이 클래스를 통해 HTTP GET, POST 등에 대한 API 테스트를 할 수 있다.
    * `mvc.perform(get("/hello"))`
        * `MockMvc`를 통해 `/hello` 주소로 HTTP GET 요청을 한다.
        * 체이닝이 지원되어 아래와 같이 여러 검증 기능을 이어서 선언할 수 있다.
    * `.andExpect(status().isok())`
        * mvc.perform의 결과를 검증한다.
        * HTTP Header 의 Status를 검증한다.
        * 우리가 흔히 알고 있는 200, 404, 500 등의 상태를 검증한다.
        * 여기선 OK 즉, 200인지 아닌지를 검증한다.
    * `.andExpect(content().string(hello))`
        * `mvc.perform`의 결과를 검증한다.
        * 응답 본문의 내용을 검증한다.
        * `Controller`에서 "hello"를 리턴하기 때문에 이 값이 맞는지 검증한다.

### 2.3 롬복 소개 및 설치하기

* 자바 개발자들의 필수 라이브러리
* Getter, Setter, 기본생성자, toString 등을 어노테이션으로 자동 생성


* build.gradle에 다음의 코드 추가

```
dependencies {
    compile('org.springframework.boot:spring-boot-starter-web')
    compile('org.projectlombok:lombok') // 추가한 코드
    testCompile('org.springframework.boot:spring-boot-starter-test')
}
```

* 플러그인 설치
* 롬복 설정
    * Settings > Build > Compiler > Annotation Processors > `Enable annotation processing` 체크
      ![롬복 설정](./readmeImages/롬복_설정창.jpg)

### 2.4 Hello Controller 코드를 롬복으로 전환하기

* web 패키지에 dto 패키지 추가 (**모든 응답 Dto는 이 Dto 패키지에 추가**)
* `HelloResponseDto`클래스 생성
    ```java
    package com.jungmin.book.springboot.web.dto;
    
    import lombok.Getter;
    import lombok.RequiredArgsConstructor;
    
    @Getter
    @RequiredArgsConstructor
    public class HelloResponseDto {
    
        private final String name;
        private final int amount;
    }
    
    ```
    * `@Getter`
        * 선언된 모든 필드의 get 메소드를 생성해 준다.
    * `@RequiredArgsConstructor`
        * 선언된 모든 final 필드가 포함된 생성자를 생성해준다.
        * final 이 없는 필드는 생성자에 포함되지 않는다.

* `HelloResponseDtoTest` 클래스 생성

    ```java
    package com.jungmin.book.springboot.web.dto;
    
    import org.junit.Test;
    
    import static org.assertj.core.api.Assertions.assertThat;
    
    public class HelloResponseDtoTest {
    
        @Test
        public void 롬복_기능_테스트() {
            // given
            String name = "test";
            int amount = 1000;
    
            //when
            HelloResponseDto dto = new HelloResponseDto(name, amount);
    
            //then
            assertThat(dto.getName()).isEqualTo(name);
            assertThat(dto.getAmount()).isEqualTo(amount);
        }
    }
    ```
    * `assertThat`
        * `assertj`라는 테스트 검증 라이브러리의 검증 메소드
        * 검증하고 싶은 대상을 메소드 인자로 받는다.
        * 메소드 체이닝이 지원되어 `isEqualTo`와 같은 메소드를 이어서 사용할 수 있다.
    * `isEqualTo`
        * `assertj`의 동등 비교 메소드
        * `assertThat`에 있는 값과 `isEqualTo`의 값을 비교해서 같을 때만 성공
    * **`assertThat`은 Junit의 기본 `assertThat`이 아닌 `assertj`의 `assertThat`을 사용!!!**
    * `assertj`의 장점
        * `CoreMatchers`와 달리 추가적으로 라이브러리가 필요하지 않는다.
            * Junit의 `assertThat`을 쓰면 `is()`와 같이 `CoreMatchers`라이브러리가 필요하다.
        * 자동완성이 좀 더 확실하게 지원된다.
            * IDE에서는 `CoreMatchers`와 같은 `Matcher`라이브러리의 자동완성 지원이 약하다.
        * [참고 사이트](http://bit.ly/30vm9Lg)


* 여기까지 왔으면 문제가 2가지가 생긴다.
    * dto 테스트 메소드가 통과되지 않는다.
      ![](https://user-images.githubusercontent.com/76673100/108456406-989bec80-72b3-11eb-924c-6cda5e1b3bc6.png)
        * 이 문제는 gradle 버전이 5 이상이면 일어나는 문제이다.
        * terminal에 `gradlew wrapper --gradle-version 4.10.2`를 작성한다. (gradle 버전 다운그래이드)
        * 현재는 gradle 버전이 6까지가 나왔고 [작가님 블로그](https://jojoldu.tistory.com/539) 에 최신 버젼에서의 설정방법이 있다. (이 밖에도 다른 모든 요소에 대한
          최신화가 있음)
        * 하지만 결국 그 방법들도 언젠가는 정상 작동하지 않을것이고, 중요한 것은 **왜 안되는지 로그를 보고 검색하는 습관이다.**
          <br><br>
    * 테스트 결과 창에 한글로 명명한 메소드 이름이 깨져서 나온다.
        * Setting > Build tools > Gradle
          ![](./readmeImages/한글깨짐1.jpg)
        * Build and run using: Intellij IDEA
        * Run tests using: Intellij IDEA
          <br>
          ![](./readmeImages/한글깨짐2.jpg)
        * 김영한 팀장님 강의에서도 이렇게 설정했는데 속도도 조금 더 빠르다고 한다.


* `HelloController`에도 새로 만든 `ResponseDto` 사용
    ```java
    package com.jungmin.book.springboot.web;
    
    import com.jungmin.book.springboot.web.dto.HelloResponseDto;
    import org.springframework.web.bind.annotation.GetMapping;
    import org.springframework.web.bind.annotation.RequestParam;
    import org.springframework.web.bind.annotation.RestController;
    
    @RestController
    public class HelloController {
    
        @GetMapping("/hello")
        public String hello() {
            return "hello";
        }
    
        @GetMapping("/hello/dto")
        public HelloResponseDto helloDto(@RequestParam("name") String name, @RequestParam("amount") int amount) {
            return new HelloResponseDto(name, amount);
        }
    }
    
    ```
    * `@RequestParam`
        * 외부에서 API로 넘긴 파라미터를 가져오는 어노테이션
        * 여기서는 외부에서 `name (@RequestParam("name"))`이란 이름으로 넘긴 파라미터를 메소드 파라미터 `name(String name)`에 저장한다.

* 추가된 API를 테스트하는 코드를 `HelloControllerTest`에 추가
    ```java
    package com.jungmin.book.springboot.web;
    
    import org.junit.Test;
    import org.junit.runner.RunWith;
    import org.springframework.beans.factory.annotation.Autowired;
    import org.springframework.boot.test.autoconfigure.web.servlet.WebMvcTest;
    import org.springframework.test.context.junit4.SpringRunner;
    import org.springframework.test.web.servlet.MockMvc;
    
    import static org.hamcrest.Matchers.is;
    import static org.springframework.test.web.servlet.request.MockMvcRequestBuilders.get;
    import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.content;
    import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.jsonPath;
    import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.status;
    
    @RunWith(SpringRunner.class)
    @WebMvcTest
    public class HelloControllerTest {
    
        @Autowired
        private MockMvc mvc;
    
        @Test
        public void hello가_리턴된다() throws Exception {
    
            String hello = "hello";
    
            mvc.perform(get("/hello"))
                    .andExpect(status().isOk())
                    .andExpect(content().string(hello));
        }
    
        @Test
        public void helloDto가_리턴된다() throws Exception {
            String name = "hello";
            int amount = 1000;
    
            mvc.perform(
                    get("/hello/dto")
                            .param("name", name)
                            .param("amount", String.valueOf(amount)))
                    .andExpect(status().isOk())
                    .andExpect(jsonPath("$.name", is(name)))
                    .andExpect(jsonPath("$.amount", is(amount)));
        }
    }
    ```
    * `param`
        * API테스트할 때 사용될 요청 파라미터를 설정
        * 단, 값은 `String`만 허용
        * 그래서 숫자/날짜 등의 데이터도 등록할 때는 문자열로 변경해야만 가능
    * `jsonPath`
        * JSON 응답값을 필드별로 검증할 수 있는 메소드
        * `$`를 기준으로 필드명을 명시
        * 여기서는 `name`과 `amount`를 검증하니 `$.name, $.amount`로 검증한다.