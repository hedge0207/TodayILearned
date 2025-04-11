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
