# 01. 스프링부트란?

## 스프링 프레임워크
- 핵심 가치: 애플리케이션 개발에 필요한 기반을 제공해 개발자가 비즈니스 로직 구현에만 집중할 수 있게 끔 하는 것
- 제어의 역전(IoC, Inversion of Control)
  ``` java
  // 기존의 코드: 사용하려는 객체를 선언하고 해당 객체의 의존성을 주입하고 객체에서 제공하는 기능 사용
  private MyService = new MyServiceImpl(); // 개발자가 직접 제어함
  ```
  하지만, 스프링에서는 다르게 동작함
  - 사용할 객체를 직접 생성하지 않고 객체의 생명주기 관리를 외부에 위임
    - 외부: Spring Container(IoC Container)
  객체의 관리를 컨테이너(외부)로 넘겨서 제어권이 넘어가는 것을 **제어의 역전**이라고 함(즉 내가 할일을 컨테이너가 알아서 객체를 주입시켜준다)
  - IoC를 통해 DI(Dependency Injection), AOP(Aspect-Oriented Programming)을 지원
  - 제어의 역전 덕분에, 개발자는 비즈니스 로직에 더 집중할 수 있게 됨
- AOP: 관점을 기준으로 묶어 개발하는 패러다임, 방식
  - Aspect: 어떤 기능을 구현할 때 그 기능을 '핵심 기능'과 '부가 기능'으로 구분해 각각을 하나의 관점으로 봄
    - 핵심 기능: 비즈니스 로직을 구현하는 과정에서 비즈니스 로직이 처리하려는 목적 기능
      - Ex) 상품 정보를 데이터베이스에 저장, 저장된 상품정보 데이터를 보여주는 코드
    - 부가 기능: 핵심 기능이 어떤 기능인지에 무관하게 로직이 수행되기 전과 후에 수행되기만 하면 되는 기능
      - Ex) 비즈니스 로직 사이에 로깅 및 트랜잭션 처리 코드
  - AOP를 구현하는 방식
    - 컴파일 과정에 삽입, 바이트코드를 메모리에 로드하는 과정에 삽입, **프록시 패턴**

## 스프링과 스프링부트
예를 들어서 JPA 구현체인 Hibernate를 사용하기 위한 과정,
- Spring: 설정파일이 복잡하고 길음
- SpringBoot: 별도의 복잡한 설정없이 개발이 편해짐

의존성 관리
- Spring: 개발에 필요한 각 모듈의 의존성을 직접 설정 및 호환 버전을 명시
- SpringBoot
  - `spring-boot-starter`의 의존성 제공으로 서로 호환되는 버전의 모듈 조합을 제공
  - `spring-boot-starter-parent` 에서 여러 라이브러리를 사용하게 될 경우, 충돌이 발생하지 않도록 검증된 조합을 제공
Auto Configuration: 스프링부트에서 지원하는 기능
- 자동설정: 애플리케이션에 추가된 라이브러리를 실행하는데 필요한 환경 설정을 알아서 찾아줌
- `@SpringBootApplication`
  - `@SpringBootConfiguration`: 스프링 설정 클래스이며 @Bean 등록
  - `@EnableAutoConfiguration`: `AutoConfigurationImportSelector` 클래스가 META-INF에 있는 자동설정 imports 정보를 읽어 @Conditional 조건을 충족할 때 빈으로 컨테이너에 등록
  - `@ComponentScan`: @Component 클래스를 Bean으로 등록
    - @Controller, @RestController, @Service, @Repository, @Configuration
스프링부트의 내장 WAS
- `spring-boot-starter-web`: 내장된 톰캣(다른 Jetty등 웹서버 사용 가능)
스프링부트의 모니터링: `SpringBoot Actuator` 로 시스템이 사용하는 스레드, 메모리, 세션등 모니터링 가능
