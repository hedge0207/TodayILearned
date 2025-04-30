# 유지 보수성

- 유지 보수성은 개발의 속도와 관련되어 있다.
  - 테스트의 실질적 도움
    - 프로덕션 코드를 변경할 때 기존 테스트를 계속 변경해야 한다면 오히려 개발 속도를 늦추게 된다.
    - 그러나 프로덕션 코드를 변경하고도 기존 테스트를 수정할 필요가 없다면 테스트가 개발에 실질적인 도움이 된다고 볼 수 있다.
  - 유지 보수성이 테스트를 얼마나 자주 변경해야 하는지 가늠하는 척도라면 그 횟수를 최소화하는 것이 중요하다.
    - 이를 위한 근본적인 원인을 파악하려면 아래 항목을 점검해 볼 필요가 있다.
    - 언제 테스트가 실패하여 변경이 필요하다는 사실을 깨닫는가?
    - 왜 테스트가 실패하는가?
    - 어떤 테스트가 실패했을 때 테스트 코드를 반드시 수정해야 하는가?
    - 테스트 코드를 반드시 수정할 필요가 없더라도 언제 테스트 코드를 변경하는가?



- 테스트 실패로 코드 변경
  - 실제 실패(true failure)와 거짓 실패(false failure)
    - 테스트 실패는 유지 보수성에 문제가 생길 수 있음을 알려주는 첫 번째 신호다.
    - 물론 실제로 프로덕션 코드에서 버그를 발견할 수도 있지만, 그렇지 않을 수도 있다.
    - 프로덕션 코드의 버그로 발생한 실패를 실제 실패라한다.
    - 프로덕션 코드의 버그가 아닌 다른 이유로 발생하는 실패를 거짓 실패라 한다.
  - 테스트의 유지 보수성 측정
    - 일정 기간 동안 발생하는 거짓 실패 횟수와 그 원인을 분석하면 유지 보수성을 측정할 수 있다.
    - 거짓 실패가 발생하는 원인 중 하나는 테스트 자체의 버그이며, 그 밖에 다른 원인들도 있다.
  - 테스트와 관련이 없거나 다른 테스트와 충돌하는 경우
    - 프로덕션 코드에 새로운 기능을 넣으면 기존 테스트와 충돌할 수 있다.
    - 이 경우 거짓 실패가 발생할 수 있다.
    - 이 경우 테스트는 버그를 찾는 용도로만 사용하지 않고 새 기능이 추가된 코드의 신규 요구 사항을 제대로 반영했는지 확인하는 용도로도 사용할 수 있다.
    - 또 코드가 변경되어 기대 결과 역시 달라졌을 수 있으니, 새로운 기능에 맞춰 작성된 테스트는 통과할 수 있지만 기존 테스트는 더 이상 유효하지 않을 수 있다.
    - 새로운 기능이 추가되어 실패한 테스트는 더 이상 유효하지 않거나 새로운 요구사항이 잘못되었을 수 있다.
    - 요구 사항이 맞는다고 가정하면 유효하지 않은 테스트는 삭제해도 된다.



