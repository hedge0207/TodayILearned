# 클래스

- 클래스 체계
  - 클래스를 정의하는 표준 자바 관례는 아래와 같다.
    - 가장 먼저 변수 목록이 나오며 그 중 static, public 상수가 있다면 맨 처음에 나온다.
    - 다음으로 static private 변수가 나오며, 이어서 private 인스턴스 변수가 나온다.
    - public 변수가 필요한 경우는 거의 없다.
    - 변수 목록 다음에는 공개 함수가 나온다.
    - private 함수는 자신을 호출하는 공개 함수 직후에 넣는다.
    - 즉, 추상화 단계가 순차적으로 내려간다.
  - 캡슐화
    - 변수와 유틸리티 함수는 가능한 공개하지 않는 편이 낫지만 반드시 숨겨야 한다는 법칙이 있는 것은 아니다.
    - 때로는 변수나 유틸리티 함수를 protected로 선언해 테스트 코드에 접근을 허용하기도 한다.
    - 같은 패키지 않에서 테스트 코드가 함수를 호출하거나 변수를 사용해야 한다면 그 함수나 변수를 protected로 선언하거나 패키지 전체로 공개한다.
    - 하지만 그 전에 비공개 상태를 유지할 온갖 방법을 강구해야 하며, 캡슐화를 풀어주는 결정은 최후의 수단이다.



- 클래스는 작아야한다.

  - 클래스를 만들 때 첫 번째 규칙은 크기다.
    - 클래스는 작아야 한다.
    - 두 번째 규칙도 크기다.
    - 클래스는 더 작아야 한다.
  - 클래스의 크기는 클래스가 맡은 책임으로 결정된다.
    - 클래스 이름은 해당 클래스 책임을 기술해야한다.
    - 작명은 클래스 크기를 줄이는 첫 번째 관문이다.
    - 간결한 이름이 떠오르지 않는다면 클래스 크기가 너무 커서 그런 것이다.
    - 클래스 이름이 모호하다면 클래스 책임이 너무 커서 그런 것이다.
    - 클래스 이름에 Processor, Manager, Super 등과 같이 모호한 단어가 있다면 클래스에다 여러 책임을 떠안겼다는 증거다.
  - 클래스 설명은 if, and, or, but을 사용하지 않고 25 단어 내외로 가능해야한다.
    - 클래스를 위 단어를 사용하지 않고 25 단어 내로 설명하지 못한다면 클래스에 책임이 너무 많은 것이다.

  - 단일 책임 원칙(Single Responsibility Principle, SRP)
    - 클래스나 모듈을 변경할 이유가 단 하나뿐이어야 한다는 원칙이다.
    - SRP는 책임이라는 개념을 정의하며 적절한 클래스 크기를 제시한다.
    - SRP는 이해하고 지키기 수월한 개념이지만, 가장 무시되는 규칙 중 하나다.
  - 단일 책임 클래스가 많아지면 전체 시스템을 파악하기 어려울 것이라 생각하는 개발자들이 있다.
    - 그러나 작은 클래스가 많은 시스템이든 소수의 큰 클래스가 있는 시스템이든 돌아가는 부품의 개수는 비슷하다.
    - 단지 작은 서랍을 많이 두고 기능과 이름이 명확한 컴포넌트를 나눠 넣을지, 큰 서랍 몇 개에 모두 던져 넣을지의 차이가 있을 뿐이다.
    - 소수의 큰 클래스로 이루어진 시스템 보다 작은 클래스 여러 개로 이루어진 시스템이 더 바람직하다.
  - 응집도
    - 클래스는 인스턴스 변수 수가 적어야 한다.
    - 각 클래스 메서드는 클래스 인스턴스 변수를 하나 이상 사용해야한다.
    - 일반적으로 메서드가 변수를 더 많이 사용할수록 메서드와 클래스는 응집도가 더 높다.
    - 모든 인스턴스 변수르 메서드마다 사용하는 클래스는 응집도가 가장 높다.
    - 일반적으로 이처럼 응집도가 가장 높은 클래스는 가능하지도, 바람직하지도 않다.
    - 그럼에도 응집도가 노파는 말은 클래스에 속한 메서드와 변수가 서로 의존하면 논리적인 단위로 묶인다는 의미이다.
    - "함수를 작게, 매개 변수 목록을 짧게"라는 전략을 따르다 보면 때때로 몇몇 메서드만이 사용하는 인스턴스 변수가 매우 많아진다.
    - 이는 대부분의 경우 새로운 클래스로 쪼개야 한다는 신호다.
    - 응집도가 높아지도록 변수와 메서드를 적절히 분리해 새로운 클래스로 쪼개야한다.
  - 응집도를 유지하면 작은 클래스 여럿이 나온다.
    - 큰 함수를 작은 함수 여럿으로 나누기만 해도 클래스 수가 많아진다.
    - 예를 들어 변수가 아주 많은 큰 함수 하나가 있다.
    - 큰 함수 일부를 작은 함수로 빼내고자 하는 코드가 함수에 정의된 변수 넷을 사용한다.
    - 그렇다면 변수 네 개를 새 함수에 인수로 넘기는 게 옳은 일일까?
    - 그렇지 않다, 만약 네 변수를 클래스 인스턴스 변수로 승격시킨다면 새 함수는 인수가 필요없으며, 그만큼 함수를 쪼개기 쉬워진다.
    - 그러나, 이렇게 할 경우 클래스가 응집력을 잃는다.
    - 몇몇 함수만 사용하는 인스턴스 변수가 점점 더 늘어나기 때문이다.
    - 그런데 몇몇 함수가 몇몇 인스턴스 변수만 사용한다면 독자적인 클래스로 분리해도 되므로, 인스턴스 변수의 증가로 인해 클래스가 응집력을 잃는다면 새로운 클래스로 분리하면 된다.
    - 이처럼 큰 함수를 작은 함수 여럿으로 쪼개다 보면 종종 작은 클래스 여럿으로 쪼갤 기회가 생긴다.
    - 그러면서 프로그램에 점점 더 체계가 잡히고 구조가 투명해진다.



