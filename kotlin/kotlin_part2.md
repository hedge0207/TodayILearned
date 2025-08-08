# 함수

- 함수 선언하기

  - `fun` keyword를 사용하여 함수를 선언할 수 있다.
    - 함수명, parameters, return type, return 값이 기본 구성 요소이다.
    - parameters와 return type, return 값은 optional 한 값이다.
    - 그러나 만일 return 값이 있다면, 반드시 return 값에 맞는 return type을 지정해줘야 한다.
    - 함수명은 변수명과 convention이 동일하다.

  ```kotlin
  fun myFunction(p1: type, p2: type2, ...): ReturnType {
      return
  }
  ```

  - 모든 함수는 반환값을 가진다.

    - 반환값을 명시적으로 지정해주지 않더라도 Kotlin에서는 `Unit`이라는 특별한 값이 반환된다.

    - 이는 함수에 반환값이 없다는 것을 의미한다.
  - Java의 `void`와 유사하다.




- Single-expression functions

  - 아래와 같이 보다 간단하게 함수를 정의하는 것도 가능하다.

  ```kotlin
  fun sum(a: Int, b: Int): Int = a + b
  
  fun sayHello(): Unit = println("Hello")
  ```

  - return type을 정의하지 않는 것도 가능하다.

  ```kotlin
  fun sum(a: Int, b: Int) = a + b
  ```

  - if 표현식이나 when 표현식을 사용하는 것도 가능하다.

  ```kotlin
  fun max(a: Int, b: Int) = if (a > b) a else b
  ```



- 함수의 parmeter

  - 함수의 매개변수에는 var나 val 키워드를 사용할 수 없다.
    - val이 자동으로 적용되어, 함수 내에서 매개 변수의 값을 변경할 수 없다.

  - `=` 연산자를 사용하여 함수의 parameter에 기본값을 줄 수 있다.
    - 기본값을 주지 않은 parameter보다 뒤에 와야 한다.
  
  ```kotlin
  fun sum(num1: Int = 0, num2: Int = 0): Int = num1 + num2
  ```
  
  -  Named argument
     - 함수 호출시에 argument를 입력할 때, 단순히 parmeter에 정의된 순서대로 넣는 것이 아니라, parameter의 이름으로 넣는 것이 가능하다.
     - 이를 통해 code의 가독성을 높일 수 있다.
     - positional argument와 함께 사용할 경우, positional argument보다 뒤에 사용하거나 순서를 지켜 사용해야 한다.
  
  ```kotlin
  fun calcEndDayAmount(startAmount: Int, ticketPrice: Int, soldTickets: Int) =
          startAmount + ticketPrice * soldTickets
  
  // 이렇게 호출 가능하다.
  val amount = calcEndDayAmount(
      ticketPrice = 10,
      soldTickets = 500,
      startAmount = 1000
  )
  
  // positional argument와 함께 사용할 경우
  // 아래와 같이 순서를 지켜 사용하거나
  calcEndDayAmount(startAmount = 1000, 10, 500)
  // positional argument보다 뒤에 사용해야한다.
  calcEndDayAmount(1000, ticketPrice=10, soldTickets=500)
  ```

  - 아래와 같이 한 parameter를 다른 parameter의 기본 값으로 주는 것도 가능하다.
    - 그러나, 기본값으로 주려는 parameter가 기본값을 주려는 parameter보다 앞에 있어야 한다.
  
  ```kotlin
  fun sum2(a: Int, b: Int = a) = a + b
   
  sum2(1)    // 1 + 1
  sum2(2, 3) // 2 + 3
  
  // 아래와 같이 쓸 수는 없다.
  fun sum2(a: Int = b, b: Int = 1) = a + b
  ```
  