- 프로덕션 코드의 API 변경

  - 테스트 대상 코드가 변경되어 함수나 객체를 사용하는 방식이 달라지는 경우에도 거짓 실패가 발생할 수 있다.
    - 이러한 거짓 실패는 가능한 피해야한다.
    - 예를 들어 아래 코드는 유지 보수성 관점에서 보면 머지않아 수정해야 할 수 도 잇는 여러 문제점이 보인다.

  ```python
  # 테스트 대상 코드
  class ILogger(ABC):
      @abstractmethod
      def info(self):
          ...
  
  class PasswordVerifier:
      def __init__(self, rules, looger: ILogger):
          self._rules = rules
          self._logger = logger
      
      def verify(self, text):
          ...
          
  # 테스트
  class MockLogger(ILogger):
      def info(self, text):
          ...
  
  def test_passes_with_zero_rules():
      verifier = PasswordVerifier([], MockLogger())
      result = verifier.verifiy("any input")
      assert result
  
  def test_fails_with_failing_rule():
      verifier = PasswordVerifier([lambda : False], MockLogger())
      result = verifier.verifiy("any input")
      assert not result
  ```

  - 코드의 수명은 길다.
    - 현재 작성 중인 코드는 때로는 10년 이상 그 자리에 있을 수도 있다.
    - 그 기간 동안 `PasswordVerifier`의 설계가 변경될 가능성은 무궁무진하다.
    - 앞으로 `PasswordVerifier`에는 아래와 같은 변경이 있을 수 있다.
    - 생성자 매개변수의 추가나 삭제
    - 생성자 매개변수의 자료형의 변경
    - `ILogger`의 함수 개수나 함수 시그니처의 변경
    - 사용 패턴의 변경으로 `PasswordVerifier`를 인스턴스화하지 않고 함수만 직접 사용.
  - 만약 프로덕션 코드의 API가 아래와 같이 변경되었다고 가정해보자.
    - `logger`는 이제 `IComplicatedLogger`의 인스턴스여야한다.
    - 아래와 같이 수정될 경우, PasswordVerifier를 직접 인스턴스화하는 모든 테스트를 수정해야한다.

  ```python
  class PasswordVerifier:
      def __init__(self, rules, logger: IComplicatedLogger):
          self._rules = rules
          self._logger = logger
      
      # ...
  ```

  - 팩토리 함수로 객체를 만드는 과정을 분리한다.
    - 유지 보수성을 떨어뜨릴 수 있는 잠재적인 문제 상황을 피하는 방법은 테스트할 코드의 생성 과정을 분리하거나 추상화하는 것이다.
    - 아래에서는 `PasswordVerifier`의 생성을 공용(centralized) 팩토리 함수로 분리했다.
    - 이렇게 하면 나중에 변경 사항이 발생하더라도 팩토리 함수만 수정하면 된다.

  ```python
  # 아래와 같이 팩토리 함수를 통해 인스턴스를 생성한다.
  def make_password_verifier(rules: list, logger: ILogger):
      return PasswordVerifier(rules, logger)
  
  
  # 추후에 변경이 필요할 경우, 모든 테스트 코드를 변경하지 않고, 팩토리 함수만 변경한다.
  def make_password_verifier(rules: list, logger: IComplicatedLogger):
      return PasswordVerifier(rules, logger)
  ```



