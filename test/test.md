# Unit Test

- 단위 테스트
  - 위키피디아는 단위 테스트를 아래와 같이 정의한다.
    - 컴퓨터 프로그래밍에서 예제 코드의 특정 모듈이 의도된 대로 정확히 작동하는지 검증하는 절차다.
    - 즉 모든 함수와 메서드에 대한 테스트 케이스를 작성하는 절차를 말한다.
    - 이를 통해 언제라도 코드 변경으로 인한 문제가 발생할 경우, 단시간 내에 이를 파악하고 바로잡을 수 있도록 해준다.
  - 켄트 벡이 smalltalk에서 단위 테스트 개념을 도입했으며, 이 개념이 다른 프로그래밍 언어로 확산되었다.
  - 테스트를 작성할 대상은 테스트 중인 주제(subject), 시스템(system), 테스트 대상(Suite Under Test, SUT)이다.
    - 무언가를 테스트할 때 테스트하고자 하는 대상을 SUT이라 한다.
    - SUT은 테스트 중인 주제, 시스템, 테스트의 모음(suite)를 의미하며, 일부 사람들은 테스트 중인 컴포넌트(component), 클래스(class), 코드(code)를 의미하는 CUT(component, class, code under test)라는 용어를 사용하기도 한다. 
  - 단위 테스트의 "단위(unit)"의 의미
    - 단위는 시스템 내 작업 단위(unit of work) 또는 사용 사례(use case)를 의미한다.
    - 작업 단위는 함수 하나만 의미할 수도 있고, 여러 함수를 의미할 수도 있다.
    - 함수뿐 아니라 모듈이나 컴포넌트도 작업 단위가 될 수 있다.



- 진입접과 종료점

  - 작업 단위에는 시작과 끝이 있으며 이를 진입접(entry point)과 종료점(exit point)이라고 한다.
  - 작업 단위는 진입점의 호출부터 하나 이상의 종료점까지 눈에 띄는 결과가 나타날 때까지 발생하는 모든 작업을 의미한다.
    - 작업 단위에는 항상 하나의 진입점과 하나 이상의 종료점이 있다.
    - 진입점은 어떤 것이 되어도 상관 없으며 설정하기 나름이다.
  - 종료점
    - 코드가 실행되면 무언가 의미 있는 작업을 하기 마련이다.
    - 여기서 의미 있는 작업이란 단순히 값을 반환하는 것부터 시작하여 어떤 상태를 변경하거나 서드 파티 코드를 호출하는 등 뭔가 눈에 띄는 동작을 의미한다.
    - 이런 동작을 종료점이라고 부른다.
    - 종료점은 작업 단위의 최종 결과를 의미한다.
  - 진입점과 종료점 예시
    - 아래 예시에서 sum() 함수는 하나의 완전한 작업 단위이다.
    - 함수는 진입점이며 그 결과로 두 숫자의 합을 반환하기 때문에 종료점 역할도 한다.
    - 함수(작업 단위)를 실행한 동일한 위치에서 반환 값도 얻으므로 진입점이 종료점이기도 하다.

  ```python
  def sum(numbers):
      a, b = numbers.split(",")
      result = int(a) + int(b)
      return result
  ```

  - 복수의 종료점이 있는 경우
    - `sum()` 함수는 아래 두 개의 종료점을 가지고 있다.
    - 숫자의 합을 반환
    - 숫자의 합을 total에 추가하는 누적 합계를 계산

  ```python
  total = 0
  
  def sum(numbers):
      global total
      a, b = numbers.split(",")
      result = int(a) + int(b)
      total += result
      return result
  ```

  - 여러 개의 종료점이 있을 때 각 종료점은 개별적인 테스트 코드로 바꿀 수 있다.
    - 여러 개의 종료점이 있을 때 각 종료점은 실제로 코드가 수행해야 하는 중요한 작업을 의미한다.
    - 따라서 같은 작업 단위에서 나오는 서로 다른 경로나 요구 사항으로 보이기도 한다.
  - 진입점과 종료점은 테스트 코드 범위를 설정하는 데 도움을 준다.
    - 진입점은 항상 테스트가 작업 단위를 실행하는 지점이다.
    - 작업 단위에는 여러 개의 종료점이 있을 수 있으며, 각 종료점은 서로 다른 테스트 코드에서 사용할 수 있다.
    - 종료점을 기준으로 종료점마다 테스트를 만들어 분리하면 각 테스트끼리 영향을 주지 않고, 더 읽기 쉬우며, 디버깅하기도 쉽다.
  - 종료점 추가하기
    - 기존 두 개의 종료점에 숫자를 출력한다는 새로운 종료점을 추가한다.
    - 종료점이 추가된다는 것은 의미 있는 작업이 추가된다는 것이다.
    - `print()` 함수는 외부에 무언가를 기록하며, 파일에 기록될 수도, 콘솔에 기록될 수도 있다.
    - 이는 의존성 호출에 해당된다.

  ```python
  total = 0
  
  def sum(numbers):
      global total
      a, b = numbers.split(",")
      print("first_num:", a, "second_num", b)
      result = int(a) + int(b)
      total += result
      return result
  ```

  - 의존성 호출
    - 의존성은 단위 테스트 중 온전히 제어할 수 없는 것을 의미한다.
    - 그 종류는 무수히 많지만 아래와 같은 것들이 있다.
    - 파일에 무언가를 기록하는 경우.
    - 네트워크와 통신하는 경우.
    - 다른 팀이 관리하는 코드일 경우.
    - DB에 접근하는 행위일 경우.
    - 오래 걸리는 계산 작업일 경우.
  - 종료점의 유형
    - 마지막으로 살펴본 코드는 세 유형의 종료점을 가지고 있다.
    - 함수가 undefined가 아닌 값을 반환한다.
    - 함수 호출 전과 후에 상태(result) 값이 달라진다.
    - 코드의 실행 주도권이 없는 함수(print)를 호출한다.
  - 종료점의 유형에 따라 테스트 방법이 다소 다르다.
    - 반환 값이 있는 종료점은 가장 테스트하기 쉬운 유형으로, 작업 단위를 실행하여 진입점을 호출하고 실행 결과를 받아 그 값을 확인하면 된다.
    - 상태 값을 변경하는 종료점은 의도한대로 상태 값이 변경되었는지를 확인해야한다.
    - 서드 파티(실행 주도권이 없는 함수)를 호출하는 종료점은 모의 객체를 만들어 테스트 결과를 임의로 조작하여 테스트해야한다.



