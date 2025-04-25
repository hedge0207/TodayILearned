# 모의 객체를 사용한 상호 작용 테스트

- 정리

  - 이전 장에서는 다른 객체에 의존하는 코드를 테스트하는 방법을 살펴봤다.
    - 스텁을 사용하여 테스트 코드가 필요로하는 모든 입력을 받아 작업 단위를 독립적으로 테스트할 수 있었다.
  - 지금까지는 작업 단위가 가질 수 있는 세 가지 종료점 중 두 가지에 대해서만 테스트를 작성했다.
    - 첫 번째는 값을 반환하는 것이고, 두 번째는 시스템의 상태를 변경하는 것이다.
    - 여기서는 세 번째 종료점에 해당하는 서드 파티 함수나 모듈, 객체를 호출하여 테스트하는 방법을 살펴볼 것이다.
    - 코드를 작성하다보면 제어할 수 없는 다른 시스템이나 라이브러리에 의존해야 할 때가 있다.
    - 예를 들어 외부 API를 호출하거나, 다른 팀이 만든 모듈을 사용하는 경우 등이 해당한다.
  - 목
    - 목은 외부로 나가는 의존성과 연결 고리를 끊는 데 사용한다.
    - 목은 가짜로 만든 모듈이나 객체 및 함수로, 단위 테스트에서 종료점을 나타낸다.
    - 목은 어떤 대상을 흉내내에 만들었기 때문에 호출되었는지 검증하는 것이 중요하다.
    - 호출 여부를 검증하지 않으면 목을 사용하지 않은 것이나 마찬가지다.
    - 유지 보수성과 가독성 때문에 보통 하나의 테스트에 목은 한 개만 사용하는 것이 일반적이다.
  - 스텁
    - 내부로 들어오는 의존성과 연결 고리를 끊는 데 사용한다.
    - 스텁은 테스트 코드에 가짜 동작이나 데이터를 제공하는 가짜 모듈, 객체, 함수를 의미한다.
    - 목과 달리 스텁은 검증할 필요가 없고, 하나의 테스트에 스텁을 여러개 사용할 수 있다.
    - 스텁은 데이터나 동작이 작업 단위로 들어오는 경유지를 나타내며 종료점은 나타내지 않는다.
    - 스텁은 상호작용이 발생하는 지점으로 볼 수 있지만, 작업 단위의 최종 결과를 나타내지 않는다.
    - 최종 결과를 달성하는 과정에서의 상호 작용일 뿐이므로 종료점으로 취급하지 않는다.

  - 상호작용 테스트
    - 상호 작용 테스트는 작업 단위가 제어할 수 없는 영역에 있는 의존성과 어떻게 상호작용하고 함수를 호출하는지 확인하는 것이다.
    - 목 함수나 목 객체를 사용하여 외부 의존성을 제대로 호출했는지 검증할 수 있다.



- 의존성을 제어할 수 없는 경우의 테스트

  - 아래와 같은 함수가 있다고 하자.
    - 함수 내부에서 로거를 통해 로그를 남긴다.
    - 아래 함수는 두 개의 종료점이 있는데, 하나는 값을 반환하는 것이고, 다른 하나는 `logging.info()` 메서드를 호출하는 것이다.
    - 테스트 코드를 작성한다 하더라도, `logging.info()` 메서드의 호출 여부를 알 수 없다.

  ```python
  import logging
  
  logger = logging.getLogger(__name__)
  logging.basicConfig(encoding='utf-8', level=logging.DEBUG)
  
  
  def verify_password(value, rules):
      failed = list(filter(lambda result: not result, map(lambda rule: rule(value), rules)))
      if len(failed) == 0:
          logging.info("PASSED")
          return True
      
      logging.info("FAIL")
      return False
  ```

  - 매개 변수를 주입하는 방식으로 변경하기
    - 위 함수를 아래와 같이 logger를 매개변수로 주입 받도록 변경한다.

  ```python
  def verify_password(value, rules, logger):
      failed = list(filter(lambda result: not result, map(lambda rule: rule(value), rules)))
      if len(failed) == 0:
          logger.info("PASSED")
          return True
      
      logger.info("FAIL")
      return False
  ```

  - 아래와 같이 테스트한다.
    - 여기서 목 객체를 생성할 클래스 이름과, 목 객체의 이름 앞에 mock을 붙여 모의 객체나 함수가 있다는 것을 알 수 있게 해줬다는 것에 주목해야한다.
    - 이렇게 이름을 지으면 코드를 읽는 사람이 쉽게 코드를 파악할 수 있게 된다.
    - 아래 테스트 코드는 테스트 대상 함수가 logger를 호출하는지 테스트한다.

  ```python
  class MockLogger:
      def __init__(self):
          self.written = ""
      
      def info(self, text):
          self.written = text
  
  
  def test_password_verifier_with_logger():
      mock_logger = MockLogger()
      verify_password("anything", [], mock_logger)
      assert "PASSED" in mock_logger.written
  ```

  - 매개변수를 추가하는 방법은 스텁을 외부에서 매개 변수로 주입할 수 있게 하는 방식과 비슷하다.
    - 단, 같은 매개변수로 주입하는 방식이라 해도, 목과 달리 스텁은 호출 여부를 검증하지는 않는다는 차이가 있다.



- 목과 스텁을 구분하는 것의 중요성

  - 목과 스텁을 구분하지 않으면 한 작업 단위 안에 여러 종료점이 있는 복잡한 함수를 테스트할 때 테스트의 가독성과 유지 보수성이 떨어질 수 있다.

  - 목과 스텁의 개수

    - 목은 작업 단위의 요구 사항을 나타내고(e.g. 로거를 호출한다, 메일을 보낸다 등), 스텁은 들어오는 정보나 동작을 나타낸다(e.g. DB query가 false를 반환한다, 특정 설정이 오류를 일으킨다 등).
    - 따라서 테스트에는 스텁을 여러개 사용해도 괜찮지만, 목은 테스트당 하나만 사용하는 것이 좋다.
    - 목이 여러 개라는 것은 하나의 테스트에서 여러 요구 사항을 테스트한다는 의미가 될 수 있다.

  - 목과 스텁을 구분하지 않고 사용하면, 테스트마다 목을 여러 개 만들거나, 스텁을 검증하는 상황이 발생할 수 있다.

    - 목은 테스트 당 하나만 사용하는 것이 바람직하며, 스텁은 검증의 대상이 아니다.
    - 이는 결국 테스트의 전체적인 품질을 낮추는 결과를 낳을 수 있다.

    - 목과 스텁의 이름을 구분하여 일관되게 사용하는 것도 중요한데, 이를 통해 가독성과 유지보수성을 높일 수 있기 때문이다.



- 모듈 스타일의 목

  - 의존할 모듈 생성하기
    - 테스트 대상 코드가 의존할 두 개의 모듈을 생성한다.

  ```python
  # logger.py
  import logging
  
  logging.basicConfig(
      level=logging.DEBUG,
      format='[%(levelname)s] %(message)s'
  )
  
  def info(text):
      logging.info(text)
  
  def debug(text):
      logging.debug(text)
  
  
  # logger_config.py
  "def get_log_level():
      return "info""
  ```

  - 모듈 의존성 예시
    - 아래 코드에서 `verfiy_password` 함수는 `info`, `debug`라는 로깅 함수와 로거 레벨을 받아오는 `get_logger_level`이라는 두 가지 외부 의존성에 의존한다.

  ```python
  from logger import info, debug
  from logger_config import get_log_level
  
  
  def log(text):
      if get_log_level() == "info":
          info(text)
      else:
          debug(text)
  
  
  def verify_password(value, rules):
      failed = [rule for rule in rules if rule(value) == False]
  
      if len(failed) == 0:
          log("PASSED")
          return True
      else:
          log("FAIL")
          return False
  ```

  - 위 코드에 대한 테스트 코드는 아래와 같이 `unittest.mock.patch` decorator를 사용하여 간단하게 작성할 수 있다.

  ```python
  import pytest
  from unittest.mock import patch
  from password_verifier import verify_password
  
  
  @patch("logger_config.get_log_level", return_value="info")
  @patch("logger.info")
  @patch("logger.debug")
  def test_password_passes_info_log(mock_debug, mock_info, mock_get_level):
      rules = [
          lambda x: len(x) >= 8,
          lambda x: any(c.isdigit() for c in x)
      ]
      result = verify_password("abc12345", rules)
  
      assert result is True
      mock_info.assert_called_once_with("PASSED")
      mock_debug.assert_not_called()
  ```