- 변경하기 쉬운 클래스

  - 대다수 시스템은 지속적인 변경이 가해지며, 변경할 때마다 시스템이 의도대로 동작하지 않을 위험이 따른다.
    - 클래스에 대한 변경은 다른 코드를 망가뜨릴 잠정적인 위험을 수반한다.

  - 아래 `Sql` 클래스는 SRP를 위반한다.
    - 새로운 SQL문을 지원하려면 클래스를 수정해야한다.
    - 또한 기존 SQL문 하나를 수정할 때도 클래스를 수정해야 한다.
    - 이처럼 변경해야 할 이유가 두 가지이므로 아래 클래스는 SRP를 위반한다.
      - 또한 구조적인 관점에서도 아래 클래스는 SRP를 위반하는데, `_select_with_criteria`와 같이 클래스 내의 다른 일부 메서드에서만 사용되는 비공개 메서드는 코드를 개선할 잠재적인 여지를 시사한다.


  ```python
class Sql:
    def create(self): ...
    def insert(self, fields): ...
    def selectAll(self): ...
    def find_by_key(self, key_column, key_value): ...
    def select(self, column, pattern): ...
    def _select_with_criteria(self, criteria): ...
  ```

  - 아래와 같이 `Sql` 클래스를 추상 클래스로 선언하고, 각 SQL문을 별도의 구상 클래스로 선언한다.
    - 모든 파생 클래스가 공통으로 사용하는 비공개 메서드는 `Where`, `ColumnList`라는 두 유틸리티 클래스에 선언한다.
    - 아래와 같이 클래스를 분리하면 메서드 하나를 수정했다고 다른 메서드가 망가질 위험도 사라진다.
    - 테스트 관점에서도 모든 논리를 증명하기도 쉬워진다.
    - 또한 update문을 추가할 때 기존 클래스를 변경할 필요 없이 `Sql`을 상속 받는 새로운 클래스를 추가하기만 하면 된다.

  ```python
from abc import abstractmethod

class Sql:
    @abstractmethod
    def generate(self): ...
        
        
class CreateSql(Sql):
    def generate(self): ...
    
    
class SelectSql(Sql):
    def __init__(self, table, columns):...
    def generate(self): ...

    
class InsertSql(Sql):
    def __init__(self, table, columns, fields): ...
    def generate(self): ...

    
class SelectWithCriteriaSql(Sql):
    def __init__(self, table, columns, criteria): ...
    def generate(self): ...
    
    
class Where:
    def __init__(self, creteria): ...
    def generate(self): ...
    

class ColumnList:
    def __init__(self, columns): ...
    def generate(self): ...
  ```

  - 변경으로부터의 격리
    - 상세한 구현에 의존하는 클라이언트 클래스느는 구현이 바뀌면 위험에 빠진다.
    - 인터페이스와 추상 클래스를 사용하면 구현의 변경이 미치는 영향을 격리한다.
    - 또한 상세한 구현에 의존하는 코드는 테스트가 어렵다.
    - 예를 들어 아래와 같이 `TokyoStockExchange`라는 구체적인 구현에 의존하는 클래스가 있다고 가정해보자.
    - `PortPolio`는 `TokyoStockExchange`로부터 5분에 한 번씩 변경된 주가 정보를 받아온다.
    - 이 때, `PortPolio` 클래스는 5분에 한 번씩 변하는 정보를 받아오므로, 이에 대한 테스트를 작성하는 것은 쉬운 일이 아니다.
    - 따라서, `TokyoStockExchange`라는 구체적인 구현에 의존하게 하지 말고, `StockExchange`라는 인터페이스를 의존하게 변경한다.
    - 변경후에는 테스트를 위해 `TokyoStockExchange`를 흉내내는 `StockExchange`구현한 테스트용 클래스를 만들 수 있다.
    - 이처럼 테스트가 가능하다는 것은 시스템의 결합도가 낮다는 의미이며 자연스럽게 유연성과 재사용성도 더욱 높아진다.

  ```python
# 변경 전
class PortPolio:
    def __init__(self, tokyo_stock_exchange: TokyoStockExchange):
        self._tokyo_stock_exchange = tokyo_stock_exchange
        
        
# 변경 후
class StackExchange:
    @abstractmethod
    def current_price(self): ...
    
class TokyoStockExchange(StackExchange):
    def current_price(self): ...
    
class PortPolio:
    def __init__(self, stock_exchange: StockExchange):
        self._stock_exchange = stock_exchange
  ```

 



# 시스템

