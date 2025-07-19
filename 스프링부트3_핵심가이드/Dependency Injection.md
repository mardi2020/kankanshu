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
    - JNDI (Java Naming and Directory Interface): 객체를 컨테이너에서 찾아 사용하는 방식