- 객체 지향 스타일의 목

  - 테스트 대상 코드
    - 의존 대상인 logger를 클래스를 생성할 때 인자로 받아 인스턴스 변수로 할당한다.
    - 이렇게 하면 객체를 생성할 때 필요한 의존성을 명확히 알 수 있다.
    - 생성자가 아닌 프로퍼티를 사용한다면 의존성은 선택사항이 되어 객체를 생성한 후 설정할 수 있다.
    - 그러나 이 경우 필요한 의존성을 설정하지 않고도 객체가 생성될 수 있어 의존성 주입이 명확하지 않게 된다.
    - 생성자를 사용하면 의존성이 필수적이라는 것을 명확히 나타낼 수 있어 코드의 가독성과 유지 보수성을 높일 수 있다.

  ```python
  class PasswordVerifier:
      def __init__(self, rules, logger):
          self._rules = rules
          self._logger = logger
      
      def verify(self, value):
          failed = list(filter(lambda result: not result, map(lambda rule: rule(value), self._rules)))
          if len(failed) == 0:
              self._logger.info("PASSED")
              return True
          
          self._logger.info("FAIL")
          return False
  ```

  - 아래와 같이 `MockLogger`르를 생성해서 테스트한다.

  ```python
  class MockLogger:
      def __init__(self):
          self.text = ""
      
      def info(self, text: str):
          self.text = text
  
  def test_verify_passed_logs_passed():
      mock_logger = MockLogger()
      verifier = PasswordVerifier([], mock_logger)
  
      verifier.verify("any-value")
  
      assert mock_logger.text == "PASSED"
  ```

  - `unittest.mock.MagicMock`을 사용하면 mock을 위한 클래스를 직접 생성하지 않아도 된다.

  ```python
  from unittest.mock import MagicMock
  
  def test_verify_passed_logs_passed():
      mock_logger = MagicMock()
      verifier = PasswordVerifier([], mock_logger)
  
      verifier.verify("any-value")
  
      mock_logger.info.assert_called_once_with("PASSED")
  ```



- 인터페이스 주입을 이용한 코드 리팩터링

  - 객체 지향 프로그램에서 인터페이스는 동일한 인터페이스를 구현하는 객체들을 서로 대체할 수 있게 하여 다형성을 이루게 해준다.
    - JavaScript나 Python 같이 덕 타이핑을 허용하는 언어에서는 굳이 인터페이스를 필요로 하지 않는다.
    - 이 언어들은 객체를 특정 인터페이스를 변환하지 않아도 되기 때문이다.
  - 아래 코드는 인터페이스와 인터페이스를 구현한 클래스, 그리고 인터페이스에 의존하는 클래스이다.

  ```python
  # 인터페이스
  class Logger(ABC):
      @abstractmethod
      def info(self, text: str):
          ...
  
  # 인터페이스를 구현한 클래스
  class SimpleLogger(Logger):
      def info(self, text: str):
          print(text)
  
  # 인터페이스에 의존하는 클래스
  class PasswordVerifier:
      def __init__(self, rules: list, logger: Logger):
          self._rules = rules
          self._logger = logger
      
      def verify(self, value):
          failed = list(filter(lambda result: not result, map(lambda rule: rule(value), self._rules)))
          if len(failed) == 0:
              self._logger.info("PASSED")
              return True
          
          self._logger.info("FAIL")
          return False
  ```

  - 위와 같이 인터페이스에 의존하는 클래스를 테스트 할 때는, 해당 인터페이스를 구현한 가짜 객체를 생성하여 사용하면 된다.

  ```python
  class FakeLogger(Logger):
      def __init__(self):
          self.written = ""
          
      def info(self, text: str):
          self.written = text
  
  
  def test_verify_passed_logs_passed():
      mock_logger = FakeLogger()
      verifier = PasswordVerifier([], mock_logger)
  
      verifier.verify("any-value")
  
      assert mock_logger.written == "PASSED"
  ```



- 복잡한 인터페이스 다루기

  - 복잡한 인터페이스 예시
    - `PasswordVerifier` 클래스가 생성자로 받는 로거는 `ComplicatedLogger` 인터페이스를 구현한 객체이며, 이는 서로 다른 함수를 네 개 포함하고 있다.
    - 각 함수는 하나 이상의 매개 변수를 받아야 하며, 모든 함수는 테스트에서 가짜로 만들어야한다.
    - 이는 코드와 테스트가 복잡해지고, 유지 보수가 어려워지는 원인이 될 수 있다.

  ```python
  from typing import Any
  
  class ComplicatedLogger(ABC):
      @abstractmethod
      def info(self, text: str): ...
      
      @abstractmethod
      def debug(self, text: str, obj: Any): ...
      
      @abstractmethod
      def warn(self, text: str): ...
      
      @abstractmethod
      def error(self, text: str, location: str, stacktrace: str): ...
      
      
  class PasswordVerifier:
      def __init__(self, rules: list, logger: ComplicatedLogger):
          self._rules = rules
          self._logger = logger
      
      # ...
  ```

  - 복잡한 인터페이스를 사용하여 테스트하기
    - 테스트에 사용하기 위해 `ComplicatedLogger`를 구현하는 클래스를 만든다.
    - 인터페이스의 모든 메서드를 오버라이드해야 하므로 반복 코드가 생기고 코드 길이도 길어진다.

  ```python
  class FakeComplicatedLogger(ComplicatedLogger):
      def __init__(self):
          self.info_written = ""
          self.debug_written = ""
          self.warn_written = ""
          self.error_written = ""
      
      def info(self, text: str):
          self.info_written = text
          
      def debug(self, text: str):
          self.debug_written = text
      
      def warn(self, text: str):
          self.warn_written = text
          
      def error(self, text: str, location: str, stacktrace: str):
          self.error_wrtten = text
  
  
  def test_verify_passed_logs_passed():
      mock_logger = FakeComplicatedLogger()
      verifier = PasswordVerifier([], mock_logger)
  
      verifier.verify("any-value")
  
      assert mock_logger.info_written == "PASSED"
  ```

  - 테스트 내에서 복잡한 인터페이스를 사용할 때 단점
    - 인터페이스의 각 메서드를 호출할 때 전달받은 매개 변수를 인스턴스 변수(`self.info_written`, `self.debug_written` 등)에 직접 저장해야하므로 메서드마다 각 호출에 대한 매개 변수를 검증하는 것이 더 번거로워진다.
    - 내부 인터페이스가 아닌 서드 파티 인터페이스에 의존할 때가 많아 시간이 지나면서 테스트가 더 불안정해질 수 있다.
    - 내부 인터페이스에 의존하더라도 긴 인터페이스는 변경될 가능성이 높아 테스트를 변경해야 할 이유도 많아진다.
  - 위와 같은 단점으로 인해, 아래 두 조건을 만족하는 경우에만 가짜 인터페이스를 사용하는 것이 좋다.
    - 인터페이스에 대해 온전히 제어권을 가지고 있다(즉, 서드 파티에서 제공하는 인터페이스가 아니어야 한다).
    - 작업 단위나 컴포넌트의 요구 사항에 맞게 설계된 인터페이스여야 한다.
  - 인터페이스 분리 원칙
    - 위 두 가지 조건 중 두 번째 조건은 ISP와 관련 있다.
    - ISP는 인터페이스에 필요한 것보다 더 많은 기능이 포함되어 있으면 필요한 기능만 포함된 더 작은 어댑터 인터페이스를 만들어야 한다는 것이다.
    - 따라서 가능한 한 함수를 더 적게 만들고 이름을 더 명확하게 짓고 매개변수를 덜 사용하도록 하면 좋다.



- 부분 모의 객체
  - 대부분의 언어와 테스트 프레임워크에서는 기존 객체와 함수를 감시(spy)하는 것이 가능하다.
  - 이를 통해 나중에 해당 객체나 함수가 호출 되었는지, 몇 번 호출되었는지, 어떤 인수로 호출되었는지를 파악할 수 있다.
    - 이 방식을 사용하면 실제 객체 일부를 모의 함수로 변환하면서 나머지 부분은 실제 객체로 유지할 수 있다.
    - 이렇게 할 경우 테스트가 더 복잡해지거나 불안정해질 수 있지만, 레거시 코드를 다루어야 할 때 괜찮은 선택이 될 수도 있다.



- 부분 모의 객체를 함수형 방식으로 풀어보기

  - 아래 코드는 부분 모의 객체를 사용하는 테스트가 어떻게 생겼는지 보여준다.
    - 실제 로거를 의미하는 `RealLogger` 인스턴스를 만든 후 기존 메서드 중 하나를 가짜 함수로 덮어쓴다.

  ```python
  def test_verify_with_logger():
      testable_logger = RealLogger()
      # lambda에서는 대입을 할 수 없으므로 가변 객체인 dictionary 사용
      logged = {"message":""}
      testable_logger.info = lambda text: logged.update({"message":text})
      verifier = PasswordVerifier([], testable_logger)
      verifier.verify("any-value")
      assert logged["message"] == "PASSED"
  ```

  - 여기서 중요한 것은 `testable_logger`는 부분 모의 객체라는 것이다.
    - 이는 `testable_logger`의 메서드 중 일부가 실제 의존성과 로직을 포함한 함수라는 의미다.