- 시스템 제작과 시스템 사용을 분리하라.

  - 제작은 사용과 매우 다르다.
    - 소프트웨어 시스템은 애플리케이션 객체를 제작하고 의존성을 서로 연결하는 준비 과정과 준비 과정 이후에 이어지는 런타임 로직을 분리해야한다.
    - 관심사 분리는 소프트웨어 개발 분야에서 가장 오래되고 가장 중요한 설계 기법 중 하나지만, 대다수 애플리케이션은 관심사를 분리하지 않는다.
    - 준비 과정 코드를 주먹구구식으로 구현할 뿐만 아니라 런타임 로직과 마구 뒤섞는다.
  - 아래는 준비 과정과 런타임 로직이 분리되지 않은 전형적인 예시이다.
    - 아래는 lazy initialization 혹은 lazy evaluation이라 불리는 기법이다.
    - 물론 아래 방식은 객체가 실제로 필요할 때 까지 객체를 생성하지 않으므로 불피요한 부하가 걸리지 않으며, 어떤 경우에도 None을 반환하지 않는다는 장점이 있다.
    - 그러나 `get_service` 메서드가 `MyService`와 `MyService`를 생성하기 위해 필요한 생성자 인수에 명시적으로 의존한다.
    - 런타임 로직에서 `MyService`를 전혀 사용하지 않더라도, 의존성을 해결하지 않으면 컴파일이 되지 않는다.
    - 테스트 또한 문제가 되는데, `MyService`가 무거운 객체라면 단위 테스트에서 `get_service` 메서드를 호출하기 전에 적절한 테스트 전용 객체(테스트 더블이나 목 객체)를 `self.service`에 할당해야한다.
    - 또한 일반 런타임 로직에 객체 생성 로직을 섞어놓은 탓에 `self.service`가 None인 경우와 아닌 경우 모두 테스트해야한다.
    - 책임이 둘이라는 말은 메서드가 작업을 둘 이상 수행한다는 의미이며, 이는 SRP를 깬다는 말이다.

  ```python
  def get_service(self):
      if self.service is None:
          self.service = MyService(...);
      return self.service
  ```

  - Main 분리
    - 시스템 생성과 시스템 사용을 분리하는 한 가지 방법으로 생성과 관련된 코드는 모두 main이나 main이 호출하는 코드로 옮기고, 나머지 시스템은 모든 객체가 생성되었고 모든 의존성이 연결 되었다고 가정하는 방법이 있다.
    - main 함수에서 시스템에 필요한 객체를 생성한 후 이를 애플리케이션에 넘기고, 애플리케이션은 그저 객체를 사용하기만 한다.
    - 애플리케이션은 main이 생성하는 객체가 생성되는 과정을 전혀 모르며, 단지 모든 객체가 적절히 생성되었다고 가정한다.
  - 팩토리
    - 때로는 객체가 생성되는 시점을 애플리케이션이 결정할 필요도 생긴다.
    - 예를 들어 주문 처리 시스템에서 애플리케이션은 `LineItem` 인스턴스를 생성해 `Order`에 추가해야한다고 가정해보자.
    - 이 때 abstract factory 패턴을 사용하면 `LineItem`을 생성하는 시점은 애플리케이션이 결정하지만, 애플리케이션은 `LineItem`을 생성하는 코드는 모른다.
    - 여기서도 애플리케이션은 `LineItem`이 생성되는 구체적인 방법은 모른다.
    - 다만 애플리케이션은 `LineItem` 인스턴스가 생성되는 시점을 완벽하게 통제하며, 필요하다면 애플리케이션에서만 사용하는 인수도 넘길 수 있다.

  - 의존성 주입
    - 사용과 제작을 분리하는 강력한 메커니즘 중 하나가 의존성 주입이다.
    - 의존성 주입은 제어의 역전 기법을 의존성 관리에 적용한 메커니즘이다.
    - 제어의 역전에서는 한 객체가 맡은 보조 책임을 새로운 객체에게 전적으로 떠넘긴다.
    - 새로운 객체는 넘겨받은 책임만 맡으므로 단일 책임 원칙을 지키게 된다.
    - 의존성 관리 맥락에서 객체는 의존성 자체를 인스턴스로 만드는 책임은 지지 않는다.
    - 대신에 이런 책임을 다른 전담 메커니즘에 넘겨야만하며, 이를 통해 제어를 역전한다.
    - 초기 설정은 시스템 전체에서 필요하므로 대개 책임질 메커니즘으로 main 루틴이나 특수 컨테이너를 사용한다.



- 확장
  - 처음부터 올바르게 시스템을 만들 수 있다는 믿음은 미신이다.
    - 대신에 우리는 오늘 주어진 사용자 스토리에 맞춰 시스템을 구현해야 한다.
    - 내일은 새로운 스토리에 맞춰 시스템을 조정하고 확장하면 된다.
    - 이것이 반복적이고 점진적인 애자일 방식의 핵심이다.
  - BDUF(Big Design Up Front)
    - 구현을 시작하기 전에 앞으로 벌어질 모든 사항을 설계하는 기법이다.
    - BDUF는 해롭기까지 하다
    - 처음에 쏟아 부은 노력을 버리지 않으려는 심리적인 저항으로 인해, 그리고 처음 선택한 아키텍처가 향후 사고 방식에 미치는 영향으로 인해, 변경을 쉽사리 수용하지 못하는 탓이다.
    - 물리적 구조는 일단 짓기 시작하면 극적인 변경이 불가능하므로 건축가는 BDUF 방식을 취한다.
    - 그러나 소프트웨어는 극적인 변화가 경제적으로 가능하다.
  - 관심사를 적절히 분리해 관리한다면 소프트웨어 아키텍처는 점진적으로 발전할 수 있다.
    - TDD와 리팩터링으로 얻어지는 깨끗한 코드는 코드 수준에서 시스템을 조정하고 확장하기 쉽게 만든다.
  - 횡단(cross-cutting) 관심사
    - 소프트웨어 여러 부분에 걸쳐 공통적으로 나타는 관심사를 의미한다.
    - 로깅, 트랜잭션, 보안, 에러 처리 등이 있으며, 이들은 비즈니스 로직과는 직접적인 연관이 없지만 여러 모듈이나 클래스에서 공통적으로 사용된다.
    - 이를 각 클래스마다 개별적으로 직접 구현할 경우 중복 코드도 많아지고 유지 보수도 어렵게 된다.
    - 따라서 AOP(Aspect-Oriented Programming) 같은 개념을 통해 비즈니스 로직과 횡단 관심사를 분리해야 한다.
  - 자바에서 사용하는 관점 혹은 관점과 유사한 매커니즘 세 가지(필요한 경우 추후 추가, p.203~p.210)
    - 자바 프록시
    - 순수 자바 AOP 프레임워크
    - AspectJ 관점



- 의사 결정을 최적화하라
  - 모듈을 나누고 관심사를 분리하면 지엽적인 관리와 결정이 가능해진다.
    - 아주 큰 시스템에서는 한 사람이 모든 결정을 내리기 어렵다.
    - 가장 적합한 사람에게 책임을 맡기는 것이 가장 좋다.
  - 때때로 가능한 마지막 순간까지 결정을 미루는 것이 최선일 수 있다.
    - 이는 게으르거나 무책임해서가 아니라 최대한의 정보를 모아 최선의 결정을 내리기 위해서다.
    - 성급한 결정은 불충분한 지식으로 내린 결정이다.
    - 너무 일찍 결정하면 더 나은 대안을 탐험할 기회가 사라진다.