- 테스트 코드 작성하기

  - 이전에 살펴본 아래 코드의 테스트 코드를 작성할 것이다.
    - 세 가지 종료점의 유형중 반환 값이 있는 종료점에 대해서만 작성할 것이다.

  ```python
  total = 0
  
  def sum(numbers):
      global total
      a, b = numbers.split(",")
      print("first_num:", a, "second_num", b)
      result = int(a) + int(b)
      total += result
      return result
  ```

  - 위 코드를 테스트 하기 위한 코드의 진입점은 sum(numbers) 함수다.
    - 일반적으로 함수의 시그니처가 함수의 진입점이 된다. 
    - sum 함수는 반환 값이 있어 진입점이자 동시에 종료점이기도 하다.

  - sum 함수를 테스트하기 위한 테스트 코드 작성하기
    - 아래 코드는 SUT인 sum 함수를 호출한 후 반환된 값을 확인한다.
    - 함수의 실행 결과가 기댓값과 다르면, 오류 정보를 출력한다.
    - 또 테스트 코드는 발생하는 모든 예외를 출력하여 이후 다른 메서드의 실행에 지장이 없도록 한다(테스트 프레임워크를 사용하면 보통 프레임워크가 자동으로 처리한다).
    - 아래 코드는 다양한 입력 값이 주어질 수 있는 여러 경우를 고려하지 않았다는 문제가 있다.

  ```python
  def parser_test():
      try:
          result = sum("1,2")
          if result == 3:
              print("parser_test example 1 PASSED")
          else:
              raise Exception(f"parser_test: expected 3 but was {result}")
      except Exception as e:
          print(e)
  ```

  - 테스트 코드 개선하기
    - 범용적인 검증 함수를 만들어 오류가 일관되게 처리되도록 수정한다.
    - `assert_equals`은 인자로 받은 두 값이 같은지를 검증하고, `check`는 테스트 이름과 콜백 함수를 인자로 받아 테스트 중에 발생하는 오류를 잡아낸다.
    - 이처럼 몇 가지 함수를 추가하면 테스트가 훨씬 읽기 쉽고 테스트 생산성도 올라가게 된다.

  ```python
  def assert_equals(expected, actual):
      if expected != actual:
          raise Exception(f"Expected {expected} but was {actual}")
  
  def check(name, implementation):
      try:
          implementation()
          print(f"{name} passed")
      except Exception as e:
          print(f"{name} FAILED", e)
          
  check("sum with 2 numbers should sum them up", lambda :assert_equals(5, sum("2,3")))
  check("sum with 2 numbers should sum them up", lambda :assert_equals(8, sum("3,6")))
  ```



- 좋은 단위 테스트
  - 좋은 테스트는 다음 항목을 만족해야 한다.
    - 테스트 작성자의 의도를 이해하기 쉬워야 한다.
    - 읽고 쓰기 쉬워야 한다.
    - 테스트를 자동화할 수 있어야 한다.
    - 같은 조건에서 실행 결과는 항상 같아야 한다.
    - 의미 있는 테스트여야 한다.
    - 구체적인 결과를 제공하여 문제를 쉽게 파악할 수 있어야 한다.
    - 누구나 쉽게 실행할 수 있어야 한다.
    - 실패할 경우 무엇이 잘못되었는지 쉽게 알 수 있어야 한다.
  - 그 중에서도 좋은 단위 테스트는 아래와 같은 특징이 있다.
    - 빠르게 실행된다.
    - 테스트 환경을 일관되게 유지하고, 테스트 결과가 항상 예측 가능하다
    - 다른 테스트와 완전히 독립적으로 실행된다.
    - 시스템 파일, 네트워크, 데이터베이스가 없어도 메모리 내에서 실행이 가능하다.
    - 가능한 한 동기적인 흐름으로 실행된다(가능하면 병렬 스레드를 사용하지 않는다).
  - 항상 모든 조건을 만족해야 하는 것은 아니다.
    - 모든 테스트가 좋은 단위 테스트의 특성을 전부 만족하는 것은 사실 불가능에 가깝다.
    - 단위 테스트 조건을 만족하기 까다로운 테스트는 통합 테스트로 만드는 것도 하나의 방법이다.
  - 의존성을 스텁으로 대체하기
    - 스텁은 실제 의존성을 흉내내는 가짜 의존성이다.
    - 스텁을 사용하면 실제 의존성에 접근하지 않고도 테스트를 수행할 수 있어 테스트를 더 예측 가능하고 안정적으로 진행할 수 있다.
  - 비동기 처리를 동기적 테스트로 흉내 내기
    - 많은 언어에서 비동기 처리가 표준이 되었다.
    - 그럼에도 테스트에서는 비동기 코드를 동기적인 방식으로 검증할 수 있다.
    - 비동기 코드를 동기적인 방식으로 검증한다는 것은 테스트에서 직접 콜백 함수를 호출하거나 비동기 작업이 완료될 때까지 기다린다는 의미다.



