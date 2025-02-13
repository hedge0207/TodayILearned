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