- 람다 함수

  - 람다 함수는 이름이 없으므로 함수명으로 호출할 수 없다.
    - 그러므로 보통 람다 함수를 변수에 대입해 사용한다.
    - 혹은 선언하자마자 사용하는 방식을 사용한다.

  ```kotlin
  // 선언과 동시에 사용
  {num1: Int, num2: Int -> num1 + num2} (1, 2)
  ```

  - 매개변수 없는 람다 함수
    - 함수에 매개 변수가 항상 있어야 하는 것은 아니다.
    - 람다 함수애서 화살표 왼쪽을 비워두면 매개 변수 없는 람다 함수가 된다.
    - 매개 변수가 없을 때는 화살표를 생략해도 된다.

  ```kotlin
  // 매개 변수 없는 람다 함수
  {-> println("lambda function call")}
  
  // 화살표 생략
  {println("lambda function call without arrow")}
  ```

  - 매개 변수가 1개인 람다 함수는 아래와 같이 작성할 수도 있다.
    - 람다 함수의 중괄호 안에 화살표가 없으므로 매개 변수가 없는 것처럼 보이지만 람다 함수 앞의 `(Int) -> Unit`이 매개 변수가 1개인 람다 함수임을 나타낸다.
    - 이처럼 람다 함수의 매개 변수가 1개일 때는 중괄호 안에서 매개 변수 선언을 생략하고 `it` 키워드로 매개 변수를 이용할 수 있다.
    - 람다 함수에서 `it`을 이용해 매개 변수를 사용하는 것은 해당 매개 변수가의 타입을 식별할 수 있을 때만 가능하다.
    - 아래 예시에서는 매개 변수의 타입을 Int로 선언했으므로 `it`이 가리키는 데이터가 Int 타입임을 알 수 있다.

  ```kotlin
  // 아래와 같은 매개 변수가 하나인 람다 함수는
  {num: Int -> println(num)}
  
  // 아래와 같이 작성할 수도 있다.
  (Int) -> Unit = {println(it)}
  ```

  - 람다 함수의 반환
    - 람다 함수도 함수이므로 자신을 호출한 곳에 결과값을 반환해야 할 때가 있따.
    - 그러나 람다 함수에서는 `return` 키워드를 사용할 수 없으며, 본문의 마지막 줄의 실행 결과가 반환값이 된다.

  ```kotlin
  val some = {num1: Int, num2: Int -> num1 * num2}
  println(some(1, 2))		// 3
  ```

  - 람다 함수와 소괄호 생략
    - 람다를 마지막 인자로 넘길 때 그 람다를 괄호 밖으로 뺄 수 있으며, 이 때 소괄호를 생략 가능하다.

  ```kotlin
  // 아래 함수는
  fun repeatTask(times: Int, task: () -> Unit) {
      for (i in 1..times) {
          task()
      }
  }
  
  // 아래와 같이 호출이 가능하다.
  repeatTask(3) {
      println("Hello")
  }
  
  // 소괄호를 생략하지 않을 경우, 아래와 같이 호출한다.
  // 일반적인 호출 방식
  repeatTask(3, {
      println("Hello")
  })
  ```



- 함수 타입과 고차 함수

  - 변수에 함수를 대입하려면 함수 타입으로 선언해야 한다.
    - 변수는 타입을 가지며 타입을 유추할 수 있을 때를 제외하고는 생략할 수 없기 때문이다.
    - 함수 타입이란 함수를 선언할 때 나타내는 매개 변수와 반환 타입을 의미한다.
    - 예를 들어 아래의 plus 함수는 Int 타입 매개 변수 2개를 받아서 Int 타입 값을 반환하는 함수이다.
    - 이를 `(Int, Int) -> Int`로 표현할 수 있다.
    - 함수를 대입할 변수를 선언할 때 이러한 함수 타입을 선언하고 그에 맞는 함수를 대입해야 한다.

  ```kotlin
  fun plus(num1: Int, num2: Int): Int {
      return num1 + num2
  }
  
  // 함수 대입
  val plus: (Int, Int) -> Int = {num1: Int, num2: Int -> num1 + num2}
  ```

  - 타입 별칭
    - 함수 타입을 `typealias`를 이용해 선언할 수 있다.
    - `typealias`는 타입의 별칭을 선언하는 키워드로 함수 타입뿐만 아니라 데이터 타입을 선언할 때도 사용할 수 있다.
    - 예를 들어 정수를 표현하는 Kotlin의 타입은 Int인데, typealias를 이용해 정수를 표현하여 새로운 별칭을 선언할 수 있다.

  ```kotlin
  typealias MyInt = Int
  val num: MyInt = 1
  ```

  - 타입 별칭은 변수보다 함수 타입을 선언하는 데 주로 사용한다.
    - 아래와 같이 타입 별칭을 사용하면 함수 타입을 보다 깔끔하게 표현할 수 있다.
    - 위와 동일한 함수를 대입할 때, 타입 별칭을 사용하면 코드가 보다 깔끔해진다.

  ```kotlin
  val MyFunType = (Int, Int) -> Int
  
  val plus: MyFunType = {num1: Int, num2: Int -> num1 + num2}
  ```

  - 매개 변수 타입 생략
    - 매개 변수의 타입을 유추할 수 있는 경우에는 매개 변수의 타입을 선언하지 않아도 오류가 발생하지 않는다.
    - 예를 들어 아래의 경우 `plus` 함수를 `MyFunType` 타입으로 선언했으므로 매개 변수의 타입일 유추할 수 있어 타입을 생략해도 된다.

  ```kotlin
  val MyFunType = (Int, Int) -> Int
  
  val plus: MyFunType = {num1, num2 -> num1 + num2}
  ```

  - 타입 유추에 따른 타입 생략은 typealias를 사용할 때뿐만 아니라 타입을 유추할 수 있는 상황이면 어디서든 통한다.

  ```kotlin
  val plus: (Int, Int) -> Int = {num1, num2 -> num1 + num2}
  ```

  - 함수의 타입을 유추할 수 있다면 변수를 선언할 때 타입을 생략할 수도 있다.
    - 이 변수에 대입한 함수를 보면 타입을 유추할 수 있으므로 굳이 변수 선언부에 타입을 명시하지 않아도 된다.

  ```kotlin
  val plus = {num1: Int, num2: Int -> num1 + num2}
  ```