- 시스템은 도메인 특화 언어가 필요하다.
  - 대다수 도메인에는 필수적인 정보를 명료하고 정확하게 전달하는 어휘, 관용구, 패턴들이 있다.
  - 소프트웨어 분야에서도 최근 들어 DSL이 새롭게 조명 받기 시작했다.
    - DSL은 간단한 스크립트 언어나 표준 언어로 구현한 API를 가리킨다.
    - DSL로 짠 코드는 도메인 전문가가 작성한 구조적인 산문처럼 읽힌다.
  - 좋은 DSL은 도메인 개념과 그 개념을 구현한 코드 사이에 존재하는 의사소통 간극을 줄여준다.
    - 도메인 전문가가 사용하는 언어로 도메인 논리를 구현하면 도메인을 잘못 구현할 가능성이 줄어든다.
  - 효과적으로 사용된 DSL은 추상화 수준을 코드 관용구나 디자인 패턴 이상으로 끌어올린다.
    - 이를 통해 개발자가 적절한 추상화 수준에서 코드 의도를 표현할 수 있다.





# 창발성

- 창발적 설계로 깔끔한 코드 구현하기
  - 켄트 백의 단순한 설계 규칙 네 가지
    - 모든 테스트를 실행한다.
    - 중복을 없앤다.
    - 프로그래머의 의도를 표현한다.
    - 클래스와 메서드 수를 최소로 줄인다.
  - 위 네 가지 규칙은 우수한 설계의 창발성을 촉진한다.



- 모든 테스트를 실행한다.
  - 설계는 의도한 대로 돌아가는 시스템을 내놓아야 한다.
  - 시스템이 의도한 대로 돌아가는지 검증할 간단한 방법이 없다면 의도한 대로 돌아가는지 확인하기 힘들다.
    - 테스트가 불가능한 시스템은 검증도 불가능하며, 검증이 불가능한 시스템은 절대 출시하면 안 된다.
  - 테스트가 가능한 시스템을 만들려고 애쓰면 설계 품질이 함께 높아진다.
    - 크기가 작고 목적 하나만 수행하는 클래스가 나온다.
    - SRP를 준수하는 클래스는 테스트가 더 쉽다.
    - 즉 응집도가 높아진다.
  - 결합도가 높으면 테스트 케이스를 작성하기 어렵다.
    - 그러므로 테스트 케이스를 많이 작성할수록 개발자는 DIP와 같은 원칙을 적용하고, 의존성 주입, 인터페이스, 추상화 등과 같은 방법을 동원해 결합도를 낮춘다.
    - 테스트 케이스가 많아질수록 결합도도 낮아진다.
  - 이처럼 테스트를 수행하는 것 만으로도 결합도는 낮추고 응집도는 높일 수 있다.



- 리팩터링

  - 단순한 설계 규칙 2~4는 리팩터링과 관련된 규칙이다.
    - 1번 규칙과도 연관이 있는데, 테스트 케이스가 있으면 리팩터링으로 인해 시스템이 깨질 것을 염려하지 않아도 된다.
  - 리팩터링 단계에서는 소프트웨어 설계 품질을 높이는 방법이라면 무엇이든 적용해도 괜찮다.
    - 응집도를 높이고, 결합도를 낮추고, 관심사를 분리하고, 시스템 관심사를 모듈로 나누고, 함수와 클래스의 크기를 줄이고 더 나은 이름을 선택하는 등 다양한 방법을 적용한다.
  - 중복을 없애라
    - 중복은 추가 작업, 추가 위험, 불필요한 복잡도를 뜻한다.
    - 중복은 여러 가지 형태로 표출되는데, 똑같은 코드 뿐 아니라 비슷한 코드 역시 더 비슷하게 고쳐주면 똑같은 코드가 되기도 한다.
    - 구현 중복도 중복의 한 형태이다.
    - 적은 양이라도 공통적인 코드를 새 메서드로 뽑으면 복잡도가 줄어든다.
    - 만약 새로 메서드를 뽑았을 때 해당 메서드가 해당 클래스에 어울리지 않는다면 다른 클래스로 이동시키는 작업을 하는 것도 좋다.
    - 템플릿 메서드 패턴은 고차원 중복을 제거할 목적으로 자주 사용하는 기법이다.
    - 예를 들어 아래 코드에서 `accrueUSDivisionVaction`와 `accrueKRDivisionVaction`는 최소 법정 일수를 계산하는 부분만 제외하면 거의 동일하다.
    - 아래 코드에 템플릿 메서드 패턴을 적용하면 중복을 제거할 수 있다.

  ```java
  public class VactionPolicy {
      public void accrueUSDivisionVaction() {
          // 근무 시간을 바탕으로 휴가 일수를 계산하는 코드
          // ...
          // 휴가 일수가 최소 미국 법정 일수를 만족하는지 확인하는 코드
          // ...
          // 휴가 일수를 급여 대장에 적용하는 코드
          // ...
      }
      
      public void accrueKRDivisionVaction() {
          // 근무 시간을 바탕으로 휴가 일수를 계산하는 코드
          // ...
          // 휴가 일수가 최소 한국 법정 일수를 만족하는지 확인하는 코드
          // ...
          // 휴가 일수를 급여 대장에 적용하는 코드
          // ...
      }
  }
  
  
  // 템플릿 메서드 패턴 적용
  public class VactionPolicy {
  	public void accrueVaction() {
          calculateBaseVactionHours();
          alterForLegalMinimums();
          applyToPayroll();
      }
      
      private void calculateBaseVactionHours() { 
      	// ...
      }
      
      abstract protected void alterForLegalMinimums();
      
      private void applyToPayroll() {
          // ...
      }
  }
  
  // 달라지는 부분은 자식 클래스에서 구현한다.
  public class USVactionPolicy extends VactionPolicy {
      @override protected void alterForLegalMinimums() {
          // 미국 법정 일수 사용
      }
  }
  
  public class KRVactionPolicy extends VactionPolicy {
      @override protected void alterForLegalMinimums() {
          // 한국 법정 일수 사용
      }
  }
  ```

  - 표현하라
    - 자신이 이해하는 코드를 짜기는 쉽지만 나중에 코드를 유지보수할 사람이 이해할 코드를 짜는 것은 쉽지 않다.
    - 코드를 변경하면서 버그를 발생시키지 않으려면 유지보수 개발자가 시스템을 정확히 이해해야한다.
    - 그러므로 코드는 개발자의 의도를 분명히 표현해야한다.
    - 우선 좋은 이름을 선택해야한다.
    - 또한 함수와 클래스의 크기를 가능한 줄여야한다.
    - 표준 명칭을 사용하는 것 역시 중요하다. 예를 들어 디자인 패턴은 의사소통과 표현력 강화가 주 목적인데, 디자인 패턴에 사용되는 이름을 사용하는 클래스가 전혀 다른 역할을 해선 안 된다.
    - 마지막으로 단위 테스트 케이스를 꼼꼼히 작성해야한다. 잘 만든 테스트 케이스를 읽어보면 클래스 기능이 한 눈에 들어온다.
  - 클래스와 메서드 수를 최소로 줄여라.
    - 클래스와 메서드 크기를 줄이자고 조그만 클래스와 메서드를 많이 만들라는 것은 아니다.
    - 목표는 함수와 클래스 크기를 작게 유지하면서 동시에 시스템 크기도 작게 유지하는 데 있다.
    - 하지만 이 규칙은 네 가지 설계 규칙 중 우선순위가 가장 낮다.







