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
      - **생성자 주입**: 객체가 생성될 때 필요한 의존성을 생성자의 파라미터로 전달받는 방식
        - final 키워드를 사용하여 의존성을 불변하게 유지 가능
        ```java
        @Service
        public class AService {
            private final ARepository aRepository;

            // Lombok @RequiredArgsConstructor를 사용하면 생략 가능
            @Autowired
            public AService(ARepository aRepository) {
                this.aRepository = aRepository;
            }
        }
        ```
        - 장점: 불변성, 테스트 시점에 의존성 주입 누락을 컴파일 타임에서 바로 감지 가능, 순환 참조 문제도 생성자 주입시 빠르게 발견 가능
          - SpringBoot 2.6 이상 버전부터는 Spring Framework 5.3 에서 순환 참조가 기본적으로 허용되지 않도록 변경됨
            - 순환참조는 잘못된 설계, Di 컨테이너가 두개 이상의 빈을 생성할 때 서로가 서로를 필요로 하면 컨테이너는 어떤 순서로 빈을 만들어야 하는지 모름
            - 순환 참조 발생시, `BeanCurrentlyInCreationException` 발생
      - 필드 주입
        ```java
        @Service
        public class AService {
            @Autowired
            private ARepository aRepository;
        }
        ```
        - 단점: 테스트할 때, `ReflectionTestUtils` 등으로 강제 주입해야 함, 생성시점에 의존성이 없다면 NPE 발생 가능
      - Setter(수정자) 주입
        ```java
        @Service
        public class AService {
            private ARepository aRepository;

            @Autowired
            public setRepository(ARepository aRepository) {
                this.aRepository = aRepository;
            }
        }
        ```
        - 단점: set 메서드로 주입후 언제든지 객체를 변경할 수 있기 때문에 불변성이 없음
    - xml이나 자바 설정 클래스로도 주입 가능
      ```java
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

## 정리

| **주입 방식**     | **특징**                                                                 | **장점**                                                                 | **단점**                                                                | **사용 권장 여부**       |
|-------------------|-------------------------------------------------------------------------|-------------------------------------------------------------------------|-------------------------------------------------------------------------|--------------------------|
| **생성자 주입**    | 생성자를 통해 의존성 주입                                                | - `final`로 불변성 보장<br>- 필수 의존성 강제<br>- 순환 참조 빨리 감지<br>- 테스트 용이 | - 생성자 파라미터가 많아질 경우 코드 복잡도 증가                        | **가장 권장**            |
| **Setter 주입**    | Setter 메서드를 통해 의존성 주입                                          | - 선택적 의존성 처리 용이<br>- 런타임에 의존성 변경 가능                 | - 변경 가능성으로 인해 불변성 깨짐<br>- 주입 누락 시 런타임 오류 발생 가능 | 보조적으로 사용          |
| **필드 주입**      | `@Autowired`를 필드에 직접 붙여 의존성 주입                              | - 코드 간결<br>- 간단한 테스트용 샘플 작성 시 편리                       | - 테스트 시 Mock 주입 어려움<br>- 불변성 보장 불가<br>- 순환 참조 늦게 발견 | **비권장 (테스트 불편)** |
| **인터페이스 주입** | 의존성을 주입하기 위한 인터페이스를 정의하고, 구현 클래스에서 호출        | - 유연성(인터페이스 기반 주입)<br>- 설계 시 명확한 의존성 표현            | - 사용 빈도가 낮고 불필요한 복잡성 추가                                  | 거의 사용 안 함          |
| **메서드 주입**    | 일반 메서드 파라미터에 `@Autowired`를 사용하여 주입                      | - 런타임에 의존성을 특정 메서드에서만 사용 가능                          | - 코드 가독성이 떨어지고 사용 사례 적음                                  | 특별한 경우만 사용        |

---
