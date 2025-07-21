# AOP
아래 코드는 비즈니스 로직과 로깅이 섞인 예시
```java
  @Service
  @slf4j
  public class OrderService() {
      public void placeOrder() {
          log.info("[INFO] Order placed."); // 로깅
          // 비즈니스 로직 시작
      }
  }
```
- 이 로깅 코드를 모든 메서드에 반복되게 작성하면 중복 코드가 늘어나고 그만큼 유지보수 추적이 어려워짐
- 따라서, 핵심 관심사인 "비즈니스 로직"과 공통 관심사가 섞여 코드 가독성도 떨어짐
- AOP는 이러한 공통 관심사를 별도 모듈(Aspect)로 분리해 비즈니스 로직과 혼재하지 않도록 함

[기존에 AOP를 적용하여 직접 구현했던 @Transactional 예제](https://github.com/mardi2020/daily-code/tree/main/transactional/transactional)

## Spring AOP의 기본 원리
> 프록시 기반으로 동작, 원본 객체를 직접 호출하지 않고 그 앞에 프록시 객체를 만들어서 메서드 실행 전후에 Advice 공통 로직 삽입
- 즉, 프록시가 호출을 가로채서 @Before, @After, @Around 등 Advice를 실행 후 Target의 실제 메서드를 호출함

### Proxy
- JDK Dynamic Proxy
  ```
    Client -> Proxy 객체 -> InvocationHandler.invoke() -> 실제 Target 메서드 실행
  ```
  - 인터페이스가 존재할 경우 기본적으로 사용됨
  - JDK의 `java.lang.reflect.Proxy` 클래스로 **인터페이스 기반 프록시를 동적**으로 생성
  - 예를 들어, Spring AOP가 OrderServiceImpl를 감싸는 Proxy 객체를 OrderService 인터페이스의 구현체로 런타임에 생성함
- CGLIB Proxy
  ```
    Client -> CGLIB Proxy (서브클래스) -> MethodInterceptor.intercept() -> Target 메서드 실행
  ```
  - 대상 클래스에 인터페이스가 없거나 `proxyTargetClass=true`로 설정했다면
  - **클래스 상속 기반 프록시** 생성(바이트 코드 조작)
  - CGLIB은 내부적으로 ASM이라는 라이브러리를 사용해 **클래스를 상속한 서브 클래스를 만들어 프록시를 구현**
  - 부모 클래스의 메서드 중 final 메서드는 오버라이딩이 불가하여 프록시가 적용되지 않음!!