- 부분 모의 객체를 객체 지향 방식으로 풀어보기

  - 실제 클래스의 메서드를 오버라이드하여 해당 메서드가 호출되었는지 검증하는 방식을 사용한다.
    - 실제 클래스인 `RealLogger`를 상속 받아 일부 메서드만 오버라이드한다.

  ```python
  class TestableLogger(RealLogger):
      def __init__(self):
          self.logged = ""
  
      def info(self, text):
          self.logged = text
  
  def test_verify_with_logger():
      testable_logger = TestableLogger()
      verifier = PasswordVerifier([], testable_logger)
      verifier.verify("any-value")
      assert testable_logger.logged == "PASSED"
  ```

  - 이 방식을 추출 및 오버라이드(extract and override)라고 한다.
    - 이 방식은 프로덕션 코드의 클래스(예시의 경우 `RealLogger`)가 상속을 허용하고 오버라이딩을 허용해야 한다는 전제가 필요하다.







# 격리 프레임워크

- 격리 프레임워크
  - 정의
    - 객체나 함수 형태의 목이나 스텁을 동적으로 생성, 구성, 검증할 수 있게 해 주는 프로그래밍 가능한 API다.
    - 격리 프레임워크를 사용하면 이러한 작업을 수작업으로 했을 때보다 더 간단하고 빠르며 코드도 더 짧게 작성할 수 있다.
  - 결국 격리 프레임워크란 런타임에 가짜 객체를 생성하고 설정할 수 있는 재사용 가능한 라이브러리를 의미한다.
    - 이러한 객체를 동적 스텁(dynamic stubs)과 동적 목(dynamic mocks)이라고 한다.
    - 격리 프레임워크라 칭하는 이유는 작업 단위를 그 의존성으로부터 격리시킬 수 있기 때문이다.
  - 격리 프레임워크는 아래와 같은 장점이 있다.
    - 손쉬운 가짜 모듈 생성으로 보일러 플에이트 코드를 제거하여 모듈 의존성을 쉽게 처리할 수 있게 도와준다.
    - 값이나 오류를 만들어 내기가 더 쉬워진다.
    - 가짜 객체 생성이 더 쉬워진다.



- 동적으로 가짜 모델 만들기

  - 아래와 같은 코드가 있다고 가정해보자.
    - 아래 코드를 테스트하려면 아래 두 가지 작업을 수행해야한다.
    - `get_log_level`함수가 반환하난 값을 스텁을 사용하여 가짜로 만들어야한다.
    - `info`, `debug` 함수가 호출되었는지 모의 함수를 사용하여 검증해야한다.
    - 즉 내부로 들어오는 의존성인 `get_log_level`는 스텁으로 만들고, 나가는 의존성(`info`, `debug`)을 목으로 만들어야한다.

  ```python
  from logger_config import get_log_level
  from logger import info, debug
  
  
  def log(text: str):
      if get_log_level() == "info":
          info(text)
      elif get_log_level() == "debug":
          debug(text)
  
  
  def verify_password(value: str, rules: list):
      failed = [rule for rule in rules if rule(value) == False]
  
      if len(failed) == 0:
          log("PASSED")
          return True
      else:
          log("FAIL")
          return False
  ```
  
  - `unittest.mock` 사용하여 스텁 생성하기
    - 내부로 들어오는 의존성인 `get_log_level`을 스텁으로 만든다.
    - `patch`는 첫 번째 인자로 경로를 받는데, 주의할 점은 실제 의존성의 경로가 아니라 의존성을 사용하고 있는 테스트 대상을 기준으로 경로를 작성해야 한다는 점이다.
    - 예를 들어 예시에서 `get_log_level`는 `logger_config` 모듈에 있으므로 `@mock.patch("logger_config.get_log_level")`과 같이 작성해야 한다고 생각할 수 있지만, 그렇지 않다.
  
  ```py
  from unittest import mock
  
  
  @mock.patch("password_verifier.get_log_level", return_value="info")
  def test_info_log_level_and_no_rules(stub):
      verify_password("anything", [])
  ```
  
  - `pytest-mock`을 사용하면 보다 단순하게 스텁을 생성 할 수 있다.
    - `pip install pytest-mock`으로 설치한다.
    - pytest-mock의 `mocker`는 fixture로 동작하기에 아래와 같이 사용이 가능하다.
  
  ```python
  def test_info_log_level_and_no_rules(mocker):
      mocker.patch("password_verifier.get_log_level").return_value="info"
      verify_password("anything", [])
  ```
  
  - 목 생성하기
    - 마찬가지로 `pytest-mock`을 사용하여 목을 생성할 수 있다.
  
  ```python
  def test_info_log_level_and_no_rules(mocker):
      mocker.patch("password_verifier.get_log_level", return_value="info")
      mock_info = mocker.patch("password_verifier.info")    
      verify_password("anything", [])
  ```
  
  - 스텁과 목을 사용하여 테스트 실행하기
    - `assert_called_with` 메서드는 어떤 인자와 함께 mock이 호출되었는지를 검증한다.
  
  ```python
  def test_info_log_level_and_no_rules(mocker):
      # stub
      mocker.patch("password_verifier.get_log_level", return_value="info")
      # mock
      mock_info = mocker.patch("password_verifier.info")
      
      verify_password("anything", [])
  
      mock_info.assert_called_with("PASSED")
  
  def test_debug_log_level_and_no_rules(mocker):
      # stub
      mocker.patch("password_verifier.get_log_level", return_value="debug")
      # mock
      mock_debug = mocker.patch("password_verifier.debug")
      
      verify_password("anything", [])
  
      mock_debug.assert_called_with("PASSED")
  
  ```
  



- 객체 지향 스타일의 동적 목과 스텁

  - 아래와 같은 코드가 있다고 가정해보자.
    - `PasswordVerifier` 클래스는 복잡한 인터페이스(`ComplicatedLogger`)에 의존한다.

  ```python
  class ComplicatedLogger(ABC):
      @abstractmethod
      def info(self, text: str, method: str): ...
      
      @abstractmethod
      def debug(self, text: str, method: str): ...
      
      @abstractmethod
      def warn(self, text: str, method: str): ...
      
      @abstractmethod
      def error(self, text: str, method: str): ...
      
  
  class PasswordVerifier:
      def __init__(self, rules: list, logger: ComplicatedLogger):
          self._rules = rules
          self._logger = logger
      
      def verify(self, value):
          failed = list(filter(lambda result: not result, map(lambda rule: rule(value), self._rules)))
          if len(failed) == 0:
              self._logger.info("PASSED", "verify")
              return True
          
          self._logger.info("FAIL", "verify")
          return False
  ```

  - 위와 같은 코드에서 복잡한 인터페이스인 `ComplicatedLogger`를 대체하는 스텁을 만들어야 하는 경우, 아래와 같이 긴 보일러 플레이트 코드를 작성해야한다.

  ```python
  
  class FakeLogger(ComplicatedLogger):
      def __init__(self):
          self.info_text = ""
          self.info_method = ""
          self.debug_text = ""
          self.debug_method = ""
          self.warn_text = ""
          self.warn_method = ""
          self.error_text = ""
          self.error_method = ""
      
      def info(self, text: str, method: str):
          self.info_text = text
          self.info_method = method
  
      def debug(self, text: str, method: str):
          self.debug_text = text
          self.debug_method = method
  
      def warn(self, text: str, method: str):
          self.warn_text = text
          self.warn_method = method
      
      def error(self, text: str, method: str):
          self.error_text = text
          self.error_method = method
  
  
  def test_verify_with_logger():
      mock_logger = FakeLogger()
      verifier = PasswordVerifier([], mock_logger)
      verifier.verify("any-value")
      assert mock_logger.info_text == "PASSED"
  ```

  - 이 역시 mock을 사용하면 보다 단순하게 테스트가 가능하다.

  ```python
  def test_verify_with_logger(mocker):
      mock_logger = mocker.Mock()
      verifier = PasswordVerifier([], mock_logger)
  
      verifier.verify("any-value")
  
      mock_logger.info.assert_called_with("PASSED", "verify")
  ```