- 고차 함수(High order function)

  - 함수를 매개 변수로 전달 받거나 반환하는 함수를 의미한다.
    - 일반적으로 함수의 매개 변수나 반환값은 데이터지만, 데이터가 아닌 함수를 매개 변수나 반환값으로 이용하는 함수를 고차 함수라 한다.
    - 함수를 매개 변수나 반환값으로 사용할 수 있는 이유는 함수를 변수에 대입할 수 있기 때문이다.
    - 아래 예시에서 `filterStrings` 함수는 `{ it.length > 5 }`라는 람다 함수를  `condition`이라는 매개 변수로 받는 고차함수이다.

  ```kotlin
  // 함수를 매개 변수로 받는 고차 함수
  fun filterStrings(strings: List<String>, condition: (String) -> Boolean): List<String> {
      return strings.filter(condition)
  }
  
  // 함수를 반환하는 고차 함수
  fun makeLengthChecker(length: Int): (String) -> Boolean {
      return { input: String -> input.length > length }
  }
  
  fun main() {
      val words = listOf("apple", "banana", "kiwi", "grape")
      val longWords = filterStrings(words) { it.length > 5 }
      println(longWords) // [banana]
      
      val checkLongerThan5 = makeLengthChecker(5)
      val checkLongerThan2 = makeLengthChecker(2)
  
      println(checkLongerThan5("apple"))   // false (5 is not > 5)
      println(checkLongerThan5("banana"))  // true  (6 > 5)
      println(checkLongerThan2("hi"))      // false
  }
  ```

  - 일급 객체(first-class citizen, first-class object)
    - Kotlin에서 함수는 일급 객체이다.
    - 일급 객체란 프로그래밍 언어에서 특정 "무언가"를 다른 일반적인 값들과 동일한 방식으로 다룰 수 있는 경우를 의미한다.
    - Kotlin에서 함수는 일급 객체이기에 함수를 변수에 대입할 수 있고, 따라서 함수를 매개 변수나 함수의 반환 값으로 사용할 수 있는 것이다.







# 객체

- 객체(Object)란 property와 method를 가질 수 있는 복합 구조체이다.
  - Kotlin에서는 모든 것이 객체이다.
    - 변수와 값은 단지 객체의 메모리상의 주소를 가리키고 있는 것 뿐이다.
    - 따라서 다른 언어들에 있는 primitive type이 존재하지 않는다.
  - property와 method
    - property란 객체의 상태에 접근할 수 있도록 해주는 것을 의미한다.
    - method란 객체의 행동을 정의한 것으로, 객체 내부에 정의된 함수를 의미한다.
    - method는 memeber function이라고도 불린다.
    - property와 method 모두 `.`을 통해 접근한다.



- 참조를 통한 복사

  - 아래와 같이 하나의 변수를 다른 변수에 할당할 때 새로운 객체가 생성되지는 않는다.
    - msg2에 msg1을 할당하면, Hi라는 String 객체를 하나 더 생성하는 것이 아니라, msg1이 가리키고 있던 Hi를 msg2도 가리키게 된다.

  ```kotlin
  val msg1 = "Hi"
  val msg2 = msg1
  ```

  - 즉, `=`를 통한 복사는 실제 객체를 복사하는 것이 아니라, 참조를 복사하는 것이다.



- Mutability

  - 만일 한 변수에 값을 할당하고, 다른 변수에 해당 변수를 할당한 뒤, 한 변수의 값을 변경하면 어떻게 될까?
    - 이는 객체의 type에 따라 달라지게 된다.
    - 예를 들어 Kotlin에서 Int는 immutable한 type이다.
    - 따라서 아래 예시에서와 같이 값을 변경하려 하면, 새로운 객체를 생성하여 재할당한다.

  ```kotlin
  var a: Int = 100
  val anotherA: Int = a
  println(a == anotherA)  // true
  println(a === anotherA) // true
  a = 200		// 즉, a가 가리키고 있는 Int 객체의 값을 변경하는 것이 아니라, 새로운 Int 객체가 생성되고, 새로 생성된 객체의 주소를 a에 할당한다.
  println(a == anotherA)  // false
  println(a === anotherA) // false
  ```

  - 다른 언어들에서 primitive type은 일반적으로 immutable한 값이다.
    - 모든 것이 객체인 Kotlin에서도, 다른 언어의 primitive type의 특성을 가져와 일부 type을 immutable하게 만들었다.
    - 즉, number, character, boolean, string 등의 기본 type은 immutable한 객체이다.
    - 이렇게 만든 이유는 위와 같은 type들을 mutable하게 만들었을 때 다양한 문제가 생길 수 있기 때문이다.

  ```kotlin
  // 예를 들어 아래와 같은 상황은 프로그래밍에서 매우 흔한 상황이다.
  // 아래와 같은 상황에서 num이 변경됐을 때 initalNum도 함께 변경된다면 예상치 못 한 결과가 나오게 된다.
  var initalNum = 0
  var num = inital_num
  
  num++
  println(initalNum)      // 0
  println(num) 			// 1
  ```



- 동일성

  - structural equality
    - 아래 예시와 같이 서로 같은 상태인 것을 structural equality라 부른다.

  ```kotlin
  var text1 = "Hi"
  var text2 = "Hi"
  println(text1==text2)	// true
  ```

  - referential equality
    - 서로가 가리키고 있는 객체가 동일한 것을 referential equality라 부른다.
  - `===`, `!==`
    - `===`는 서로 같은 객체를 가리키고 있으면 true, 아니면 false를 반환하는 연산자이다(`!==`는 그 반대).
    - 즉 referential equality를 확인할 때 사용하는 연산자이다.





