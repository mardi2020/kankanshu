# spring-boot-starter

## 역할
- 의존성 관리 간소화
  - `spring-boot-starter-web`을 추가하면 Spring MVC, Jackson, Tomcat 등의 라이브러리가 자동으로 포함됨
  - 개발자가 하나하나 spring-webmvc, jackson-databind, tomcat 등을 dependency로 추가할 필요가 없음
- 버전 충돌 방지 - Dependency Management
  - SpringBoot는 **Spring Boot BOM(Bill Of Materials)** 를 사용해 스타터 내 라이브러리 버전을 일괄 관리
    - BOM: 특정 라이브러리들의 호환되는 버전을 미리 정의해 놓은 설정 파일
    - 호환되는 버전들이 자동 적용되어 버전 충돌 문제를 줄임
- 빠른 애플리케이션 구성 (Convention over Configuration)
  - 스타터만 추가하면 기본적인 설정이 자동 적용되어 빠르게 서비를 시작할 수 있음
    - `spring-boot-starter-data-jpa` 추가 시, Hibernate + JPA 관련 설정을 자동구성

## spring-boot-starter-parent와의 차이
- `spring-boot-starter-*`: 필요한 의존성들의 모음
- `spring-boot-starter-parent`: Maven 프로젝트에서 부모 POM으로 사용해 버전 관리 및 플러그인 설정을 자동으로 가져옴
  - Ex) `spring-boot-starter-web`에서 별도로 버전을 지정하지 않아도 parent에서 알맞는 버전 조합을 알려줌
  - parent POM이 dependencyManagement 섹션에서 버전을 미리 지정해 둠
  - gradle에서는 maven과 다르게 parent의 개념이 없음
    - `io.spring.dependency-management` 플러그인이 BOM 역할 대신으로 사용하거나 `spring-boot-gradle-plugin`을 사용해 직접 BOM을 관리함
    - `implementation("org.springframework.boot:spring-boot-starter-web")`