- 목과 스텁을 사용한 객체 지향 예제

  - 비밀번호 검증기에 아래와 같은 요소를 추가한다.
    - 소프트웨어가 업데이트되는 유지 보수 기간 동안 비밀번호 검증기가 비활성화 된다.
    - 유지 보수 기간 중에는 비밀번호 검증기의 verify()를 호출하면 `logger.info()`에 "Under Maintenance"라는 메시지를 전달한다.
    - 유지 보수 기간이 아닐 때는 `logger.info()`에 passed 또는 failed 결과를 전달한다.
  - 변경된 코드는 아래와 같다.
    - `MainTenanceWindow` 인터페이스는 생성자 매개변수로 주입되어 비밀번호 검증을 실행할지 여부를 결정한다.

  ```python
  class MainTenanceWindow:
      def is_under_maintenance(self) -> bool:
          # ...
  
  class PasswordVerifier:
      def __init__(self, rules: list, logger: ComplicatedLogger, maintenance_window: MainTenanceWindow):
          self._rules = rules
          self._logger = logger
          self._maintenance_window = maintenance_window
      
      def verify(self, value):
          if self._maintenance_window.is_under_maintenance():
              self._logger.info("Under Maintenance", "verify")
              return False
          
          failed = list(filter(lambda result: not result, map(lambda rule: rule(value), self._rules)))
          if len(failed) == 0:
              self._logger.info("PASSED", "verify")
              return True
          
          self._logger.info("FAIL", "verify")
          return False
  ```

  - 아래와 같이 테스트한다.
    - 기존과 마찬가지로 logger를 mock으로 대체한다.
    - `MainTenanceWindow.is_inder_maintenance`의 반환값을 스텁으로 대체해야한다.

  ```python
  def test_verify_during_maintenance_with_logger(mocker):
      s
      stub_maintain_window.is_under_maintenance.return_value=True
      mock_logger = mocker.Mock()
      verifier = PasswordVerifier([], mock_logger, stub_maintain_window)
  
      verifier.verify("any-value")
  
      mock_logger.info.assert_called_with("Under Maintenance", "verify")
  
  
  def test_verify_outside_maintenance_with_logger(mocker):
      stub_maintain_window = mocker.Mock()
      stub_maintain_window.is_under_maintenance.return_value=False
      mock_logger = mocker.Mock()
      verifier = PasswordVerifier([], mock_logger, stub_maintain_window)
  
      verifier.verify("any-value")
  
      mock_logger.info.assert_called_with("PASSED", "verify")
  ```



- 격리 프레임워크의 함정
  - 대부분의 경우 모의 객체가 필요하지 않다.
    - 격리 프레임워크의 가장 위험하면서도 무시하기 힘든 함정은 무엇이든 쉽게 가짜로 만들 수 있다는 것과 애초에 모의 객체가 필요하다고 생각하게 하는 것이다.
    - 스텁이 필요하지 않다는 것은 아니지만, 모의 객체는 대부분의 단위 테스트에서 기본적으로 사용해서는 안 된다.
    - 작업 단위에는 반환 값, 상태 변화, 서드 파티 의존성 이렇게 세 가지 종류의 종료점이 있을 수 있다는 점을 항상 기억하자.
    - 이 중 단 하나의 유형만 테스트에서 모의 객체를 사용했을 때 이점을 누릴 수 있고, 나머지는 그렇지 않다.
    - 테스트를 정의하면서 모의 객체나 모의 함수가 호출되었는지 검즈아기 전에 모의 객체 없이도 동일한 기능을 검증할 수 있는지 잠시 생각해보자.
    - 모의 객체나 스텁을 사용하면 외부 의존성이 영향을 받아 테스트 난이도가 올라갈 수 있다.
    - 의도한 대로 제대로 동작하는지 확인하는 과정 자체가 힘들 수 있기 때문이다.
    - 대신에 반환 값을 검증하거나 작업 단위의 동작 변화를 외부에서 확인하는 것이 훨씬 쉬울 수 있다.
    - 예를 들어 함수가 실행되었는지 확인하는 대신 해당 함수가 예상한 대로 예외를 발생시키는지 확인할 수 있다.
  - 읽기 어려운 테스트 코드
    - 테스트에서 목을 사용하면 테스트 가독성이 조금 떨어지지만, 여전히 코드를 읽는 사람 입장에서는 큰 문제없이 이해할 수 있다.
    - 그러나 하나의 테스트에 많은 목을 만들거나 검증 단계를 너무 많이 추가하면 테스트 가독성이 떨어져 유지 보수가 어렵고, 무엇을 테스트하고 있는지 이해하기도 힘들다.
  - 잘못된 대상 검증
    - 모의 객체를 사용하면 인터페이스의 메서드나 함수가 호출되었는지 확인할 수 있지만, 그렇다고 해서 항상 올바른 대상을 테스트하고 있는 것은 아니다.
    - 아래와 같은 경우는 검증하지 않아야 할 대상을 검증하는 것이다.
    - 내부 함수가 다른 내부 함수를 호출했는지 검증(종료점이 아닌 대상을 검증).
    - 스텁이 호출되었는지 검증(들어오는 의존성을 검증).
  - 테스트당 하나를 초과하는 목을 사용
    - 하나의 테스트에 목을 두 개 이상 사용하는 것은 동일한 작업 단위의 여러 종료점을 한꺼번에 테스트하는 것과 같다.
    - 각 종료점마다 별도의 테스트를 작성하는 것이 좋다.
  - 테스트의 과도한 명세화
    - 테스트에 검증 항목이 너무 많으면 아주 작은 프로덕션 코드 변경에도 쉽게 깨질 수 있다.
    - 상호 작용을 테스트하는 것은 양날의 검과 같아서 너무 많이 테스트하면 전체 기능이라는 큰 그림을 놓치게 되고, 너무 적게 테스트하면 작업 단위 간 중요한 상호 작용을 놓치게 된다.
    - 이를 균형있게 유지하기 위해선 아래 사항을 고려해야한다.
    - 목 대신 스텁을 사용한다. 전체 테스트 중 모의 객체를 사용하는 테스트가 5% 이상이라면 너무 많이 사용하고 있는 것일지도 모른다.
    - 가능한 스텁을 목으로 사용하지 않는다. 스텁에서 메서드 호출 여부는 검증하지 않아야한다.







# 비동기 코드 단위 테스트

- 통합 테스트

  - 아래와 같이 비동기 방식으로 특정 웹사이트로부터 데이터를 가져오는 코드를 작성했다.
    - 메인 URL에서 콘텐츠를 가져와 "illustrative"라는 단어가 포함되어 있는지를 확인하여 웹 사이트가 정상인지 판단한다.
    - 아래 코드는 많은 것이 하드코딩 되어 있다.

  ```python
  import aiohttp
  import asyncio
  
  async def is_website_alive():
      try:
          async with aiohttp.ClientSession() as session:
              async with session.get("http://example.com") as resp:
                  if resp.status != 200:
                      raise Exception(resp.reason)
                  
                  text = await resp.text()
                  if "illustrative" in text:
                      return {"success": True, "status": "ok"}
                  raise Exception("text missing")
      except Exception as err:
          return {"success": False, "status": str(err)}
  
  
  async def main():
      result = await is_website_alive()
      print(result)
  
  if __name__ == "__main__":
      asyncio.run(main())
  ```

  - 단위 테스트를 실행하기 전에 통합테스트를 먼저 실행해본다.
    - pytest는 비동기 함수를 테스트하기 위한 `pytest-asyncio` 패키지를 지원한다.
    - `pip install pytest-asyncio`로 설치한다.

  ```python
  import pytest
  
  from fetch_data import is_website_alive
  
  
  @pytest.mark.asyncio
  async def test_is_website_alive():
      response = await is_website_alive()
      assert response["success"]
  ```



- 통합 테스트의 한계
  - 긴 실행 시간
    - 단위 테스트에 비해 훨씬 속도가 느리다.
  - 불안정성
    - 환경에 따라 실행 시간이 달러지거나 실패와 성공이 일관되지 않을 수 있다.
  - 테스트와는 관계없는 코드나 환경 검증
    - 테스트와 직접적인 관련이 없거나 개발자가 신경쓰지 않는 환경까지도 테스트한다.
    - 네트워크 상태, 방화벽, 외부 웹 사이트 기능 등.
  - 파악하는 데 더 많은 시간이 걸린다
    - 통합 테스트가 실패하면 실패 원인으로 의심되는 것이 다양하다.
    - 때문에 어디가 잘못되었는지 찾아보고 디버깅하는 데 더 많은 시간이 필요하다.
  - 상황을 재현하기 어려움.
    - 잘못된 웹 사이트 콘텐츠, 웹 사이트 다운, 네트워크 다운 등 부정적인 상황을 재현하는 것이 필요 이상으로 어렵다.
  - 결과를 신뢰하기 어려움
    - 통합 테스트 실패가 외부 문제라고 생각할 수 있지만 실제로는 코드 내부에 버그가 있을 수 있다.