# Class

- Kotlin에서 class를 선언하는 방법

  - 다른 언어와 동일하게 `class` keyword를 사용한다.

  ```kotlin
  class Foo {
      
  }
  ```

  - Kotlin의 경우 위와 같이 body가 없는 class를 생성할 경우, 중괄호를 생략 가능하다.

  ```kotlin
  class Foo
  ```



- Instance 생성하기

  - 위에서 작성한 `Foo` class의 instance를 생성하기

  ```kotlin
  val foo: Foo = Foo()
  ```

  - Type을 지정하지 않아도 된다.

  ```kotlin
  val foo = Foo()
  ```



- Class member

  - Class member는 method와 property(혹은 field)를 모두 일컷는 말이다.
  - Property 작성하기
    - 반드시 기본값을 지정해줘야한다.
    - 주지 않을 경우 instance가 생성 될 때 값이 null이 되게 되는데 Kotlin에서는 non-nullable한 값에 null을 할당하는 것을 허용하지 않기 때문이다.

  ```kotlin
  Class Car {
      var model: String = "Unknown"
      var brand: String = "Unknown"
      var capacity: Int = 2
  }
  ```

  - Property에 접근하기

  ```kotlin
  var myCar = Car()
  println(myCar.model)		// Unknown
  println(myCar.capacity)		// 2
  ```

  - Property 변경하기

  ```kotlin
  var myCar = Car()
  myCar.model = "spark"
  myCar.brand = "KIA"
  myCar.capacity = 4
  ```




- Constructor

  - 새로운 object를 생성할 때 자동으로 호출되는 class memeber이다.
    - 새로 생성되는 object의 property들을 설정해주는 역할을 한다.
  - 모든 클래스는 constructor를 필요로 한다.
    - 아래와 같이 object를 생성하는 과정은 사실은 constructor를 호출하는 것이다.
    - 만일 따로 constructor를 정의해주지 않았을 경우, compiler는 기본 constructor를 생성한다.
    - 기본 compiler를 사용하려 할 경우, 반드시 property에 기본값을 설정해줘야한다.
  
  ```kotlin
  class Size {
      val width: Int = 1
      val height: Int = 1
  }
  
  // constructor 호출
  val size = Size()
  ```
  
  - Primary constructor
    - class와 property들을 initialize하는 데 사용하는 constructor이다.
    - 오직 property의 값을 설정하는 데만 사용할 수 있으며, 다른 code는 포함할 수 없다.
    - 아래와 같이 class 명 뒤에 괄호를 통해 초기화 하고자 하는 값을 받는다.
    - 일반적으로 constructor를 정의할 때, `constructor`라는 keyword를 class명과 괄호 사이에 넣어야 하지만, primary constructor의 경우 생략 가능하다.
  
  ```kotlin
  class Size(width: Int, height: Int) {
      val width: Int = width
      val height: Int = height
  }
  
  // constructor 키워드를 생략하지 않은 경우
  class Size constructor(width: Int, height: Int) {
      val width: Int = width
      val height: Int = height
  }
  ```
  
  - Property 선언
    - 괄호 안에 `var` 혹은 `val` 키워드를 넣어, property를 선언할 수 있다.
    - 기본값을 지정해주는 것도 가능하다.
  
  ````kotlin
  class Size(val width: Int = 1, height: Int) {
      val height: Int = height
      val area: Int = width * height
  }
  ````
  
  - Single line class
    - primary constructor에 선언된 property를 제외하고, 다른 class memeber가 없다면, 괄호를 생략 가능하다.
    - 이를 이용하여, primary constructor로 single line class를 작성하는 것이 가능하다.
    - 주로 data class를 선언하기위해 사용한다.
  
  ```kotlin
  // primary constructor에 선언된 width, height를 제외하면 다른 class member가 없다.
  class Size(val width: Int, val height: Int)
  ```
  
  - `init`
    - property의 값을 설정하는데에만 사용할 수 있는 primary constructor와는 달리, 추가적인 code를 작성할 수 있는 initializer block을 작성하는 것이 가능하다.
    - Initializer block을 생성할 때 `init` 키워드를 사용한다.
    - 하나의 class 내에 여러 개의  initializer block을 생성할 수 있다.
  
  ```kotlin
  class Size(_width: Int, _height: Int) {
      var width: Int = 0
      var height: Int = 0
  
      init {
          width = if (_width >= 0) _width else {
              println("Error, the width should be a non-negative value")
              0
          }
          height = if (_height >= 0) _height else {
              println("Error, the height should be a non-negative value")
              0
          }
      }
  }
  ```
  
  - Primary constructor의 매개 변수를 var 또는 val 키워드로 선언하면 클래스의 멤버 변수가 된다.
  
    - 원래 primary constructor의 매개 변수는 constructor 안에서만 사용할 수 있는 지역 변수이나, 이 방법을 통해 클래스의 멤버 변수로 선언이 가능하다.
    - Kotlin에서는 함수의 매개 변수를 선언할 때 var나 val을 붙일 수 없으나, 이 경우가 유일한 예외다.
  
  
  ```kotlin
  class User(val name: String, val count Int) {
      fun printUserInfo() {
          println("name: $name, count: %count")
      }
  }
  ```



