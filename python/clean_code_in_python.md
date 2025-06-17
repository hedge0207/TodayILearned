# SOLID

- SRP

  - 아래 예시에서 `SystemMonitor` 클래스는 SRP를 지키지 않고 있다.
    - 독립적인 동작을 하는 메서드드들을 하나의 인터페이스에 정의하여 사용하고 있다.
    - 각각의 동작은 나머지 부분과 독립적으로 수행할 수 있다.
    - 각 메서드는 클래스의 책임을 의미하며, 각각의 책임마다 수정 사유가 발생한다.
    - 즉, 소스에서 이벤트를 가져오는 방식이 변경되거나, 파싱하는 방식이 변경되거나, 외부 에이전트로 전송하는 방식이 변경되면 클래스를 수정해야한다

  ```python
  class SystemMonitor:
      def load_activity(self):
          """소스에서 처리할 이벤트 가져오기"""
      
      def identify_events(self):
          """가져온 데이터를 파싱하여 도메인 객체 이벤트로 변환"""
      
      def stream_events(self):
          """파싱한 이벤트를 외부 에이전트로 전송"""
  ```

  - 책임 분산
    - 모든 메서드를 각기 다른 클래스로 분리하여 각 클래스마다 단일한 책임을 갖게한다.

  ```python
  class ActivityWatcher:
      def load(self): ...
      
      
  class SystemMonitor:
      def identify(self): ...
      
  class Output:
      def stream(self): ...
  ```



- OCP

  - 아래와 같은 코드가 있다고 가정해보자.
    - 다른 시스템에서 발생하는 이벤트를 분류하는 기능을 위한 코드이다.
    - 각 클래스는 수집한 데이터를 기반으로 어떤 타입의 이벤트인지 정확히 분류해야한다.

  ```python
  @dataclass
  class Event:
      raw_data: dict
  
  class UnknownEvent(Event): 
      """데이터만으로 식별할 수 없는 이벤트"""
      
  class LoginEvent(Event): 
      """로그인 이벤트"""
      
  class LogoutEvent(Event): 
      """로그아웃 이벤트"""
      
  class SystemMonitor:
      def __init__(self, raw_data: dict):
          self.raw_data = raw_data
      
      def identify(self) -> Event:
         	if self.raw_data["before"]["session"] == 0 and self.raw_data["after"]["session"] == 1:
              return LoginEvent(self.raw_data)
          elif self.raw_data["before"]["session"] == 1 and self.raw_data["after"]["session"] == 0:
              return LogoutEvent(self.raw_data)
          return UnknownEvent(self.raw_data)
  ```

  - 위 코드는 OCP를 위반한다.
    - 이벤트 유형을 결정하는 로직이 단일 메서드에 중앙 집중화된다.
    - 지원하려는 이벤트가 늘어날수록 메서드도 커질 것이므로 결국 매우 큰 메서드가 될 수 있다.
    - 새로운 유형의 이벤트를 시스템에 추가할 때마다 메서드를 수정해야한다.
  - 개선하기
    - 위 코드의 문제점은 `SystemMonitor` 클래스가 분류하려는 구체 클래스와 직접 상호작용한다는 점이다.
    - `SystemMonitor`가 추상적인 클래스와 협력하도록 변경하고, 구체 클래스에 대응하는 개별 로직은 각 구체 클래스에 위임한다.

  ```python
  class Event:
      def __init__(self, raw_data: dict):
          self._raw_data = raw_data
      
      @staticmethod
      def meets_condition(raw_data: dict) -> bool:
          return False
      
      
  class UnknownEvent(Event):
      ...
      
  
  class LoginEvent(Event):
      @staticmethod
      def meets_condition(raw_data: dict) -> bool:
          return raw_data["before"]["session"] == 0 and raw_data["after"]["session"] == 1
  
  
  class LogoutEvent(Event):
      @staticmethod
      def meets_condition(raw_data: dict) -> bool:
          return raw_data["before"]["session"] == 1 and raw_data["after"]["session"] == 0
      
      
  class SystemMonitor:
      def __init__(self, raw_data: dict):
          self._raw_data = raw_data
      
      def identify_event(self):
          for event_cls in Event.__subclasses__():
              try:
                  if event_cls.meets_condition(self._raw_data):
                      return event_cls(self._raw_data)
              except KeyError:
                  continue
          return UnknownEvent(self._raw_data)
  ```

  - 위 코드는 변경에는 닫혀 있으면서 확장에는 열려있다.
    - 새로운 유형의 이벤트가 추가되더라도, 기존 이벤트를 수정할 필요가 없다.
    - 또한 새로운 이벤트를 추가해야하면 단순히 `Event`를 상속 받는 클래스를 생성 후 `meets_condition` 메서드를 오버라이드하기만 하면 된다.