- 코드를 단위 테스트에 적합하게 만들기
  - 코드를 단위 테스트로 테스트할 때 주로 사용되는 두 가지 패턴이 있다.
  - 진입점 분리 패턴
    - 프로덕션 코드에서 순수 로직 부분을 별도의 함수로 분리하여 그 함수를 테스트의 시작점으로 사용하는 패턴.
  - 어댑터 분리 패턴
    - 본질적으로 비동기적인 요소를 분리하고 이를 추상화하여 동기적인 요소로 대체할 수 있게 하는 패턴.



- 진입점 분리 패턴

  - 이 패턴에서는 특정 비동기 작업을 두 부분으로 나눈다.
    - 비동기 부분(이 부분은 그대로 둔다).
    - 비동기 작업이 끝난 후 실행할 부분
  - 비동기 작업이 끝난 후 실행할 부분을 새로운 함수로 분리하여 순수 논리 작업 단위의 진입점으로 사용한다.
    - 비동기 작업이 끝난 후 실행되던 부분을 분리하여 새로운 진입점으로 사용한다.

  ```python
  # website_verifier.py
  import aiohttp
  import asyncio
  
  def throw_if_response_not_ok(resp):
      if resp.status != 200:
          raise Exception(resp.reason)
  
  # 새로운 진입점
  def process_fetch_content(text):
      if "illustrative" in text:
          return {"success": True, "status": "ok"}
      return {"success": False, "status": "missing text"}
  
  # 새로운 진입점
  def process_fetch_error(err):
      raise err
  
  async def is_website_alive():
      try:
          async with aiohttp.ClientSession() as session:
              async with session.get("http://example.com") as resp:
                  throw_if_response_not_ok(resp)
                  text = await resp.text()
                  return process_fetch_content(text)
      except Exception as err:
          process_fetch_error(err)
  
  async def main():
      try:
          result = await is_website_alive()
          print(result)
      except Exception as e:
          print("에러 발생:", e)
  
  if __name__ == "__main__":
      asyncio.run(main())
  
  ```
  
  - 분리된 진입점에 대한 단위 테스트를 작성한다.
    - 이제 비동기가 아닌 부분에 대해서는 async/await와 관련된 키워드를 추가하거나 코드 실행을 기다릴 필요가 없다.
    - 이는 로직 작업 단위를 비동기 부분과 분리했기 때문에 가능하다.
  
  ```python
  def test_on_fetch_success_with_good_content():
      result = process_fetch_content("illustrative")
      assert result["success"]
      assert result["status"] == "ok"
  
  def test_on_fetch_success_with_bad_content_returns_false():
      result = process_fetch_content("text not on site")
      assert not result["success"]
      assert result["status"] == "missing text"
  
  def test_on_fetch_fail_raise():
      with pytest.raises(Exception):
          process_fetch_error(Exception)
  ```



- 어댑터 분리 패턴

  - 비동기 코드를 의존성 처럼 여기는 전략이다.
    - 테스트에서 더 쉽게 제어하고 테스트 시나리오를 다양하게 만들기 위해 대체하고 싶은 대상으로 본다.
    - 논리 코드를 별도의 진입점으로 분리하는 대신 기존 코드에서는 의존성으로 있던 비동기 코드를 분리하여 어댑터로 감싸고, 이를 다른 의존성처럼 주입할 수 있게 한다.
  - 어댑터를 사용하는 쪽 필요에 맞게 단순화된 특별한 인터페이스를 만드는 것도 일반적이다.
    - 이를 인터페이스 분리원칙(ISP)이라고도 한다.
    - 이 경우 실제 데이터를 가져오는 기능을 숨기고 자체적인 함수를 가진 네트워크 어댑터 모듈을 만들 수 있다.
    - 아래 코드는 네트워크 어댑터에 해당하는 모듈을 간단하게 표현한 것이다.
    - 함수의 이름과 기능을 모두 단순화한다.
    - URL에서 상태와 텍스트를 가져오는 과정을 숨기고, 이를 더 사용하기 쉬운 하나의 함수로 추상화한다.
  
  ```python
  # network_adapter.py
  import aiohttp
  
  
  async def fetch_url_text(url):
      async with aiohttp.ClientSession() as session:
          async with session.get(url) as resp:
              if resp.status == 200:
                  text = await resp.text()
                  return {"ok": True, "text": text}
              return {"ok": False, "text": resp.reason}
  ```
  
  - 위에서 만든 어댑터를 모듈 형식으로 사용하기
    - 이후 테스트에서는 실제 네트워크 어댑터 모듈을 가짜 모듈로 대체한다.
  
  ```python
  # website_verifier.py
  from network_adapter import fetch_url_text
  
  async def is_website_alive():
      try:
          result = await fetch_url_text("http://example.com")
          if not result["ok"]:
              raise Exception(result["text"])
          text = result["text"]
          return process_fetch_content(text)
      except Exception as err:
          process_fetch_error(err)
  ```
  
  - 모듈 형식으로 사용했을 때의 테스트 코드 작성하기
    - 네트워크 어댑터를 가짜로 대체한다.
  
  ```python
  @pytest.mark.asyncio
  async def test_with_good_content_return_true(mocker):
      mocker.patch("website_verifier.fetch_url_text", return_value={"ok": True, "text": "illustrative"})
      result = await is_website_alive()
      assert result["success"]
      assert result["status"] == "ok"
  
  @pytest.mark.asyncio
  async def test_with_bad_content_return_false(mocker):
      mocker.patch("website_verifier.fetch_url_text", return_value={"ok": True, "text": "Hello World!"})
      result = await is_website_alive()
      assert not result["success"]
      assert result["status"] == "missing text"
  ```
  
  - 어댑터를 객체 지향 형식으로 만들기.
    - 매개 변수 주입을 생성자 주입 패턴으로 확장한다.
  
  ```python
  from abc import ABC, abstractmethod
  
  from pydantic import BaseModel
  import aiohttp
  
  
  class NetworkAdapterFetchResults(BaseModel):
      ok: bool
      text: str
  
  
  class INetworkAdapter(ABC):
      @abstractmethod
      def fetch_url_text(self, url: str) -> NetworkAdapterFetchResults:
          ...
  
  
  class NetworkAdapter(INetworkAdapter):
      async def fetch_url_text(self, url: str) -> NetworkAdapterFetchResults:
          with aiohttp.ClientSession() as session:
              async with session.get(url) as resp:
                  if resp.status == 200:
                      text = await resp.text()
                      return NetworkAdapterFetchResults(**{"ok": True, "text": text})
                  return NetworkAdapterFetchResults(**{"ok": False, "text": resp.reason})
  ```
  
  - 어댑터를 사용하는 코드도 객체지향 형식으로 변경한다.
  
  ```python
  from pydantic import BaseModel
  
  from network_adapter import INetworkAdapter
  
  
  class WebsiteAliveResult(BaseModel):
      success: bool
      status: str
      
  
  class WebsiteVerifier:
      def __init__(self, network: INetworkAdapter):
          self._network = network
      
      def _process_fetch_content(self, text) -> WebsiteAliveResult:
          if "illustrative" in text:
              return WebsiteAliveResult(**{"success": True, "status": "ok"})
          return WebsiteAliveResult(**{"success": False, "status": "missing text"})
      
      def _process_fetch_error(self, err):
          raise err
      
      async def is_website_alive(self) -> WebsiteAliveResult:
          try:
              result = await self._network.fetch_url_text("http://example.com")
              if not result.ok:
                  raise Exception(result.text)
              text = result.text
              return self._process_fetch_content(text)
          except Exception as err:
              self._process_fetch_error(err)
  ```
  
  - 객체 지향 방식의 코드를 테스트하기
    - 가짜 네트워크 어댑터를 생성하고, 생성자를 사용하여 주입한다.
    - `unittest.mock.create_autospec`를 사용하면 간편하게 생성이 가능하다.
  
  ```python
  from website_verifier import WebsiteVerifier
  from network_adapter import INetworkAdapter, NetworkAdapterFetchResults
  
  from unittest.mock import create_autospec
  
  
  def make_stub_network_with_result(fake_result: NetworkAdapterFetchResults):
      stub_network = create_autospec(INetworkAdapter)
      stub_network.fetch_url_text.return_value = fake_result
      return stub_network
  
  @pytest.mark.asyncio
  async def test_with_good_content_return_true():
      stub_network = make_stub_network_with_result(NetworkAdapterFetchResults(**{"ok": True, "text": "illustrative"}))
      web_verifier = WebsiteVerifier(stub_network)
      result = await web_verifier.is_website_alive()
      assert result.success
      assert result.status == "ok"
  
  @pytest.mark.asyncio
  async def test_with_bad_content_return_false():
      stub_network = make_stub_network_with_result(NetworkAdapterFetchResults(**{"ok": True, "text": "Hello World!"}))
      web_verifier = WebsiteVerifier(stub_network)
      result = await web_verifier.is_website_alive()
      assert not result.success
      assert result.status == "missing text"
  ```







