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

- 오류 처리는 깨끗한 코드를 작성하는 데 중요하다.
  - 상당수 코드 기반은 전적으로 오류 처리 코드에 좌우된다.
    - 여기서 좌우된다는 것은 코드 기반이 오류만 처리한다는 의미가 아니다.
    - 여기저기 흩어진 오류 처리 코드 때문에 실제 코드가 하는 일을 파악하기가 거의 불가능하다는 의미다.
  - 오류 처리는 중요하지만 오류 처리 코드로 인해 프로그램 논리를 이해하기 어려워진다면 깨끗한 코드라 부르기 어렵다.



- 오류 코드보다 예외를 사용하라

  - 아래는 호출자에게 오류 코드를 반환하는 방식으로 오류를 처리하는 코드이다.
    - 아래와 같은 방식은 함수를 호출한 즉시 오류를 확인해야 하기 때문에 호출자 코드가 복잡해진다.

  ```java
  public class DeviceController {
    // ...
    public void sendShutDown() {
      DeviceHandle handle = getHandle(DEV1);
      if (handle != DeviceHandle.SUSPENDED) {
        retrieveDeviceRecord(handle);
        if (record.getStatus() != DEVICE_SUSPENDED) {
          pauseDevice(handle);
          clearDeviceWorkQueue(handle);
          consoleDevice(handle);
        } else {
          logger.log("Device suspended. Unable to shut down");
        }
      } else {
        logger.log("Invalid handle for: " + DEV1.toString());
      }
    }
  }
  ```

  - 오류가 발생하면 아래와 같이 예외를 던지는 편이 낫다.
    - 호출자 코드가 더 깔끔해진다.
  - 메인 로직을 처리하는 부분과 오류를 처리하는 부분이 분리되어 각 부분을 독립적으로 살펴보고 이해할 수 있다.

  ```java
  public class DeviceController {
    public void sendShutDown() {
      try {
        tryToShutDonw();
      } catch (DeviceShutDownError e) {
        logger.log(e);
      }
    }
  
    private void tryToShutDonw() throws DeviceShutDownError {
      DeviceHandle handle = getHandle(DEV1);
      DeviceRecord record = retrieveDeviceRecord(handle);
      pauseDevice(handle);
      clearDeviceWorkQueue(handle);
      consoleDevice(handle);
    }
  
    private DeviceHandle getHandle(DeviceID id) {
      // ...
      throw new DeviceShutDownError("Invalid handle for: " + id.toString());
      // ...
    }
  }
  ```




- Try-Catch-Finally 부터 작성해라

  - 어떤 면에서 try 블록은 트랜잭션과 유사하다.
    - try 블록에서 무슨 일이 생기든지 catch 블록은 프로그램 상태를 일관성 있게 유지해야 한다.
    - 따라서 예외가 발생할 코드를 짤 때는 try-catch-finally문으로 시작하는 것이 좋다.
    - 그러면 try 문에서 무슨 일이 생기든지 호출자가 기대하는 상태를 정의하기 쉬워진다.
  - 아래는 파일이 없으면 예외를 던지는지 알아보는 단위 테스트다.

  ```java
  @Test(expected = StorageException.class)
  public void retrieveSectionShouldThrowOnInvalidFileName() {
    sectionStore.retrieveSection("invalid-file");
  }
  ```

  - 단위 테스트에 맞춰 아래와 같은 코드를 구현했다.
    - try-catch-finally를 작성하지 않았다.
    - 아래 코드는 예외를 던지지 않으므로 단위 테스트는 실패한다.

  ```java
  public List<RecordedGrip> retrieveSection(String sectionName) {
    return new ArrayList<RecordedGrip>();
  }
  ```

  - 아래와 같이 예외를 던지도록 코드를 수정한다.
    - 이제 테스트는 성공하게 된다.

  ```java
  public List<RecordedGrip> retrieveSection(String sectionName) {
  try {
    FileInputStream stream = FileInputStream(sectionName);
    stream.close();
  } catch (FileNotFoundException e) {
    throw new StorageException("retrieval error", e);
  } 
  return new ArrayList<RecordedGrip>();
  }
  ```



