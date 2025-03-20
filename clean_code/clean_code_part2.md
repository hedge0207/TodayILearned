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









