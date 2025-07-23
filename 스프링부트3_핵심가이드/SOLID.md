# SOLID

### 1. SRP
단일 책임 원칙(Single Responsibility Principle)

- 하나의 클래스는 하나의 책임만 가져야 한다.
- 하나의 책임이라는 것은 모호하다. 클 수도 있고 작을 수도 있으며 문맥과 상황에 따라 다를 수 있다.
- 중요한 기준은 변경이다. 변경되었을 때, 파급 효과가 적을 경우 단일 책임 원칙을 잘 따랐다고 말할 수 있다.
  - 변경 사유가 한가지를 초과한다면 SRP 위반이라고 볼 수 있음

Ex) UI 변경, 객체의 생성과 사용 분리

Before - Report 클래스는 리포트 생성, 파일 저장, 이메일 전송의 3가지 책임을 갖고 있음
- Report 클래스의 변경 사유는?
  - 리포트 포맷 변경시 generateReport() 수정
  - 저장 경로 정책 변경시 saveToFile() 수정
  - 이메일 전송 방식 변경시 sendEmail() 수정
  으로 총 3가지 변경 사유가 있으므로 SRP 위반

**After** - Report 클래스가 갖고 있던 2가지 책임을 ReportFileSaver, ReportEmailSender 클래스로 분리하여 하나의 책임만을 가짐
- Report 클래스의 책임(변경 사유)
  - Report: 리포트의 생성만 담당
- ReportFileSaver 클래스: 리포트 저장만 담당
- ReportEmailSender 클래스: 리포트 전송만 담당

### 2. OCP
개방-폐쇄 원칙(Open/Closed Principle)

- 소프트웨어 요소는 확장에는 Open, 변경에는 Closed
- 다형성을 활용
- 인터페이스를 구현한 새로운 클래스를 하나 만들어서 새로운 기능을 구현한다.
- 기존 코드를 변경하는 행위가 아님
- 문제점
  - 클라이언트가 인터페이스의 구현 클래스를 직접 선택할 때, 구현 객체를 변경하려면 클라이언트의 코드를 변경해야 한다.

Before - 새로운 등급이 추가될 떄마다 if-else 조건을 계속 수정해야 함(즉, 변경에 닫혀있지 않은 코드임)
```java
public class DiscountService {
  public double getDiscountPrice(double price, String grade) {
      if ("VIP".equals(grade)) {
          return price * 0.8;
      } else if ("GOLD".equals(grade)) {
          return price * 0.9;
      } else {
          return price;
      }
  }
}
```
**After** - 인터페이스로 다형성 도입, **새로운 등급이 추가되면 새로운 구현체만 추가**하면 되므로 DiscountService를 수정할 필요가 없어짐
``` java
public interface DiscountPolicy {
    double discount(double price);
}

public class VipDiscountPolicy implements DiscountPolicy {
    @Override
    public double discount(double price) {
        return price * 0.8;
    }
}

public class GoldDiscountPolicy implements DiscountPolicy {
    @Override
    public double discount(double price) {
        return price * 0.9;
    }
}

public class DiscountService {
  private final DiscountPolicy discountPolicy;

  public DiscountService(DiscountPolicy discountPolicy) {
      this.discountPolicy = discountPolicy;
  }

  public double getDiscountPrice(double price) {
      return discountPolicy.discount(price);
  }
}
```

### 3. LSP
리스코프 치환 원칙(Liskov Substitution Principle)

- 프로그램의 객체는 프로그램의 정확성을 깨뜨리지 않으면서 하위 타입의 인스턴스로 바꿀 수 있어야 한다.
- 다형성에서 하위 클래스는 인터페이스 규약을 다 지켜야 한다는 것
- 다형성을 위한 원칙
- 상속은 `IS-A` 관계를 만족해야 함(즉, 상위 클래스의 규약을 다 만족해야 함)

Ex) Car 인터페이스의 엑셀 기능은 앞으로 가는 역할을 맡는데, 뒤로 가게 구현하면 LSP를 위배했다고 볼 수 있다.

Before - Square는 Rectangle로 대체할 수 없음, Rectangle의 규약(setWidth, setHeight가 독립적으로 동작한다)를 위반함
```java
class Rectangle {
    protected int width;
    protected int height;
    
    public void setWidth(int width) { this.width = width; }
    public void setHeight(int height) { this.height = height; }
    
    public int getArea() { return width * height; }
}

class Square extends Rectangle {
    @Override
    public void setWidth(int width) {
      this.width = width;
      this.height = width; // 정사각형이므로 높이도 동일하게 강제
    }
    
    @Override
    public void setHeight(int height) {
        this.width = height;
        this.height = height;
    }
}
```