- LSP

  - LSP의 핵심은 클라이언트가 특별히 주의를 기울이지 않고도 부모 클래스를 대신하여 하위 클래스를 그대로 사용할 수 있어야 한다는 것이다.
    - 즉 클라이언트는 부모 타입 대신에 어떠한 하위 타입을 사용해도 정상적으로 동작해야한다.
    - 클라이언트는 자신이 사용하는 클래스의 계층 구조가 변경되는 것에 대해 알 필요가 없으며, 그 변경 사항에 대해 완전히 독립적이어야한다.
  - 코드에 타입 어노테이션을 적용한 상태에서 mypy, pylint등을 사용하면 LSP를 준수하는지 여부를 확인할 수 있다.
    - 예를 들어 아래 코드를 mypy로 검사하면 'error: Argument 1 of"meets_condition" incompatible with supertype "Event"'와 같은 메시지가 출력된다.

  ```python
  class Event:
      def meets_condition(self, raw_data: dict) -> bool:
          ...
          
  class LoginEvent:
      def meets_condition(self, raw_data: list) -> bool:
          ...
  ```

  - 사전조건과 사후조건
    - 하위 클래스는 상위 클래스에 정의된 것 보다 엄격한 사전조건을 만들면 안 된다.
    - 하위 클래스는 상위 클래스에 정의된 것 보다 완화된 사후조건을 만들면 안 된다.
    - 이 유형의 LSP 위반은 mypy 등으로 감지할 수 없다.
  - 사전조건 예시
    - 아래 예시에서 사전 조건은 `Event.validate_precondition` 메서드에 명시되어 있다.
    - `"before"`와 `"after"`가 필수이고, 그 값은 dict 타입이어야 한다는게 사전 조건의 전부다.

  ```py
  from collections.abc import Mapping
  
  class Event:
      def __init__(self, raw_data):
          self._raw_data = raw_data
          
      @staticmethod
      def meets_condition(raw_data: dict) -> bool:
          return False
      
      @staticmethod
      def validate_precondition(raw_data: dict):
          """사전 조건"""
          if not isinstance(raw_data, Mapping):
              raise ValueError(f"{raw_data} is not dict type")
          for moment in ("before", "after"):
              if moment not in raw_data:
                  raise ValueError(f"{moment} does no included in {raw_data}")
              if not isinstance(raw_data[moment], Mapping):
                  raise ValueError(f"{raw_data[moment]} is not dict type")
                  
  
  class SystemMonitor:
      def __init__(self, raw_data):
          self._raw_data = raw_data
      
      def identify_event(self):
          Event.validate_precondition(self._raw_data)
          event_cls = next(
              (
                  event_cls 
                  for event_cls in Event.__subclasses__() 
                  if event_cls.meets_condition(self._raw_data)
              ),
              UnknownEvent
          )
          return event_cls(self._raw_data)
  ```

  - 더 엄격한 사전 조건
    - 아래 코드에서 `meets_condition`은 `Event`와 `SystemMonitor` 사이에 합의된 사전조건 보다 더 엄격한 사전 조건을 요구한다.
    - `"after"`와 `"before"`의 값은 `"session"`이라는 키도 가져야한다.
    - 이는 더 엄격한 사전조건으로 명백한 LSP 위반이다.

  ```python
  class LoginEvent(Event):
      @staticmethod
      def meets_condition(raw_data: dict) -> bool:
          return raw_data["before"]["session"] == 0 and raw_data["after"]["session"] == 1
  
  
  class LogoutEvent(Event):
      @staticmethod
      def meets_condition(raw_data: dict) -> bool:
          return raw_data["before"]["session"] == 1 and raw_data["after"]["session"] == 0
  ```

  - 개선하기
    - 아래와 같이 `dict.get` 메서드를 사용하도록 변경함으로써 개선이 가능하다.
    - 이제는 `"after"`, `"before"`의 값은 `"session"`이라는 키를 가지지 않아도 에러가 발생하지 않는다.

  ```python
  class LoginEvent(Event):
      @staticmethod
      def meets_condition(raw_data: dict) -> bool:
          return raw_data["before"].get("session") == 0 and raw_data["after"].get("session") == 1
  
  
  class LogoutEvent(Event):
      @staticmethod
      def meets_condition(raw_data: dict) -> bool:
          return raw_data["before"].get("session") == 1 and raw_data["after"].get("session") == 0
  ```



- ISP

  - Python에서 인터페이스는 메서드의 형태를 보고 암시적으로 정의된다.
    - 이는 Python이 duck typing을 지원하는 언어이기 때문이다.
    - Duck typing에서 객체의 본질을 정의하는 것은 궁극적으로 객체가 가지고 있는 메서드이다.
    - 오랫동안 duck typing은 Python에서 인터페이스를 정의하는 유일한 방법이었으나, PEP-3119에서 인터페이스를 다른 방식으로 정의하는 추상 기본 클래스 개념이 도입되었다.
  - 추상 기본 클래스
    - 파생 클래스가 반드시 구현해야 하는 것을 명시적으로 가리키기 위한 도구이다.
    - `abc` 모듈에는 가상 서브클래스를 계층 구조의 일부로 등록하는 방법도 포함되어 있다.
  - ISP의 핵심은 아래와 같다.
    - 여러 메서드를 가진 인터페이스가 있다면, 정확하고 구체적인 구분에 따라 더 적은 수의 메서드를 가진 여러 개의 인터페이스로 분할하는 것이 좋다.
    - 이를 통해 재사용성을 높일 수 있으며, 각 클래스는 매우 명확한 동작과 책임을 지니게 되어 응집력이 높아진다.
  - 거대한 인터페이스 예시
    - 여러 데이터 소스에서 이벤트를 파싱하는 인터페이스이다.
    - 아래 인터페이스를 구현하는 구체 클래스는 모든 메서드를 구현해야만한다.
    - 문제는 어떤 클래스는 XML을 파싱할 필요가 없고, 또 다른 클래스는 JSON을 파싱할 필요가 없더라도 모두 구현을 해야한다는 것이다.
    - 이는 클래스의 응집력을 낮춘다.

  ```python
  class EventParser(ABC):
      @abstractmethod
      def from_xml(self): ...
      
      @abstractmethod
      def from_json(self): ...
      
      @abstractmethod
      def from_csv(self): ...
  ```

  - 위 인터페이스는 각각 하나의 메서드를 가진 세 개의 인터페이스로 분리하는 것이 좋다.

  ```python
  class XMLEventParser(ABC):
      @abstractmethod
      def from_xml(self): ...
  
  class JSONEventParser(ABC):
      @abstractmethod
      def from_json(self): ...
      
  class CSVEventParser(ABC):
      @abstractmethod
      def from_csv(self): ...
  ```

  - 만약 둘 이상의 기능이 필요할 경우 두 인터페이스를 구현하는 클래스를 만들면 된다.

  ```python
  class EventParser(XMLEventParser, JSONEventParser):
      def from_xml(self): ...
      
      def from_json(self): ...
  ```



