# Spring Container
``` java
// 자바 설정 클래스 기반으로 스프링 컨테이너 생성
// AnnotationConfigApplicationContext은 ApplicationContext의 구현체이다.
ApplicationContext ac = new AnnotationConfigApplicationContext(AppConfig.class);
```
**BeanFactory와 ApplicationContext**

<img width="191" height="301" alt="images_mardi2020_post_1dbda601-39ea-4f91-8ba2-a10d7dde3aa7_무제" src="https://github.com/user-attachments/assets/fa8c94f3-0175-406f-b968-646e90b52573" />

- BeanFactory: 스프링 컨테이너의 최상위 인터페이스, 스프링 빈을 관리하고 조회하는 역할
- ApplicationContext(스프링 컨테이너)
  <img width="931" height="431" alt="images_mardi2020_post_a139a438-97cb-423f-97e0-ead61d546a96_무제" src="https://github.com/user-attachments/assets/26da5812-fe28-4de7-9f4f-ada8547ce9b4" />

  - `BeanFactory`을 상속받아 추가 기능까지 제공
    - MessageSource: 국제화(i18n) 지원
    - ApplicationEventPublisher: ApplicationEvent 발행 및 ApplicationListener로 이벤트 구독
    - Environment 정보 제공: Environment와 PropertySource를 통해 외부 설정(*.properties, *.yml 등)을 읽어옴
      - `context.getEnvironment().getProperty("app.name")`
    - ResourceLoader: classpath:, file:, url: 등의 접두사를 사용해 외부 리소스를 로드함
      - `context.getResource("classpath:app.properties")
  - @Configuration 설정정보 클래스를 구성 정보로 사용
  - 설정정보 클래스에 정의된 @Bean 메서드를 빈으로 등록

### Spring container 생성 과정

1. 스프링 컨테이너 생성

   <img width="682" height="391" alt="images_mardi2020_post_f52c82bd-5310-411f-8243-df9715ceb14a_d" src="https://github.com/user-attachments/assets/5ae71c10-dd43-4bc4-966c-a4de803c98c3" />

  - `new AnnotationConfigApplicationContext(AppConfig.class)`에 구성정보 AppConfig.class 지정

2. 스프링 빈 등록

   <img width="551" height="261" alt="images_mardi2020_post_09a9664b-1204-4603-8406-ce9e01d17dd3_d" src="https://github.com/user-attachments/assets/e27f8257-df49-4bcd-a5d8-fac0fde95a4d" />

  - 스프링 컨테이너는 매개변수로 넘어온 설정 클래스 정보를 이용해 스프링 빈 등록
    - AppConfig 클래스에 정의된 @Bean 메서드를 호출하여 반환된 객체를 빈으로 등록

3. 스프링 빈 의존관계 설정

   <img width="551" height="571" alt="images_mardi2020_post_338ee621-3b63-45fd-8a65-7eb773437d7e_d" src="https://github.com/user-attachments/assets/aefa8d7f-8479-4fef-841b-9fdc9cd94b92" />

  - 스프링 컨테이너는 설정 정보를 기반으로 의존성 주입