- 단위 테스트 체크리스트
  - 오래 전에 작성한 테스트가 여전히 잘 돌아가는가?
    - 코드를 변경한 후 이전에 잘 돌아가던 테스트를 실행할 수 없게 되면 코드에 버그가 생겨도 알아차리지 못할 수 잇다.
    - 이를 회귀(regression)라고 한다.
    - 회귀는 한때 작동하던 하나 이상의 작업 단위가 이제는 작동하지 않는 것이다.
  - 내가 작성한 테스트를 다른 팀원이 실행했을 때 문제 없이 결과를 받을 수 있는가?
    - 오래된 레거시 코드를 수정하는 것에 부담을 느끼는 이유는 코드를 바꾸면 다른 코드에 어떤 형태로 영향을 주는지 분명히 알 수 없기 때문이다.
    - 단위 테스트는 낯선 코드와 마주칠 때 두려움을 덜어준다.
  - 테스트가 수분 내로 전부 실행되는가?
    - 단 몇 초 빨라진다고 해도 테스트는 빨라져야한다.
    - 테스트를 빠르게 실행할 수 없다면 결국 자주 돌려 보지 않게 된다.
  - 간단하게 모든 테스트를 실행할 수 있는가?
    - 테스트가 완전 자동화되지 않으면 테스트를 실행하는 것이 점점 귀찮아지게 된다.
    - 좋은 테스트는 수작업으로 설정할 필요 없이 바로 실행할 수 있어야 한다.
  - 테스트 코드가 외부 의존성에서 독립적인가?
    - 아래와 같은 질문을 통해 외부 의존성에 독립적인지를 판단할 수 있다.
    - 다른 팀 코드에 버그가 있어도 내 테스트는 통과하는가?
    - 내 테스트는 다른 실행환경에서 실행해도 동일한 결과를 보장하는가?
    - DB나 network, 배포 없이도 테스트가 동작하는가?
    - 테스트 결과가 항상 동일한 이유는 시스템에 대한 간접 입력을 우리가 제어할 수 있기 때문이다.
    - 간접 입력이란 외부 의존성을 의미한다.
    - 우리는 실제가 아닌 테스트의 목적에 부합하는 가짜 의존성을 조작해서 사용할 수 있다.
  - 하나의 테스트에 변경 사항이 생겨도 다른 테스트는 영향을 받지 않는가?
    - 테스트 사이에 의존성이 있어선 안 된다.



- 통합 테스트
  - 위에서 살펴본 좋은 단위 테스트의 조건 중 하나라도 충족하지 못하는 모든 테스트를 의미한다.
    - 예를 들어 테스트가 실제 network, 실제 REST API, 실제 시스템 시간, 실제 파일 시스템 또는 실제 DB를 사용한다면 이는 통합 테스트에 해당한다.
    - 예를 들어 테스트 코드에서 현재 시간을 가져오는 `newDate()`를 사용하면 매번 테스트를 실행할 때 마다 다른 시간이 적용되므로 본질적으로는 다른 테스트가 된다.
  - 통합 테스트가 나쁘다는 것은 아니다.
    - 통합 테스트는 단위 테스트에서 검증하지 못하는 시스템 전체의 상호 작용을 확인할 수 있는 중요한 역할을 한다.
    - 하지만 단위 테스트와 분리는 되어야한다.
  - 통합 테스트의 문제는 한 번에 너무 많이 테스트한다는 것이다.
    - 개발자 대부분은 앱의 전체 기능, REST API의 응답 결과, UI의 동작을 통해 기능을 테스트한다.
    - 이는 단일 기능이나 모듈이 아니라 여러 구성 요소가 함께 작동하여 최종 결과를 만들어 내는 방식을 테스트하는 것이다.
    - 테스트가 실패하면 이 소프트웨어 구성 요소 중 어느 부분이 잘못되어 실패한 것인지 파악하기 어려울 수 있다.
    - 결국 통합 테스트는 다른 팀이 만든 모듈, 외부 API나 서비스, 네트워크, DB, 스레드 등 실제 의존성을 완전히 제어할 수 없는 상태에서 작업 단위를 테스트하는 것이다.
  - 요약하자면 통합 테스트와 단위 테스트의 차이는 아래와 같다.
    - 통합 테스트는 실제 의존성을 사용한다. 
    - 단위 테스트는 작업 단위를 의존성에서 격리시켜 항상 일관된 결과를 받을 수 있도록 하여 작업 단위의 모든 측면을 쉽게 조작할 수 있게 한다.



- 단위 테스트의 정의
  - 위에서 살펴본 내용들을 토대로 내린 단위 테스트의 정의는 아래와 같다.
    - 진입점을 통해 작업 단위를 호출한 후 그 종료점을 확인하는 자동화된 코드다.
    - 잘 작성된 단위 테스트는 신뢰성이 높고 가독성이 좋아 유지보수하기에 용이하다.
    - 운영하는 코드가 변경되지 않는 한 동일한 결과를 보장한다.
  - 단위 테스트의 대상은 제어 흐름이 포한된 코드이다.
    - 예를 들어 게터와 세터는 보통 조건문이나 계산 같은 논리는 포함하지 않기 때문에 테스트할 필요가 없다.
    - 이러한 코드는 테스트하고자 하는 작업 단위에서 사용될 수는 있지만 직접 테스트할 필요는 없다.
    - 하지만 게터나 세터에 조건문, 반복문, 계산, 데이터 변환 같은 논리를 추가하면 그 논리가 제대로 동작하는지 확인해야 한다는 점에 주의해야한다.



- 테스트 주도 개발(TDD)
  - 기능을 개발하기 전에 단위 테스트를 먼저 작성하는 개발 방식이다.
    - 먼저 실패하는 테스트 케이스를 작성하고, 해당 테스트를 통과할 수 있는 코드를 작성한다.
    - 테스트가 통과되는 것을 확인한 후 필요에 따라 리팩터링을 진행하고, 다음 테스트를 작성한다.
  - TDD는 단위 테스트의 대체재가 아니다.
    - 코드 작성 후 단위 테스트를 작성하면 보통은 코드가 잘 돌아가고 있을 때만 테스트가 통과하는지 확인한다.
    - 반면 TDD를 하면 테스트가 실패할 때와 통과하는 과정 모두를 겪는다.
    - 이 과정에서 개발자는 테스트가 정확히 작동하는지 확인할 수 있어 디버깅을 덜 하게 된다.
  - TDD를 잘 하는 세 가지 핵심 기법
    - 좋은 테스트를 작성하는 방법을 안다.
    - 코드보다 테스트를 먼저 작성한다.
    - 테스트와 프로덕션 코드를 자 설계한다.



