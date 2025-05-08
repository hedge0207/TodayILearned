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