# 신뢰할 수 있는 테스트

- 좋은 테스트는 아래 세 가징 특성을 만족해야한다.
  - 신뢰성
    - 아무런 의심과 걱정 없이 테스트 결과를 받아볼 수 있어야한다.
    - 신뢰할 수 있는 테스트란 버그가 없고 올바른 대상을 테스트함을 의미한다.
  - 유지 보수성
    - 유지 보수성이 떨어지는 테스트는 항상 골칫거리다.
    - 프로젝트 일정이 촉박한 경우 아무도 챙길 여유가 없어 그대로 방치될 수 있다.
    - 코드가 조금만 바뀌어도 테스트를 계속 수정해야 한다면 결국 유지 보수에 지친 개발자들이 손을 놓는다.
  - 가독성
    - 가독성은 테스트를 읽을 수 있는 것 외에도 테스트가 잘못된 경우 문제를 파악할 수 있는 능력을 의미한다.
    - 가독성이 없으면 신뢰성이나 유지 보수성도 큰 힘을 발휘하지 못한다.
    - 테스트를 유지 보수하기 어려워지고 이해하지 못하므로 결국 테스트를 신뢰할 수 없게 된다.



- 테스트를 신뢰할 수 있는지 판단하는 방법
  - 일반적으로 아래 상황에서는 테스트를 신뢰하지 않는다.
    - 테스트는 실패했지만 이는 거짓 양성 때문이라고 생각한다.
    - 테스트가 가끔 통과하거나 테스트가 현재 작업과 관련 없다고 생각하거나 테스트에 버그가 있다고 느껴 테스트 결과를 무시한다.
    - 테스트가 통과했지만 의심스러워 거짓 음성이라고 생각한다.
    - 만약에 대비하여 직접 디버깅하거나 프로그램을 테스트할 필요를 느낀다.
  - 아래 상황에서는 테스트를 신뢰한다.
    - 테스트가 실패했을 때, 코드의 무언가가 잘못되었을까 봐 진심으로 걱정하여 쉽게 넘어가지 않고 테스트가 틀렸다고 생각한다.
    - 테스트가 통과하면 따로 수동으로 테스트하거나 디버깅할 필요가 없다고 여긴다.





## 테스트가 실패하는 이유

- 테스트가 실패하는 이유

  - 단위 테스트를 포함한 모든 종류의 테스트 실패는 합당한 이유가 있어야 납득할 수 있다.
    - 합당한 이유란 실제 버그가 프로덕션 코드에서 발견된 경우를 의미한다.
  - 테스트는 다양한 이유로 실패할 수 있다.
    - 실제 버그 외의 다른 이유로 테스트가 실패한다면 그 테스트는 신뢰할 수 없는 것으로 간주해야 한다.
    - 모든 테스트가 동일한 방식으로 실패하는 것은 아니므로 실패하는 이유를 파악하면 각 상황에 맞는 대응 방안을 마련하는 데 도움이 된다.

  - 테스트가 실패하는 이유는 아래와 같다.
    - 프로덕션 코드에서 실제 버그가 발견된 경우.
    - 테스트가 거짓 실패를 일으키는 경우.
    - 기능 변경으로 테스트가 최신 상태가 아닌 경우.
    - 테스트가 다른 테스트와 충돌하는 경우.
    - 테스트가 불안정한 경우.
  - 위 이유 중에서 첫 번째 이유를 제외한 나머지 이유는 모두 현재 단계에서는 테스트를 신뢰할 수 없음을 의미한다.



- 테스트가 거짓 실패를 일으키는 경우

  - 테스트 자체에 버그가 있으면 거짓 실패(프로덕션 코드에는 문제가 없지만 테스트가 실패)가 발생할 수 있다.
    - 종료점의 예상 결과를 잘못 설정했거나 테스트 대상 시스템(SUT)을 잘못 사용한 것이 원인일 수 있다.
    - 아니면 테스트 환경을 잘못 설정했거나 테스트해야 하는 내용을 잘못 이해했을 수도 있다.
    - 테스트에 버그가 있으면 테스트가 통과하더라도 실제로는 문제가 있는 상황을 발견하지 못할 수도 있다.

  - 테스트에 버그가 있는지 찾아내는 방법.
    - 테스트가 실패했는데 프로덕션 코드를 디버깅해도 버그를 찾을 수 없을 때는 테스트 코드 자체를 의심해야한다.
    - 아래 항목에 해당하면 거짓 실패를 일으킨다고 볼 수 있다.
    - 잘못된 항목이나 잘못된 종료점을 검증하는 경우.
    - 잘못된 값을 진입점에 전달하는 경우.
    - 진입점을 잘못 호출하는 경우.
  - 해결 방법
    - 테스트의 버그를 수정하고 다시 실행하여 통과하는지 확인한다.
    - 이 때 통과했다고 해서 바로 안심하지 말고, 프로덕션 코드에 일부러 버그를 넣어 테스트가 실패하는지도 확인한다.
  - 예방법
    - TDD 방식으로 코드를 작성하면 코드의 어디가 잘못되었는지 쉽게 찾을 수 있어 버그를 미리 막을 수 있다.
    - 테스트 주도 개발은 이러한 목적으로 사용하기에 가장 좋은 방법 중 하나이다.



- 기능 변경으로 테스트가 최신 상태가 아닌 경우
  - 기능이 변경되면 테스트가 현재 기능과 맞지 않아 실패할 수 있다.
    - 예를 들어 로그인 기능의 이전 버전에서는 사용자 이름과 비밀번호를 입력해야 로그인이 가능했다.
    - 그러나 새로운 버전에서는 이중 인증 방식이 도입되었다면, 기존 테스트는 로그인 함수에 필요한 값을 제대로 제공하지 못해 실패한다.
  - 해결 방법
    - 테스트를 새로운 기능에 맞게 수정한다.
    - 새로운 기능을 대상으로 새 테스트를 만들고 기존 테스트는 삭제한다.



- 테스트가 다른 테스트와 충돌하는 경우
  - 결코 동시에 성공할 수 없는 두 개의 테스트가 있을 수 있다.
    - 이 경우 일반적으로 통과하는 테스트는 눈에 띄지 않기 때문에 우리는 실패하는 테스트만 보게 된다.
    - 예를 들어 어떤 테스트는 새로운 기능과 호환이 되지 않아 실패하는데, 또 다른 테스트는 새로운 기능이 있어야 하는데 그 기능이 없어 실패할 수 있다.
  - 이러한 문제가 발생하는 원인.
    - 근본적인 원인은 두 테스트 중 하나가 더 이상 쓸모없어졌다는 것이다.
  - 해결 방법
    - 어떤 기능이 필요한지, 어떤 기능이 서비스의 요구 사항을 만족하는지를 파악하여 쓸모없어진 기능을 테스트하는 코드를 없애야한다.
  - 예방법
    - 테스트와 기능이 발전하는 과정에서 자연스러운 현상이다.
    - 따라서 굳이 예방하려 하지 않아도 된다.



- 테스트가 불안정한 경우
  - 테스트가 불규칙하게 실패할 때가 있다.
    - 프로덕션 코드가 바뀌지 않았는데도 테스트가 갑자기 실패했다가 다시 통과하고 또다시 실패하는 경우가 있다.
    - 이러한 테스트를 불안정한 테스트라고 한다.
  - 이는 특수한 경우이므로 추후에 다시 다룬다.





## 단위 테스트에서 불필요한 로직 제거하기

- 테스트에 로직을 많이 넣을수록 테스트에 버그가 생길 확률이 기하급수적으로 증가한다.
  - 아래와 같은 경우 테스트에 버그가 생길 확률이 높아진다.
    - 간단하게 끝날 테스트가 동적인 로직을 갖는다.
    - 난수를 생성한다.
    - 스레드를 만든다.
    - 파일을 쓴다.
  - 테스트 코드도 코드이기에 버그가 있을 수 있다는 사실을 항상 염두에 두어야한다.
    - 철저한 테스트를 만들기 위해 테스트가 복잡해질수록 테스트 자체의 버그가 발생할 확률도 올라간다는 사실을 기억해야한다.
    - 물론 로직이 있는 테스트가 전혀 가치가 없다는 것은 아지만 가능한 지양해야 한다.
  - 아래 내용이 단위 테스트에 포함되어 있다면 불필요한 로직이 포함된 것이므로 이를 줄이거나 완전히 없애는 것이 좋다.
    - switch, if, else문
    - foreach, for, while 루프
    - 문자열 연결 등
    - try / catch 블록