- 단위 테스트 프레임워크
  - 일반적으로 단위 테스트 프레임워크는 아래와 같은 역할을 한다.
    - 테스트를 작성할 때 사용하는 테스트 라이브러리 역할.
    - 테스트 내에서 테스트 결과를 검증(assertion)하는 라이브러리 역할.
    - 테스트를 실행하는 테스트 러너 역할.
    - 테스트 실행 결과를 보여주는 테스트 리포터 역할.
  - 테스트 코드를 직접 작성하지 않고, 단위 테스트 프레임워크를 사용하는 것은 아래와 같은 이점을 제공한다.
    - 테스트 코드를 일관된 형식으로 작성할 수 있다.
    - 새로운 테스트를 작성하는 작업을 쉽게 반복할 수 있다.
    - 보다 신뢰할 수 있는 테스트를 작성할 수 있다.
    - 테스트 코드를 작성하는 시간을 절약할 수 있다.
    - 팀 간 소통이 더욱 원활해질 수 있다.
  - xUnit
    - 과거에는 대부분의 단위 테스트 프레임워크 프레임워크가 xUnit이라 불렸다.
    - xUnit의 시초는 Smalltalk의 단위 테스트 프레임워크인 SUnit이다.
    - xUnit 프레임워크의 이름은 C++은 CppUnit, Java는 JUnit Haskell은 HUnit과 같이 보통 해당 언어의 첫 글짜를 따서 지었다.
    - 이름뿐 아니라 xUnit을 기반으로 한 단위 테스트 프레임워크들은 대부분 동일한 구조를 가지고 있어 한 언어의 xUnit 프레임워크를 배우면 다른 xUnit 프레임워크도 쉽게 사용할 수 있다.





# 테스트 코드 작성하기

> 아래 예시에서는 pytest를 사용하여 테스트한다.

- 함수 하나 테스트하기

  - 아래와 같이 비밀번호를 검증하는 함수를 작성한다.
    - 해당 함수는 `rules`를 매개변수로 받으며, 이는 또 다른 함수다.
    - 매개변수로 전달된 규칙 함수 배열을 반복하면서 각 함수를 입력값(`password`)에 적용한다.

  ```python
  # password_verifier.py
  def verify_password(password, rules):
      errors = []
      for rule in rules:
          result = rule(password)
          if not result["passed"]:
              errors.append(f"error {result["reason"]}")
      return errors
  ```

  - 매개변수 `rules`로 전달되는 함수는 아래 데이터 형태를 반환값으로 보내야한다.

  ```json
  {
      "passed": {boolean},
      "reason": "{string}"
  }
  ```

  - 테스트 코드 작성하기

  ```python
  # test_password_verifier.py
  from password_verifier import verify_password
  
  
  def test_verify_password():
      # 준비
      fake_rule = lambda password : {"passed":False, "reason": "fake reason"}
      # 실행
      errors = verify_password("any value", [fake_rule])
      # 검증
      assert errors[0] == "error fake reason"
  ```

  - 테스트 실행하기

  ```bash
  $ pytest test_password_verifier.py 
  ```

  - 준비 - 실행 - 검증 패턴(Arrange - Act - Assert, AAA)
    - 위 테스트 코드는 AAA패턴으로 작성되었다.
    - AAA 패턴은 테스트의 가독성을 높여 테스트를 쉽게 이해할 수 있게 해준다.
  - 함수에 버그 심기
    - `verify_password()` 함수에 일부러 버그를 심어 테스트 결과가 어떻게 변하는지 살펴본다. 
    - `errors`에 error 메시지가 추가되지 않게 변경한다.

  ```python
  def verify_password(password, rules):
      errors = []
      for rule in rules:
          result = rule(password)
          if not result["passed"]:
              pass
              # errors.append(f"error {result["reason"]}")
      return errors
  ```

  - 다시 테스트를 수행하면 테스트는 실패하게 된다.
    - 이후에 다시 원래대로 코드를 수정하면 테스트에 성공한다.
    - TDD를 하지 않았을 때는 이처럼 코드의 특정 부분을 수정하면서 테스트가 성공하는지 보는 것도 신뢰성을 얻기에 좋은 방법이다.

  ```bash
  $ pytest test_password_verifier.py 
  ```

  - USE 전략
    - 테스트 코드에서 이름을 잘 짓는 것은 무엇보다 중요한데, 테스트 이름만 읽고도 무엇을 테스트 하는지 바로 알 수 있어야한다.
    - 테스트 코드의 이름을 지을 때 고려해야 할 세 가지 요소는 아래와 같다.
    - 테스트하려는 대상(Unit): 예시의 경우 `verify_password()`함수
    - 입력 값이나 상황에 대한 설명(Senario): 결과로 False를 반화하는 상황
    - 기댓 값이나 결과에 대한 설명(Expectation): 에러 메시지를 반환
    - 아래는 USE 전략에 따라 수정한 버전이다.

  ```python
  # test_password_verifier.py
  from password_verifier import verify_password
  
  
  def test_verify_password_given_a_failing_rule_return_error():
      fake_rule = lambda password : {"passed":False, "reason": "fake reason"}
      errors = verify_password("any value", [fake_rule])
      assert errors[0] == "error fake reason"
  ```