- DIP

  - 강한 의존성 예시
    - 아래 코드는 저수준의 내용(`Syslog`)에 따라 고수준의 클래스가 변경되어야 하므로 별로 좋은 디자인이 아니다.
    - 만약 `SysLog`로 데이터를 보내는 방식이 변경되면, `EventStreamer`를 수정해야한다.
    - 만약 다른 데이터에 대해서는 전송 목적지를 변경하거나 새로운 데이터를 추가하려면 `EventStream.stream()`메서드를 이러한 요구사항에 따라 지속적으로 수정해야 하므로 문제가 생긴다.

  ```python
  class Syslog:
      def send(self, data: str):
          print(f"[Syslog] {data} sent to syslog server")
  
  
  class EventStreamer:
      def __init__(self):
          self.syslog = Syslog()  # 고수준 클래스가 저수준 구현에 직접 의존
      def stream(self, event: str):
          self.syslog.send(event)
  ```

  - 의존성 역전 시키기
    - `EventStream`이 구체 클래스가 아닌 `DataTarget`라는 인터페이스에 의존하도록 변경한다.
    - 이제 `EventStream`은 구체적인 구현과 관련이 없어졌으며, 구현 내용이 바뀌어도 수정할 필요가 없어진다.

  ```python
  class DataTarget(ABC):
      @abstractmethod
      def send(self, data: str): ...
  
  
  class Syslog(DataTarget):
      def send(self, data: str):
          print(f"[Syslog] {data} sent to syslog server")
  
  
  class EventStreamer:
      def __init__(self, sender: DataTarget):
          self.sender = sender
  
      def stream(self, event: str):
          self.sender.send(event)
  ```

  - 의존성 주입
    - 위 코드를 보면 `EventStreamer`가 인터페이스에 의존하게 된 것 외에도 추가적인 변경 사항이 있는데, 이는 `DataTarget` 객체를 직접 생성하지 않고 생성자 함수의 인자로 받는다는 것이다.
    - 이를 통해 보다 유현한 사용이 가능하다.







# Python과 decorator

- 함수 데코레이터

  - 함수에 데코레이터를 사용하면 다양한 종류의 로직을 추가할 수 있다.
    - 일반적으로 유효성 검사, 사전조건 검사, 함수이 시그니처 변경, 함수의 결과 캐싱 등에 사용된다.
    - 아래 코드는 특정 예외에 대해 특정 횟수만큼 재시도하는 데코레이터이다.

  ```python
  def retry(func):
      @wraps(func)
      def wrapped(*args, **kwargs):
          last_raise = None
          RETRIES_LIMIT = 3
          for _ in range(RETRIES_LIMIT):
              try:
                  return func(*args, **kwargs)
              except Exception as e:
                  logger.info("retry")
                  last_raised = e
          raise last_raised
      return wrapped
  ```

  - 위에서 작성한 데코레이터는 아래와 같이 적용할 수 있다.
    - `@retry`는 `run_operation = retry(run_operation)`을 실행해주는 문법적 설탕이다.

  ```python
  @retry
  def run_operation(task):
      return task.run()
  ```



- 클래스 데코레이터

  - 클래스 데코레이터 사용의 이점
    - 코드 재사용과 DRY 원칙의 모든 이점을 공유한다.
    - 클래스 데코레이터를 사용하면 여러 클래스가 특정 인터페이스나 기준을 따르도록 강제할 수 있다.
    - 데코레이터를 통해 클래스의 점진적인 보강이 가능하다.
  - 클래스 데코레이터를 사용하지 않은 코드
    - 아래 코드는 시간이 지남에 따라 아래와 같은 문제가 발생할 수 있다.
    - 이벤트와 직렬화 클래스가 1:1로 매핑되어 있으므로 직렬화 클래스가 점점 많아지게 된다.
    - `LoginEvnt.serialize()` 메서드는 모든 이벤트 클래스가 구현해야한다.

  ```python
  class LoginEventSerializer:
      def __init__(self, event):
          self._event = event
      
      def serialize(self) -> dict:
          return {
              "username": self._event.username,
              "password": "private_information",
              "ip": self._event.ip,
              "timestamp": self._event.timestamp.strftime("%Y-%m-%d %H:%M")
          }
  
  @dataclass
  class LoginEvnt:
      # 이벤트와 직렬화 클래스가 1:1로 매핑된다.
      SERIALIZER = LoginEventSerializer
      
      username: str
      password: str
      ip: str
      timestamp: datetime
      
      def serialize(self) -> dict:
          return self.SERIALIZER(self).serialize()
  ```

  - 데코레이터를 사용하도록 변경하기
    - 데코레이터를 사용하면 다른 클래스의 코드를 확인하지 않고도 각 필드가 어떻게 처리되는지 쉽게 알 수 있다.
    - 클래스 데코레이터에 전달된 인수를 읽는 것만으로도 username과 ip는 수정되지 않고, password는 숨겨지고, timestamp는 포매팅 된다는 것을 알 수 있다.

  ```python
  from datetime import datetime
  from dataclasses import dataclass
  
  
  def hide_field(field) -> str:
      return "private_information"
  
  def format_time(field_timestamp: datetime) -> str:
      return field_timestamp.strftime("%Y-%m-%d %H:%M")
  
  def show_original(event_field):
      return event_field
  
  
  class EventSerializer:
      def __init__(self, serialization_fields: dict) -> None:
          self.serialization_fields = serialization_fields
      
      def serialize(self, event) -> dict:
          return {
              field: transformation(getattr(event, field))
              for field, transformation
              in self.serialization_fields.items()
          }
  
  
  class Serialization:
      def __init__(self, **transformations):
          self.serializer = EventSerializer(transformations)
          
      def __call__(self, event_class):
          def serialize_method(event_instance):
              return self.serializer.serialize(event_instance)
          event_class.serialize = serialize_method
          return event_class
  
  
  @Serialization(
      username=show_original,
      password=hide_field,
      ip=show_original,
      timestamp=format_time
  )
  @dataclass
  class LoginEvent:
      username: str
      password: str
      ip: str
      timestamp: datetime
      
      
  
  login_event = LoginEvent(username="John", password="1q2w3e4r", ip="127.0.0.1", timestamp=datetime.now())
  print(login_event.serialize())	# {'username': 'John', 'password': 'private_information', 'ip': '127.0.0.1', 'timestamp': '2025-05-09 14:35'}
  ```



