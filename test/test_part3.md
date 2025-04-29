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

