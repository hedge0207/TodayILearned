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