- 중첩 함수를 사용한 데코레이터

  - 데코레이터는 간단히 말해 함수를 파라미터로 받아 함수를 반환하는 함수이다.
    - 함수형 프로그래밍에서 함수를 받아서 함수를 반환하는 함수를 고차 함수라 부르는데, 이와 같은 개념이다.
  - 데코레이터에 파라미터를 추가하려면 다른 수준의 간접 참조가 필요하다.
    - 첫 번째 함수는 파라미터를 받아서 내부 함수에 전달하고, 두 번째 함수는 데코레이터가 될 함수이며, 세 번째 함수는 데코레이팅의 결과를 반환하는 함수다.
    - 즉 최소 세 단계의 중첩 함수가 필요하다.
  - @ 구문은 데코레이팅 객체에 대한 연산 결과를 반환한다.
    - 즉 `@retry(arg1, arg2, ...)`는 의미상 아래와 같다.
    - `<original_function> = retry(args1, args2, ...)(<original_function>)`
  - 중첩 함수를 사용하여 파라미터를 받도록 변경한 코드는 아래와 같다.

  ```python
  from typing import Optional, Sequence
  from functools import wraps
  
  
  _DEFAULT_RETRIES_LIMIT = 3
  
  def with_retry(retries_limit: int = _DEFAULT_RETRIES_LIMIT, allowed_exceptions: Optional[Sequence[Exception]] = Exception):    
      def retry(func):
          @wraps(func)
          def wrapped(*args, **kwargs):
              last_raise = None
              for _ in range(retries_limit):
                  try:
                      return func(*args, **kwargs)
                  except allowed_exceptions as e:
                      print("retry")
                      last_raised = e
              raise last_raised
          return wrapped
      return retry
  ```

  - 위 데코레이터는 아래와 같이 적용할 수 있다.

  ```python
  @with_retry(retries_limit=2, allowed_exceptions=(ZeroDivisionError, AttributeError))
  def run_with_custom_params(task):
      return task.run()
  ```



- 객체를 사용한 데코레이터

  - 앞에서는 데코레이터에 파라미터를 추가하기 위해 세 단계의 중첩된 함수를 사용했다.
  - 클래스를 사용하여 데코레이터를 정의하면 보다 깔끔한 방식으로 데코레이터에 파라미터를 전달할 수 있다.
    - `__init__` 메서드에 파라미터를 전달한 다음 `__call__`메서드에 데코레이터의 로직을 구현하면 된다.

  ```python
  _DEFAULT_RETRIES_LIMIT = 3
  
  class WithRetry:
      def __init__(self, retires_limit: int = _DEFAULT_RETRIES_LIMIT, allowed_exceptions: Optional[Sequence[Exception]] = Exception):
          self._retires_limit = retries_limit
          self._allowed_exceptions = allowed_exceptions
      
      def __call__(self, func):
          @wraps(func)
          def wrapped(*args, **kwargs):
              last_raise = None
              for _ in range(retires_limit):
                  try:
                      return func(*args, **kwargs)
                  except allowed_exceptions as e:
                      print("retry")
                      last_raised = e
              raise last_raised
          return wrapped
  ```

  - 사용 방법은 이전과 거의 유사하다.

  ```python
  @WithRetry(retries_limit=2, allowed_exceptions=(ZeroDivisionError, AttributeError))
  def run_with_custom_params(task):
      return task.run()
  ```



- 기본값을 가진 데코레이터

  - 데코레이터의 파라미터에 기본값이 있다고 하더라도, 대상 함수에 데코레이터를 추가할 때 괄호가 없으면 기본값이 적용되지 않는다.
    - 괄호가 없는 경우, 첫 번째 파라미터로 함수가 전달되지만, 괄호가 있는 경우 첫 번째 파라미터로 None이 전달되기 때문이다.

  ```python
  # 아래와 같이 아무 파라미터를 넘기지 않더라도 괄호를 달면 기본값이 적용되지만
  @retry()
  def my_function(): ...
  
  # 아래와 같이 괄호를 달지 않으면 기본값이 적용되지 않는다.
  @retry
  def my_function(): ...
  ```

  - 괄호 없이도 기본값이 적용되도록 데코레이터 생성하는 방법1
    - 가장 간단한 방법은 조건문으로 첫 번째 인자로 오는 함수가 None인지에 따라 분기하는 방법이다.

  ```python
  from functools import wraps
  
  
  def decorator(func=None, *, x=3, y=4):
      if func is None:
          def decorated(func):
              @wraps(func)
              def wrapped():
                  return func(x, y)
              return wrapped
          return decorated
      else:
          @wraps(func)
          def wrapped():
              return func(x, y)
          return wrapped
  
  
  @decorator
  def sum_(x, y):
      return x + y
  
  print(sum_())
  ```

  - 괄호 없이도 기본값이 적용되도록 데코레이터 생성하는 방법2
    - 래핑된 데코레이터의 일부를 추상화한 다음 `functools.partial`을 사용해 함수의 일부분에 적용하는 것이다.

  ```python
  def decorator(func=None, *, x=3, y=4):
      if function is None:
          return lambda f: decorator(f, x=x, y=y)
      
      @wraps(func)
      def wrapped():
          return func(x, y)
      return wrapped
  ```

  - 위 방식은 모두 데코레이터의 파라미터를 키워드 인자로 받는다는 공통점이 있다.
    - 데코레이터의 파라미터는 기본값을 가지고 있는지에 관계 없이 키워드 인자로 설정하는 것이 좋다.
    - 데코레이터를 적용할 때 각 값이 하는 일에 대한 컨텍스트 정보를 제공할 수 있기 때문이다.



