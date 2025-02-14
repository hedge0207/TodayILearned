# 객체와 자료 구조

- 자료 추상화

  - 아래 두 개의 클래스를 살펴보자
    - 앞의 클래스는 구현을 외부로 노출하고, 다른 클래스(인터페이스)는 구현을 완전히 숨긴다.

  ```java
  public class Point {
      public double x;
      public double y;
  }
  
  public interface Point {
      double getX();
      double getY();
      void setCartesian(double x, double y);
      double getR();
      double getTheta();
      void setPolar(double r, double theta)
  }
  ```

  - 두 번째 클래스는 자료 구조 이상을 표현한다.
    - 클래스가 메서드 접근 정책을 강제한다.
    - 좌표를 읽을 때는 각 값을 개별적으로 읽어야 하지만 좌표를 설정할 때는 두 값을 한 번에 설정해야 한다.
  - 변수와 사용자 사이에 함수라는 계층을 넣는다고 구현이 저절로 감춰지지 않는다.
    - 따라서 변수를 private으로 선언하더라도 각 값마다 조회(get) 함수와 설정(set) 함수를 제공한다면 구현을 외부로 노출하는 셈이다.
    - 구현을 감추려면 추상화가 필요하다.
    - 조회 함수와 설정 함수로 변수를 다룬다고 클래스가 되는 것은 아니며, 추상 인터페이스를 제공해 사용자가 구현을 모른 채 자료의 핵심을 조작할 수 있어야 진정한 의미의 클래스다.
  - 자료를 세세하게 공개하기 보다는 추상적인 개념으로 표현하는 것이 좋다.



- 자료/객체 비대칭

  - 객체와 자료 구조
    - 객체는 추상화 뒤로 자료를 숨긴 채 자료를 다루는 함수만 공개한다.
    - 자료 구조는 자료를 그대로 공개하며 별다른 함수는 제공하지 않는다.
    - 위의 두 정의는 본질적으로 상반된다.
  - 절차적인 도형 클래스
    - 아래 클래스가 객체지향적이지 않고 절차 지향적이라 비판한다면 맞는 말이다.
    - 그러나 그런 비판이 완전히 옳다고 말하기는 어렵다.
    - 만약 `Geometry` 클래스에 둘레 길이를 구하는 `perimeter()` 메서드를 추가해야 한다고 가정해보자.
    - 새로운 메서드를 추가하더라도 도형 클래스는 아무 영향도 받지 않으며, 도형 클래스에 의존하는 다른 클래스도 마찬가지다.
    - 그러나 새 도형을 추가해야 할 경우 `Geometry` 클래스에 속한 함수를 모두 고쳐야한다.

  ```java
  public class Squre {
      public Point topLeft;
      public double side;
  }
  
  public class Rectangle {
      public Point topLeft;
      public double height;
      public double width;
  }
  
  public class Circle {
      public Point center;
      public double radius;
  }
  
  public class Geometry {
      public final PI = 3.142592;
   
      public double area(Object shape) throws NoSuchShapeException {
          if (shape instanceof Squre) {
              Squre s = (Squre)shape;
              return s.side * s.side;
          }
          else if (shape instanceof Recatanble) {
              Rectangle r = (Ractangle)shape;
              return r.height * r.width;
          }
          else if (shape instanceof Circle) {
              Circle c = (Circle)shape;
              return PI * c.radius * c.radius;
          }
          throw new NoSuchShapeException();
      }
  }
  ```

  - 객체 지향적인 도형 클래스
    - `aria()`는 다형(polymorphic) 메서드다.
    - Geometry 클래스는 필요 없으며, 따라서 새 도형을 추가해도 기존 함수에 아무런 영향을 미치지 않는다.
    - 반면에 새 함수를 추가하고 싶다면 도형 클래스를 전부 고쳐야 한다.

  ```java
  public class Squre implements Shape{
      public Point topLeft;
      public double side;
      
      public double area() {
          return side * side
      }
  }
  
  public class Rectangle implements Shape{
      public Point topLeft;
      public double height;
      public double width;
      
      public double area() {
          return height * width;
      }
  }
  
  public class Circle implements Shape{
      public Point center;
      public double radius;
      public final PI = 3.142592;
      
      public double area() {
          return PI * radius * radius;
      }
  }
  ```

  - 두 코드의 차이
    - 자료 구조를 사용하는 절차적인 코드는 기존 자료 구조를 변경하지 않으면서 새 함수를 추가하기 쉬운 반면, 객체 지향 코드는 기존 함수를 변경하지 않으면서 새 클래스를 추가하기 쉽다.
    - 자료 구조를 사용하는 절차적인 코드는 새로운 자료 구조를 추가하려면 모든 함수를 고쳐야해서 새로운 자료 구조를 추가하기 어렵고, 객체 지향 코드는 새로운 함수를 추가하려면 모든 클래스를 고쳐야 하므로 새로운 함수를 추가하기 어렵다.
    - 다시 말해, 객체 지향 코드에서 어려운 변경은 절차적인 코드에서 쉬우며, 절차적인 코드에서 어려운 변경은 객체 지향 코드에서 쉽다.
  - 분별 있는 프로그래머는 모든 것이 객체라는 생각이 미신임을 잘 안다.
    - 때로는 단순한 자료 구조와 절차적인 코드가 가장 적합한 상황도 있다.