# 동시성

> 동시성과 클린 코드가 양립하기는 매우 어렵다.

- 동시성이 필요한 이유
  - 동시성은 결합을 없애는 전략이다.
    - 즉 무엇과 언제를 분리하는 전략이다.
    - 스레드가 하나인 프로그램은 무엇과 언제가 서로 밀접하다.
    - 따라서 호출 스택을 살펴보면 프로그램 상태가 바로 드러난다.
    - 흔히 단일 스레드 프로그램을 디버깅하는 프로그래머는 일련의 정지점(breakpoint)를 정한 후 어느 정지점에 결렸는지 살펴보면서 시스템의 상태를 파악한다.
    - 무엇과 언제를 분리하면 애플리케이션의 구조와 효율이 극적으로 개선된다.
    - 구조적인 관점에서 프로그램은 거대한 루프가 아니라 작은 협력 프로그램 여럿으로 보이므로, 시스템을 이해하기 쉽고 문제를 분리하기도 쉽다.
  - 응답 시간과 작업 처리량 개선을 위해 동시성을 채택하기도 한다.
    - 위에서 살펴본 구조적인 개선만을 위해 동시성을 채택하는 것은 아니다.
    - 대량의 정보를 병렬로 처리하거나 많은 사용자의 요청을 동시에 처리하면 처리 속도나 처리량을 늘릴 수 있다.
  - 미신과 오해
    - 동시성이 항상 성능을 높여주는 것은 아니다.
    - 대기 시간이 아주 길어 여러 스레드가 프로세서를 공유할 수 있거나, 여러 프로세서가 동시에 처리할 독립적인 계산이 충분히 많은 경우에만 성능이 높아지는데, 어느 쪽도 일상적으로 발생하는 상황은 아니다.
    - 동시성을 구현하면 설계도 변해야한다.
    - 단일 스레드 시스템과 다중 스레드 시스템은 설계가 판이하게 다르다.
    - 일반적으로 무엇과 언제를 분리하면 시스템 구조가 크게 달라딘다.
  - 동시성과 관련된 타당한 생각들
    - 동시성은 다소 부하를 유발한다.
    - 동시성은 복잡하다.
    - 일반적으로 동시성 버그는 재현이 어렵다.
    - 동시성을 구현하려면 근본적인 설계 전략을 재고해야한다.



- 동시성을 구현하기 어려운 이유

  - 두 개의 스레드가 같은 변수를 동시에 참조할 경우 문제가 발생할 수 있다.
  - 예를 들어 아래와 같이 단순한 코드도, 동시성을 적용하면 문제가 발생할 수 있다.
    - `X`의 인스턴스를 생성하고, `lastIdUsed`필드를 42로 설정한 다음, 두 스레드가 해당 인스턴스를 공유한다고 가정해보자.
    - 두 스레드가 `getNextId()`를 호출한다면 결과는 셋 중 하나다.
    - 한 스레드는 43을 받고, 다른 스레드는 44를 받으며, `lastIdUsed`는 44가 된다.
    - 한 스레드는 44를 받고, 다른 스레드는 43을 받으며, `lastIdUsed`는 44가 된다.
    - 한스레드는 43을 받고, 다른 스레드도 43을 받으며, `lastIdUsed`는 43이 된다.

  ```java
  public class X {
      private int lastIdUsed;
      
      public int getNextId() {
          return ++lastIdUsed;
      }
  }
  ```

  - 위와 같은 문제가 발생하는 원인
    - 두 스레드가 자바 코드 한 줄을 거쳐가는 경로는 수 없이 많은데, 그 중에서 일부 경로가 잘못된 결과를 내놓기 때문이다.
    - JIT 컴파일러가 바이트 코드를 처리하는 방식과 자바 메모리 모델이 원자로 간주하는 최소 단위를 알아야 경로의 개수를 알 수 있는데, 답만 말하면, 바이트 코드만 고려했을 때, 두 스레드가 `getNextId` 메서드를 실행하는 잠재적인 경로는 최대 12,870개에 달한다.
    - `lastIdUsed` 변수가 int가 아니라 long이라면 조합 가능한 경로 수는 2,704,156개로 증가한다.
    - 물론 대다수 경로는 올바른 결과를 내놓지만, 잘못된 결과를 내놓는 일부 경로가 문제가 된다.



