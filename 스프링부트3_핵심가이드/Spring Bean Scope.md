[Bean Scopes](https://docs.spring.io/spring-framework/reference/core/beans/factory-scopes.html)
# Bean Scopes

## Singleton scope
하나의 싱글톤 빈(shared instance)만 관리되며, 해당 빈 정의의 ID 또는 ID들과 일치하는 요청은 모두 Spring 컨테이너에 의해 동일한 특정 빈 인스턴스를 반환함
- 즉, singleton scope로 빈을 정의하면, Spring IoC 컨테이너는 해당 빈 정의에 의해 정의된 객체를 정확히 한 번만 생성함

이 단일 인스턴스는 싱글톤 빈의 캐시에 저장되고 이후 해당 이름의 빈에 대한 모든 요청과 참조는 캐시된 객체를 반환

<img width="800" height="398" alt="singleton" src="https://github.com/user-attachments/assets/564a5180-b709-479a-a1ca-30db2b696b50" />

Spring의 싱글톤은 GoF 디자인 패턴의 싱글톤 패턴과 다름
- GoF 싱글톤: 특정 클래스의 인스턴스가 ClassLoader당 하나만 생성되도록 객체의 범위를 하드코딩
- Spring의 싱글톤: 컨테이너당 하나, 특정 Spring 컨테이너 내에서 한 클래스에 대해 하나의 빈을 정의하면, Spring 컨테이너가 그 빈 정의에 의해 정의된 클래스의 인스턴스를 정확히 하나만 생성함
  - 싱글톤 범위는 Spring에서 default scope

XML에서 빈을 싱글톤으로 정의하려면, 다음 예제와 같이 빈을 정의할 수 있음
```xml
<bean id="accountService" class="com.something.DefaultAccountService"/>

<!-- the following is equivalent, though redundant (singleton scope is the default) -->
<bean id="accountService" class="com.something.DefaultAccountService" scope="singleton"/>
```

## Prototype Scope
prototype 범위로 빈을 설정하면, 해당 빈에 대한 요청이 발생할 때마다 새로운 빈 인스턴스가 생성됨
- 즉, 해당 빈이 다른 빈에 주입되거나 컨테이너의 `getBean()` 메서드 호출을 통해 요청될 때마다 새로운 인스턴스가 만들어짐
- 일반적으로, **상태를 가지는(stateful) 빈**에는 프로토타입 범위를 사용하고, **상태를 가지지 않는(stateless) 빈**에는 싱글톤 범위를 사용함

<img width="800" height="397" alt="prototype" src="https://github.com/user-attachments/assets/76e038a8-31fc-4a0c-b8ab-dfbef6704245" />

그외에 다양한 scope가 있으나 두가지만 설명하겠음! 나머지는 필요할 때 하나씩 볼 것

### 정리
| **특징**                          | **Singleton Scope**                                                 | **Prototype Scope**                                              |
|----------------------------------|----------------------------------------------------------------------|-------------------------------------------------------------------|
| **빈 인스턴스 생성 횟수**         | 한 번만 생성됨                                                       | 요청 시마다 새로 생성됨                                           |
| **범위 정의**                    | 컨테이너당 한 번 생성됨                                              | 요청한 대상(빈 주입, `getBean()` 호출 등)마다 새 인스턴스 생성    |
| **캐싱 여부**                    | Spring 컨테이너의 싱글톤 캐시에 저장                                  | 캐싱되지 않으며, 매번 새로운 인스턴스가 생성됨                   |
| **사용 대상**                    | 상태를 가지지 않는(Stateless) 빈                                      | 상태를 가지는(Stateful) 빈                                       |
| **기본 스코프 여부**             | Spring의 기본 스코프 (별도 설정이 없어도 싱글톤으로 동작)             | 기본 스코프가 아니므로 명시적으로 설정해야 사용 가능              |
| **사용 예시**                    | - 서비스, DAO, 공통 유틸리티 클래스 같은 공유 객체<br>- 재사용 가능한 로직 | - 사용자 세션, 요청 단위의 데이터 처리 객체                     |
| **XML 설정 예제**                | `<bean id="myBean" class="com.example.MyBean" scope="singleton"/>`   | `<bean id="myBean" class="com.example.MyBean" scope="prototype"/>` |
| **장점**                        | - 메모리 사용 효율적<br>- 인스턴스 재사용 가능<br>- 성능 향상          | - 매번 새 인스턴스 생성으로 인해 안전한 상태 관리 가능           |
| **단점**                        | - 상태를 가질 경우 동시성 문제 발생 가능                             | - 너무 자주 생성하면 메모리 사용량 과다 발생 가능                |