- 다른 테스트가 변경되었을 경우

  - 테스트는 항상 다른 테스트와 독립적으로 실행되어야 한다.
    - 같은 기능을 검증하더라도 각 테스트는 서로 격리된 공간에서 실행되어야 한다.
    - 테스트가 제대로 분리되지 않으면 각 테스트가 서로 간섭하게 된다.
    - 그러면 코드가 매우 복잡해져 단위 테스트에 많은 시간을 소모하게 된다.
  - 순서가 정해진 테스트
    - 이전 테스트의 실행 여부에 따라 결과가 달라지는 테스트를 말한다.
    - 이는 테스트가 다른 테스트 때문에 설정 값이 바뀌거나 공유 변수 같은 자원의 상태 값이 변경되는 것에 영향을 받기 때문이다.
  - 예를 들어 아래와 같은 코드가 있다고 가정해보자.
    - `UserCache`는 애플리케이션과 테스트에서 데이터를 임시로 저장하는 단일 인스턴스(싱글톤)을 사용한다.

  ```python
  class UserDetails(BaseModel):
      key: str
      password: str
  
  
  class UserCacheInterface(ABC):
      @abstractmethod
      def add_user(self, user: UserDetails):
          ...
          
      @abstractmethod
      def get_user(self, key: str):
          ...
          
      @abstractmethod
      def reset(self):
          ...
          
          
  class UserCache(UserCacheInterface):
      def __init__(self):
          self._users = {}
      
      def add_user(self, user: UserDetails):
          if self._users.get(user.key) is None:
              self._users[user.key] = user
          else:
              raise Exception("user aleady exists")
      
      def get_user(self, key: str) -> UserDetails:
          return self._users.get(key)
      
      def reset(self):
          self._users = {}
          
  
  cache = None
  def get_user_cache() -> UserCache:
      global cache
      if cache is None:
          cache = UserCache()
      return cache
  
  
  class SpecialApp:
      def login_user(self, key:str, pass_: str) -> bool:
          _cache = get_user_cache()
          found_user = _cache.get_user(key)
          if found_user is not None and found_user.password == pass_:
              return True
          return False
  ```

  - 위 코드의 테스트 코드를 아래와 같이 작성했다.
    - 아래 코드는 첫 번째 테스트와 세 번째 테스트가 모두 두 번째 테스트에 의존하고 있다.
    - 첫 번째 테스트는 cache가 비어 있어야 성공하므로 두 번째 테스트가 먼저 실행되면 안 된다.
    - 세 번째 테스트는 두 번째 테스트가 유저 정보를 cache에 넣어야 성공하므로 두 번째 테스트가 먼저 실행되어야 한다. 

  ```python
  def test_no_user_login_fails():
      app = SpecialApp()
      result = app.login_user("foo", "bar")
      assert not result
      
  def test_can_only_cache_each_user_once():
      get_user_cache().add_user(UserDetails(**{"key":"foo", "password":"bar"}))
      with pytest.raises(Exception, match=r"user aleady exists"):
          get_user_cache().add_user(UserDetails(**{"key":"foo", "password":"bar"}))
  
  def test_users_exists_login_succeed():
      app = SpecialApp()
      result = app.login_user("foo", "bar")
      assert result
  ```

  - 리팩터링하기
    - 위와 같은 안티 패턴은 보통 테스트 일부를 헬퍼 함수로 빼지 않고 사용하려 할 때 발생한다.
    - 다른 테스트가 먼저 실행되어 현재 테스트에 필요한 설정 값을 초기화 해 주길 바라는 것이다.
    - 이를 해결하려면 헬퍼 함수를 만들어야한다.
    - 아래와 같이 수정하면, 테스트 실핼 순서가 테스트 결과에 영향을 미치지 않는다.

  ```python
  # cache에 사용자를 추가해주는 헬퍼 함수
  def add_default_user():
      get_user_cache().add_user(UserDetails(**{"key":"foo", "password":"bar"}))
  
  # SpecialApp 인스턴스도 팩토리 함수로 생성하도록 변경한다.
  def make_special_app() -> SpecialApp:
      return SpecialApp()
  
  # 각 테스트가 실행되기 전에 실행되어 cache를 초기화해주는 함수
  @pytest.fixture
  def reset_cache():
      get_user_cache().reset()
  
  def test_no_user_login_fails(reset_cache):
      app = make_special_app()
      result = app.login_user("foo", "bar")
      assert not result
      
  def test_can_only_cache_each_user_once(reset_cache):
      add_default_user()
      with pytest.raises(Exception, match=r"user aleady exists"):
          get_user_cache().add_user(UserDetails(**{"key":"foo", "password":"bar"}))
  
  def test_users_exists_login_succeed(reset_cache):
      add_default_user()
      app = make_special_app()
      result = app.login_user("foo", "bar")
      assert result
  ```