- 동시성 방어 원칙
  - 단일 책임 원칙
    - 동시성과 관련된 코드는 다른 코드와 분리해야한다.
    - 동시성은 복잡성 하나만으로도 따로 분리할 이유가 충분하다.
    - 그런데 불행히도 동시성과 관련이 없는 코드에 동시성을 곧바로 구현하는 사례가 너무 흔하다.
    - 동시성 코드를 구현할 때는 아래와 같은 것들을 고려해야한다.
    - 동시성 코드는 독자적인 개발, 변경, 조율 주기가 있다.
    - 동시성 코드에는 다른 코드에서 겪는 난관과는 다른 독자적고 훨씬 어려운 난관이 있다.
    - 주변에 있는 다른 코드가 발목을 잡지 않더라도 잘못 구현한 동시성 코드는 다양한 방식으로 실패한다.
  - 따름 정리(corollary): 자료 범위를 제한하라.
    - 앞서 확인한 것 처럼 객체 하나를 공유한 후 동일 필드를 수정하던 두 스레드가 서로 간섭하며 예상치 못 한 결과가 발생하기도 한다.
    - 이런 문제를 해결하는 방안으로 공유 객체를 사용하는 코드 내 임계 영역을 synchronized 키워드로 보호하라고 권장한다.
    - 이런 임계 영역의 수를 줄이는 기술이 중요하다.
    - 공유 자료를 수정하는 위치가 많을 수록 아래의 상황이 발생할 가능성도 높아진다.
    - 보호할 임계 영역을 빼먹는 상황.
    - 모든 임계 영역을 올바로 보호했는지 확인하느로 똑같은 노력과 수고를 반복하는 상황.
    - 버그글 더욱 찾기 어려워지는 상황
  - 따름 정리: 자료 사본을 사용하라
    - 공유 자료를 줄이려면 처음부터 공유하지 않는 방법이 제일 좋다.
    - 경우에 따라 객체를 복사해 읽기 전용으로 사용하거나 각 스레드가 객체를 복사해 사용한 후 스레드가 해당 사본에서 결과를 가져오는 방법도 가능하다.
    - 객체의 공유를 막으면 코드가 문제를 일으킬 가능성도 아주 낮아진다.
    - 물론 객체를 복사하는 시간과 부하가 걱정될수도 있지만, 복사본으로 동기화를 피할 수 있다면 내부 잠금을 없애 절약한 수행 시간이 사본 생송과 가비지 컬렉션에 드는 부하를 상쇄할 가능성이 크다.
  - 따름 정리: 스레드는 가능한 독립적으로 구현하라
    - 다른 스레드와 자료를 공유하지 않는 스레드를 구현하는 것이 좋다.
    - 각 스레드는 클라이언트 요청 하나를 처리하며, 모든 정보는 비공유 출처에서 가져와 로컬 변수에 저장하는 것이 좋다.
    - 독자적인 스레드로, 가능하면 다른 프로세서에서 돌려도 괜찮도록 자료를 독립적인 단위로 분할하라.



- 라이브러리를 이해하라
  - 자신이 사용하는 언어에서 지원하는 라이브러리, 패키지, 자료 구조들이 스레드에 사용해도 안전한지 확인하라.
  - 자신이 사용하는 언어가 어떤 방식으로 동시성 프로그래밍을 지원하는지 이해해라
    - 자바의 경우 `java.util.concurrent`, `java.util.concurrent.atomic`, `java.util.concurrent.locks` 등이 있다.



- 실행 모델을 이해하라
  - 기본 용어 정리
    - 한정된 자원(Bound Resource): 다중 스레드 환경에서 사용하는 자원으로, 크기나 숫자가 제한적이다. 데이터베이스 연결, 길이가 일적한 읽기/쓰기 버퍼 등이 예다.
    - 상호 배제(Mutual Exclusion): 한 번에 한 스레드만 공유 자료나 공유 자원을 사용할 수 있는 경우를 가리킨다.
    - 기아(Starvation): 한 스레드나 여러 스레드가 굉장히 오랫동안 혹은 영원히 자원을 기다리는 현상이다.
    - 데드락(Deadlock): 여러 스레드가 서로 끝나기를 기다리는 상황으로, 모든 스레드가 각기 필요한 자원을 다른 스레드가 점유하여 어느 쪽도 더 이상 진행하지 못한다.
    - 라이브락(Livelock): 락을 거는 단계에서 각 스레드가 서로 방해하는 상황으로, 스레드는 계속해서 진행하려 하지만, 공명(resonance)으로 인해, 굉장히 오랫동안 혹은 영원히 진행하지 못한다.

  - 생산자-소비자(Producer-Consumer) 모델
    - 하나 이상 생산자 스레드가 정보를 생성해 버퍼나 대기열에 넣고, 하나 이상 소비자 스레드가 대기열에서 정보를 가져와 사용한다.
    - 생산자 스레드와 소비자 스레드가 사용하는 대기열은 한정된 자원이다.
    - 생산자 스레드는 대기열에 빈 공간이 있어야 정보를 채울 수 있으므로 빈 공간이 없을 경우 빈 공간이 생길 때까지 기다린다.
    - 소비자 스레드는 대기열에 정보가 있어야 가져올 수 있으므로 대기열에 정보가 들어올 때까지 기다린다.
    - 대기열을 올바로 사용하고자 생산자 스레드와 소비자 스레드는 서로에게 시그널을 보낸다.
    - 생산자 스레드는 대기열에 정보를 채운 후 소비자 스레드에 시그널을 보내며, 소비자 스레드는 대기열을 확인한 후 정보가 없다면 생성자 스레드에 시그널을 보낸다.
    - 따라서 잘못 하면 생산자 스레드와 소비자 스레드가 둘 다 진행 가능함에도 불구하고 동시에 서로에게서 시그널을 기다릴 가능성이 있다.

  - 읽기-쓰기(Readers-Writers)
    - 읽기 스레드를 위한 주된 정보원으로 공유 자원을 사용하지만, 쓰기 스레드가 이 공유 자원을 이따금 갱신한다고 하자.
    - 이런 경우 처리율이 문제의 핵심이다.
    - 처리율을 강조하면 기아 현상이 생기거나 오래된 정보가 쌓인다.
    - 갱신을 허용하면 처리율에 영향을 미친다.
    - 쓰기 스레드가 버퍼를 갱신하는 동안 읽기 스레드가 버퍼를 읽지 않으려면, 마찬가지로 읽기 스레드가 버퍼를 읽는 동안 쓰기 스레드가 버퍼를 갱신하지 않으려면 복잡한 균형 잡기가 필요하다.
    - 대개는 쓰기 스레드가 버퍼를 오랫동안 점유하는 바람에 여러 읽기 스레드가 버퍼를 기다리느라 처리율이 떨어진다.
    - 따라서 읽기 스레드의 요구와 쓰기 스레드의 요구를 적절히 만족시켜 처리율도 적당히 높이고 기아도 방지하는 해법이 필요하다.
    - 간단한 전략은 읽기 스레드가 없을 때까지 갱신을 원하는 쓰기 스레드가 버퍼를 기다리는 방법이다.
    - 하지만 읽기 스레드가 계속 이어진다면 쓰기 스레드는 기아 상태에 빠진다.
    - 반면 쓰기 스레드에게 우선권을 준 상태에서 쓰기 스레드가 계속 이어진다면 처리율이 떨어진다.

  - 식사하는 철학자들(Dining Philosophers)
    - 둥근 식탁에 철학자 한 무리가 둘러앉아 있으며, 각 철학자 왼쪽에는 포크가 놓여있다.
    - 식탁 한 가운데에는 음식이 있으며 철학자들은 배가 고프지 않으면 생각하면서 시간을 보낸다.
    - 배가 고프면 양 손에 포크를 들고 음식을 먹지만 양손에 포크를 쥐지 않으면 먹지 못한다.
    - 왼쪽 철학자나 오른쪽 철학자가 포크를 사용하는 중이라면 그쪽 철학자가 먹고 나서 포크를 내려놓을 때까지 기다려야 한다,
    - 음식을 먹고 나면 포크를 내려놓고 배가 고플 때가지 다시 생각에 잠긴다.
    - 여기서 철학자는 스레드를, 포크는 자원을 나타낸다.