- 함수 리팩터링 후 테스트하기

  - `verify_password()` 함수를 아래와 같이 리팩터링한다.
    - 함수에서 메서드를 가진 클래스로 변경했다.
    - 기존 함수는 상태를 유지하지 않았지만, 이제는 `_rules`를 상태로 유지하고 있는다.

  ```python
  class PasswordVerifier:
      def __init__(self):
          self._rules = []
      
      def add_rule(self, rule):
          self._rules.append(rule)
      
      def verify(self, password):
          errors = []
          for rule in self._rules:
              result = rule(password)
              if not result["passed"]:
                  errors.append(f"error {result["reason"]}")
          return errors
  ```

  - 테스트 코드 작성하기
    - 위 코드에 대한 테스트를 작성할 때 고려해야 할 점은 작업 단위의 범위가 넓어졌다는 것이다.
    - 이제는 `fake_rule`를 사용하여 실패하도록 만든 시나리오를 테스트하려면 `add_rule`과 `verify`라는 두 개의 함수를 호출해야한다.
    - 상태 값(`_rules`)을 다루는 코드 특성상 두 함수가 반드시 결합(`coupling`)되어야한다.
    - 내부 상태를 노출하지 않고도 효과적으로 테스트하려면 두 함수를 사용해야 한다.

  ```python
  from password_verifier import PasswordVerifier
  
  
  def test_verify_password_given_a_failing_rule_return_error():
      verifier = PasswordVerifier()
      fake_rule = lambda input : {"passed":False, "reason": "fake reason"}
      verifier.add_rule(fake_rule)
      errors = verifier.verify("any value")
      assert errors[0] == "error fake reason"
  ```

  - 검증 추가하기
    - 오류가 하나만 있는지 확인하고 싶어 아래와 같이 새로운 검증을 추가했다.
    - 아래와 같은 테스트는 문제가 발생할 확률을 높인다.

  ```python
  from password_verifier import PasswordVerifier
  
  
  def test_verify_password_given_a_failing_rule_return_error():
      verifier = PasswordVerifier()
      fake_rule = lambda input : {"passed":False, "reason": "fake reason"}
      verifier.add_rule(fake_rule)
      errors = verifier.verify("any value")
      assert len(errors) == 1		# 추가된 검증 코드
      assert errors[0] == "error fake reason"
  ```

  - 검증 룰렛
    - 위처럼 종료점이 여러 개이거나 동일한 종료점에서 여러 값을 테스트하고자 할 때 문제가 생길 수 있다.
    - 위 예시에서 만약 새롭게 추가한 검증 코드가 실패한다면, 테스트 러너는 오류를 받고 다음 테스트로 이동하므로 두 번째 검증 코드(error reason을 확인하는 코드)는 실행되지 않는다.
    - 그럼에도 두 번째 검증 코드가 통과했는지 확인해보기 위해 첫 번째 검증 코드를 주석처리 하는 것은 좋은 테스트 방식이 아니다.
    - 제라드 메스자로스의 <xUnit 테스트 패턴>에서는 이를 검증 룰렛이라고 한다.
    - 이는 테스트를 실행할 때 많은 혼란과 잘못된 false positive를 초래할 수 있다.
  - 검증 룰렛을 피하기 위해서는 추가 검증 코드를 별도의 테스트 케이스로 분리해야한다.
    - 추가 검증 코드를 별도의 테스트 케이스로 분리했다.
    - 이제는 두 테스트 사이에 중복 코드가 너무 많다는 새로운 문제가 발생한다.

  ```python
  from password_verifier import PasswordVerifier
  
  
  def test_verify_password_given_a_failing_rule_return_error():
      verifier = PasswordVerifier()
      fake_rule = lambda input : {"passed":False, "reason": "fake reason"}
      verifier.add_rule(fake_rule)
      errors = verifier.verify("any value")
      assert errors[0] == "error fake reason"
  
  def test_verify_password_has_exactly_one_error():
      verifier = PasswordVerifier()
      fake_rule = lambda input : {"passed":False, "reason": "fake reason"}
      verifier.add_rule(fake_rule)
      errors = verifier.verify("any value")
      assert len(errors) == 1
  ```

  - 중복 코드 제거하기
    - 유닛 테스트 프레임워크 중에는 각 테스트에 필요한 특정 상태를 설정하거나 해제하는 기능을 제공하는 것들이 있다.
    - Jest는 `beforeEach()`, `afterEach()`함수를 제공하며, pytest도 `fixture`를 제공한다.
    - 아래 코드는 `pytest.fixture`를 사용하여 중복 코드를 제거한 것이다.
  
  ```python
  import pytest
  
  from password_verifier import PasswordVerifier
  
  
  @pytest.fixture(autouse=True)
  def errors():
      verifier = PasswordVerifier()
      fake_rule = lambda input : {"passed":False, "reason": "fake reason"}
      verifier.add_rule(fake_rule)
      errs = verifier.verify("any value") 
      return errs
  
  def has_a_error_message_based_on_reason(errors): 
      assert errors[0] == "error fake reason"
  
  def has_exactly_one_error(errors):
      assert len(errors) == 1
  ```
  
  - 주의 사항
    - 아래와 같이 몇 개의 테스트가 추가되었다고 가정해보자.
    - 중복 코드 제거를 위해 fixture를 사용했지만, 여러 fixture들 사이에도 중복 코드가 발생하게 되었다.
    - 또 어떤 테스트가 어떤 fixture를 사용하는지 확인하는 것도 불편해졌다.
    - 실제 프로젝트를 진행함에 따라 fixture와 같은 setup 함수는 일종의 쓰레기통으로 전락할 위험이 있다.
    - 초기화 코드부터 특정 테스트에만 필요한 코드나 모든 테스트에 영향을 미치는 코드 등 온갖 것을 다 집어넣기 때문이다.
  
  ```python
  import pytest
  
  from password_verifier import PasswordVerifier
  
  
  @pytest.fixture(autouse=True)
  def single_error():
      verifier = PasswordVerifier()
      fake_rule = lambda input : {"passed":False, "reason": "fake reason"}
      verifier.add_rule(fake_rule)
      errs = verifier.verify("any value") 
      return errs
  
  def test_has_a_error_message_based_on_reason(single_error): 
      assert single_error[0] == "error fake reason"
  
  def test_has_exactly_one_error(single_error):
      assert len(single_error) == 1
  
  
  @pytest.fixture(autouse=True)
  def no_error():
      verifier = PasswordVerifier()
      fake_rule = lambda input : {"passed":True, "reason": ""}
      verifier.add_rule(fake_rule)
      errs = verifier.verify("any value")
      return errs
  
  def test_has_no_error(no_error):
      assert len(no_error) == 0
  
  
  @pytest.fixture(autouse=True)
  def errors():
      verifier = PasswordVerifier()
      fake_passing_rule = lambda input : {"passed":True, "reason": ""}
      verifier.add_rule(fake_passing_rule)
      fake_failed_rule = lambda input : {"passed":False, "reason": "fake reason"}
      verifier.add_rule(fake_failed_rule)
      errs = verifier.verify("any value")
      return errs
  
  
  def test_has_one_error(errors):
      assert len(errors) == 1
  
  
  def test_error_text_belongs_to_failed_rule(errors):
      assert "fake reason" in errors[0]
  ```
  
  - 팩토리 함수
    - 팩토리 함수는 객체나 특정 상태를 쉽게 생성하고 여러 곳에서 동일한 로직을 재사용할 수 있게 해주는 간단한 헬퍼 함수다.
    - 팩토리 함수를 사용하면 중복 코드를 줄이고 코드 직관성을 높일 수 있다.
    - 아래와 같이 팩토리 함수를 사용하도록 변경한다.
  
  ```python
  from password_verifier import PasswordVerifier
  
  # 팩토리 함수
  def make_verifier_with_passing_rule():
      verifier = PasswordVerifier()
      fake_rule = lambda input : {"passed":True, "reason": ""}
      verifier.add_rule(fake_rule)
      return verifier
  
  # 팩토리 함수
  def make_verifier_with_failed_rule(reason):
      verifier = PasswordVerifier()
      fake_rule = lambda input : {"passed":False, "reason": reason}
      verifier.add_rule(fake_rule)
      return verifier
  
  
  def test_has_a_error_message_based_on_reason(): 
      verifier = make_verifier_with_failed_rule("fake reason")
      errs = verifier.verify("any value") 
      assert errs[0] == "error fake reason"
  
  
  def test_has_exactly_one_error():
      verifier = make_verifier_with_failed_rule("fake reason")
      errs = verifier.verify("any value") 
      assert len(errs) == 1
  
  
  def test_has_no_error():
      verifier = make_verifier_with_passing_rule()
      errs = verifier.verify("any value")
      assert len(errs) == 0
  
  
  def test_has_one_error():
      verifier = make_verifier_with_failed_rule("fake reason")
      verifier.add_rule(lambda input : {"passed":True, "reason": ""})
      errs = verifier.verify("any value")
      assert len(errs) == 1
  
  
  def test_error_text_belongs_to_failed_rule():
      verifier = make_verifier_with_failed_rule("fake reason")
      verifier.add_rule(lambda input : {"passed":True, "reason": ""})
      errs = verifier.verify("any value")
      assert "fake reason" in errs[0]
  ```
  
  - 코드 길이는 변경 이전과 비슷하지만 가독성이 높아졌다.
    - 더 이상 어느 테스트가 어느 fixture를 사용하는지 일일이 확인하지 않아도 된다.
    - 또한 각 테스트 함수가 자체 상태를 캡슐화하고 있다.



