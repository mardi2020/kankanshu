# DI(Dependency Injection, 의존성 주입)
- 스프링 이전의 의존성 주입
  - 대부분 직접 객체를 생성하고 의존 관계를 코드에서 개발자가 직접 처리
    ``` java
      public class Service {
          private Repository repository;

          // 내부에서 직접 의존 객체 관리 - 강한 결합도
          public Service() {
              this.repository = new Repository();
          }
      
          // 외부에서 생성자를 호출하여 의존 객체 초기화
          public Service(Repository repository) {
              this.repository = repository;
          }
      }
    ```
  - 내부에서 직접 의존 객체를 관리하게 될 경우, 결합도가 높아져 유지보수 및 테스트가 어려워짐
  - 효율적인 DI를 위한 시도들
    - Container/Factory pattern: 객체 생성과 관리를 분리
      - Factory pattern의 문제점
        - 팩토리 자체가 모든 객체에 의존: 팩토리 클래스에서 객체 생성 로직을 담당하므로 점점 비대해질 수 있고 객체간의 의존성을 팩토리 클래스에 전부 정의해야함
        - 테스트 어려움: Mock 객체를 주입하기가 어려움
        - 환경 설정의 유연성 부족: 팩토리 클래스의 소스 코드를 직접 수정해야 함
    - JNDI (Java Naming and Directory Interface): 객체를 컨테이너에서 찾아 사용하는 방식
      - WAS에서 객체를 lookup하여 사용자가 가져오는 방식으로 강한 결합 발생
      - 잘못된 이름으로 lookup하면 runtime error 발생
    - EJB (Enterprise Java Beans): 대규모 엔터프라이즈 애플리케이션에서 컴포넌트 간의 의존관계를 관리하는 방식, 복잡하고 무거움
      - 개발 생산성 저하: 배포, 실행, 설정 과정이 매우 복잡하고 많은 xml의 설정 파일이 필요함
      - 강한 결합: 비즈니스 로직과 인프라 코드가 뒤섞임
- 스프링
  - 경량 컨테이너와 어노테이션기반 의존성 주입 등 편리함 제공
  - 객체 생성 및 생명주기 관리 위임
    - 개발자가 직접 객체를 `new` 로 객체를 생성하지 않아도 됨 ->   `Spring Container`에서 Bean을 대신 생성하고 관리함
    - xml이나 java 설정 클래스(@Configuration), 어노테이션(@Component, @Service 등)으로 객체 정의
  - 의존성 주입
    - `@Autowired`, `@Qualifier`, `@Inject` 등을 통해 자동으로 필요한 Bean을 주입받음
    - 생성자 주입, 수정자 주입, 필드 주입도 가능
    - xml이나 자바 설정 클래스로도 주입 가능
      ```
      @Configuration
      pulic class AppConfig {
          @Bean
          public Clazz clazz() {
              return new Clazz();
          }
          // ...
      }
      ```
  - 강한 결합에서 느슨한 결합
    - 객체가 서로 직접 생성하지 않고 인터페이스 기반으로 DI를 받으므로 코드 변경없이 구현체 교체가 가능