- 동기화하는 메서드 사이에 존재하는 의존성을 이해하라
  - 동기화하는 메서드 사이에 의존성이 존재하면 동시성 코드에 찾아내기 어려운 버그가 생긴다.
    - 공유 클래스 하나에 동기화된 메서드가 여럿이라면 구현이 올바른지 다시 한 번 확인해야한다.
  - 공유 객체 하나에 여러 메서드가 필요한 상황도 생기는데, 이 때는 아래 세 가지 방법을 고려한다.
    - 클라이언트에서 잠금: 클라이언트에서 첫 번째 메서드를 호출하기 전에 서버를 잠그고, 마지막 메서드를 호출할 때까지 잠금을 유지한다.
    - 서버에서 잠금: 서버에다 서버를 잠그고 모든 메서드를 호출한 후 잠금을 해제하는 메서드를 구현하며, 클라이언트는 이 메서드를 호출한다.
    - 연결(Adapated) 서버: 잠금을 수행하는 중간 단계를 생성한다. 서버에서 잠금 방식과 유사하지만 원래 서버는 변경하지 않는다.



- 동기화하는 부분을 작게 만들어라
  - 락은 스레드를 지연시키고 부하를 가중시킨다.
    - 같은 락으로 감싼 모든 코드 영역은 한 번에 한 스레드만 실행이 가능하다.
    - 그러므로 여기저기서 락을 거는 것은 바람직하지 않다.
  - 반면 임계 영역은 반드시 보호해야한다.
    - 따라서 코드를 짤 때는 임계 영역 수를 최대한 줄여야 한다.
    - 그러나 임계 영역의 개수를 줄이기 위해 거대한 임계 영역 하나로 구현해서는 안 된다.
    - 필요 이상으로 임계 영역 크기를 키우면 스레드 간에 경쟁이 늘어나고 프로그램 성능이 떨어진다.



- 올바른 종료 코드는 구현하기 어렵다
  - 영구적으로 돌아가는 시스템을 구현하는 방법과 잠시 돌다 깔끔하게 종료되는 시스템을 구현하는 방법은 다르다.
  - 깔끔하게 종료하는 코드는 올바로 구현하기 어렵다.
  - 가장 흔히 발생하는 문제가 데드락이다.
    - 즉 스레드가 절대 오지 않을 시그널을 기다린다.
    - 예를 들어 부모 스래드가 자식 스레드를 여러 개 만든 후 모두가 끝가리를 기다렸다 자원을 해제하고 종료하는 시스템이 있다고 가정하자.
    - 만약 자식 스레드 중 하나가 데드락에 걸렸다면 부모 스레드는 영원히 기다리고, 시스템은 영원히 종료하지 못한다.
    - 이번에는 유사한 시스템이 사용자에게서 종료하라는 지시를 받았다고 가정하자.
    - 부모 스레드는 모든 자식 스레드에게 작업을 멈추고 종료하라는 시그널을 전달한다.
    - 그런데 자식 스레드 중 두 개가 생산자/소비자 관계라면 생산자 스레드는 재빨리 종료했는데, 소비자 스레드가 생산자 스레드에서 오는 메시지를 기다리고 있는 상황이 발생할 수 있다.
    - 이 경우 소비자 스레드는 차단(blocked) 상태에 있으므로 종료하라는 시그널을 받지 못해 생산자 스레드를 영원히 기다리고, 부모 스레드는 소비자 스레드를 영원히 기다린다.