- Custom constructor(Secondary constructor)

  - 하나의 class에 여러 개의 constructor를 사용할 수 있게 해준다.
    - Primary constructor와 별개로 custom constructor를 선언할 수 있다.
  - Class 내부에 `constructor` keyword를 사용하여 선언한다.

  ```kotlin
  class Size {
      var width: Int = 0
      var height: Int = 0
  
      constructor(_width: Int, _height: Int) {
          width = _width
          height = _height
      }
  }
  
  val size = Size(3, 4)
  ```

  - 만일 secondary constructor를 선언했을 경우, 암묵적 constructor(기본 constructor)를 호출할 수는 없다.
  
  ```kotlin
  class Size {
      var width: Int = 0
      var height: Int = 0
  
      constructor(_width: Int, _height: Int) {
          width = _width
          height = _height
      }
  }
  
  // 이렇게 암묵적 construcotor를 호출할 수는 없다.
  val size = Size()
  
  
  // 만일 위와 같이 사용해야 한다면, 아래와 같이 빈 constructor를 선언해야 한다.
  // 아래와 같이 primary constructor를 선언하거나
  class Size() { 
      var width: Int = 0
      var height: Int = 0
  }
  
  // 또 다른 secondary constructor를 선언한다.
  class Size {
      var width: Int = 0
      var height: Int = 0
  
      constructor() {
      }
  }
  ```
  
  - 여러 개의 constructor 선언하기
    - 어떤 constructor가 선언될지는 parameter에 따라 달라지게 된다.

  ```kotlin
  class Size {
      var width: Int = 0
      var height: Int = 0
  
      constructor(_height: Int) {
          height = _height
      }
  
      constructor(_width: Int, _height: Int) {
          width = _width
          height = _height
      }
  
      constructor(_width: Int, _height: Double) {
          width = _width
          height = _height.toInt()
      }
  
      constructor(_height: Double, _width: Int) {
          width = _width
          height = _height.toInt()
      }
  }
  ```
  
  - 여러개의 constructor가 선언됐을 경우, compiler는 각 constructor들의 argument name이 아니라 argument의 type으로 어떤 constructor가 실행될지를 판단한다.
    - 즉, 아래 두 개의 constructor는, 비록 argument의 이름이 다를지라도, compiler가 보기에는 동일한 constructor이며, error가 발생하게 된다.
  
  ```kotlin
  constructor(width: Int, height: Int) {}
  constructor(x: Int, y: Int) {}
  ```

  - `this` keyword를 반드시 사용해야 하는 경우
    - 현재 object를 가리키는 `this` keyword는 생략이 가능하다.
    - 그럼에도 반드시 사용해야 하는 경우가 있는데, 아래와 같이 constructor의 parameter의 값과 property의 이름이 동일한 경우에는 property에 반드시 `this`를 붙여줘야 한다.
  
  ```kotlin
  class Size {
      var width: Int = 0
      var height: Int = 0
  
      constructor(width: Int, height: Int) {
          this.width = width
          this.height = height
      }
  }
  ```
  
  - Constructor의 역할은 초기화지 재할당이 아니다.
    - 아래 예시에서 `val` keyword로 선언된 property들은 재할당이 불가능하다.
    - 그러나 constructor를 통해 값을 할당할 수 있는데, 이는 변수의 초기화지 재할당이 아니기 때문이다.
  
  ```kotlin
  class Size {
      val width: Int
      val height: Int
      val area: Int
  
      constructor(width: Int, height: Int) {
          this.width = width
          this.height = height
          this.area = width * height
      }
  }
  ```
  
  - Custom constructor의 parameter에 `val`, `var`를 붙이는 방식으로 property를 선언할 수는 없다.
  
  ```kotlin
  class Size {
      // 아래와 같이 하는 것은 불가능하다.
      constructor(val width: Int, val height: Int) {
      }
  }
  ```
  
  - Constructor delegation
    - 만일 primary constructor가 존재할 경우, secondary constructor는 반드시 primary constructor를 호출해야한다.
    - 이를 delegation이라 한다.
    - `this`를 사용하여 primary constructor를 호출할 수 있다.
    - Secondary constructor의 가장 첫 번째 문이 되므로, secondary constructor가 실행되기 전에 delegation이 먼저 실행된다.
    - 만일 primary constructor가 없을 경우, delegation은 암묵적으로 발생한다.
  
  ```kotlin
  class Size(val width: Int, val height: Int) {
      var area: Int = width * height
  	// secondary construcotr에서 primary constructor를 호출하려면, 아래와 같이 this를 사용한다.
      constructor(width: Int, height: Int, outerSize: Size) : this(width, height) {
          outerSize.area -= this.area
          println("Updated outer object's area is equal to ${outerSize.area}")
      }
  }
  ```
  
  - Secondary constructor의 실행 순서
    - 아래와 같은 code가 있을 경우, Primary constructor가 가장 먼저 실행 되고, `init` block이 실행된 후, secondary constructor가 실행된다.
  
  ```kotlin
  class Size(val width: Int, val height: Int) {
      var area: Int = width * height
  
      init {
          println("Object with area equal to $area is created")
      }
  
      constructor(width: Int, height: Int, outerSize: Size) : this(width, height) {
          outerSize.area -= this.area
          println("Updated outer object's area is equal to ${outerSize.area}")
      }
  }
  
  fun main() {
      val obj = Size(2, 3, outerObject)
  }
  ```