- 코루틴을 위한 데코레이터

  - Python의 비동기 프로그래밍 구문은 일반 구문과 차이가 있으며, 이는 데코레이터도 마찬가지다.
    - 간단하게 생각하면 코루틴에 대한 데코레이터를 작성하려면 새로운 구문(래핑 할 객체를 `def`이 아닌 `async def`으로 선언하고, 래핑된 코루틴을 `await`한다)을 사용하면 된다.
    - 문제는 우리가 만든 데코레이터를 함수와 코루틴에 모두 적용하고자 하는 경우이다.
    - 각각을 따로 지원하는 두 개의 데코레이터를 만드는 것이 가장 좋은 방법일 수도 있지만, 데코레이터 사용자에게 보다 간단한 인스턴스를 제공하기 위해 내부적으로 어떤 데코레이터를 사용해야 하는지 분기해주는 디스패치 래퍼를 만들 수 있다.
    - 이는 마치 데코레이터를 위한 facade를 만드는 것과 같다.
  - 코루틴에 대해 한 가지 주의할 점이 있다.
    - 데코레이터는 호출 가능한 인자를 받아서 파라미터와 함께 그것을 호출하는데, 코루틴을 받았을 경우에도 await 없이 호출한다는 점이다.
    - 이렇게 하면 코루틴 객체(이벤트 루프에 전달할 작업)가 생성되지만 작업이 끝날 때까지 기다리지는 않는다.
    - 즉 나중에 `await coro()`를 호출하는 곳에서 결과를 알 수 있다.
    - 일반적으로는 전달 받은 코루틴에 대한 처리를 하는 것을 권장한다.
  - 예시
    - 두 번째 래퍼는 코루틴을 위해 필요한데, 이 부분이 없다면 두 가지 문제가 발생한다.
    - 첫 번째 문제는 await 없이 코루틴을 호출하면 실제로는 작업이 끝나길 기다린 것이 아니므로 정확한 결과가 아니다.
    - 두 번째 문제는 반환하는 dictionary에서 result키에 대한 값은 실제 결과 값이 아니라 코루틴이라는 것이다.
    - 결과적으로 나중에 await 없이 result 키에 대한 값을 사용하려고 하면 에러가 발생한다.

  ```python
  from functools import wraps
  import time
  import inspect
  
  def timing(callable):
      @wraps(callable)
      def wrapped(*args, **kwargs):
          start = time.time()
          result = callable(*args, **kwargs)
          latency = time.time() - start
          return {"latency": latency, "result": result}
      
      @wraps(callable)
      async def warpped_coro(*args, **kwargs):
          start = time.time()
          result = await callable(*args, **kwargs)
          latency = time.time() - start
          return {"latency": latency, "result": result}
      
      if inspect.iscoroutinefunction(callable):
          return warpped_coro
      return wrapped
  ```



- 데코레이터를 위한 확장 구문

  - Python 3.9에서는 PEP-614를 받아들여 보다 일반화된 문법을 지원하는 새로운 데코레이터 문법이 추가되었다.
    - 기존에는 @를 사용할 수 있는 범위가 제한적이어서 모든 함수 표현식에 적용할 수 없었다.
    - Python 3.9부터는 이러한 제약이 해제되어 보다 복잡한 표현식을 데코레이터에 사용할 수 있게 되었다.
  - 예시
    - 아래는 함수에서 호출된 파라미터의 로그를 남기는 데코레이터이다.
    - 내부 함수의 정의를 데코레이터 호출 시 두 개의 람다 표현식으로 대체했다.

  ```python
  def _log(f, *args, **kwargs):
      print(f"function_name: {f.__qualname__!r}, params: {args=}, {kwargs=}")
      return f(*args, **kwargs)
  
  @(lambda f: lambda *args, **kwargs: _log(f, *args, **kwargs))
  def func(x):
      return x + 1
  
  func(3)
  ```





## 데코레이터 활용 사례

- 데코레이터가 가장 유용하게 사용되는 경우는 아래와 같다.
  - 파라미터 변환
    - 파라미터가 어떻게 처리되는지 세부사항을 숨기면서 함수의 서명을 변경하는 경우에 사용한다.
    - 명확한 의도를 가지고 사용할 경우에만 유용하기에 주의해서 사용해야 한다.
    - 즉 기존의 다소 복잡한 함수에 대해 데코레이터를 사용해서 보다 간결한 시그니처를 제공하는 경우라면 권장할 만한 방법이다.
  - 코드 추적
    - 파라미터와 함께 함수의 실행 경로를 로깅하려는 경우이다.
    - 이는 기존 코드를 건드리지 않고 외부의 기능을 통합할 수 있는 강력한 추상화이자 좋은 인터페이스로 기능한다.
  - 파라미터 유효성 검사
    - 파라미터의 값이나 데이터 타입이 유효한지 검사하는 데 사용할 수 있다.
    - 계약에 의한 디자인을 따르면서 추상화의 전제조건을 강요하도록 할 수 있다.
  - 재시도 로직 구현
    - 이전에 직접 구현해본 retry 데코레이터가 이에 해당한다.
  - 일부 반복 작업을 데코레이터로 이동하여 클래스 단순화.
    - DRY 원칙과 관련이 있다.



- 함수의 시그니처 변경

  - 레거시 코드에 복잡한 시그니처(많은 파리미터나 보일러 플레이트 코드 등)를 가진 함수가 많이 있는 경우 데코레이터를 유용하게 사용할 수 있다.
    - 데코레이터를 사용하면 기존 함수의 변경은 최소화하면서 보다 깔끔한 인터페이스를 제공할 수 있다.
  - 아래와 같은 함수를 여러 곳에서 호출하는 경우를 상상해보자.
    - 이 함수를 여러 곳에서 사용하고 있어서 모든 파라미터의 처리 부분을 캡슐화하고 애플리케이션에 알맞은 동작으로 변환하는 추상화를 하려고 한다.
    - 아래 코드를 보면 첫 번째 줄에서 동일한 객체를 반복해서 생성하는 보일러 플레이트 코드가 있고, 나머지 코드는 오직 이를 통해 생성한 도메인 객체와 교류하고 있다.

  ```python
  def resolver_function(root, args, context, info):
      helper = DomainObject(root, args, context, info)
      # ...
      helper.process()
  ```

  - 데코레이터를 통해 함수의 시그니처를 변경하여 도메인 객체가 직접 전달되는 것 처럼 할 수 있다.
    - 데코레이터는 원래의 파라미터을 가로채서 도메인 객체를 만들고, 데코레이팅된 함수에 도메인 객체(helper)를 전달한다.
    - 이제 원래의 함수는 이미 초기화된 helper 객체를 가진 것 처럼 시그니처를 변경할 수 있다.

  ```python
  @DomainArgs
  def resolver_function(helpers):
      # ...
      helper.process()
  ```

  - 이러한 원리는 반대 방향으로 활용할 수도 있다.
    - 즉 기존의 레거시 코드가 객체를 받아서 많은 파라미터로 분해하고 있는 상황이다.
    - 이 때 기존 코드를 모두 리팩터링하는 것보다는 데코레이터를 중간 레이어로 활용할 수 있다.