- 디미터 법칙

  - 디미터 법칙은 모듈은 자신이 조작하는 객체의 속사정을 몰라야 한다는 법칙이다.
    - 앞에서 봤듯이 객체는 자료를 숨기고 함수를 공개한다.
    - 즉 객체는 조회 함수로 내부 구조를 공개하면 안 된다는 의미다.

  - 좀 더 정확히 표현하면 디미터 법칙은 클래스 C의 메서드 f는 다음과 같은 객체의 메서드만 호출해야 한다는 법칙이다.
    - 클래스 C
    - f가 생성한 객체
    - f의 인수로 넘어온 객체
    - C의 인스턴스 변수에 저장된 객체
    - 단, 이 객체들의 메서드가 반환하는 객체의 메서드는 호출하면 안 된다.

  - 기차 충돌(train wreck)
    - 아래 코드는 디미터 법칙을 어기고 있는 것이다.
    - 흔히 아래와 같은 코드를 기차 충돌이라 부른다.
    - 여러 객차가 한 줄로 이어진 기차처럼 보이기 때문이며, 일반적으로 조잡하다 여겨지는 방식이므로 피하는 것이 좋다.


  ```java
  final String outputDir = ctxt.getOptions().getScratchDir().getAbsolutePath();
  ```

  - 위 코드는 아래와 같이 나누는 것이 좋다.

  ```java
  Options opts = ctxt.getOptions();
  File scratchDir = opts.getScratchDir();
  final String outputDir = scratchDir.getAbsolutePath();
  ```

  - 나누어진 코드도 디미터 법칙을 위반하는가?
    - 위 코드는 `ctxt` 객체가 Options를 포함하고, Options가 ScratchDir을 포함하며, ScratchDir이 AbsolutePath를 포함한다는 사실을 안다.
    - 즉 함수 하나가 아는 지식이 굉장히 많으며, 따라서 위 코드를 사용하는 함수는 많은 객체를 탐색할 줄 알아야 한다는 말이다.
    - 위 예제게 디미터 법칙을 위반하는지 여부는 ctxt, Options, ScratchDir이 객체인지 자료구조인지에 달렸다.
    - 객체라면 내부 구조를 숨겨야 하므로 확실히 디미터 법칙을 위반한다.
    - 반면 자료 구조라면 당연히 내부 구조를 노출하므로 디미터 법칙이 적용되지 않는다.
    - 그런데 위 예제는 조회 함수를 사용하는 바람에 혼란을 일으킨다.
    - 코드를 아래와 같이 구현했다면 디미터 법칙을 거론할 필요가 없었을 것이다.

  ```java
  final String outputDir = ctxt.options.scratchDir.absolutePath;
  ```

  - 잡종 구조
    - 자료 구조는 무조건 함수 없이 공개 변수만 포함하고, 객체는 비공개 변수와 공개 함수를 포함한다면 문제는 훨씬 간단해진다.
    - 그러나 단순한 자료 구조에도 조회 함수와 설정 함수를 정의하라 요구하는 프레임워크와 표준(e.g. bean)이 존재한다.
    - 이런 혼란으로 인해 때때로 절반은 객체, 절반은 자료 구조인 잡종 구조가 나온다.
    - 잡종 구조는 중요한 기능을 수행하는 함수도 있고, 공개 변수나 공개 조회/설정 함수도 있다.
    - 공개 조회/설정 함수는 비공개 변수를 그대로 노출한다.
    - 이로 인해 다른 함수가 절차적인 프로그래밍의 자료 구조 접근 방식처럼 비공개 변수를 사용하고픈 유혹에 빠지기 십상이다.
    - 이런 잡종 구조는 자료 구조와 객체의 단점만 모아놓은 구조다. 즉 새로운 함수는 물론이고 새로운 자료 구조도 추가하기 어렵다.
    - 그러므로 잡종 구조는 되도록 피하는 것이 좋다.
    - 잡종 구조는 프로그래머가 타입을 보호할지 공개할지 확신하지 못해 어중간하게 내놓은 설계에 불과하다.
  - 구조체 감추기
    - 만약 ctxt, options, scratchDir이 진짜 객체라면 내부 구조를 감춰야 하므로 앞의 예시처럼 줄줄이 엮어서는 안 된다.
    - 아래는 내부 구조를 감추기 위한 방법들이다.
    - 첫 번째 방법은 ctxt 객체에 공개해야하는 메서드가 너무 많아진다.
    - 두 번째 방법은 getScratchDirectoryOption()이 객체가 아니라 자료 구조를 반환한다고 가정한다.
    - 두 방법 다 훌륭한 방법은 아니다.
    - ctxt가 객체라면 뭔가를 하라고 말해야지 속을 드러내라고 말하면 안 된다.
    - 세 번째 방법은 추상화 수준을 뒤섞어 놓아 다소 불편하다. 점, 슬래시, 파일 확장자, File 객체가 뒤섞여있다.
    - 네 번째 방법이 구조체를 감추기 위한 가장 좋은 방법이다.
    - 코드의 목적이 임시 파일을 생성하기 위해 임시 디렉터리의 절대 경로를 얻는 것이므로, 그냥 ctxt 객체에게 임시 파일을 생성하라고 시킨다.

  ```java
  // 1
  ctxt.getAbsolutePathOfScratchDirectoryOption();
  
  // 2
  ctx.getScratchDirectoryOption().getAbsolutePath();
  
  // 3
  String outFile = outputDir + "/" + className.replace(".", "/") + ".class";
  FileOutputStream fout = new FileOutputStream(outFile);
  BufferedOutputStream bos = new BufferedOutputStream(fout);
  
  // 4
  BufferedOutputStream bos = ctxt.createScratchFileStream(classFileName);
  ```