- 스레드 코드 테스트하기
  - 코드가 올바르다고 증명하기는 현실적으로 불가능하다.
    - 테스트가 정확성을 보장하지는 않지만, 충분한 테스트는 위험을 낮춘다.
    - 적어도 스레드가 하나인 프로그램에서는 그렇다.
    - 같은 코드와 같은 자원을 사용하는 스레드가 둘 이상으로 늘어나면 상황은 급격히 복잡해진다.
  - 말이 안 되는 실패는 잠정적인 스레드 문제로 취급하라.
    - 다중 스레드 코드는 때때로 말이 안 되는 오류를 일으킨다.
    - 대다수 개발자는 스레드가 다른 스레드와 교류하는 방식을 직관적으로 이해하지 못한다.
    - 스레드 코드에 잠입한 버그는 수백만 번에 한 번씩 드러나기도 하므로, 실패를 재현하기 아주 어렵다.
    - 그래서 많은 개발자가 일회성 문제로 치부하고 무시한다.
    - 일회성 문제를 계속 무시한다면 잘못된 코드 위에 코드가 계속 쌓인다.
  - 다중 스레드를 고려하지 않은 순차 코드부터 제대로 돌게 만들자.
    - 스레드 환경 밖에서 코드가 제대로 도는지 반드시 확인한다.
    - 스레드 환경 밖에서 생기는 버그와 스레드 환경에서 생기는 버그를 동시에 디버깅하지 마라.
    - 먼저 스레드 환경 밖에서 코드를 올바로 돌려라.
  - 다중 스레드를 쓰는 코드 부분을 다양한 환경에 쉽게 끼워 넣을 수 있게 스레드 코드를 구현하라
    - 다중 스레드를 쓰는 코드를 다양한 설정으로 실행하기 쉽게 구현하라.
    - 한 스레드로 실행하거나, 여러 스레드로 실행하거나, 실행 중 스레드 수를 바꿔본다.
    - 스레드 코드를 실제 환경이나 테스트 환경에서 돌려본다.
    - 테스트 코드를 빨리, 천천히, 다양한 속도로 돌려본다.
    - 반복 테스트가 가능하도록 테스트 케이스를 작성한다.
  - 다중 스레드를 쓰는 코드 부분을 상황에 맞게 조율할 수 있게 작성하라
    - 적절한 스레드 개수를 파악하려면 상당한 시행착오가 필요하다.
    - 처음부터 다양한 설정으로 프로그램의 성능 측정 방법을 강구한다.
    - 스레드 개수를 조율하기 쉽게 코드를 구현한다.
    - 프로그램이 돌아가는 도중에 스레드 개수를 변경하는 방법도 고려한다.
    - 프로그램 처리율과 효율에 따라 스스로 스레드 개수를 조율하는 코드도 고민한다.
  - 프로세서 수보다 많은 스레드를 돌려보라.
    - 시스템이 스레드를 스와핑할 때도 문제가 발생한다.
    - 스와핑을 일이키려면 프로세서 수보다 많은 스레드를 돌린다.
    - 스와핑이 잦을수록 임계영역을 빼먹은 코드나 데드락을 일이키는 코드를 찾기 쉬워진다.
  - 다른 플랫폼에서 돌려보라
    - 다중 스레드는 플랫폼에 따라 다르게 돌아가므로, 코드가 돌아갈 가능성이 있는 모든 플랫폼에서 테스트를 수행해야한다.
    - 처음부터 모든 플랫폼에서 코드를 실행해봐라.
  - 코드에 보조 코드(instrument)를 넣어 돌려라, 강제로 실패를 일이키게 하라.
    - 스레드 코드는 오류를 찾기가 쉽지 않다.
    - 간단한 테스트로는 버그가 드러나지 않으며, 몇 주가 지나서야 한 번씩 문제를 일으키는 경우도 흔하다.
    - 스레드 버그가 산발적이고 재현이 어려운 이유는 코드가 실행되는 수천가지 경로 중에 아주 소수만 실패하기 때문이다.
    - 보조 코드를 추가해 코드가 실행되는 순서를 바꾸면 드물게 발생하는 오류를 좀 더 자주 일으킬 수 있다.
    - 예를 들어 `Object.wait()`, `Object.sleep()`, `Object.yield()`, `Object.priority()` 등과 같은 메서드를 추가해 다양한 순서로 실행한다.
    - 각 메서드는 스레드가 실행되는 순서에 영향을 미치므로 버그가 드러날 가능성도 높아진다.



- 코드에 보조 코드를 추가하는 방법

  - 직접 구현하기
    - 코드에다 직접 `wait()`, `sleep()`, `yield()`, `priority()` 함수를 추가한다.
    - 특별히 까다로운 코드를 테스트할 때 적합하다.
    - 아래 코드에서 `yield()`를 삽입하면 코드가 실행되는 경로가 바뀌어 이전에 실패하지 않았던 코드가 실패할 가능성을 열어준다.
    - 코드가 실패한다면 `yield()`를 추가했기 때문이 아니며(물론 추가 때문일 수도 있다), 원래 잘못된 코드인데 증거가 드러났을 분이다.

  ```java
  public synchronized String nextUrlOrNull() {
      if (hasNext()) {
          String url = urlGenerator.next();
          Thread.yield();  // 테스트를 위해 추가
          updateHasNext();
          return url;
      }
      return null;
  }
  ```

  - 직접 구현하는 방식의 문제점
    - 보조 코드를 삽입할 적정 위치를 직접 찾아야 한다.
    - 어떤 함수를 어디서 호출해야 적당한지를 알기 어렵다.
    - 배포 환겨에서 보조 코드를 그대로 남겨두면 프로그램 성능이 떨어진다.
    - 오류가 드러날 가능성이 무작위적이다.

  - 자동화하기
    - 보조 코드를 자동으로 추가하려면 AOF(Aspect-Oriented Framework), CGLIB, ASM 등과 같은 도구를 사용한다.
    - 예를 들어 아래 `ThreadJigglePoint`는 메서드가 하나인 클래스다.
    - 다양한 위치에 `ThreadJigglePoint.jiggle()`호출을 추가한다.
    - `ThreadJigglePoint.jiggle()`는 무작위로 sleep이나 yield를 호출하며 때로는 아무 동작도 하지 않는다.

  ```java
  public class ThreadJigglePoint {
      public static void jiggle() {}
  }
  
  
  public synchronized String nextUrlOrNull() {
      if (hasNext()) {
          ThreadJigglePoint.jiggle();
          String url = urlGenerator.next();
          ThreadJigglePoint.jiggle();
          updateHasNext();
          ThreadJigglePoint.jiggle();
          return url;
      }
      return null;
  }
  ```

  - `ThreadJigglePoint` 클래스를 두 가지로 구현하면 편리하다.
    - 하나는 `jiggle()` 메서드를 비워두고 배포 환경에서 사용하고, 다른 하나는 무작위로 nop, sleep, yield 등을 테스트 환경에서 수행한다.
    - 둘째 구현으로 테스트를 수천 번 실행하면 스레드 오류가 드러날지 모른다.
    - 코드가 수천 번에 이르는 테스트를 통과한다면 나름대로 할 만큼 했다고 마래도 되겠다.