- 유지 보수성을 높이는 리팩터링 방법

  - private 또는 protected 메서드를 직접 테스트하지 않는다.
    - 이들은 보통 특정한 이유로 메서드에 대한 접근을 제한하는 용도로 사용한다.
    - 이는 구현 세부 사항을 숨겨 나중에 구현이 변경되더라도 외부에서 보이는 동작은 바뀌지 않도록 하기 위함이다.
    - private 메서드를 테스트하는 것은 시스템 내부의 약속이나 규칙을 테스트하는 것과 같다.
    - 테스트 할 때는 외부의 규칙만 신경쓰면 된다.
    - private 메서드를 테스트하는 것은 내부적으로 어떤 동작을 발생시킬지 몰라 전체 코드의 동작을 테스트하는 것에 영향을 줄 수 있다.
    - private 메서드는 홀로 존재하지 않으며, 결국에는 이를 호출하는 public 메서드가 있고, 그렇지 않더라도 실행의 연결 고리 중에는 반드시 어딘가에 public 메서드가 있다.
    - 즉, private 메서드는 항상 외부 인터페이스로 시작하는 더 큰 작업의 일부이다.
    - 따라서 private 메서드를 보면 시스템 내에서 이 메서드를 호출하는 public 메서드나 기능을 찾아야한다.
    - private 메서드가 제대로 동작한다고 해도, 외부 인터페이스(public 메서드)에서 이를 잘못 사용하고 있으면 시스템 전체가 제대로 동작하지 않을 수 있다.
    - 결국 중요한 것은 외부 동작(public 메서드)이 올바르게 작동하는지 확인하는 것이다.
    - public 메서드로 private 메서드를 간접적으로 테스트하는 것이 더 효과적이다. 
    - 테스트는 외부에서 관찰할수 있는 동작을 검증해야한다.
  - 메서드를 public으로 만들기
    - 이 방법이 객체 지향 원칙에 어긋나는 것처럼 보일 수 있지만, 항상 그런 것은 아니다.
    - 어떤 메서드를 테스트하고 싶다는 것은 그 메서드가 일정하게 동작하거나 역할이 중요하다는 의미일 수 있다.
    - 메서드를 public으로 만들면 이를 명확하게 드러내는 것이다.
    - 반면에 private으로 두면 나중에 그 코드를 유지 보수할 때 해당 private 메서드를 사용하는 코드는 걱정할 필요 없이 private 메서드의 구현부를 마음대로 변경할 수 있다.
  - 메서드를 새로운 클래스나 모듈로 분리하기
    - 메서드에 독립적으로 동작할 수 있는 로직이 많이 포함되어 있거나, 해당 메서드와 관련된 특정 상태 값을 사용한다면 메서드를 시스템 내에서 특정 역할을 하는 새로운 클래스나 모듈로 분리하는 것이 좋다.
    - 이렇게 하면 그 클래스만 따로 테스트할 수 있다.
  - Stateless한 private 메서드를 public이나 static 메서드로 만들기
    - 메서드가 stateless하다면 이 기능을 지원하는 언어에서는 이를 static으로 변경하는 것이 좋다.
    - 이렇게 하면 테스트하기가 훨씬 쉽고 메서드 이름으로 유틸리티 메서드라는 점을 더 명확히 드러낼 수 있다.

  - 테스트에서도 DRY 원칙 고수해야한다.
    - 단위 테스트에서 중복은 프로덕션 코드에서의 중복만큼이나 해롭다.
    - 헬퍼 함수를 사용하면 테스트에서 중복되는 부분을 줄일 수 있다.
    - 다만, 중복을 제거하는 것이 지나치면 가독성을 해칠 수도 있다.

  - 초기화 함수를 사용을 지양해야한다.
    - 테스트 전에 한 번씩 실행되는 `beforeEach()` 함수(초기화 함수)는 중복을 제거하려는 목적으로 자주 사용되지만, 이 보다는 헬퍼 함수를 사용하는 것이 더 나을 수 있다.
    - 초기화 함수는 잘못 사용되기 쉬워 개발자가 이를 원래 의도와는 다르게 사용할 가능성이 높아, 테스트 유지 보수가 어려워질 수 있기 때문이다.
  - 초기화 함수는 아래와 같은 조건을 만족해야한다.
    - 일부 테스트에서만 사용하는 객체를 초기화 함수에서 초기화하면 안 된다.
    - 길고 이해하기 어려운 초기화 코드를 작성하면 안 된다. 
    - 초기화 함수 내에서 목이나 가짜 객체를 만들어선 안 된다.
    - 초기화 함수는 매개 변수나 반환 값을 가져서는 안 된다.
    - 초기화 함수는 값을 반환하는 팩터리 매서드로 사용해선 안 된다.
    - 초기화 함수는 모든 테스트에 동일한 초기화 작업을 적용해야 하므로 특정 상황에 맞춘 세밀한 조정 로직이 들어가선 안 된다.
  - 매개 변수화된 테스트로 중복 코드를 제거한다.
    - 초기화 함수를 사용하지 않고 코드를 전개할 수 있는 다른 좋은 방법은 모든 테스트가 비슷하게 작성되었다는 가정하에 매개 변수화된 테스트를 사용하는 것이다.
    - 매개 변수화 패턴은 원래 초기화 함수에 있어야 할 설정을 각 테스트의 준비 단계로 옮길 수 있다.