- 파라미터 유효성 검사
  - 데코레이터를 사용하여 파라미터의 유효성을 검사할 수 있다.
    - 계약에 의한 설계의 원칙에 따라 사전조건 혹은 사후조건을 강제할 수도 있다.
  - 유사한 객체를 반복적으로 생성하거나 추상화를 위해 유사한 변형을 반복하는 경우 데코레이터를 만들어 사용하면 이 작업을 쉽게 처리할 수 있다.



- 코드 추적
  - 특정 함수를 추적하는 데 사용할 수 있다.
  - 주로 아래와 같은 내용을 추적한다.
    - 함수의 실행 경로 추적
    - 함수 지표 모니터링
    - 함수의 실행 시간 측정
    - 함수가 실행되는 시점
    - 함수에 전달된 파라미터 확인.





## 데코레이터 사용시 주의할 점

- 래핑된 원본 객체의 데이터를 보존해야한다.

  - 데코레이터를 함수에 적용할 때 가장 많이 하는 실수 중 하나는 원본 함수의 일부 속성을 유지하지 않아 부작용을 유발하는 것이다.
  - 예를 들어 아래와 같이 함수가 실행될 때 로그를 남기는 데코레이터가 있다고 가정해보자.

  ```python
  def trace_decorator(func):
      def wrapped(*args, **kwrags):
          print(f"{func.__qualname__} 실행")
          return func(*args, **kwrags)
     	return wrapped
  ```

  - 위 데코레이터를 사용하는 함수는 아래와 같다.
    - Docstring을 가지고 있다.

  ```python
  @trace_decorator
  def process_account(account_id: int):
      """id별 계정 처리"""
      print(f"{account_id} 계정 처리")
  ```

  - 위에서 데코레이터를 사용하는 함수의 help와 호출 정보, 그리고 어노테이션 정보를 확인하면 아래와 같다.
    - 데코레이터가 원본 함수를 wrapped라 불리는 새로운 함수로 변경해버렸기에 원본 함수의 속성이 아닌 새로운 함수의 속성이 출력된다.
    - 또 다른 문제는 docstring또한  warpped 함수의 docstring으로 덮어씌워진다는 것이다.

  ```python
  help(process_account)
  """
  Help on function wrapped in module __main__:
  
  wrapped(*args, **kwrags)
  """
  
  print(process_account.__qualname__)	# trace_decorator.<locals>.wrapped
  print(process_account.__annotations__) # {}
  ```

  - 이 문제를 해결하려면 래핑된 함수에 `@wraps` 데코레이터를 적용하여 실제로는 func 파라미터 함수를 래핑한 것이라고 알려주는 것이다.

  ```python
  from functools import wraps
  
  def trace_decorator(func):
      @wraps(func)
      def wrapped(*args, **kwargs):
          print(f"{func.__qualname__} 실행")
          return func(*args, **kwargs)
      return wrapped
  ```

  - 위와 같이 수정한 후 다시 확인해보면, 원래 함수의 정보가 정상적으로 출력되는 것을 확인할 수 있다.

  ```python
  help(process_account)
  """
  Help on function process_account in module __main__:
  
  process_account(account_id: int)
      id별 계정 처리
  """
  
  print(process_account.__qualname__)		# process_account
  print(process_account.__annotations__) 	# {'account_id': <class 'int'>}
  ```

  - 이것이 앞의 모든 예제에서 `functools.wraps`를 사용한 이유이다.



- 잘못 선언한 데코레이터

  - 데코레이터 함수를 구현할 때 반드시 지켜야하는 단 하나의 조건은 구현 함수가 가장 안쪽에 위치해야 한다는 것이다.
    - 아래와 같이 함수의 실행 시간을 로깅하는 데코레이터가 있다.

  ```python
  from functools import wraps
  import time
  
  def trace_function_wrong(func):
      print(f"{func} 실행")
      start_time = time.time()
      
      @wraps(func)
      def wrapped(*args, **kwargs):
          result = func(*args, **kwargs)
          print(f"{func}의 실행 시간: {time.time()-start_time}")
          return result
      return wrapped
  ```

  - 위 데코레이터에는 중요한 버그가 하나 있다.
    - 아래와 같이 데코레이터를 적용한 함수가 있을 때, 해당 함수가 실행되지 않더라도 데코레이터가 실행된다.
    - 이는 다른 모듈에서 해당 데코레이터를 import만 해도 동일하게 실행된다.

  ```python
  from functools import wraps
  import time
  
  
  def trace_function_wrong(func):
      print(f"{func} 실행")
      start_time = time.time()
      
      @wraps(func)
      def wrapped(*args, **kwargs):
          result = func(*args, **kwargs)
          print(f"{func}의 실행 시간: {time.time()-start_time}")
          return result
      return wrapped
  
  
  @trace_function_wrong
  def hello_world():
      time.sleep(1)
      print("hello world!")
  ```

  - 위 문제 외에도 아래와 같은 문제도 있다.
    - 데코레이터로 수식한 함수의 실행 시간은 비슷해야하는데, 함수를 호출할수록 더 오랜 시간이 걸리는 것으로 출력된다.
    - 이는 실제 시간이 더 걸리는 것이 아니라 데코레이터에서 실행 시간을 잘 못 출력하고 있는 것이다.

  ```python
  hello_world()
  hello_world()
  hello_world()
  
  """
  <function hello_world at 0x1048fe020> 실행
  hello world!
  <function hello_world at 0x1048fe020>의 실행 시간: 1.0050990581512451
  hello world!
  <function hello_world at 0x1048fe020>의 실행 시간: 2.010247230529785
  hello world!
  <function hello_world at 0x1048fe020>의 실행 시간: 3.0154359340667725
  """
  ```

  - 이 문제의 원인은 아래와 같다.
    - `@trace_function_wrong`은 `hello_world = trace_function_wrong(hello_world)`를 의미한다.
    - 이 문장은 모듈이 실행될 때실행되며, 데코레이터에 설정된  `start_time`은 모듈이 실행되는 시점이 된다.
    - 함수을 연속적으로 호출하면 함수의 실행 시간으로 모듈이 실행된 시간과의 시간차를 계산한다.
    - 이는 함수가 실제로 호출된 순간부터 계산한 것이 아니기에 잘못된 값이다.
  - 수정하기
    - 래핑된 함수 내부로 코드를 이동시킨다.

  ```python
  from functools import wraps
  import time
  
  
  def trace_function_wrong(func):    
      @wraps(func)
      def wrapped(*args, **kwargs):
          print(f"{func} 실행")
      	start_time = time.time()
          result = func(*args, **kwargs)
          print(f"{func}의 실행 시간: {time.time()-start_time}")
          return result
      return wrapped
  ```