- Member function

  - class 내부에 있는 함수를 member function이라 부른다.
    - 일반적으로는 method라 불리지만 kotlin에서는 member function이라 부른다.
    - member function은 아래와 같이 property에 접근이 가능하다.
    - `this`는 class의 instance 그 자체를 의미하며, optional한 keyword로 빼도 된다.

  ```kotlin
  class MyClass(var property: Int) {
      fun printProperty() {
          println(this.property)
      }
  }
  ```

  - Member function을 호출하기 위해서는 member function이 정의된 class의 instance를 생성해야 한다.

  ```kotlin
  val myObject = MyClass(10)
  myObject.printProperty()
  ```

  - 만일 같은 이름의 member function이 정의될 경우 compile에 실패한다.



- Extension function

  - 이미 존재하는 class를 확장하여 사용할 수 있게 해주는 기능이다.
    - Programmning을 하다보면 다른 사람이 작성한 class를 사용해야 할 때가 있다.
    - 문제는 만일 해당 class에 원하는 기능이 없다면, 해당 기능을 추가해야 한다는 점이다.
    - Kotlin은 이런 상황을 위한 syntactic sugar를 가지고 있다.

  ```kotlin
  fun <className>.<methodName>()[: returnType] {
      // body
  }
  ```

  - 예시
    - 예를 들어 String class에는 `repeated`이라는 member function이 존재하지 않는다.
    - 만일 우리가 String class에 `repeated` 메서드를 추가하고 싶다면 아래와 같이 하면 된다.

  ```kotlin
  fun String.repeated(): String = this + this
  
  "ha".repeated()
  ```

  - 확장할 class를 receiver type이라 부르고, 확장 한 class로 생성한 instance를 receiver object라 부른다.

  ```kotlin
  class Client(val name: String, val age: Int)
  
  fun Client.getInfo() = "$name $age" // Client는 receiver type
  
  
  val client = Client("John", 32)
  print(client.getInfo()) // client는 receiver object이다.
  ```

  - Member function과의 차이
    - 만일 class에 숨겨진 member가 있다면, extension function은 해당 member에 까지는 접근할 수 없다.
    - 그러나, member function은 모든 member에 제한 없이 접근 가능하다.
    - 또한, 만일 member function과 동일한 이름을 가진 extension function을 정의하여 호출하더라도, member function이 실행되고, extension function은 무시된다.
    - 그러나, 만일 이름이 동일하더라도, parameter가 다르다면, 다른 argument에 따라 각기 다른 함수를 호출할 수 있다.



- Getter와 setter
  - Object 내의 data를 가져오거나, data를 설정하는 데 사용되는 메서드를 getter, setter라 부른다.
    - 이를 사용하여 보다 안전하게 data의 조회 및 수정이 가능하다.
  - Property getter
    - Kotlin에서는 getter라 불리는 특별 함수를 정의할 수 있다.
    - `get`이라는 이름으로 함수를 선언하면 되는데, `get`함수는 인자로 아무 것도 받지 않고, `field`라는 keyword를 반환한다.
    - `field`는 Kotlin의 모든 property가 가지는 backing field로, property의 값을 담고 있다.
    - 만일 따로 정의해주지 않을 경우, Kotlin에서 자동으로 아래와 같이 생성해주며, 추가해야 할 code가 있을 경우에만 직접 선언해서 사용하면 된다.

  ```kotlin
  class Client {
      val name = "Unknown"
          get() {
              return field
          }
  }
  
  // 아래와 같이 괄호를 생략하는 것도 가능하다.
  class Client {
      val name = "Unknown"
          get() = field
  }
  
  
  println(client.name)	// Unknown
  ```

  - Custom getter
    - 만일 추가해야 할 code가 있을 경우 아래와 같이 `get` 함수 내부에 추가해주면 된다.
    - `field`는 read-only로, 변경이 불가능하다.

  ```kotlin
  class Client {
      val name = "Unknown"
          get() {
              if (field == "Unknown") {
                  return "Foo"
              } else {
                  return field
              }
          }
  }
  
  val client = Client()
  println(client.name)	// foo
  ```

  - Property setter
    - `get` 함수와 마찬가지로, setter라 불리는 `set` 함수를 선언하여 property를 명시적으로 수정하도록 할 수 있다.
    - `get`과 달리 하나의 arguement를 받으며, 반환값은 없다. argument의 이름은 `value`로 하는 것이 관례다.
    - 이 역시 따로 선언하지 않을 경우, Kotlin이 자동으로 선언해주며, 추가해야 할 code가 있을 경우에만 직접 선언해서 사용하면 된다.

  ```kotlin
  class Client {
      var name = "Unknown"
          set(value) {
              field = value
          }
  }
  ```

  - Custom setter
    - 만일 추가해야 할 code가 있을 경우 아래와 같이 `set` 함수 내부에 추가해주면 된다.

  ```kotlin
  class Client {
      var name = "Unknown"
      var age = 18
          set(value) {                      
              field = if (value < 0) {
                  println("Age cannot be negative. Set to $defaultAge")
                  defaultAge
              } else
                  value
          }
      val defaultAge = 18
  }
  
  val client = Client()
  client.age = -1      // Age cannot be negative. Set to 18.
  println(client.age)  // 18
  ```

  - Constructor와 함께 사용할 경우 아래와 같이 사용하면 된다.
    - Property를 다른 property에 할당한 후 해당 property에 `get` 또는 `set` 함수를 선언한다.

  ```kotlin
  class Client(name: String, age: Int) {
      var fullName: String = name
          set(value) {
              println("The name is changing. Old value is $field. New value is $value.")
              field = value
          }
      // 아래 age는 위 age와 다른 새로운 property다.
      var age: Int = age
          set(value) {
              println("The age is changing. Old value is $field. New value is $value.")
              field = value
          }
  }
  ```

  - Instance가 initializing 될 때는 setter가 실행되지 않는다.

  ```kotlin
  class Client(name: String) {
      var name: String = name
          set(value) {
              println("The name is changing. Old value is $field. New value is $value.")
              field = value
          }
  }
  
  val client = Client("Annie")  // set 함수는 실행되지 않는다.
  client.name = "Ann"           // The name is changing. Old value is Annie. New value is Ann.
  ```

  - `val`로 선언한 변수에는 setter를 사용할 수 없다.
    - 단, val property 내부의 property는 수정이 가능하다.

  ```kotlin
  class Passport(number: String) {
      var number = number
      set(value) {
          println("Passport number has changed.")
          field = value
      }
  }
  
  class Client {
      val passport = Passport("1234567")
  }
  
  val client = Client()
  println(client.passport.number)       // 1234567
  /*
  client.passport = Passport("2345678") // 수정할 수 없다.
  */
  client.passport.number = "2345678"    // val property 내부의 property는 수정이 가능하다.
  println(client.passport.number)       // 2345678
  ```