- 과잉 명세된 테스트
  - 과잉 명세된 테스트는 단순히 코드의 외부 동작을 확인하는 것이 아니라 코드 내부가 어떻게 구현되어야 하는지까지 검증하는 테스트를 의미한다.
    - 단순히 관찰 가능한 동작(종료점)이 올바른지 확인하는 것을 넘어서 코드 내부의 구현 세부 사항까지 확인한다.
    - 예를 들어 메서드의 내부 로직이나 특정 상태 변화를 검증하는 테스트가 이에 해당한다.
    - 이는 테스트가 지나치게 구체적이고 복잡해서 코드를 변경할 때 테스트를 자주 수정해야 하는 문제가 생길 수 있다.
  - 아래는 단위 테스트가 과잉 명세되었다고 볼 수 있는 조건이다.
    - 테스트가 객체의 내부 상태만 검증한다.
    - 테스트가 목을 여러 개 만들어 사용한다.
    - 테스트가 스텁을 목처럼 사용한다.
    - 테스트가 필요하지 않은데도 특정 실행 순서나 정확한 문자열 비교를 포함한다.



- 목을 사용한 내부 동작 과잉 명세

  - 작업 단위의 종료점을 확인하는 대신 클래스나 모듈의 내부 함수가 호출되었는지만 검증하는 것이다.
    - 예를 들어 아래 테스트 코드에서는 종료점을 검증하지 않으며, 내부 메서드를 호출하는지만 확인한다.
    - 이는 아무 의미 없는 테스트이다.

  ```python
  class PasswordVerifier:
      def __init__(self, rules: list):
          self._rules = rules
      
      def verify(self, value):
          failed = self._find_failed_rules(value)
          if len(failed) == 0:
              return True
          return False
      
      # 내부 메서드
      def _find_failed_rules(self, value):
          return list(filter(lambda result: not result, map(lambda rule: rule(value), self._rules)))
      
      
  def test_check_failed_rules_is_called(mocker):
      verifier = PasswordVerifier([])
      failed_mock = Mock()
      failed_mock.return_value=[]
      verifier._find_failed_rules = failed_mock
      verifier.verify("foo")
      failed_mock.assert_called()
  ```

  - 테스트는 내부 메서드에 신경 쓸 필요가 없다.
    - `_find_failed_rules`메서드와 같이 값을 반환하는 메서드는 모의 함수로 만들지 않는 것이 좋은데, 이는 메서드 호출 자체는 종료점을 나타내지 않기 때문이다.
    - 위 코드에서 종료점은 `verify()` 메서드가 반환하는 값이며, 내부 메서드가 존재하는지 여부는 중요하지 않다.
    - 종료점이 아닌 protected 메서드를 모의 함수로 만들어 검증하는 것은 별다른 이점 없이 테스트 코드 내부 구현의 결합도만 올리는 행위다.
    - 메서드 내에서 클래스 내부 메서드 호출이 변경될 때마다 관련된 모든 테스트를 수정해야 하는데, 이는 바람직하지 않다.