- 자료 전달 객체(Data Transfer Object, DTO)

  - 자료 구조체의 전형적인 형태는 공개 변수만 있고 함수가 없는 클래스이며, 이런 자료 구조체를 때로는 자료 전달 객체라 한다.
  - DTO는 매우 유용한 구조체다.
    - 특히 DB와 통신하거나 소켓에서 받은 메시지의 구문을 분석할 때 유용하다.
    - 흔히 DTO는 DB에 저장된 가공되지 않은 정보를 애플리케이션 코드에서 사용할 객체로 변환하는 일련의 단계에서 가장 처음으로 사용하는 구조체다.
  - 좀 더 일반적인 형태는 bean 구조다.
    - Bean은 비공개 변수를 조회/설정 함수로 조작한다.
    - 일종의 사이비 캡슐화로, 일부 OO 순수주의자나 만족시킬뿐 별다른 이익을 제공하지 않는다.

  ```java
  public class Address {
      private String street;
      private String streetExtra;
      private String city;
      private String state;
      private String zip;
      
      public Address(String street, String streetExtra, String city, String state, String zip) {
          this.street = street;
          this.streetExtra = streetExtra;
          this.city = city;
          this.state = state;
          this.zip = zip;
      }
      
      public String getStreet() {
          return street;
      }
      
      public String getStreetExtra() {
          return streetExtra;
      }
      
      // ... 다른 조회 함수들
  }
  ```

  - 활성 레코드
    - 활성 레코드는 DTO의 특수한 형태다.
    - 공개 변수가 있거나 비공개 변수에 조회/설정 함수가 있는 자료 구조지만, 대개 save나 find 같은 탐색 함수도 제공한다.
    - 활성 레코드는 DB 테이블이나 다른 소스에서 자료를 직접 변환한 결과다.
    - 불행히도 활성 레코드에 비즈니스 규칙 메서드를 추가해 이런 자료 구조를 개체로 취급하는 개발자가 흔하다.
    - 하지만 이는 바람직하지 않다.
    - 이 경우 잡종 구조가 나오기 때문이다.
    - 따라서 활성 구조체는 자료 구조로 취급해야 하며, 비즈니스 규칙을 담으면서 내부 자료를 숨기는 객체는 따로 생성해야한다.







# 오류 처리