- 다양한 입력 값으로 테스트하기

  - 아래와 같은 비밀번호 규칙이 있다.

  ```python
  def one_uppercase_rule(value):
      return {
          "passed": value.lower() != value,
          "reason": "at least one uppercase needed"
      }
  ```

  - 위 규칙을 다양한 조건으로 테스트한다.
    - 아래 테스트를 보면 알 수 있듯, 두 번째와 세 번째 테스트는 입력값만 다를 뿐 근본적으로는 같은 테스트이다.
    - 아래 테스트를 통해 테스트하고자 하는 것은 대문자가 어디에 위치했는지가 아닌 대문자를 포함하고 있는지 여부다.
    - 이러한 중복 코드는 나중에 대문자를 검사하는 로직이 변경되거나, `one_upper_case_rule()`함수를 사용하는 코드가 변경될 경우 문제가 될 수 있다.

  ```python
  def test_given_no_uppercase_it_fails():
      result = one_uppercase_rule("abc")
      assert not result["passed"]
  
  def test_given_one_uppercase_it_passed():
      result = one_uppercase_rule("Abc")
      assert result["passed"]
  
  def test_given_a_different_uppercase_it_passed():
      result = one_uppercase_rule("aBc")
      assert result["passed"]
  ```

  - 유닛 테스트 프레임워크들은 다양한 입력 값을 테스트할 수 있는 기능을 제공한다.
    - Jest의 경우 `test.each()`가 있으며, pytest의 경우 입력값을 parameter화 하는 기능을 제공한다.
    - 아래와 같이 입력값을 parameter화 하면 하나의 테스트에서 모두 테스트가 가능하다.

  ```python
  import pytest
  
  
  @pytest.mark.parametrize("value,expected", [("abc", False), ("Abc", True), ("aBc", True)])
  def test_one_uppercase_rule(value, expected):
      result = one_uppercase_rule(value)
      assert result["passed"] == expected
  ```

  - 실제 테스트에서는 분리하는 것이 나을 수도 있다.
    - 보통 하나의 테스트에서 동일한 유형의 입력 값에 대해 일관된 시나리오를 유지하는 것이 좋다.
    - 예를 들어 위에서 통합된 테스트는 대문자가 없는 경우와 대문자가 있는 경우를 모두 테스트한다.
    - 그러나 대문자가 없는 경우와 있는 경우는 두 가지 서로 다른 시나리오 이므로 서로 다른 테스트 두 개로 분리하는 것이 더 낫다.

  ```python
  import pytest
  
  
  @pytest.mark.parametrize("value,expected", [("abc", False)])
  def test_no_uppercase_rule(value, expected):
      result = one_uppercase_rule(value)
      assert result["passed"] == expected
  
  @pytest.mark.parametrize("value,expected", [("Abc", True), ("aBc", True)])
  def test_one_uppercase_rule(value, expected):
      result = one_uppercase_rule(value)
      assert result["passed"] == expected



- 예상한 예외가 발생하는지 확인하기

  - 비밀번호를 검증하는  `verify()` 메서드를 아래와 같이 수정한다.
    - 규칙이 설정되지 않을 경우 예외를 발생시킨다.

  ```python
  class PasswordVerifier:
      # ...
      
      def verify(self, password):
          if len(self._rules) == 0:
              raise Exception("There are no rules configured")
          
          errors = []
          for rule in self._rules:
              result = rule(password)
              if not result["passed"]:
                  errors.append(f"error {result["reason"]}")
          return errors
  ```

  - 아래와 같이 try, except 구문을 사용하여 예외가 발생하는지 테스트할 수 있다.

  ```python
  def make_verifier_with_no_rule():
      verifier = PasswordVerifier()
      return verifier
  
  def test_with_no_rules_raise_exception():
      verifier = make_verifier_with_no_rule()
      try:
          verifier.verify("value")
      except Exception as e:
          assert "no rules configured" in e.__str__()
  ```

  - 대부분의 유닛 테스트 프레임워크는 try, except 구문 대신 보다 간편하게 발생한 예외를 검증할 수 있는 방법을 제공한다.
    - `Jest`의 `expect().toThrowError()`나 `pytest`의 `raises()` 등이 이러한 방법을 제공한다.

  ```python
  def test_with_no_rules_raise_exception():
      verifier = make_verifier_with_no_rule()
      with pytest.raises(Exception, match=r"no rules configured"):
          verifier.verify("value")