- 잘못 선언한 데코레이터 활용하기

  - 때로는 의도적으로 데코레이터를 잘못 선언하여 데코레이턱 실제 실행되는 시점까지 기다리지 않는 경우도 있다.
    - 대표적인 예로 모듈의 공용 레지스트리에 객체를 등록하는 경우가 있다.
    - 예를 들어 아래 예시의 경우 각 클래스마다 처리 여부를 flag로 표시하는 대신 데코레이터를 사용해 명시적으로 레지스트리에 등록할 수 있다.
    - 아래 모듈이 어떤 방식으로든 실행되면(직접 실행하거나 import하거나) `EVENT_REGISTRY`에 클래스들이 등록된다.

  ```python
  EVENT_REGISTRY = {}
  
  def register_event(event_cls):
      EVENT_REGISTRY[event_cls.__name__] = event_cls
      return event_cls
  
  class UserEvent:
      TYPE = "user"
  
  @register_event
  class UserLoginEvent(UserEvent):
      ...
      
  @register_event
  class UserLogOutEvent(UserEvent):
      ...
  
  print(EVENT_REGISTRY)	# {'UserLoginEvent': <class '__main__.UserLoginEvent'>, 'UserLogOutEvent': <class '__main__.UserLogOutEvent'>}
  ```

  - 이런 동작 방식이 문제가 되는 경우도 있지만, 경우에 따라 필요한 경우도 있다.
    - 사실 많은 웹 프레임워크나 널리 알려진 라이브러리들은 이 방식으로 객체를 노출하거나 활용하고 있다.
    - 단, 이러한 방식이 예상치 못한 부수 효과를 일으킬 수 있다는 것을 항상 염두에 둬야한다.



- 데코레이터는 범용적이면 좋다.

  - 특정한 함수, 특정한 객체제 적용할 수 있는 데코레이터보다는 재사용을 고려해 다양한 곳에서 사용할 수 있는 데코레이터가 좋다.
    - 다만, 데코레이터의 파라미터에 `*args, **kwrags`를 정의하면 모든 곳에서 사용할 수 있지만, 아래의 이유로 수식하는 함수의 시그니처와 비슷하게 데코레이터를 정의하는 것이 나을 수 있다.
    - 원래의 함수와 모양이 비슷하기에 읽기가 쉽다.
    - 파라미터를 받아서 뭔가를 처리해야 하는 경우 `*args, **kwargs`보다는 파라미터 자체를 받는 것이 편하다.
  - 아래와 같은 데코레이터가 있다고 가정해보자.
    - 파라미터를 받아 특정 객체를 생성하는 경우가 많다고 가정해보자.
    - 예를 들어 문자열을 받아서 드라이버 객체를 초기화하는 경우가 있을 수 있다.
    - 이 경우 파라미터를 변환해주는 데코레이터를 만들어 중복을 제거할 수 있다.

  ```python
  from functools import wraps
  
  class DBDriver:
      def __init__(self, db_string: str):
          self._db_string = db_string
      
      def execute(self, query: str) -> str:
          return f"query {query} at {self._db_string}"
      
  def inject_db_driver(func):
      @wraps(func)
      def wrapped(db_string):
          return func(DBDriver(db_string))
      return wrapped
  
  @inject_db_driver
  def run_query(driver):
      return driver.execute("test_function")
  
  print(run_query("test_ok"))		# query test_function at test_ok
  ```

  - 이제 같은 기능을 하는 데코레이터를 클래스 메서드에 재사용하려고 한다.
    - 아래 코드를 실행하면 에러가 발생한다.
    - 에러가 발생하는 원인은 클래스의 메서드에는 `self`라는 추가 파라미터가 있기 때문이다.
    - 하나의 파라미터만 받도록 설계된 위 데코레이터는 `db_string` 자리에 `self`를 전달하고, 두 번째 파라미터는 전달받지 못해서 에러가 발생한다.

  ```python
  class DataHandler:
      @inject_db_driver
      def run_query(self, driver):
          return driver.execute(self.__class__.__name__)
  
  print(DataHandler().run_query("test_fails"))
  ```

  - 위 문제를 해결하려면 메서드와 함수에 대해 동일하게 동작하는 데코레이터를 만들어야한다.
    - 데코레이터를 클래스 객체로 구현하고 `__get__` 메서드를 구현한 디스크립터(추후 설명) 객체를 만들 것이다.
    - 이제 일반 함수와 메서드 모두에서 정상 동작한다.

  ```python
  from functools import wraps
  from types import MethodType
  
  class inject_db_driver:
      def __init__(self, func):
          self._func = func
          wraps(self._func)(self)
      
      def __call__(self, db_string):
          return self._func(DBDriver(db_string))
      
      def __get__(self, instance, owner):
          if instance is None:
              return self
          return self.__class__(MethodType(self._func, instance))
      
  
  print(run_query("test_ok"))			# query test_function at test_ok
  print(DataHandler().run_query("test_fails"))	# query DataHandler at test_fails
  ```