- Assert문에서 로직

  - 아래는 잘 못 작성한 테스트 코드의 예시이다.
    - "hello"와 name을 연결하는 방식이 테스트 코드와 테스트 대상 코드에서 모두 반복된다.
    - 문자열을 연결하는 것은 매우 단순하긴 하지만 분명한 로직이며, 테스트 대상 코드에서 사용하는 로직이 테스트 코드에서도 그대로 사용되고 있다.
    - 만약 테스트 대상 코드에 버그가 있다면, 테스트에서도 동일한 버그가 발생할 것이고, 테스트는 버그를 잡아내지 못하고 오히려 잘못된 결과를 내놓게 된다.

  ```python
  # 테스트 대상 코드
  def make_greeting(name: str):
      return f"hello {name}"
  
  # 테스트 코드
  def test_return_correct_greeting_for_name():
      name = "foo"
      result = make_greeting(name)
      assert result == "hello "+ name
  ```

  - 테스트 코드가 실제 코드와 동일한 로직을 사용하면 테스트는 신뢰할 수 없다.
    - 실제 코드에 버그가 있어도 테스트가 이를 감지하지 못하고 통과할 수 있기 때문이다.
    - 테스트는 항상 실제 코드와 다른 방식으로 기댓값을 설정해야 버그를 제대로 잡아낼 수 있다.

  - 검증 단계에서 기댓값을 동적으로 생성하지 말고 가능하면 하드코딩된 값을 사용해야 한다.
    - 하드코딩하면 "foo"가 두 번 반복되어 유지보수성이 낮아진다고 생각할 수 있다.
    - 그러나 유지 보수성보다 신뢰성이 훨씬 더 중요하다.

  ```python
  def test_return_correct_greeting_for_name():
      name = "foo"
      result = make_greeting(name)
      assert result == "hello foo"
  ```



- 다른 형태의 로직

  - 아래는 잘 못 작성한 테스트 코드이다.
    - 아래 테스트는 여러 입력값을 사용하고 있는데, 이는 입력값을 반복해서 사용함으로써 테스트를 복잡하게 한다.
    - 또 값에 따라 공백이 있는 경우와 없는 경우를 처리하는데 if/else문이 필요하다.
    - 또한 프로덕션 코드에서 이미 사용한 로직을 테스트 코드에서 반복하고 있다.
    - 마지막으로 테스트 이름이 너무 모호한데, 이는 여러 테스트 시나리오와 테스트 기댓값을 모두 포함하다 보니 이름을 "잘 동작한다" 정도밖에는 지을 수 없기 때문이다.

  ```python
  # 테스트 대상 코드
  def is_name(value: str):
      return len(value.split(" ")) == 2
  
  # 테스트 코드
  def test_collectly_find_out_name():
      names_to_test = ["fisrt_only", "first second", ""]
      for name in names_to_test:
          result = is_name(name)
  
          if " " in name:
              assert result
          else:
              assert not result
  ```

  - 테스트가 복잡해지면 아래 문제가 생길 수 있다.
    - 테스트를 읽고 이해하기 어렵다.
    - 테스트를 재현하기가 어렵다.
    - 테스트에 버그가 있을 가능성이 높아지거나 잘못된 것을 검증할 수 있다.
    - 테스트가 여러 가지 일을 하므로 이름 짓기가 어렵다.
  - 복잡한 테스트는 다양한 상황을 다루다 보니 특정 문제를 정확히 파악하기 어려워 실제 버그를 놓칠 가능성이 높다.



- 로직이 더 많이 포함된 경우
  - 테스트에는 다양한 로직이 추가될 수도 있다.
    - 로직은 테스트뿐만 아니라 테스트에 필요한 헬퍼 함수, 직접 작성한 가짜 객체, 테스트 유틸리티 클래스에서도 찾아볼 수 있다.
    - 이러한 곳에 로직을 추가할수록 코드를 읽기 어렵게 만들고 테스트에서 사용하는 헬퍼 함수에 버그가 발생할 가능성도 높다.
  - 어떤 이유로든 테스트에 복잡한 로직이 필요하다면 최소한 유틸리티 함수의 로직을 검증하는 몇 가지 테스트를 추가하는 것이 좋다.





## 테스트가 통과하더라도 끝이 아니다.

- 모든 테스트가 통과했다고 해서 테스트를 신뢰할 수 있는 것은 아니다.
  - 잘못된 신뢰(false trust)
    - 신뢰하지 말아야 할 테스트를 신뢰하지만 그 사실을 아직 모르는 상태를 의미한다.
    - 테스트를 검토하여 잘못된 신뢰를 찾아내는 것은 매우 중요하다.
  - 테스트가 통과하더라도 그 테스트를 믿지 못하는 이유는 아래와 같다.
    - 검증 부분이 없는 경우.
    - 테스트를 이해할 수 없는 경우.
    - 단위 테스트가 불안정한 통합 테스트와 섞여 있는 경우.
    - 테스트가 여러 가지를 한꺼번에 검증하는 경우.
    - 테스트가 자주 변경되는 경우



- 검증 부분이 없는 경우
  - 무언가가 참인지 거짓인지 확인하지 않는 테스트는 그다지 도움이 되지 않는다.
    - 유지 보수, 리팩터링, 코드를 읽는 데 시간이 든다.
    - 때로는 프로덕션 코드의 API 변경으로 불필요한 수정이 필요하다.
  - 테스트에 검증(assert) 부분이 없으면 함수 호출 내 검증 로직이 숨어 있을 수 있다.
    - 그러나 함수 이름에 어떤 설명도 포함하지 않으면 가독성이 떨어질 수밖에 없다.
  - 누군가는 단순히 코드를 실행하여 예외가 발생하지 않는지 확인하는 테스트를 작성하기도 한다.
    - 이런 테스트도 어느 정도 가치는 있지만 테스트 이름에  "예외 없음" 같은 용어를 포함하여 이를 명확히 해야 한다.
    - 
    - 그러나 이를 표준으로 삼기보다는 정말 특별한 경우에만 사용하는 것이 좋다.



- 테스트를 이해할 수 없는 경우
  - 아래와 같은 테스트들은 이해하기 어렵다.
    - 이름이 적절하지 않은 테스트
    - 코드가 너무 길거나 복잡한 테스트
    - 변수 이름이 헷갈리게 되어 있는 테스트
    - 숨어 있는 로직이나 이해하기 어려운 가정을 포함한 테스트
    - 결과가 불분명한 테스트(실패도 아니고 통과도 아닌 경우)
    - 충분한 정보를 제공하지 않는 테스트 메시지
  - 중요한 문제이므로 이후에 자세히 다룬다.



- 단위 테스트가 불안정한 통합 테스트와 섞여 있는 경우
  - 통합 테스트는 단위 테스트보다 의존성이 많아 불안정할 가능성이 더 높다.
    - 이러한 테스트가 같은 폴더에 있거나 하나의 테스트 실행 명령어로 함께 실행된다면 이를 의심해보아야 한다.
    - 불안정한 테스트와 안정적인 테스트가 섞여 있으면 테스트가 실패할 경우 불안정한 테스트 때문으로 치부하고 버그를 찾으려는 노력을 기울이지 않을 수 있다.
  - 안정적인 테스트 영역(safe green zone)을 만들어야한다.
    - 안정적인 테스트와 불안정한 테스트를 분리해야한다.
    - 이렇게 하면 테스트가 실패했을 때 테스트가 불안정하다고 탓할 수 있는 여지가 사라지게 된다.
    - 안정적인 테스트 영역에는 빠르고 신뢰할 수 있는 테스트만 포함되어야한다.
    - 이 영역의 일부 테스트가 통과하지 않을 경우 개발자는 더 신경써서 문제를 살펴보게 된다.