# 의존성 분리와 스텁

- 의존성 유형
  - 외부로 나가는 의존성
    - 작업 단위의 종료점을 나타내는 의존성이다.
    - 예를 들어 로거 함수 호출, DB에 저장, 이메일 발송, API나 웹훅에 알림을 보내는 작업 등이 이에 해당한다.
    - 이러한 작업은 모두 동사("호출", "저장", "전송", "알림")로 표현되며, 작업 단위에서 외부로 흘러가는 일종의 fire-and-forget 시나리오를 의마한다.
    - 각 작업은 종료점을 나타내거나 작업 단위 내 특정 논리 흐름의 끝을 의미한다.
  - 내부로 들어오는 의존성
    - 종료점을 나타내지 않는 의존성이다.
    - 작업 단위의 최종 동작에 대한 요구 사항을 나타내지 않는다.
    - 단지 테스트에 필요한 특수한 데이터나 동작을 작업 단위에 제공하는 역할을 한다.
    - 예를 들어 쿼리 결과, 파일 시스템의 파일 내용, 네트워크 응답 결과 등이 있다.
    - 이러한 의존성들은 모두 이전 작업의 결과로, 작업 단위로 들어오는 수동적인 데이터 조각이라고 할 수 있다.
  - 혼합된 의존성
    - 외부로 나가는 의존성이기도 하면서 동시에 내부로 들어오는 의존성도 있다.
    - 즉 어떤 테스트에서는 종료점을 나타내고 다른 테스트에서는 애플리케이션으로 들어오는 데이터가 될 수도 있다.
    - 이러한 의존성은 흔하진 않지만 아예 없는 것도 아니다.
    - 예를 들어 외부 API가 응답 결과로 성공 및 실패 응답을 받아 서드 파티를 호출하는 경우가 이에 해당한다.



- 스텁, 목, 페이크

  - 스텁(Stub)
    - 테스트 대상 코드에 미리 정의된 가짜 데이터를 제공한다.
    - 예를 들어 고객이 "채식주의자" 책을 검색할 때 실제 DB에 접근하지 않고도 결과를 반환하도록 하기 위해 스텁을 사용할 수 있다.
    - 스텁을 통해 "채식주의자"라는 책을 반환하도록 설정하면 DB 상태와 무관하게 검색 기능을 테스트할 수 있다.
    - 이처럼 스텁은 테스트 독립성을 높여 주지만, 호출 여부는 검증하지 않는다.
    - 스텁은 내부로 들어오는 의존성(간접 입력)을 끊어준다.
    - 스텁은 가짜 모듈이나 객체 및 가짜 동작이나 데이터를 코드 내부로 보내는 가짜 함수를 의미한다.
    - 이를 통해 테스트 대상 코드가 외부 시스템이나 데이터에 의존하지 않고도 동작할 수 있게 한다.
    - 하나의 테스트에서 여러 스텁을 사용할 수 있다.
  - 목(Mock)
    - 외부 시스템과 상호 작용을 시뮬레이션하고, 호출 여부나 호출된 인수를 검증하는 데 사용된다.
    - 스텁과 달리 호출 여부를 검증한다.
    - 예를 들어 고객이 "채식주의자" 책을 검색하고 이를 장바구니에 추가할 때, 실제로 DB에 책이 추가되지 않더라도 책 추가 함수가 제대로 호출되었는지, 어떤 인수로 호출되었는지 등을 확인하는 데 목을 사용한다.
    - 목은 하나의 테스트에서 하나만 사용하는 것이 좋다.
    - 목은 외부로 나가는 의존성(간접 출력 또는 종료점)을 끊어준다.
    - 목은 가짜 모듈이나 객체 및 호출 여부를 검증하는 함수를 의미한다.
    - 목은 테스트 단위에서 종료점을 나타낸다.
  - 페이크
    - 실제 구현을 대체하는 가벼운 버전의 구성 요소다.
    - 예를 들어 실제 DB를 사용하는 대신 인메모리 DB를 사용하여 테스트를 수행할 수 있다.
    - 페이크는 실제 구성 요소와 동일한 인터페이스를 제공하지만, 훨씬 가볍고 빠르게 동작한다.
    - 실제가 아닌 모든 것을 지칭할 때 페이크라는 단어를 사용하기도 한다.
  - 정리

  | 카테고리 | 패턴          | 목적                                                         | 사용법                                                       |
  | -------- | ------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
  |          | 테스트 더블   | 스텁과 목을 포함한 일반적인 이름.                            | 페이크라고도 부른다.                                         |
  | 스텁     | 더미 객체     | 테스트에서 사용될 값을 지정하는 데 사용.                     | 진입점의 매개변수로 보내거나 준비 단계에서 사용한다.         |
  |          | 테스트 스텁   | 다른 소프트웨어 구성 요소의 간접 입력에 의존해야할 때 독립적으로 로직을 검증하는 데 사용. | 의존성으로 주입하고 SUT에 특정 값이나 동작을 반환하도록 구성한다. |
  | 목       | 테스트 스파이 | 다른 소프트웨어 구성 요소에 간접 출력을 보낼 때 독립적으로 로직을 검증하는 데 사용. | 실제 객체의 메서드를 오버라이드하고, 오버라이드한 함수가 예상대로 호출되었는지 확인한다. |
  |          | 모의 객체     | 다른 소프트웨어 구성 요소에 대한 간접 출력에 의존해야 할 때 독립적으로 로직을 검증하기 위해 사용. | 가짜 객체를 SUT 의존성으로 주입하고, 가짜 객체가 예상대로 호출되었는지 확인한다. |