- 결과와 순서를 지나치게 세밀하게 검증

  - 반환된 값의 순서와 구조를 지나치게 세분화하는 것은 바람직하지 않다.
    - 전체 목록괴 각 항목을 한꺼번에 검증하려고 하면 목록의 사소한 변경에도 테스트를 수정해야 하는 부담이 생긴다.
    - 한 번에 모든 것을 검증하려고 하지 말고, 검증을 여러 작은 검증 구문으로 나누어 각각의 측면을 명확하게 확인하는 것이 좋다.

  - 예를 들어 아래와 같은 코드를 테스트하려고 한다.

  ```python
  class PasswordVerifier:
      def __init__(self, rules: list):
          self._rules = rules
      
      def verify(self, values: list[str]) -> list[Result]:
          failed_results = list(map(self._check_single_input, values))
          return failed_results
      
      def _check_single_input(self, value: str) -> Result:
          failed = self._find_failed_rules(value)
          return Result(result=len(failed)==0, input_=value)
      
      def find_result_for(self, results: list[Result], value: str) -> bool:
          found_result = None
          for result in results:
              if result.input_ == value:
                  found_result = result
                  break
          return found_result.result if found_result else False
      
      def _find_failed_rules(self, value: str):
          return list(filter(lambda result: not result, map(lambda rule: rule(value), self._rules)))
      
      def _find_failed_rules(self, value: str):
          return list(filter(lambda result: not result, map(lambda rule: rule(value), self._rules)))
  ```

  - 아래와 같이 결과의 순서와 구조를 단일 검증문을 이용하여 확인하도록 테스트 코드를 작성한다.
    - 아래 테스트는 결과값뿐만 아니라 결과의 순서와 구조까지도 검증한다.
    - 즉, 한 번에 너무 많은 것을 검증한다.
    - `results`의 배열 길이가 변경되거나, 각 객체에 속성이 추가되거나 삭제되거나, 결과의 순서가 변경되면 테스트도 함께 변경해줘야한다.
    - 심지에 테스트에서 해당 속성을 사용하지 않더라도 속성이 추가되거나 삭제되면 테스트도 변경해야한다.

  ```python
  def test_overspecify_order_and_schema():
      verifier = PasswordVerifier([lambda x: "abc" in x])
      results = verifier.verify(["a", "ab", "abc", "abcd"])
      assert results == [
          Result(result=False, input_='a'), 
          Result(result=False, input_='ab'), 
          Result(result=True, input_='abc'), 
          Result(result=True, input_='abcd')
      ]
  ```

  - 구조는 무시하고 순서와 결과만 확인하도록 변경하면 아래와 같다.
    - 이 코드 역시 결과 값의 순서가 바뀌면 여전히 문제가 될 수 있다.

  ```python
  def test_overspecify_order_and_schema():
      verifier = PasswordVerifier([lambda x: "abc" in x])
      results = verifier.verify(["a", "ab", "abc", "abcd"])
      assert len(results) == 4
      assert results[0].result == False
      assert results[1].result == False
      assert results[2].result == True
      assert results[3].result == True
  ```

  - 순서가 그다지 중요하지 않다면 결과 값에 특정 값이 포함되어 있는지만 확인하면 된다.

  ```python
  def test_overspecify_order_and_schema():
      verifier = PasswordVerifier([lambda x: "abc" in x])
      results = verifier.verify(["a", "ab", "abc", "abcd"])
      assert len(results) == 4
      assert verifier.find_result_for(results, "a") == False
      assert verifier.find_result_for(results, "ab") == False
      assert verifier.find_result_for(results, "abc") == True
      assert verifier.find_result_for(results, "abcd") == True
  ```

  - 단위 테스트에서 반환값이나 속성에 대해 하드코딩된 문자열을 사용하여 검증하는 것 역시 쉽게 저지를 수 있는 실수이다.
    - 사실 필요한 것은 문자열으 특정 부분만 확인하는 것이라면 문자열이 완벽히 일치하는지보다는 필요한 내용의 포함 유무를 확인 가능한지 생각해봐야한다.
    - 문자열이 정확히 일치하는지 확인하는 것 보다는 원하는 대로 문자열이 구성되었는지를 확인하는 것이 중요하다.
    - 문자열이 정확히 일치하는지 확인하는 경우, 띄어쓰기, 쉼표, 마침표 하나에도 테스트가 실패할 수 있다.