- 테스트가 여러 가지를 한꺼번에 검증하는 경우

  - 종료점(또는 관심사)은 하나의 작업 단위에서 나오는 최종 결과를 의미한다
    - 반환 값, 상태 값 변경, 서드 파티 호출 등이 이에 해당한다.

  - 아래는 종료점 두 개, 즉 관심사를 두 개 가진 함수 예시이다.
    - 이 함수는 값을 반환하면서 동시에 매개변수로 전달된 콜백 함수를 실행한다.

  ```python
  def trigger(x, y, callback):
      callback("I'm triggered")
      return x + y
  ```

  - 위 두 가지 종료점을 동시에 확인하는 테스트는 다음 예제처럼 작성할 수 있다.
    - 하나의 테스트에서 여러 가지를 검증하면서 테스트 이름이 모호해졌다.
    - 더 큰 문제는 대부분의 단위 테스트 프레임워크에서 검증이 실패하면 테스트 프레임워크가 특별한 예외를 던진다는 것이다.
    - 테스트 프레임워크가 그 예외를 잡아내면 테스트는 실패한 것으로 처리된다.
    - 대부분의 프로그래밍 언어는 예외가 발생하면 코드 실행이 중단되므로 이후의 테스트는 실행되지 않는다.
    - 예를 들어 아래 코드에서 `assert result == 2`와 같이 작성해서 검증이 실패한다면 그 아래의 테스트는 실행되지 않는다.
    - 따라서 검증이 실패한 이후의 테스트는 성공 여부를 알 수 없게 된다.

  ```python
  from unittest.mock import Mock
  
  def test_works():
      callback = Mock()
      result = trigger(1, 2, callback)
      assert result == 3
      callback.assert_called_with("I'm triggered")
  ```

  - 각 검증은 별도의 테스트로 나누어서 실행하여 무엇이 실제로 실패했는지 명확히 확인하는 것이 좋다.
    - 여러 가지를 한꺼번에 확인하는 것은 복잡성만 늘릴 뿐 별로 도움이 되지 않는다.

  ```python
  def triggers_given_callback():
      callback = Mock()
      trigger(1, 2, callback)
      callback.assert_called_with("I'm triggered")
  
  def sum_up_given_values():
      result = trigger(1, 2, lambda x: x)
      assert result == 3
  ```

  - 단, 여러 관심사를 다루지 않을 경우 한 테스트에서 여러 가지를 검증해도 괜찮을 수 있다.
    - 아래 테스트에서 검증하는 name과 age는 모두 같은 객체의 일부로, 같은 관심사에 해당하기에 함께 검증해도 무방하다.
    - 첫 번째 검증(name 검증)이 실패하면 객체를 만드는 과정에서 큰 문제가 발생했을 가능성이 있으므로 두 번째 검증(age 검증)은 그다지 중요하지 않게 된다.
    - 만약 첫 번째 검증이 실패했을 때, 다음 검증 결과가 여전히 중요하다면 각 검증을 서로 다른 테스트 두 개로 독립적으로 진행하는 것이 좋다.

  ```python
  def make_person(name: str, age: int):
      return {
          "name": name,
          "age": age,
          "type": "person"
      }
      
  def test_create_person_given_passed_in_values():
      result = make_person("foo", 38)
      assert result["name"] == "foo"
      assert result["age"] == 38
  ```



- 테스트가 자주 변경되는 경우
  - 아래와 같은 테스트들은 실행할 때마다 다른 테스트가 된다고 할 수 있다.
    - 현재 날짜와 시간을 사용하는 테스트.
    - 난수, 컴퓨터 이름, 외부 환경 변수 값을 가져오는 테스트
    - 그 밖에 동적으로 만든 값을 사용하는 테스트
  - 이러한 테스트들은 결과가 일관되지 않을 가능성이 크기 때문에 불안정할 수 있다.
    - 테스트가 불안정하면 테스트 결과를 신뢰하기 어렵다.
    - 동적으로 만든 값을 테스트에서 사용하는 것은 문제가 생길 여지가 많다.
    - 이 경우 입력값을 미리 알 수 없기 때문에 예상 기댓값도 함께 계산해야한다.
    - 그러나 이는 프로덕션 코드의 로직을 반복하게 되어 테스트에서도 동일한 버그를 발생시킬 수 있다.





## 불안정한 테스트 다루기

- 불안정한 테스트
  - 코드에는 아무런 변화가 없는데도 일관성 없는 결과를 반환하는 테스트를 설명하는 용어이다.
    - 테스트 수준이 높아질수록(즉 유닛테스트에서 통합테스트로 갈수록)실제 의존성을 더 많이 사용하게 된다.
    - 상위 단계로 갈수록 변동 요소가 많아 테스트가 불안정해질 가능성도 높다.
    - 더 많은 의존성이 포함된 테스트는 시스템이 제대로 작동한다는 신뢰성은 높이지만 동시에 불안정성도 증가시킨다.
  - 낮은 수준의 테스트(단위 테스트)
    - 가장 낮은 수준에 해당하는 단위 테스트에서는 테스트가 모든 의존성을 완전히 제어할 수 있어 변동 요소가 없다.
    - 이는 의존성을 가짜로 만들거나 메모리에서만 실행되기 때문이다.
    - 모든 초기 상태와 의존성이 반환하는 값이 미리 설정되어 있어 코드가 거의 고정된 방식으로 실행되므로 코드 실행을 완전히 예측할 수 있다.
    - 예상과 다른 결과가 나온다면 이는 프로덕션 코드의 실행 경로나 로직에 중요한 변화가 생겼음을 의미한다.
  - 높은 수준의 테스트(통합 테스트)
    - 테스트 수준이 올라갈수록 스텁과 모의 객체를 덜 사용하고 DB, 네트워크, 환결 설정 등 실제 의존성을 더 많이 사용한다.
    - 그 결과 제어할 수 없는 변동 요소가 많아지는데 실행 경로가 변경되거나, 예상치 못한 값을 반환하거나, 실행 자체가 실패할 수 있음을 의미한다.
    - 최상위 수준의 테스트는 서드 파티, 보안 및 네트워크 계층, 환경 설정 등의 모든 의존성을 실제로 사용한다.
    - 상위 단계로 갈수록 변동 요소가 많아 테스트가 불안정해질 가능성도 높다.
    - 또한 상위 수준으로 갈수록 추가되는 의존성은 테스트 실행 시간을 늘리며, 테스트가 왜 실패하는지 파악하는 시간도 늘린다.



- 불안정한 테스트를 발견했을 때 할 수 있는 일
  - 불안정한 테스트는 없애야한다.
    - 불안정한 테스트는 장기적으로 팀, 조직, 회사에 큰 비용을 발생시킬 수 있다.
    - 불안정한 테스트를 수정하여 안정적으로 만들거나, 불필요할 경우 제거해야한다.
  - 문제 정의하기
    - 불안정이 무엇을 의미하는지 명확히 정의해야 한다.
    - 예를 들어 프로덕션 코그를 변경하지 않고 테스트 케이스를 열 번 실행한 후 결과가 일관되지 않은 테스트를 모두 확인한다.
    - 불안정하다고 판단된 테스트는 별도의 폴더에 따로 모아 실행할 수 있도록 한다.
    - 이를 통해 이 테스트들을 메인 배포 파이프라인에서 제외시켜 배포시에 불필요한 잡음을 줄일 수 있다.
  - 수정
    - 가능한 경우 의존성을 제어하여 테스트 안정성을 높인다.
    - 예를 들어 DB에 특정 데이터가 필요하다면 테스트 과정에서 해당 데이터를 DB에 삽입한다.
  - 리랙터링
    - 의존성을 제거하거나 제어하여 테스트를 더 낮은 수준의 테스트로 변환해서 불안정성을 제거한다.
    - 예를 들어실제 서버 대신 스텁을 사용하여 네트워크 요청을 처리하게 할 수 있다.
  - 삭제
    - 테스트로 얻는 이점이 유지 보수 비용을 감당하고도 남을 만큼 충분한지 고민해볼 필요가 있다.
    - 오래되고 불안정한 테스트는 차라리 없애는 것이 나을 수 있다.



- 상위 수준의 테스트에서 안정성을 유지하는 방법
  - 고수준 테스트 안정성을 지속적으로 유지하려면 어떤 환경에서도 배포 후에 테스트가 반복적으로 실행될 수 있도록 하는 것이 중요하다.
  - 이를 위해 아래 방법을 고려해볼 필요가 있다.
    - 테스트가 DB나 네트워크 같은 외부 시스템을 변경했으면 변경한 내용을 롤백한다.
    - 다른 테스트가 외부 시스템의 상태를 변경하지 않도록 한다.
    - 외부 시스템과 의존성을 제어할 수 있어야한다.
    - 예를 들어 해당 시스템을 언제든지 다시 만들 수 있도록 하거나([인프라를 코드로 관리하기] 참고) 제어 가능한 더미 시스템을 만들거나, 테스트 전요 계정을 만들어 안전하게 관리한다.
  - 외부 시스템을 다른 회사에서 관리하는 경우 외부 의존성을 제어하기가 힘들거나 불가능할 수 있는데, 이 상황에서는 아래 방법을 고려해 볼 수 있다.
    - 저수준 테스트가 이미 특정 기능이나 동작을 검증하고 있다면 일부 고수준 테스트를 삭제한다.
    - 일부 고수준 테스트를 저수준 테스트로 바꾼다.
    - 새로운 테스트를 작성할 때는 배포 파이프라인에 적용하기 쉬운 테스트 전략과 방법을 고려한다.