- 상속

  - Kotlin의 상속 문법을 위해서는 상속 할 class 앞에 `open` keyword를 붙여야한다.
    - Kotlin에서 모든 class와 method는 기본적으로 final로 선언된다.
    - 따라서 class를 상속 가능하게 하기 위해서는 아래와 같이 `open` keyword를 붙여야한다.

  ```kotlin
  open class Parent {}
  ```

  - Kotlin의 상속 문법은 아래와 같다.
    - `:` 뒤에 상속 받을 클래스의 이름을 입력한다.

  ```kotlin
  class Child: Parent() {}
  ```

  - 만약 상속 받을 클래스의 constructor에 매개 변수가 있다면, 매개 변수를 전달해야한다.

  ```kotlin
  open class Parent(name: String) {}
  
  class Child(name: String): Super(name) {}
  ```



- 오버라이딩

  - 부모 클래스의 메서드를 오버라이딩 하기 위해서는 오버라이딩 할 메서드에 `open` keyword를 붙여야한다.
    - 앞서 말했듯 모든 메서드는 기본적으로 final로 선언된다.
    - 따라서 메서드를 오버라이딩 하기 위해서는 `open` keyword를 붙여야한다.
    - 메서드 뿐 아니라 변수의 오버라이딩 역시 마찬가지다.
  - 오버라이딩하여 새로 작성한 메서드에는 `override` keyword를 붙여야한다.

  ```kotlin
  open class Parent() {
      open var name = "John"
      open fun sayHello() {
          println("Hi, I'm $name. I'm a Parent")
      }
  }
  
  class Child: Parent() {
      override var name = "Mike"
      override fun sayHello() {
          println("Hi, I'm $name. I'm a Child")
      }
  }
  ```



- 접근제한자(visibility modifier)

  - 클래스의 멤버에 접근할 수 있는 대상을 제한하는 keyword이다.
    - 접근 제한자를 생략할 경우 public이 된다.

  | 접근제한자 | 최상위에서 사용       | 클래스 멤버에서 사용               |
  | ---------- | --------------------- | ---------------------------------- |
  | public     | 모든 파일에서 가능    | 모든 클래스에서 가능               |
  | internal   | 같은 모듈 내에서 가능 | 같은 모듈 내에서 가능              |
  | protected  | 사용 불가             | 상속 관계의 하위 클래스에서만 가능 |
  | private    | 파일 내부에서만 가능  | 클래스 내부에서만 가능             |

  - 예시

  ```kotlin
  open class Super {
      var public_num = 1
      protected var protected_num = 2
      private var private_num = 3
  }
  
  class Sub: Super() {
      fun test() {
          public_num++
          protected_num++
          private_num++		// error
      }
  }
  ```