**After** - 상속 관계를 재설계하거나 공통 인터페이스를 도입하여 Rectangle과 Square는 서로 대체할 필요가 없고 **Shape 인터페이스로 묶어 다형성 보장**
```java
interface Shape {
    int getArea();
}

class Rectangle implements Shape {
    private int width;
    private int height;

    public Rectangle(int width, int height) {
        this.width = width;
        this.height = height;
    }

    @Override
    public int getArea() {
        return width * height;
    }
}

class Square implements Shape {
    private int side;
    
    public Square(int side) {
        this.side = side;
    }
    
    @Override
    public int getArea() {
        return side * side;
    }
}
```
### 4. ISP
인터페이스 분리 원칙(Interface Segregation Principle)

- 특정 클라이언트를 위한 인터페이스 여러 개가 범용 인터페이스 하나보다 낫다.
  (하나의 인터페이스에 많은 기능이 있으면 복잡해지므로)
- 인터페이스가 명확해지고 대체 가능성이 높아진다.
- 인터페이스에 너무 많은 내용이 포함되어 있어 특정 구현체에서는 필요없는 메서드까지 구현하도록 강제함
  - 불필요한 의존성이 생겨서 의미없는 코드를 작성하거나 `throw new UnsupportedOperationException()` 예외 작성
  - 인터페이스 변경시 모든 구현체에 영향을 받으므로 유지보수성 저하

Before - 로봇은 밥을 먹지않으므로 eat() 메서드를 구현할 필요가 없는데 Worker 인터페이스에 정의되어 있어 구현을 강제받음
```java
public interface Worker {
    void work();
    void eat();
}

public class Robot implements Worker {
    public void work() { //... }
    public void eat() {
        throw new UnsupportedOpertationException("로봇은 밥을 먹지 않음");
    }
}
```

After - `Workable`, `Eatable` 인터페이스로 분리하여 필요한 기능만 구현할 수 있도록 유연한 설계 적용
```java
public interface Workable {
    void work();
}

public interface Eatable {
    void eat();
}

public class Robot implements Workable {
    public void work() { //... }
}

public class Human implements Workable, Eatable {
    public void work() { //... }
    public void eat() { //... }
}
```

### 5. DIP
의존관계 역전 원칙(Dependency Inversion Principle)

- 프로그래머는 추상화에 의존하지, 구체화에 의존하면 안된다. 의존성 주입은 이 원칙을 따르는 방법 중 하나이다.
- 구현 클래스에 의존하지 말고 인터페이스에 의존하라는 의미이다.
- 역할에 의존하게 해야 한다는 것과 같다. 객체 세상도 클라이언트가 인터페이스에 의존해야 유연하게 구현체를 변경할 수 있기 때문이다. 구현 클래스에 의존하게 되면 변경이 어려워진다.

Before - UserService는 MySQLDatabase라는 구현 클래스에 직접 의존중
```java
public class MySQLDatabase {
    public void connect() {
        System.out.println("MySQL DB 연결");
    }
}
    
public class UserService {
    private MySQLDatabase database = new MySQLDatabase(); // 구체적 구현에 의존
    
    public void process() {
        database.connect();
    }
}
```

**After** - DB 벤더를 MySQL 에서 PostgreSQL로 변경해도 **UserService의 코드는 변경하지 않아도 됨**(즉, 추상화에 의존하게 됨)
``` java
public interface Database {
    void connect();
}

public class MySQLDatabase implements Database {
    @Override
    public void connect() {
        System.out.println("MySQL DB 연결");
    }
}
    
public class PostgreSQLDatabase implements Database {
    @Override
    public void connect() {
        System.out.println("PostgreSQL DB 연결");
    }
}

public class UserService {
    private final Database database;
    
    // 의존성 주입 (Constructor Injection)
    public UserService(Database database) {
        this.database = database;
    }
    
    public void process() {
        database.connect();
    }
}
```

## 마무리
객체 지향의 핵심은 다형성이다. 

다형성 만으로 구현 객체를 변경할 때 클라이언트 코드도 함께 변경되므로 OCP, DIP 원칙을 지킬 수 없다. 

무엇인가 더 필요해지는데, 여기서 스프링은 Dependency Injection, DI 컨테이너를 제공함으로써 OCP, DIP 원칙을 지킬 수 있도록 지원해 준다.(클라이언트 코드의 변경 없이 기능을 확장할 수 있도록 보장해줌)
- DIP + IoC + DI
  - DIP를 실현하려면 의존성 주입은 거의 필수적임
  - Spring에서는 IoC container를 통해 DIP를 구현함(Ex: @Autowired, @Bean, @Configuration 등)

- SRP: 클래스/메서드/모듈은 한가지 역할(책임)을 가져야 한다.
- OCP: 새로운 기능을 추가할 떄 기존 코드를 수정하지 않고 확장으로 대응할 수 있어야 한다.
- LSP: 상위 클래스의 인스턴스를 사용하는 코드에서 하위 클래스의 인스턴스로 바꿔도 프로그램이 정상적으로 동작해야 한다.
- ISP: 하나의 큰 인터페이스를 여러 개의 작은 인터페이스로 분리하여, 클라이언트가 꼭 필요한 메서드만 알 수 있도록 설계해야 한다.
- DIP: 추상화는 세부 사항에 의존하지 않고, 세부 사항이 추상화에 의존해야 한다.