- 스텁을 사용하는 이유

  - 아래와 같은 코드를 테스트해야 한다고 가정해보자.
    - 토요일, 일요일에는 비밀번호 검증을 하지 않기로 했다.
    - 요일을 다루기 위해서 `datetime` 패키지를 사용했다.
    - `datetime` 패키지를 사용함으로써 패키지에서 제공하는 값을 사용해야 하므로, 우리는 날짜나 시간을 직접 통제할 수 없다.
    - 이는 테스트의 실행 결과가 현재 요일에 따라 달라질 수 있음을 의미한다.

  ```python
  from datetime import datetime
  
  SUNDAY = 0
  SATURDAY = 6
  
  def verfiy_password(input, rules):
      weekday = datetime.today().weekday()
      if weekday in [SUNDAY, SATURDAY]:
          raise Exception("It's the weekend!")
      # 다른 코드
      return []
  ```

  - 위 코드의 테스트 코드를 작성한다.
    - 아래 코드는 현재 요일이 토요일이나 일요일일 때만 검증된다.

  ```python
  def test_on_weekend_throw_exceptions():
      weekday = datetime.today().weekday()
      if weekday in [SUNDAY, SATURDAY]:
          with pytest.raises(Exception, match=r"It's the weekend!"):
              verfiy_password("anything", [])
  ```

  - 위 테스트의 문제점
    - 테스트는 언제 실행하든 항상 같은 결과를 보장해야한다.
    - 테스트 내에서 사용된 값은 변하지 않아야 하고 검증도 매번 동일해야 한다.
    - 테스트 코드나 프로덕션 코드를 변경하지 않는 한 이 법칙은 유지해야 한다.
  - 이와 같이 테스트가 외부적인 요인에 의존해야 할 때 일관된 테스트 결과를 얻기 위해 스텁을 사용할 수 있다.



- 스텁을 사용하는 일반적인 설계 방식

  - 위에서 살펴본 테스트 대상 코드를 변경한다. 
    - 현재 날짜를 의미하는 `current_weekday`를 매개변수로 추가한다.
    - 이렇게 하면 함수 내에서 `datetime` 패키지를 사용할 필요가 없고, 테스트에서 시간을 하드코딩 방식으로 설정하여 인수로 넘겨주면 테스트의 일관성을 유지할 수 있다.

  ```python
  def verfiy_password(input, rules, current_weekday):
      if current_weekday in [SUNDAY, SATURDAY]:
          raise Exception("It's the weekend!")
      # 다른 코드
      return []
  ```

  - `current_weekday` 매개변수를 추가함으로써 시간 값의 통제권을 함수 호출자에 넘겨주었다.
    - 작업 단위에 시간이라는 의존성을 주입하는 방식으로 변경했으며, 새로운 의존성이 매개변수로 주입되면서 작업 단위의 진입점 역시 바뀌었다.
    - 우리가 앞으로 주입할 것은 더미(dummy)라는 단순한 데이터 조각으로, 별다른 동작은 없는 코드 조각이다.
    - 여기서는 이 더미가 스텁의 역할을 하게 된다.
    - 단순히 요일을 나타내는 숫자값인 `current_weekday`를 스텁이라고 부르는 것이 이상하게 느껴질 수 있다.
    - 그러나 스텁이 꼭 복잡해야 할 필요는 없으며, 함수에 전달되는 특정 입력 값이나 동작을 모방하기만 하면 스텁이라고 볼 수 있다.
  - 이러한 변화는 제어의 역전(Inversion of Control)이라고도 볼 수 있다.
    - 테스트에서 의존성이란 제어할 수 없어 테스트를 어렵게 만드는 요소를 의미한다.
    - 제어란 의존성의 동작 방식을 결정할 수 있는 능력을 의미하며, 의존성을 생성하는 주체가 그 의존성을 제어한다고 할 수 있다.
    - 예를 들어 변경 이전의 코드에서 `weekday`에 대한 제어권은 우리가 작성한 코드가 아닌 `datetime` 패키지가 가지고 있었다.
    - 제어의 역전이란 의존성을 내부에서 생성하지 않고 외부에서 주입받도록 코드 설계를 변경하는 것을 의미한다.
    - 변경 이후의 코드에서 `weekday`는 더 이상 `datetime` 패키지가 생성하지 않고, 매개 변수로 받음으로써 우리가 작성한 코드가 `weekday`를 제어할 수 있게 되었다.
  - 이제 테스트 코드도 아래와 같이 변경하면 된다.

  ```python
  def test_on_weekend_throw_exceptions():
      with pytest.raises(Exception, match=r"It's the weekend!"):
          verfiy_password("anything", [], SUNDAY)



- 함수를 이용한 주입 방법
  - 이전 예시에서는 `weekday`