## 데코레이터와 클린 코드

- 상속보다 구성(composition)

  - 아래와 같은 상황을 해결해야한다.
    - 특정 패키지에 속한 클래스를 사용하려고 하는데, 해당 클래스의 모든 attribute에는 `resolve_`가 prefix로 붙어있다.
    - 해당 패키지를 사용하는 코드에서는 패키지 클래스의 attribute들에 접근할 때 `resolve_`를 일일이 붙여서 접근하고싶지 안하.
  - Mixin을 사용하여 위 문제를 해결할 수 있다.
    - Mixin을 생성하고, 해당 mixin을 상속 받게 한다.

  ```python
  from dataclasses import dataclass
  
  
  class BaseResolverMixin:
      def __getattr__(self, attr: str):
          if attr.startswith("resolve_"):
              *_, actual_attr = attr.partition("resolve_")
          else:
              actual_attr = attr
              
          try:
              return self.__dict__[actual_attr]
          except KeyError as e:
              raise AttributeError from e
              
              
  @dataclass
  class Customer(BaseResolverMixin):	# 상속
      customer_id: str
      name: str
  
  
  customer = Customer("1", "John")
  print(customer.resolve_customer_id)
  ```

  - 데코레이터를 사용해도 해결이 가능하다.

  ```python
  from dataclasses import dataclass
  
  
  def _resolver_method(self, attr):
      if attr.startswith("resolve_"):
          *_, actual_attr = attr.partition("resolve_")
      else:
          actual_attr = attr
          
      try:
          return self.__dict__[actual_attr]
      except KeyError as e:
          raise AttributeError from e
              
  def with_resolver(cls):
      cls.__getattr__ = _resolver_method
      return cls
  
  
  @dataclass
  @with_resolver		# 데코레이터
  class Customer:
      customer_id: str
      name: str
  
  
  customer = Customer("1", "John")
  print(customer.resolve_customer_id)
  ```

  - 데코레이터를 사용한 방식이 상속을 사용하는 방식보다 낫다.
    - 데코레이터를 사용한다는 것은 상속 대신에 구성을 사용하는 것이므로, 결합도를 낮출 수 있다.
    - 또한 위와 같은 상황에서 상속을 사용하는 것은 적절한 해결 방법이 아니다.
    - 첫 번째로, 상속을 단순히 코드 재사용을 위해서 사용했기 때문이고, 둘 째로 개념적으로 `Customer` 클래스가 `BaaseResolverMixin` 클래스의 하위에 있지도 않기 때문이다.
  - 위 예시에서 데코레이터가 구성을 활용했다고 볼 수 있는지?
    - 일반적으로 구성은 `A has a B`와 같이  한 객체가 다른 객체를 가지고 있음을 의미한다.
    - 그런 면에서 위 예시는 일반적인 구성에 딱 들어맞지는 않지만, `_resolver_method`라는 메서드를 가지게 된다(`Customer has a _resolver_method`)라는 맥락에서는 구성이라고 볼 수 있다.
    - 즉, 부모 클래스의 메서드를 상속 받는 것이 아니라, 주입을 통해 가지게 되는 구성이라고 볼 수 있다.



- 데코레이터와 DRY 원칙
  - 데코레이터를 사용하면 곳에 중복으로 필요한 로직을 한 번만 구현하여 여러 곳에서 재사용하게 할 수 있다.
  - 다만 데코레이터는 복잡성을 증가시키므로, 아래 내용을 지키면서 사용해야한다.
    - 처음부터 데코레이터를 만들지 않고, 패턴이 생기고 데코레이터에 대한 추상화가 명확해지면 그 때 리펙터링을 하면서 데코레이터를 만든다.
    - 데코레이터가 각기 다른 곳에서 적어도 3회 이상 필요한 경우에만 구현한다.
    - 데코레이터 코드를 최소한으로 유지한다.



- 데코레이터와 관심사의 분리

  - 아래와 같은 데코레이터가 있다고 가정해보자.

  ```python
  def traced_function(func):
      @wraps(func)
      def wrapped(*args, **kwargs):
          print(func.__qualname__, "함수 실행")
          start_time = time.time()
          result = func(*args, **kwargs)
          print("함수 실행 시간:", time.time()-start_time)
       	return result
      return wrapped
  ```

  - 위 데코레이터에는 문제가 있다.
    - 하나 이상의 작업을 수행하고 있다.
    - 특정 함수가 방금 호출된 것을 기록하고, 실행하는데 걸린 시간도 기록한다.
    - 따라서 위 코드는 좀 더 구체적이고 제한적인 책임을 지닌 더 작은 데코레이터로 분류되어야한다.

  - 개선
    - 더 작은 데코레이터로 분리한다.

  ```python
  def log_execution(func):
      @wraps(func)
      def wrapped(*args, **kwargs):
          print(func.__qualname__, "함수 실행")
          return func(*args, **kwargs)
      return wrapped
  
  def measure_time(func):
      @wraps(func)
      def wrapped(*args, **kwargs):
          start_time = time.time()
          result = func(*args, **kwargs)
          print("함수 실행 시간:", time.time()-start_time)
          return result
      return wrapped
  
  @mesure_time
  @log_execution
  def opertaion(): ...
  ```




- 좋은 데코레이터 분석
  - 좋은 데코레이터는 아래와 같은 특성을 갖춰야한다.
    - 캡슐화: 데코레이터의 내부 구현을 모르더라도 사용이 가능해야한다.
    - 관심사의 분리: 실제로 하는 일과 장식하는 일의 책임을 명확하게 구분해야한다.
    - 독립성: 데코레이터는 자신이 장식하는 객체와 최대한 분리되어야한다.
    - 재사용성: 하나의 함수 인스턴스에만 적용되는 것이 아니라 여러 유형에 적응 가능한 형태가 바람직하다.
  - 좋은 데코레이터는 깔끔한 인터페이스를 제공하고, 사용자가 내부 동작 원리를 자세히 몰라도 기대하는 바를 정확히 성취하게 해준다.