- 미확인 예외를 사용하라.

  > 이 부분의 서문이 "논쟁은 끝났다"인데, [엘레강트 오브젝트]의 자자가 이에 반박하듯 자신의 블로그에 [The Debate Is Not Over](https://www.yegor256.com/2015/07/28/checked-vs-unchecked-exceptions.html)라는 제목으로 확인 예외의 필요성을 주장하는 글을 올렸다.

  - 확인된(checked) 예외와 미확인(unchecked) 예외
    - 확인된 예외는 다른 언어에는 없는 Java만의 예외로, 컴파일러가 예외 처리를 강제하는 예외이다.
    - 확인된 예외를 처리하는 코드를 작성하지 않을 경우 컴파일 타임에 컴파일 에러가 발생하게 된다.
    - `Exception` class의 자손들 중 `RuntimeException`의 자손이 아닌 class들이 확인 예외에 해당하며 `FileNotFoundException`가 대표적인 확인된 예외다.
    - 반면에 미확인 예외는 다른 언어들에도 존재하는 예외로, 처리하지 않아도 컴파일 에러가 발생하지 않는다.
  - 확인된 예외와 비확인 예외 중 무엇을 사용해야 하는지에 관해 Java 개발자들 사이에서 오랫동안 논쟁이 있었다.
    - 확인된 예외가 견고한 애플리케이션을 만드는 데 도움을 준다는 주장과 생산성을 떨어뜨린다는 주장이 대립했다.
    - 그러나 애플리케이션의 규모가 점차 거대해지면서 모든 예외를 예측해서 처리하는 것이 불가능에 가까워졌고, 오히려 예외 처리에 더 많은 시간을 할애하게 된다는 단점으로 인해 확인된 예외는 요즘에는 거의 사용되지 않는다.
    - Java와 완전히 호환되도록 제작된 Kotlin은 확인된 예외를 강제하지 않으며, Spring 역시 확인된 예외를 처리하지 않는다.
  - 확인된 예외는 OCP를 위반한다.
    - 메서드에서 확인된 예외를 던졌는데 catch 블록이 세 단계 위에 있다면 그 사이 메서드 모두가 선언부에 해당 예외를 정의해야한다.
    - 즉, 하위 단계에서 코드를 변경하면 상위 단계 메서드 선언부를 전부 고쳐야 한다는 말이다.
    - 모듈과 관련된 코드가 전혀 바뀌지 않았더라도 모듈을 다시 빌드한 다음 배포해야 한다는 말이다.
  - 확인된 예외는 대규모 시스템에도 부적절하다.
    - 대규모 시스템에서 호출이 일어나는 방식을 생각해보면, 최상위 함수가 아래 함수를 호출하고, 아래 함수는 그 아래 함수를 호출한며, 단계를 내려갈수록 호출하는 함수 수는 늘어난다.
    - 이제 최하위 함수를 변경해 새로운 오류를 던진다고 가정하자
    - 확인된 오류를 던진다면 함수는 선언부어 throws 절을 추가해야한다.
    - 그러면 변경한 함수를 호출하는 모두가 catch 블록에서 새로운 예외를 처리하거나, 선언부에 throw절을 추가해야한다.
    - 결과적으로 최하위 단계에서 최상위 단계까지 연쇄적인 수정이 일어난다.
    - throws 경로에 위치하는 모든 함수가 최하위 함수에서 던지는 예외를 알아야 하므로 캡슐화가 깨진다.
    - 오류를 원거리에서 처리하기 위해 예외를 사용한다는 사실을 감안하면 이처럼 확인된 예외가 캡슐화를 깨버리는 현상은 부적절하다.
  - 때로는 확인된 예외가 유용할 수도 있다.
    - 예를 들어 아주 중요한 라이브러리를 작성한다면 모든 예외를 잡아야 할 수 있다.
    - 그러나 일반적인 애플리케이션은 의존성이라는 비용이 이익보다 크다.



- 예외에 의미를 제공해야한다.
  - 예외를 던질 때는 전후 상황을 충분히 덧붙여야한다.
    - 이를 통해 오류가 발생한 원인과 위치를 찾기 쉬워진다.
    - Java는 모든 예외에 호출 스택을 제공한다.
    - 하지만 실패한 코드의 의도를 파악하려면 호출 스택만으로는 부족한다.
  - 오류 메시지에 정보를 담아 예외에 함께 던져야한다.
    - 실패한 연산 이름과 실패 유형도 언급해야한다.
    - 애플리케이션이 로그를 남긴다면 catch 블록에서 오류를 기록하도록 충분한 정보를 넘겨줘야 한다.



- 호출자를 고려해 예외 클래스를 정의하라.

  - 아래 코드는 오류를 형편없이 분류한 예시이다.
    - 외부 라이브러리를 호출하는 try-catch-finally 문을 포함한 코드로, 외부 라이브러리가 던질 예외를 모두 잡아낸다.
    - 아래 코드는 중복이 심하지만, 오류를 처리하는 방식 자체는 우리가 오류를 처리하는 일반적인 방식이다.
    - 대다수 상황에서 우리가 오류를 처리하는 방식은 오류를 일으킨 원인과 무관하게 비교적 일정하다.
    - 먼저 오류를 기록하고, 오류에도 불구하고 프로그램을 계속 수행할 수 있는지 확인한다.

  ```java
  ACMEPort port = new ACMEPort(12);
  
  try {
      port.open();
  } catch (DeviceResponseException e) {
      reportPortError(e);
      logger.log("Device response exception", e);
  } catch (ATM1212UnlockedException e) {
      reportPortError(e);
      logger.log("Unlock exception", e);
  } catch (GMXError e) {
      reportPortError(e);
      logger.log("Device response exception");
  } finally {
      // ...
  }
  ```

  - 위 코드의 문제점
    - 호출자 입장에서 각기 다른 예외를 모두 개별적으로 처리해 줘야한다.
    - 만약 예외를 개별적으로 처리하지 않고, 단일한 방식으로 처리할 수 있다면 호출자 입장에서는 예외를 처리하기가 훨씬 간단해진다.

  - 위 코드를 아래와 같이 수정한다.
    - 위 코드는 예외에 대응하는 방식이 예외 유형과 무관하게 거의 동일하다.
    - 그래서 코드를 간결하게 고치기 아주 쉽다.
    - 호출하는 라이브러리 API를 감써면서 예외 유형 하나를 반환한다.

  ```java
  LocalPort port = new LocalPort(12);
  
  try {
      port.open();
  } catch (PortDeviceFailure e) {
      reportPortError(e);
      logger.log(e.getMessage(), e);
  } finally {
      // ...
  }
  ```

  - 위에서 `LocalPort` 클래스는 단순히 `ACMEPort` 클래스가 던지는 예외를 잡아 변환하는 wrapper 클래스일 뿐이다.
    - `LocalPort` 클래스처럼 `ACMEPort`를 감싸는 클래스는 매우 유용하다. 
    - 실제로 외부 API를 사용할 때는 감싸기 기법이 최선이다.
    - 외부 API를 감싸면 외부 라이브러리와 프로그램 사이에서 의존성이 크게 줄어든다.
    - 나중에 다른 라이브러리로 갈아타도 비용이 적다.
    - 또한 감싸기 클래스에서 외부 API를 호출하는 대신 테스트 코드를 넣어주는 방법으로 프로그램을 테스트하기도 쉬워진다.

  ```java
  public class LocalPort {
      private ACMEPort innterPort;
      
      public LocalPort(int portNumber) {
          innerPort = new ACMEPort(portNumber);
      }
      
      public void open() {
          try {
              innerPort.open();
          } catch (DeviceResponseException e) {
              throw new PortDeviceFailure(e);
          } catch (ATM1212UnlockedException e) {
              throw new PortDeviceFailure(e);
          } catch (GMXError e) {
              throw new PortDeviceFailure(e);
          } finally {
              // ...
          }
      }
  }
  ```



- 정상 흐름을 정의하라

  - 아래 예시는 비용 청구 애플리케이션에서 총계를 계산하는 코드다.
    - 식비를 비용으로 청구했다면 직원이 청구한 식비를 총계에 더한다.
    - 식비를 비용으로 청구하지 않았다면(`expenseReportDAO.getMeals()` 메서드가 `MealExepnsesNotFound`를 발생시킨다면) 일일 기본 식비를 총계에 더한다.
    - 그런데 예외가 논리를 따라가기 어렵게 만든다.

  ```java
  try {
      MealExpenses expense = expenseReportDAO.getMeals(employee.getID());
      m_total += expenses.getTotal();
  } catch(MealExepnsesNotFound e) {
      m_total += getMealPerDiem;
  }
  ```

  - 특수 상황을 처리할 필요가 없다면 코드는 아래와 같이 훨씬 더 간결해진다.

  ```java
  MealExpenses expenses = expenseReportDAO.getMeals(employee.getID());
  m_total += expenses.getTotal();
  ```

  - 위와 같이 특수 상황을 처리할 필요가 없게 만들려면, `expenseReportDAO.getMeals()` 메서드가 항상 `MealExpenses` 객체를 반환하게 만들면 된다.
    - 예를 들어 청구한 식비가 없다면 아래와 같이 `MealExpenses` 구현하는 클래스를 만들고, 일일 기본 식비를 반환하는 메서드를 구현하면 된다.
    - 이제 `expenseReportDAO.getMeals()`가 호출될 때 청구한 식비가 없다면, 아래에서 정의한 `PerDiemMEalExpenses`의 인스턴스를 반환하게 한다.
    - 그럼 `MealExepnsesNotFound`가 던져지지 않으므로 예외 처리를 할 필요가 없어진다.

  ```java
  public class PerDiemMEalExpenses implements MealExpenses {
      public int getTotal() {
          // 기본값으로 기본 식비를 반환한다.
      }
  }
  ```

  - 위와 같은 방식을 특수 사례 패턴(special case pattern)이라 부른다.
    - 클래스를 만들거나 객체를 조작해 특수 사례를 처리하는 방식이다.
    - 예를 들어 위 예시에서는 식비를 비용으로 청구하지 않은 특수 사례를 기존에는 예외를 던져 처리했으나, 특수 사례 패턴을 적용하여`PerDiemMEalExpenses` 클래스를 만들어, 해당 클래스에서 식비를 비용으로 청구하지 않은 경우 기본 식비를 반환하게 만들었다.
    - 특수 사례 패턴을 사용하면 클래스나 객체가 예외적인 상황을 캡슐화해서 처리하므로 클라이언트 코드가 예외적인 상황을 처리할 필요가 없어진다.



- null을 반환하지 마라

  - 아래와 같이 null인지 자주 확인하는 코드는 좋지 않은 코드다.

  ```java
  public void registerItem(Item item) {
      if (item != null) {
          ItemRegistry registry = perinstentStore.getItemRegistry();
          if (registry != null) {
              Item existing = registry.getItem(item.getId());
              if (existing.getBillingPeriod().hashRetailOwner()){
                  existing.register(item)
              }
          }
      }
  }
  ```

  - null을 반환하면 안 되는 이유
    - null을 반환하는 코드는 호출자로 하여금 반환값이 null인지 확인하게 만든다.
    - 이는 호출자의 코드가 복잡해지는 것을 의미한다.
    - 또한 null 확인을 잘 못 할 경우 버그가 발생할 수 도 있으며, 이 책임은 null을 반환한 함수가 아닌 호출자가 지게 된다.
  - 메서드에서 null을 반환하고픈 유혹이 든다면 그 대신 예외를 던지거나 특수 사례 객체를 반환하는 것이 낫다.
    - 사용하려는 외부 API가 null을 반환한다면 감싸기 메서드를 구현해 예외를 던지거나 특수 사례 객체를 반환하는 방식을 고려해야한다.
    - 많은 경우 특수 사례 객체가 손쉬운 해결책이다.
  - 예를 들어 아래 코드를 보자
    - 아래 코드에서 `getEmployees`는 null도 반환한다.
    - 따라서 호출자는 `getEmployees`의 반환값이 null인지 확인하는 과정을 거쳐야한다.

  ```java
  List<Employee> employees = getEmployees();
  if (employees != null) {
      for (Employee e : employees) {
          totalPay += e.getPay();
      }
  }
  ```

  - 위 코드에서 `getEmployees`가 null을 반환해야 할 때 대신 빈 배열을 반환하게 만들면, 호출자의 코드는 아래와 같이 보다 깔끔해진다.

  ```java
  List<Employee> employees = getEmployees();
  for(Employee e : employess) {
      totalPay += e.getPay();
  }
  ```



- null을 전달하지 마라

  - 메서드에서 null을 반환하는 방식도 나쁘지만 메서드에 null을 전달하는 방식은 더 나쁘다.
    - 정상적인 인자로 null을 기대하는 API가 아니라면 메서드로 null을 전달하는 코드는 최대한 피해야한다.
  - 예를 들어 아래 코드에서 `xProjection` 메서드의 인자로 null이 전달된다면, NullPointException이 발생할 것이다.

  ```java
  public class MetricCalculator {
      public double xProjection(Point p1, Point p2) {
          return (p2.x - p1.x) * 1.5;
      }
  }
  ```

  - NullPointException이 발생하는 것을 방지하기 위해 아래와 같이 코드를 변경할 수도 있다.
    - 두 인자 중 하나라도 null이 들어온다면 InvalidArguemtnException를 방생시킨다.
    - 문제는 호출자가 InvalidArguemtnException를 받아서 처리해야 한다는 점이다.

  ```java
  public class MetricsCalculator {
      public double xProjection(Point p1, Point p2) {
          if (p1 == null || p2 == null) {
              throw InvalidArguemtnException("...")
          }
          return (p2.x - p1.x) * 1.5; 
      }
  }
  ```

  - 또 다른 대안으로는 assert 문을 사용하는 방식도 있다.
    - 코드를 읽기는 더 편할 수 있지만 이 역시 문제를 해결하지는 못하며, 인자로 null을 전달하면 여전히 오류가 발생한다.

  ```java
  public class MetricCalculator {
      public double xProjection(Point p1, Point p2) {
          assert p1 == null : "p1 should not be null";
          assert p2 == null : "p2 should not be null";
          return (p2.x - p1.x) * 1.5; 
      } 
  }
  ```

  - 애초에 null을 넘기지 못하도록 금지하는 정책이 합리적이다.
    - 위에서 살펴본 것 처럼 대부분의 프로그래밍 언어는 호출자가 실수로 넘기는 null을 적절히 처리하는 방법이 없다.





# 경계

- 외부 코드 사용하기

  - 인터페이스 제공자와 인터페이스 사용자
    - 패키지 또는 프레임워크 제공자는 적용성을 최대한 넗히려 애쓴다.
    - 더 많은 환경에서 실행이 가능해야 더 많은 고객이 이용하기 때문이다.
    - 반면 사용자는 자신의 요구 사항에 집중하는 인터페이스를 원한다.
    - 인터페이스 제공자와 사용자 사이의 이러한 차이로 인해 시스템 경계에서 문제가 생길 소지가 다분하다.
  - `java.util.Map`의 경우
    - `Map`은 굉장히 다양한 인터페이스로 수많은 기능을 제공하는데, 이러한 기능성과 유연성은 유용하지만 위험도 크다.
    - 예를 들어 프로그램에서 Map을 만들어 여기저기 넘긴다고 가정하자.
    - 넘기는 쪽에서는 아무도 `Map` 내용을 삭제하지 않으리라 믿고 넘기지만 `Map`은 내용을 지울 수 있는 `clear` 메서드를 지원하므로, `Map`을 사용하는 사용자라면 누구나 내용을 삭제할 수 있다.
    - 또 다른 예로 설계 시 `Map`에 특정 객체 유형만 저장하기로 결정했다고 가정하자.
    - 그렇지만 `Map`은 객체 유형을 제한하지 않으며, 사용자는 어떤 객체 유형이든 `Map`에 추가할 수 있다.

  ```java
  // 누군가 아래와 같이 Sensor 유형의 객체를 담으려고 sensors라는 Map의 인스턴스를 생성했다.
  Map sensors = new HashMap();
  
  // Sensor 객체가 필요한 코드는 아래와 같이 Sensor 개체를 가져온다.
  // 이 때, `Map`이 반환하는 객체를 올바른 유형으로 변환할 책임은 `Map`을 사용하는 클라이언트에 있다.
  Sensor s = (Sensor)sensors.get(sensorId);
  ```

  - 제네릭스를 사용하면 가독성을 높일 수는 있다.
    - 그러나 `Map<String, Sensor>`이 사용자에게 필요하지 않은 기능까지 제공한다는 문제는 해결하지 못한다.
    - 또, 아래 코드에서 `Map<String, Sensor>` 인스턴스를 여기저기로 넘긴다면 `Map` 인터페이스가 변경될 경우, 수정할 코드가 상당히 많아진다.
    - 인터페이스가 변할 가능성이 없다고 생각할수 있지만, Java5가 제네릭스를 지원하면서 `Map` 인터페이스가 변한적이 있다.

  ```java
  // 
  Map<String, Sensor> sensors = new HashMap<Sensor>();
  // ...
  Sensor s = sensors.get(sensorId);
  ```

  - 아래는 `Map`을 좀 더 깔끔하게 사용한 코드다.
    - 경계 인터페이스인 `Map`을 `Sensors` 안으로 숨겨 `Map` 인터페이스가 변하더라도 나머지 프로그램에는 영향을 미치지 않게 한다.
    - `Sensors` 클래스 안에서 객체 유형을 관리하고 변환하기 때문에 제네릭스를 사용하든 사용하지 않든 더 이상 문제가 안 된다.
    - 또한 `Sensors` 클래스는 프로그램에 필요한 인터페이스만 제공하므로 코드는 이해하기 쉽고 오용하기 어렵다.

  ```java
  public class Seonsors {
      private Map sensors = new HashMap();
      
      public Sensor getById(String id) {
          return (Sensor) sensors.get(id);
      }
  }
  ```

  - 핵심은 `Map`과 같은 경계 인터페이스를 여기저기 넘기지 말라는 말이다.
    - 모든 경계 클래스를 위와 같이 캡슐화하라는 말이 아니다.
    - 경계 인터페이스를 사용할 때는 이를 이용하는 클래스나 클래스 계열 밖으로 노출되지 않도록 주의해야한다.
    - 또한 경계 인터페이스를 공개 API의 인수로 넘기거나 반환값으로 사용해선 안 된다.



- 경계 살피고 익히기
  - 학습 테스트
    - 외부 코드를 익히고 외부 코드를 우리가 작성한 코드와 통합하는 것은 어려운 일이다.
    - 시간을 들여서 외부 코드의 사용법을 익히고 우리쪽 코드에서 외부 코드를 사용하는 코드를 작성한 후 예상대로 동작하는 지 확인하는 것은 시간이 오래 걸릴뿐더러 버그가 발생했을 때, 우리 코드의 문제인지 외부 코드의 문제인지 파악하는 것도 쉽지 않다.
    - 따라서 외부 코드를 우리 코드에 바로 적용하기 보다는 먼저 간단한 테스트 케이스를 작성해 외부 코드를 익히는 것이 도움이 될 수 있다.
    - 이러한 테스트를 학습 테스트라 부른다.
    - 학습 테스트는 프로그램에서 사용하려는 방식대로 외부 API를 호출한다.
    - 즉 외부 코드의 전체를 테스트하는 것이 아니라, 우리가 사용하고자 하는 부분만 테스트하는 것이다.
    - 테스트 코드를 작성하는 과정에서 외부 코드에 대한 학습도 할 수 있으며, 버그가 발생했을 때 외부 코드와 우리가 작성한 코드 중 어느 부분에 문제가 있는 것인지도 쉽게 파악할 수 있다.
  - 학습 테스트는 공짜 이상이다.
    - 학습 테스트에 드는 비용은 없다.
    - 오히려 필요한 지식만 확보하는 손쉬운 방법이다.
    - 또한 외부 코드의 새로운 버전이 나왔을 때에도 학습 테스트를 실행해 새로운 버전과 우리가 작성한 코드에 어떤 차이가 있는지도 확인할 수 있다.



- 아직 존재하지 않는 코드를 사용해야 할 경우
  - 다른 개발자와 협업을 하다 보면 다른 개발자의 작업 진행도에 내 작업의 진행도가 종속되는 경우가 생길 수 있다.
    - 예를 들어 협업하는 개발자가 API를 개발해줘야 내가 해당 API를 호출하여 작업을 할 수 있는 경우 등이 있을 수 있다.
  - 이럴 경우 adapter 패턴을 사용하면 다른 개발자의 진행도와 무관하게 내 작업이 가능하다.
    - 먼저 내가 바라는 API의 인터페이스를 정의하고, 해당 인터페이스를 기반으로 작업을 진행한다.
    - 이후에 협업자가 API 개발을 완료하면 adapter를 추가하여 내가 정의한 인터페이스와 협업자가 작성한 API 사이를 중재한다.



- 깨끗한 경계
  - 경계에서는 변경이 이루어진다.
    - 소프트웨어 설계가 우수하다면 변경하는데 많은 투자와 재작업이 필요하지 않다.
    - 통제할 수 없는 코드를 사용할 때는 너무 많은 투자를 하거나 향후 변경 비용이 지나치게 커지지 않도록 각별히 주의해야한다.
  - 경계에 위치하는 코드는 깔끔히 분리해야 한다.









# 단위 테스트

- TDD 법칙
  - TDD는 실제 코드를 짜기 전에 단위 테스트부터 짜라고 요구한다.
  - 세 가지 법칙
    - 실패하는 단위 테스트를 작성할 때까지 실제 코드를 작성하지 않는다.
    - 컴파일은 실패하지 않으면서 실행이 실패하는 정도로만 단위 테스트를 작성한다.
    - 현재 실패하는 테스트를 통과할 정도로만 실제 코드를 작성한다.
  - 위 법칙을 모두 지키면서 일하면 실제 코드와 맞먹을 정도로 방대한 테스트 코드가 나올 수 있으며, 이는 심각한 관리 문제를 유발하기도 한다.



- 깨끗한 테스트 코드 유지하기

  - 지저분한 테스트 코드는 테스트를 안 하는 것과 별 차이가 없다.
    - 테스트 코드가 복잡할 수록 실제 코드를 작성하는 시간보다 테스트 케이스를 추가하는 시간이 더 걸리기마련이다.
    - 테스트 코드가 더러울수록 실제 코드 변경 이후 테스트 케이스를 통과시키기기는 점점 더 어려워진다.
    - 새 버전을 출시할 때마다 팀이 테스트 케이스를 유지보수하는 비용도 늘어나며, 점차 팀 내에서 테스트 코드 자체가 비난의 대상이 되어간다.
    - 결국 테스트 슈트를 폐기하기로 결정하는 상황에 이르게 된다.
  - 테스트 슈트가 없으면 개발자는 자신이 수정한 코드가 제대로 동작하는지 확인할 방법이 없다.
    - 테스트 슈트가 없으면 시스템이 수정 후에도 안전하다는 사실을 검증하지 못한다.
    - 테스트 슈트가 없어지면 결함율이 높아지며, 의도하지 않은 결함 수가 많아지면 개발자는 변경을 주저하게 된다.
    - 변경하면 득보다 해가 크다 생각해 더 이상 코드를 정리하지 않는다.
  - 결국 지저분한 테스트 코드는 테스트의 중단을 낳고, 이는 테스트를 위해 쏟은 노력도 허사로 만들게 된다.
    - 테스트 코드를 깨끗하게 유지하지 않으면 결국은 테스트를 하지 않게 되며, 이는 코드를 유연하게 만드는 버팀목이 사라진다는 의미이다.

  - 테스트는 유연성, 유지보수성, 재사용성을 제공한다.
    - 테스트 케이스가 없다면 개발자는 버그가 숨어들 수도 있다는 두려움에 변경을 주저하게 된다.
    - 하지만 테스트 커버리지가 증가할수록 이러한 두려움은 줄어든다.
    - 이를 통해 아키텍처가 부실한 코드나 설계가 모호하고 엉망인 코드라도 별다른 우려 없이 변경할 수 있다.



- 깨끗한 테스트 코드
  - 깨끗한 테스트 코드를 만들기 위해서는 세 가지가 필요하다.
    - 가독성, 가독성, 가독성이 그것이다.
    - 어떤 의미에서 가독성은 실제 코드에서 중요한 것 보다 테스트 코드에서 더 중요하다.
    - 테스트 코드에서 가독성을 높이는 것은 여느 코드와 마찬가지로 명료성, 단순성, 풍부한 표현력 등이다.
  - 테스트 코드는 최소의 표현으로 많은 것을 나타내야한다.
    - 테스트 코드에는 잡다하고 세세한 부분들이 포함되어서는 안 된다.
    - 테스트 코드는 바로 본론에 돌입해 진짜 필요한 자료 유형과 함수만 사용해야한다.
    - 이를 통해 코드를 읽는 사람은 코드가 수행하는 기능을 바로 파악할 수 있게 된다.
  - 깨끗한 테스트 코드를 작성하기 위해서라면 어느 정도는 비효율적인 코드를 작성하는 것이 용인된다.
    - 이를 이중 표준이라 하는데, 실제 코드에서는 표준이 될 수 없는 것들이, 테스트 코드에서는 표준이 될 수 있다는 의미이다.
    - 예를 들어 테스트 환경에서는 깨끗한 테스트 작성을 위해 메모리나 CPU의 효율성 것들을 적당히 포기할 수 있다.



- 깨끗한 테스트 코드는 FIRST 법칙을 따른다.

  - Fast
    - 테스트는 빨라야 한다.
    - 테스트가 느리면 자주 실행할 엄두를 내지 못한다.

  - Independent
    - 각 테스트는 서로 의존하면 안 된다.
    - 한 테스트가 다음 테스트가 실행될 환경을 준비해서는 안 된다.
    - 각 테스트는 독립적으로 그리고 어떤 순서로 실행해도 무방해야한다.
    - 테스트가 서로에게 의존하면 하나가 실패할 때 나머지도 잇달아 실패하므로 원인을 찾기 어려워진다.
  - Reapeatable
    - 테스트는 어떤 환경(OS, 장소, 시간 등)에서도 반복 가능해야한다.
    - 테스트가 돌아가지 않는 환경이 하나라도 있다면 테스트가 실패한 이유를 둘러댈 변명이 생긴다.
  - Self-Validating
    - 테스트는 bool 값으로 결과를 내야 한다.
    - 성공 아니면 실패를 정확히 반환해야하며, 통과 여부를 알기 위해 로그 파일을 읽게 만들어서는 안 된다.
    - 통과 여부를 보려고 텍스트 파일 두 개를 수작업으로 비교하게 만들어서도 안 된다.
    - 테스트가 스스로 성공과 실패를 가늠하지 않는다면 판단은 주관적이 되며, 지루한 수작업이 필요해진다.
  - Timely
    - 테스트는 적시에 작성해야한다.
    - 단위 테스트는 테스트하려는 실제 코드를 구현하기 직전에 구현한다.
    - 실제 코드를 구현한 다음에 테스트 코드를 만들면 실제 코드가 테스트하기 어렵다는 사실을 발견하게 될 수도 있다.