- Enum

  - 상수들의 논리적 집합이다.
    - 상수들을 관리하기 위해 사용한다.
  - 기본 Enum
    - `enum` keyword를 사용하여 enum class를 생성할 수 있다.
    - Naming convention은 undersocre로 구분된 대분자를 사용하거나(`MY_CONSTANT`), 파스칼 케이스를 사용하는 것이다(`MyConstant`).

  ```kotlin
  enum class Animal {
      DOG, CAT, TIGER, LION, ELEPHANT
  }
  ```

  - 값 부여하기
    - enum class 내부의 각 요소들은 enum class의 인스턴스이기에, constructor에 값을 줌으로써 초기화가 가능하다.
    - 마지막 요소 뒤에는 `;`을 붙여야한다.

  ```kotlin
  enum class Animal(val age: Int) {
      DOG(3), CAT(2), TIGER(6), LION(3), ELEPHANT(5);
  }
  ```

  - 값에 접근하기
    - 아래와 같이 접근이 가능하다.

  ```kotlin
  val animalAge = Animal.DOG.age
  ```

  - 메서드 추가하기

  ```kotlin
  enum class Animal(val age: Int) {
      DOG(3), CAT(2), TIGER(6), LION(3), ELEPHANT(5);
      
      fun printAge() {
          println(age)
      }
  }
  
  dog = Animal.DOG
  dog.printAge()		// 3
  ```

  - 인스턴스의 이름에 접근하기

  ```kotlin
  val cat: Animal = Animal.CAT
  println(cat.name)		// CAT
  ```

  - 몇 번째 인스턴스인지 확인하기

  ```kotlin
  val tiger: Animal = Animal.TIGER
  println(tiger.ordinal)		// 2
  ```

  - Instance들을 순회하기
    - `values()` 메서드는 enum class의 instance들의 배열을 반환한다.

  ```kotlin
  for (enum in Animal.values()) {
      println(enum.age)
  }
  ```



- Data class

  - 자주 사용하는 데이터를 객체로 묶게 해주는 클래스이다.
    - VO(value-object) 클래스를 편리하게 이용할 수 있게 해준다.
    - `data` keyword로 선언한다.
    - 데이터 클래스는 데이터를 다루는 데 편리한 기능을 제공하는 것이 주목적이므로 주 생성자에 val, var 키워드로 매개변수를 선언해 클래스의 멤버 변수로 활용하는 것이 일반적이다.

  ```kotlin
  data class UserInfo(val name: String, val email: String, val age: Int)
  ```

  - `equals()`
    - 객체의 데이터를 비교하는 함수이다.
    - 객체가 같은지가 아니라 객체의 데이터가 같은지를 비교할 때 사용한다.
    - 주 생성자에 선언한 멤버 변수의 데이터만 비교 대상으로 삼는다.

  ```kotlin
  val user_info1 = UserInfo("foo", "foo@gmail.com", 28)
  val user_info2 = UserInfo("foo", "foo@gmail.com", 28)
  
  println("${user_info1.equals(user_info2)}")		// true
  ```

  - `toString()`
    - 객체의 데이터를 반환하는 함수이다.
    - 일반 클래스에도 사용이 가능하지만, 데이터 클래스에 사용할 경우 객체가 포함하는 멤버 변수의 데이터를 출력한다.
    - 주 생성자의 매개 변수에 선언된 데이터만 출력된다.

  ```kotlin
  val user_info = UserInfo("foo", "foo@gmail.com", 28)
  println("${data.toString()}")		// UserInfo(name="foo", emaol="foo@gmail.com", age=28)
  ```



- 오브젝트 클래스

  - 익명 클래스(anonymous class)를 만들 목적으로 사용하는 클래스이다.
    - 익명 클래스는 문자 그대로 이름이 없는 클래스를 말한다.
    - 클래스의 이름이 없으므로 클래스를 선언하면서 동시에 객체를 생성해야 한다.
    - `object` keyword를 사용하여 선언한다.

  ```kotlin
  val obj = object {
      var num = 10
      fun sayHello() {
          println("Hello World!")
      }
  }
  ```

  - 단, 위에서 생성한 객체의 멤버에 접근하려고 하면 에러가 발생한다.
    - 이는 객체의 타입 때문으로, 오브젝트 클래스를 선언할 때 타입을 지정하지 않았으므로 이렇게 생성된 객체는 최상위 타입은 `Any`로 취급된다.
    - 그런데 `Any`에는 `num`, `sayHello()` 등의 멤버가 없으므로 에러가 발생하게 된다.

  ```kotlin
  println(obj.num)	// error
  obj.sayHello()		// error
  ```

  - 따라서 일반적으로는 아래와 같이 타입을 지정해준다.

  ```kotlin
  open class Super {
      open var num = 10
      open fun sayHello() {
          println("Hello World!")
      }
  }
  
  val obj = object: Super() {
      override var num = 10
      override fun sayHello() {
          println("Hello World!")
      }
  }
  ```



- 컴패니언 클래스

  - 클래스의 멤버 변수나 메서드에 객체가 아닌 클래스를 통해 접근하고자 할 때 사용한다.
    - 컴패니언 클래스는 객체를 생성하지 않고도 클래스를 통해 멤버에 접근이 가능하다.
    - 클래스 내부에서 `companion object`를 사용하여 선안하면, 이 클래스를 감싸는 클래스를 통해 멤버에 접근이 가능하다.

  ```kotlin
  class CompanionClass {
      companion object {
          var num = 10
          fun sayHello() {
              println("Hello World!")
          }
      }
  }
  
  
  fun main() {
      CompanionClass.num = 20
      CompanionClass.sayHello()
  }
  ```

  - Java의 static을 대체한다고 생각하면 된다.
    - Kotlin은 Java와 달리 static keyword를 지원하지 않는다.
    - 따라서 컴패니언 클래스는 결과적으로 Java의 static을 대체한다.