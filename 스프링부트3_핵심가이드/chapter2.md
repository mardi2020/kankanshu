# chatper2

<img width="2081" height="1451" alt="Image" src="https://github.com/user-attachments/assets/190c954b-7134-4b55-8f81-654dcddb1135" />

- [GoF 디자인 패턴](https://refactoring.guru/ko/design-patterns)
- Servlet: 클라이언트의 요청을 처리하고 응답을 생성, 서블릿 컨테이너에서 동작
  - HttpServletRequest: 클라이언트 요청
  - HttpServletRespose: 요청에 대한 비즈니스 로직 처리 결과
  - 흐름(Ex: GET 동작)
    - client request(GET /hello?user=mardi2020) 

      -> Web Server
      
      -> Servlet Container(Tomcat)
      
      -> HttpServletReqeust, HttpServletResponse 객체 생성 

      -> HttpServlet.service() 호출 

      -> doGet() 메서드 실행 

      -> req.getParameter("user") 

      -> resp.getWriter().write("hello mardi2020") 

      -> servlet container에서 client로 response 전송(200, "hello mardi2020")
         
- springboot에서 servlet을 직접사용하지 않아도 되는 이유?
  - Spring MVC와 내장 서블릿 컨테이너가 서블릿의 동작을 추상화하고 auto-configuration으로 감춰두었기 때문
  - springboot는 `DispatcherServlet`을 자동으로 등록해 모든 요청을 처리하도록 구성함
  - HttpServletReqeust, HttpServletResponse 를 직접 다루는 대신에 `@RestController, @Controller`를 사용하여 요청과 응답을 추상화
    - @RequestParam, @RequestBody, @ResponseBody 등으로 매핑해서 사용
  - 서블릿 관련 설정(생성, mapping, 수명 주기 등)은 springboot의 auto-configuration으로 가능
  - **DispatcherServlet** 으로 모든 HTTP 요청이 들어옴
    - client 요청 -> 내장 Tomcat -> DispatcherServlet -> HandlerMapping -> Controller -> JSON 응답
  
