# 예외처리

- 예외처리
  - 프로그래밍을 하다 보면 수 없이 많은 오류와 마주하게 된다.
  - 오류가 발생하면 오류가 발생한 줄 아래로는 실행이 되지 않는데 때로는 오류가 발생하더라도 실행해야 할 때가 있다.
  - 그럴 때 사용하기 위한 것이 예외처리 기법이다.



- `try`, `except` 문

  - 기본 구조
    - try 블록 실행 중 오류가 발생하면 except 블록이 실행된다.
    - except문에 발생 오류를 적어놓으면 해당 오류가 발생했을 때에만 except 블록이 실행된다.

  ```python
  # 괄호 안의 내용은 모두 생략이 가능하다.
  try:
      실행할 문장
  except [발생 오류 as 오류 메세지 담을 변수]:
      실행할 문장
  ```

  - 예시

  ```python
  # arr을 설정해 준 적 없기에 아래 문장은 error을 발생시킨다.
  print(arr)
  print("에러가 발생했으므로 출력이 안됩니다.")
  """
  Traceback (most recent call last):
    File "c:\Users\pract.py", line 6, in <module>
      print(arr)
  NameError: name 'arr' is not defined
  """
  
  # 아래와 같이 예외 처리가 가능하다.
  try:
      print(arr)
  except:
      print("에러 발생")
  print("에러가 발생했지만 실행이 됩니다.")
  """
  에러 발생
  에러가 발생했지만 실행이 됩니다.
  """
  
  # error를 특정하는 것도 가능하다.
  try:
      print(arr)
  except NameError as e:
      print("e에는 error message가 담겼습니다.")
      print(e)
  print("에러가 발생했지만 실행이 됩니다.")
  """
  e에는 error message가 담겼습니다.
  name 'arr' is not defined
  에러가 발생했지만 실행이 됩니다.
  """
  
  # error를 특정할 경우 특정하지 않은 에러가 발생하면 except로 빠지지 않는다.
  try:
      3//0
  except NameError as e:
      print("에러 종류가 다르므로 빠지지 않습니다.")
      print(e)
  """
  Traceback (most recent call last):
    File "c:\Users\pract.py", line 2, in <module>
      3//0
  ZeroDivisionError: integer division or modulo by zero
  """
  ```

  

- `finally`

  - try문 수행 도중 예외 발생 여부와 무관하게 수행된다.
  - 보통 사용한 리소스를 close해야 할 때 사용한다.

  ```python
  try:
      3//0
  finally:
      print("ZeroDivisionError가 발생했지만 finally 블록은 실행됩니다.")
  """
  ZeroDivisionError가 발생했지만 finally 블록은 실행됩니다.
  Traceback (most recent call last):
    File "c:\Users\pract.py", line 2, in <module>
      3//0
  ZeroDivisionError: integer division or modulo by zero
  """    
  ```




- 복수의 오류 처리하기

  - except를 여러 개 사용하면 된다.
  - 가장 먼저 진입한 except문만 실행 되고 다음 try 블록의 다음 줄은 실행 되지 않는다.

  ```python
  try:
      3//0
      print(arr)
  except ZeroDivisionError as e:
      print(e)	# integer division or modulo by zero 
  except NameError as e:
      print(e)
  
  
  
  # 아래와 같이 하는 것도 가능하다.
  try:
      3//0
      print(arr)
  except (ZeroDivisionError,NameError) as e:
      print(e)	# integer division or modulo by zero
  ```



- 오류 회피하기

  - except 블록에 pass를 쓰면 에러를 그냥 지나칠 수 있다.

  ```python
  try:
      3//0
      print(arr)
  except ZeroDivisionError as e:
      pass
  print("pass 됐습니다.")
  
  # pass 됐습니다.
  ```




- 오류 강제로 발생시키기

  - `raise`를 사용한다.
  - 프로그래밍을 하다 보면 예외를 강제로 발생시켜야 할 때가 있다.
  - Dog이라는 class를 생속 받는 자식 클래스는 반드시 bark라는 메서드를 구현하도록 하고 싶은 경우 아래와 같이 할 수 있다.
    - NotImplementedError는 python 내장 에러로 꼭 구현해야 할 부분이 구현되지 않았을 경우 발생한다.

  ```python
  class Dog:
      def bark(self):
          raise NotImplementedError
          
  class Bichon(Dog):
      pass
  
  bcn = Bichon()
  bcn.bark()			# NotImplementedError
  ```

  - 다음과 같이 bark 메서드를 오버라이딩하여 생성하면 에러가 발생하지 않는다.

  ```python
  class Dog:
      def bark(self):
          raise NotImplementedError
          
  class Bichon(Dog):
      def bark(self):
          print("멍멍")
  
  bcn = Bichon()
  bcn.bark()		# 멍멍
  ```




- 예외 만들기

  - Exception을 상속받는 class 생성

  ```python
  class CustomException(Exception):
      def __init__(self):
          super().__init__('CustomException occur!')
  ```

  - 예외 발생시키기

  ```python
  from custom_exception import CustomException
  
  
  msg = "Bye"
  
  try:
      if msg=="Bye":
          raise CustomException
  except Exception as e:
      print("Exception:",e)	# Exception: CustomException occur!
  ```

  - 혹은 상속만 받고 예외를 발생시킬 때 메시지를 넣어줄 수도 있다.

  ```python
  class CustomException(Exception):
      pass
  ```

  - 예외 발생시키기

  ```python
  from custom_exception import CustomException
  
  
  msg = "Bye"
  
  try:
      if msg=="Bye":
          raise CustomException("CustomException occur!")
  except Exception as e:
      print("Exception:",e)	# Exception: CustomException occur!
  ```

  - class를 아래와 같이 작성하여 추가적인 인자를 받을 수도 있다.
    - `__str__`이 반환하는 내용이 출력되게 된다.

  ```python
  class CustomException(Exception):
      def __init__(self, code, name):
          self.code  = code
          self.name  = name
      
      def __str__(self):
          return f'{self.code}, {self.name} error occur!'
  ```

  - 예외 발생시키기
    - 또한 args를 통해 직접 작성한 예외 클래스의 인자들을 확인 가능하다.

  ```python
  from custom_exception import CustomException
  
  
  msg = "Bye"
  
  try:
      if msg=="Bye":
          raise CustomException(1, "some exception")
  except Exception as e:
      print("Exception:",e)	# Exception: 1, some exception error occur!
      print(e.args)			# (1, 'some exception')
  ```



- traceback

  - Exception을 출력하는 것으로는 traceback까지는 확인이 불가능한데, traceback 모듈을 사용하면 traceback도 확인이 가능하다.
  - `format_exc()`
    - traceback을 문자열로 변환

  ```python
  import traceback
  
  try:
      1//0
  except Exception as e:
      print(e)	# integer division or modulo by zero
      print(traceback.format_exc())
  """
  Traceback (most recent call last):
    File "test.py", line 4, in <module>
      1//0
  ZeroDivisionError: integer division or modulo by zero
  """
  ```

  - `extract_tb()`와 `sys.exc_info()`를 활용하면 최초에 Exception이 발생한 파일을 추적 가능하다.

  ```python
  # error를 발생시킬 파일
  # error_occur.py
  def print_hello():
      print(msg)
  
  
  # error가 발생한 파일을 추적하는 코드
  # test.py
  import os
  import sys
  import traceback
  
  from test2 import print_hello
  
  
  try:
      print_hello()
  except Exception as e:
      error_file_path = traceback.extract_tb(sys.exc_info()[-1])[-1].filename
      print(error_file_path)
  ```



- error가 발생한 모듈 이름을 얻는 방법

  - `traceback` 모듈의 `extract_tb()` 메서드를 사용한다.
  - my_module.py

  ```python
  def foo():
      print(1//0)
  ```

  - test.py

  ```python
  import sys
  import os
  import traceback
  
  import my_module
  
  
  try:
      module.foo()
  except Exception as e:
      print(traceback.extract_tb(sys.exc_info()[-1])[-1].filename)
  ```



- try, execpt, else

  - else는 except 절이 실행되지 않았을 경우 실행되는 절이다.

  ```python
  try:
      print("hello world")
  except:
      print("error!")
  else:
      print("no error!")	# 실행된다.
  
      
  try:
      1//0
  except:
      print("error!")
  else:	
      print("no error!")	# 실행되지 않는다.
  ```



- Python의 모든 Exception class는 `BaseException` class를 상속받은 것이다.

  - pydoc을 통해 확인이 가능하다.

  ```bash
  $ pydoc builtins
  ```

  - `BaseException` class는 `__new__`메서드(`__init__` 메서드가 아니다)의 인자로 `args`를 받는다.
    - 따라서 `BaseException`을 상속 받은 하위 class에서 `__init__` 메서드를 실행하더라도 `args`는 여전히 존재하게 된다.
    - CPython의 `BaseException` code의 일부를 Python code로 변환하면 아래와 같다.

  ```python
  class BaseException(Exception):
      def __new__(cls, *args):
          # self = self.에 object를 생성하여 할당하는 code
          self.args = args
          return self
  ```

  - `args`가 **유일하게** 사용되는 곳은 `__str__` 메서드로, `args`를 string으로 변환하여 반환하는 역할을 한다.
    - CPython의 `BaseException`내부의 `__str__` 메서드를 python code로 변환하면 아래와 같다.

  ```python
  def __str__(self):
      if len(self.args) == 0:
          return ""
      if len(self.args) == 1:
          return str(self.args[0])
      return str(self.args)
  ```

  - 확인해보기

  ```python
  try:
      raise Exception
  except Exception as e:
      print(e)	#
  try:
      raise Exception("hello")
  except Exception as e:
      print(e)	# "hello"
  try:
      raise Exception("hello", "world", 42)
  except Exception as e:
      print(e)	# ('hello', 'world', 42)
  ```



- 새로운 Exception 만들기

  - Exception을 만들어야하는 이유
    - 개발을 하다보면 Python에서 제공하는 built in exception로 처리할 수 없는 예외들이 존재한다.
    - 따라서 개발자가 이러한 예외들을 직접 만들어서 처리해줘야한다.
  - 항상 `Exception` class를 상속받아야 한다.
    - `Exception` class는 `BaseException` class를 상속 받는 class이다.

  ```python
  class MyException(Exception):
      pass
  ```

  - 반드시 어떤 예외가 발생한 것인지 정확히 할 수 있도록 정확한 메시지를 남겨야한다.

  ```python
  class MyException(Exception):
      def __init__(self, msg):
          super(MyException, self).__init__(msg)
  ```

  - 굳이 부모 class의  `__init__` 메서드를 실행해야 하는 이유는?
    - 사실 위와 같이 직접 작성한 Exception class가 인자를 하나만 받을 때는 굳이 부모 class의 `__init__` 메서드를 실행시키지 않아도 된다.
    - 부모 class의 `__init__`메서드가 실행되며, msg가 넘어가기 때문이다.
    - 그러나 아래와 같이 default msg를 설정해줘야 하거나 인자를 둘 이상 받을 때는 반드시 `__init__` 메서드를 실행해야한다.

  ```python
  # default msg를 설정하고자 하는 경우
  class MyException(Exception):
      def __init__(self, msg=None):
          if msg is None:
              msg = "Default detail message"
          super(MyException, self).__init__(msg)
          
  # 인자를 둘 이상 받아야 하는 경우
  class MyException(Exception):
      def __init__(self, foo, msg=None):
          print(foo)
          if msg is None:
              msg = "Default detail message"
          super(MyException, self).__init__(msg)
  
  # 만일 인자를 둘 이상 받을 때, 부모 class의 __init__메서드를 실행시키지 않으면, 모든 인자가 부모 class의 args인자로 넘어가게 된다.
  # 아래 예시에서 실제 exception 메시지로 사용하려 한 것은 world인데, world뿐 아니라 hello까지 부모 class의 __init__ 메서드의 args로 넘어가게 된다.
  class MyException(Exception):
      def __init__(self, foo, msg=None):
          pass
  
  try:
      raise MyException("hello", "world")
  except MyException as e:
      print(e)	# ('hello', 'world')
  ```

  - 인자로 받은 값을 활용하기
    - 인자로 받은 값을 활용하여 보다 구체적인 예외 처리가 가능하다.

  ```python
  class MyException(Exception):
      def __init__(self, msg, foo):
          self.msg = msg
          self.foo = foo
          super(MyException, self).__init__(msg)
    
  try:
  
      raise MyException("Exception!!", False)
  except Exception as e:
      if e.foo:
          do_something()
  ```



- Python에서 Error를 다루는 두 가지 방식

  - LBYL

  ```python
  if os.path.exists(file_path):
      os.remove(file_path)
  else:
      print(f"Error: file {file_path} does not exist!")
  ```

  - LBYL을 사용한 위 방식에는 실질적인 문제가 있다.
    - 위 방식의 경우 file을 삭제하기 위해 충족되어야 하는 모든 가정이 충족되는지 확인한 후에 `os.remove()` method를 실행해야 한다는 점이다.
    - 예를 들어 위에서는 file의 유무만 확인을 했지만, 입력된 경로가 file이 아닌 directory일 수도 있고, 다른 user의 소유라서 삭제 권한이 없을 수도 있으며, file이 저장된 disk가 read-only volume으로 mount되어 있을 수도 있고, 다른 process가 file을 사용중이라 lock이 걸려있을 수도 있다.
    - File을 삭제한다는 단순한 작업을 위해 위의 모든 가정을 충족하는지 확인하는 조건문을 넣는 것은 매우 비효율적이다.
    - 이처럼 LBYL 방식을 사용하면서 견고한 logic을 작성하는 것은 상당히 어려운 일이다.
  - LBYL과 race condition
    - LBYL을 사용할 때의 또 다른 문제는 race condition이다.
    - 조건문을 통해 가정을 모두 충족되는지 확인을 한 후, 최종적으로 수행하기 직전에 상황이 바뀔 경우 실행이 실패하게 된다.
    - 예를 들어 위 예시에서, file이 있는 것을 확인했을 때는 file이 있었으나, `os.remove()`가 실행되기 직전에 삭제된다면, error가 발생하게 된다.
  - EAFP

  ```python
  try:
      os.remove(file_path)
  except OSError as error:
      print(f"Error deleting file: {error}")
  ```

  - 대부분의 상황에서 LBYL보다 EAFP가 낫다.
    - 정상 실행 여부를 확인하고, error를 보고하는 역할은 target function(예시의 경우 `os.remove`)이 맡는다.
    - 우리는 그저 호출자로서 함수를 호출하고 호출된 함수가 그 결과를 우리에게 알려주게 된다.
  - EAFP의 경우 어떤 exception이 발생할 수 있는지를 알아야한다.
    - `except` 절에 catch할 exception들을 나열해야하기 때문이며, 예상치 못한 exception이 발생할 경우 application에 crush가 발생하게 된다.
    - 위 예시의 경우 파일 삭제와 관련해서 발생할 수 있는 모든 error는 `OSError`이거나 이를 상속 받는 error이기에 위 처럼만 해도 모든 error를 catch할 수 있다.
    - 상황에 따라 어떤 예외가 발생할 수 있는지를 알아야한다.
  - 모든 exception을 전부 catch하려고 해선 안 된다.
    - 대부분의 bug는 예상치 못한 방식으로 발생한다.
    - 모든 exception을 전부 catch할 경우 이러한 예상치 못 한 bug를 잡지 못한 채 지나칠 수 있다.
    - 따라서 가능한 가장 적은 수의 exception class만 사용하여 exception을 catch해야한다.
    - 물론 예외적인 상황도 있다.



- Error의 종류

  > https://blog.miguelgrinberg.com/post/the-ultimate-guide-to-error-handling-in-python

  - New Error와 Bubbled-Up Error
    - Code에서 문제를 찾아서 새로운 error를 생성해야 할 때, 이러한 error를 new error라 부를 것이다.
    - Code가 실행되면서 호출한 함수에서 error가 발생했을 때, 이러한 error를 bubbled-up error라 부를 것이다.
    - Bubbled-up이라 부르는 이유는 error가 발생할 경우 error를 처리할 함수를 만날때 까지 call stack을 따라 올라가기 때문이다.
  - Recoverable Error외 Non-Recoverable Error
    - Recoverable error는 해당 error를 처리하는 code가 error 발생 후에도 다음 logic을 계속 실행할 수 있는 error이다.
    - 예를 들어 file을 삭제하려 하는데, 삭제하려는 file이 없어서 error가 발생한 경우, 해당 error를 처리하는 code는 file을 삭제할 file이 이미 없으므로 error를 무시해도 무방하다.
    - Non-recoverable error는 error를 처리하는 code가 더 이상 다음 logic을 수행할 수 없는 error이다.
    - 예를 들어 DB로부터 data를 읽어와서 수정하는 function이 있을 때, 이 function은 읽어오는 데 실패하면 다음 단계인 수정도 불가능하다.
  - 위와 같이 종류별로 2개씩 총 4개의 error가 존재하며, 두 종류의 error가 어떻게 조합되느냐에 따라 처리 방법도 달라진다.
    - New Recoverable
    - Bubbled-up Recoverable
    - New Non-Recoverable
    - Bubbled-up Non-Recoverable



- New Recoverable

  - DB에 data를 삽입하는 function이 있다고 가정해보자.
    - DB schema에서 year는 null이어선 안 되는 상황이다.
    - LBYL 방식에 따라 아래와 같이 year의 값이 None인지 check하고, None일 경우 임의의 값을 넣어주어 DB에 삽입할 때 error가 발생하지 않게 한다.

  ```python
  def add_song_to_database(song):
      # ...
      if song.year is None:
          song.year = 'Unknown'
      # ...
  ```

  - 이처럼 복구가 가능한 error가 있고 굳이 error를 raise시키지 않아도 된다면, 굳이 error를 raise하지 않고 적절한 처리를 실행한 후 다음 logic을 계속 실행한다.



- Bubbled-Up Recoverable

  - 마찬가지로 DB에 data를 삽입하는 function이 있다고 가정해보자.
    - `artist` 정보를 DB에서 받아와서 이 정보를 활용해야하는 상황이다.
    - EAFP 방식에 따라 error를 catch하고, 적절한 처리를 한 후 계속 진행한다.

  ```python
  def add_song_to_database(song):
      # ...
      try:
          # DB에 artist 정보가 없으면
          artist = get_artist_from_database(song.artist)
      except NotFound:
          # DB에 artist 정보를 추가한다.
          artist = add_artist_to_database(song.artist)
      # ...
  ```

  - 첫 번째 사례와 마찬가지로, 복구가 가능한 error이므로, 호출함 함수에서 반환한 error를 catch하여 이를 적절히 처리한 후 다음 logic을 계속 실행한다.
    - 예외 처리가 마무리 되었으므로, call stack의 더 위에 있는 계층에서는, error가 발생했다는 것을 알 필요가 없다.
    - 따라서 error의 bubbling up도 이 단계에서 종료된다.



- New Non-Recoverable

  - 앞의 두 경우와 다르게, 이 경우는 error 처리하기 위해 뭘 해야하는지 모르는 경우이다.
    - 이 경우에 할 수 있는 유일한 일은 call stack의 한 단계 위에 있는 level에게 경고를 보내고, 해당 단계가 error를 처리할 수 있기를 바라는 것이다.
    - 대부분의 경우에 non-recoverable error는 call stack을 따라 올라가다보면 recoverable한 상태가 되기에, 이 전략은 잘 동작한다.
    - 즉 call stack을 따라 올라가다보면 new non-recoverable error는 bubbled-up recoverable error가 된다.
    - 따라서 어떻게 처리해야하는 지 알 수 있게 되고, 적절하게 처리할 수 있게 된다.
  - DB에 data를 삽입하는 function이 있다고 가정해보자.
    - 첫 번째 유형에서는 year가 없는 경우를 가정했으나, 이번에는 name이 없는 경우를 가정할  것이다.
    - name은 year와는 달리 단순히 null이어서는 안 되는 값이 아니라 반드시 있어야 하는 값인 상황이다.
    - 현재 단계에서는 name이 없을 경우 어떻게 처리해야 하는지 알 수 없으므로, 아래와 같이 error를 raise하여 상위 call stack으로 보낸다.

  ```python
  def add_song_to_database(song):
      # ...
      if song.name is None:
          raise ValueError('The song must have a name')
      # ...
  ```

  - 만약 Python 내장 exception중 적절한 것이 없다면, 아래와 같이 exception class를 생성하면 된다.

  ```python
  class ValidationError(Exception):
      pass
  
  # ...
  
  def add_song_to_database(song):
      # ...
      if song.name is None:
          raise ValidationError('The song must have a name')
      # ...
  ```

  - 주목해야할 점은 `raise` keyword가 function의 실행을 중단시킨다는 것이다.
    - Error를 처리할 수 있는 방법을 모르는 상황이므로 function의 실행을 중단시키는 것이 자연스럽다.
    - Exception을 raise하여 현재 함수의 실행이 중단되고, 해당 exception을 처리할 수 있는 계층에 도달할 때 까지 call stack을 따라 올라가게 된다.



- Bubbled-Up Non-Recoverable Error

  - 아래와 같이 특정 function을 호출하는코드가 있다고 가정해보자.
    - 호출한 함수에서 exception이 raise되고, 이를 어떻게 처리해야하는지 모르는 상황이다.
    - 이 상황에서는 아무것도 하지 않아야한다.

  ```python
  def new_song():
      song = get_song_from_user()
      add_song_to_database(song)
  ```

  - Error를 hadnling하지 않는 것은 좋은 error handling 전략일 수 있다.
    - `new_song()`에서 호출되는 두 개의 function은 실패후에 exception을 raise할 수 있다.
    - 만약 우리가 이 error들을 처리할 방법을 모른다면 error를 catch할 이유도 없다.
    - 다만, 이것이 error를 무시해야한다는 것을 의미하지는 않는다.
    - 아무것도 하지 않고, 그저 error가 call stack을 따라 올라가도록 함으로써, 해당 error를 처리할 수 있는 단계에 도달하도록 해야한다.
    - 가능한 한 많은 function이 error 처리를 신경쓰지 않도록 하고, error 처리와 관련된 code를 상위 계층으로 옮기는 것은 clean code를 작성하는 좋은 전략이다.
  - 관심사의 분리와 error handling
    - 누군가는 그래도 error와 관련된 log라도 남겨서 사용자에게 error가 발생했다는 사실을 알려야 하는 것이 아니냐고 물을 수 있다.
    - 그러나, 만약 application이 CLI가 아닌 GUI라면, 혹은 web application이라면 print를 통해 stdout을 남기는 것은 소용이 없다.
    - GUI라면 alert 창이나 modal 창을 통해 error message를 보여줘야 할 것이고, web application이라면 HTTP 통해 error를 응답으로 반환해야 할 것이다.
    - 문제는, function 그 자체는 현재 자신이 CLI에서 실행되는지, GUI에서 실행되는지, web application에서 실행되는지 알 필요가 없다는 것이다.
    - Error가 발생했다는 것을 사용자에게 알리는 방식은 application의 종류에 따라 달라지게 되지만, function은 application의 종류와 무관하게 동일한 logic을 수행해야한다.
    - 따라서 관심사의 분리를 통해 사용자에게 error 발생 사실을 알리는 것은 더 상위 계층에서 처리하도록 하고, 하위 계층의 function들은 자신이 맡은 역할만 수행하게 하는 것이 바람직하다.



- Catching All Exceptions

  - 정말 bubbled-up non-recoverable error 아무 처리 없이 그냥 둬도 되는가
    - 이런 불안감을 느끼는 이유는, error가 처리되지 않은 채로 call stack을 타고 올라가다 결국 아무 계층에서도 처리되지 않고 맨 위 계층까지 도달하여 application에 crash를 일으킬 것이라 생각하기 때문이다.
    - 이는 할만한 걱정이지만 간단하게 해결이 가능하다.
  - Error가 가장 높은 계층에서는 무조건 잡을 수 있도록 application을 설계해야한다.
    - 최상위 계층에 모든 exception을 catch할 수 있는 `try`/`catch`문을 추가한다.
    - 아래와 같이 최상위 계층에 모든 error를 잡을 수 있는 `try`/`catch`를 추가하고, 사용자에게 message를 출력하며, exit code 1과 함께 exit 되도록 해 shell이나 다른 parent process가 application이 실패했다는 것을 알 수 있게 한다.

  ```python
  import sys
  
  def my_cli()
      # ...
  
  if __name__ == '__main__':
      try:
          my_cli()
      except Exception as error:
          print(f"Unexpected error: {error}")
          sys.exit(1)
  ```

  - 모든 exception을 catch하는 것은 부절절한 것 아닌가?
    - 위에서 말했듯 모든 exception을 catch하는 것이 적절한 예외가 존재하며, 이 경우가 그에 해당한다.
    - 실제로 상위 계층에서 모든 exception을 catch하는 것은 흔한 pattern이며 많은 application에서 사용하고 있다.
  - 모든 errror를 catch하는 계층이 있다는 것을 활용하지 않으면, 아래와 같은 code를 작성하게 된다.
    - 아래 코드는 DB에 데이터를 추가하는 코드다.
    - 예외가 발생할 경우 log에 error가 발생했다는 사실을 남기고, rollback을 수행한다.
    - 그런데, rollback도 실패할 수 있으므로, 이 경우에도 마찬가지로 log에 error가 발생했다는  사실을 남기고 500 error를 반환한다.

  ```python
  @app.put('/songs/{id}')
  def update_song(id):
      # ...
      try:
          db.session.add(song)
          db.session.commit()
      except SQLAlchemyError:
          current_app.logger.error('failed to update song %s, %s', song.name, e)
          try:
              db.session.rollback()
          except SQLAlchemyError as e:
              current_app.logger.error('error rolling back failed update song, %s', e)
          return 'Internal Service Error', 500
      return '', 204
  ```

  - 위 방식은 아래와 같은 문제점들이 있다.
    - Rollback이 실패했다는 것은 DB에 심각한 문제가 생겼다는 것이므로, DB와 관련된 log를 남기는 것이 의미가 없다.
    - 또한 commit이 실패했을 때도 log에 stack trace와 관련된 정보를 남기지 않아 무엇 때문에 commit이 실패했는지 추적이 어렵다.
  - 반면 모든 errror를 catch하는 계층이 있다는 것을 활용하면 보다 깔끔한 코드를 작성할 수 있다.
    - 아래와 같이 작성할 경우, stack  trace는 framework에서 로그로 남겨주므로 보다 깔끔한 코드를 작성할 수 있다.

  ```python
  @app.put('/songs/{id}', status_code=204)
  def update_song(id):
      # ...
      db.session.add(song)
      db.session.commit()
      return
  ```



- 올바른 추상화 단계에서 예외 처리를 해야 한다.

  - 예외는 오직 한 가지 일을 하는 함수의 한 부분이어야 한다.
    - 함수가 처리하는(또는 발생시키는) 예외는 함수가 캡슐화하고 있는 로직에 대한 것이어야 한다.
  - 아래는 서로 다른 수준의 추상화를 혼합하는 예제이다.
    - `deliver_event()` 메서드가 예외를 처리하는 방법에는 몇 가지 문제가 있다.
    - 우선 `ConnectionError`와 `ValueError` 사이에는 별 관계가 없으며, `ConnectionError`는 `connect()`메서드 내에서 처리하는 것이 바람직하다.
    - 이렇게 하면 행동을 명확하게 분리할 수 있다.
    - 반대로 `ValueError`는 `Event.decode()`에 속한 에러로, 이 역시 여기서 처리하게 하면 `deliver_event()` 메서드에서는 더 이상 예외를 처리할 필요가 없어진다. 

  ```python
  class DataTransport:
      _RETRY_BACKOFF = 5
      _RETRY_TIMES = 3
      
      def __init__(self, connector: Connector) -> None:
          self._connector = connector
          self.connection = None
          
      def deliver_event(self, event: Event):
          try:
              self.connect()
              data = event.decode()
              self.send(data)
          except ConnectionError as e:
              logger.info("Connection Error")
              raise
          except ValueError as e:
              logger.error("Invalid data detected in event payload.")
              raise
      
      def connect(self):
          for _ in range(self._RETRY_TIMES):
              try:
                  self.connection = self._connector.connect()
              except ConnectionError as e:
                  logger.info("Connection Error")
                  time.sleep(self._RETRY_BACKOFF)
              else:
                  return self.connection
          raise ConnectionError
      
      def send(self, data: bytes):
          return self.connection.send(data)
  ```

  - 위 코드를 아래와 같이 개선할 수 있다.
    - 이전에 `deliver_event`가 처리하던 예외는 각각의 내부 메서드에서 처리하거나 의도적으로 예외가 발생하도록 내버려둔다.
    - `deliver_event` 메서드는 다른 메서드나 함수로 분리한다.
    - 연결을 관리하는 것은 별도의 함수로 작성하여, 연결과 관련된 비즈니스 로직, 예외 처리, 로깅을 담당하게 한다.

  ```python
  def connect_with_retry(connector: Connector, retry_n_times: int, retry_backoff: int = 5):
      for _ in range(retry_n_times):
          try:
          	return connector.connect()
          except ConnectionError as e:
              logger.info("Connection failed")
              time.sleep(retry_backoff)
          exc = ConnectionError("Connection failed")
          logger.exception(exc)
          raise exc
  
  
  class DataTransport:
      _RETRY_BACKOFF = 5
      _RETRY_TIMES = 3
      
      def __init__(self, connector: Connector) -> None:
          self._connector = connector
          self.connection = None
          
      def deliver_event(self, event: Event):
          self.connection = connect_with_retry(srlf._connector, self._RETRY_TIMES, self._RETRY_BACKOFF)
          self.send(event)
  	
      def send(self, event: Event):
          try:
              return self.connection.send(event.decode())
          except ValueError as e:
              logger.error("Invalid data detected in event payload.")
  ```



- 비어있는 except 블록은 지양해야한다.

  - 이는 Python의 안티 패턴 중에서도 가장 악마 같은 패턴이다.
    - 일부 오류에 대비하여 프로그램을 방어하는 것은 좋은 일이지만, 너무 방어적인 것은 더 심각한 문제로 이어질 수 있다.
    - 특히 너무 방어적이어서 아무것도 하지 않은 채로 조용히 지나쳐버리는 비어있는 except 블록은 가장 안 좋은 예이다.

  - 아래와 같은 코드의 문제는 어떠한 예외도 발견할 수 없다는 것이다.
    - "Errors should never pass silently"라는 Python의 철학(The Zen of Python)에 정면으로 위배된다.

  ```python
  try:
      do_something()
  except:
      pass
  ```

  - 예외는 구체적으로 catch해야하고, catch한 예외는 적절히 처리해야한다.
    - 보다 구체적인 예외를 사용하면 사용자는 무엇을 기대하는지 알게 되기 때문에 프로그램을 더욱 유지보수하기 쉽다.
    - 또한 다른 종류의 예외가 발생하면 바로 버그로 판단할 수 있으므로 쉽게 대응할 수 있다.
  - 만약 명시적으로 특정 에러를 무시하고자 한다면 `contextlib.suppress` 함수를 사용해야한다.

  ```python
  import contextlib
  
  with contextlib.suppress(KeyError):
      do_somethin()
  ```



- Assertion 사용하기

  - Assertion은 절대로 일어나지 않아야 하는 상황에 사용된다.
    -  `assert` 문에 사용된 표현식은 불가능한 조건을 의미한다.
    - 이 상태가 된다는 것은 소프트웨어에 결함이 있음을 의미한다.
  - Assertion의 목적
    - 어떤 특정 상황(더 이상 극복할 수 없는 오류이거나 프로그램 내에서 스스로 치유할 수 있는 다른 방법을 찾기 어려운 경우)이 발생했을 때는 더 이상 프로그램을 실행하는 것이 의미가 없을 수도 있다.
    - 이런 경우는 빨리 실패하고 수정이 가능하도록 그 상황을 알려주는 것이 더 나은 선택일 수 있다.
    - Assertion은 잘못된 시나리오에 도달하여 프로그램이 더 큰 피해를 입지 않도록 하는 데 목적이 있다.
  - Assertion을 비즈니스 로직과 섞거나 소프트웨어의 제어 흐름 메커니즘으로 사용해서는 안 된다.
    - 문법상으로 assertion은 항상 참이어야만 하는 조건이다.
    - 만약 이 조건이 거짓이 되어 AssertionError가 발생했다면, 프로그램에서 극복할 수 없는 치명적인 결함이 발견되었다는 뜻이다.
    - 아래와 같이 작성해서는 안 된다.
    - AssertionError는 더 이상 처리가 불가능한 상황을 의미하므로 catch 후에 프로그램을 계속 실행하면 단 된다.
    - 특정 조건에 대한 검사가 필요하다면 보다 구체적인 오류를 발생시켜야한다.

  ```python
  try:
      assert condition.hold()
  except AssertionError:
      alternative_procedure()
  ```

  - AssertionError를 catch해야 하는 경우는 오직 프로그램을 gracefully fail하도록 하려는 경우뿐이다.
    - 프로그램이 갑자기 종료되도록 하는 대신, AssertionError를 catch하여 일반적인 에러 메시지를 보여주고, 상세 에러를 내부 시스템에 기록하고 종료시키는 방식으로 사용할 수 있다.
    - Assertion에 실패하면 프로그램이 종료되는지 반드시 확인해야한다.
  - Assertion과 예외 처리의 관계
    - 일반적으로 예외는 예상치 못한 상황을 처리하기 위한 것이고, assertion은 정확성을 보장하기 위해 스스로 체크하기 위한 것이다.
    - 이런 이유로 예외를 발생시키는 것이 assert 구문을 사용하는 것보다 훨씬 일반적이다.
    - assert문은 항상 변하지 않는 고정된 조건에 대해 검증할 때 사용한다.
    - 이 조건이 깨진다면 무엇인가 잘못 구현되었거나 문제가 발생했음을 의미한다.



- 예외 처리시 고려할 점들

  - 예외를 go-to문처럼 사용해선 안 된다.
    - 정상적인 시나리오나 비즈니스 로직을 예외처리하려고 하면 프로그램의 흐름을 읽기가 어려워진다.
    - 이는 예외를 go-to문처럼 사용하는 것과 같다.
    - 예외가 호출 스택의 여러 단계에서 사용될 경우 올바른 위치에서 추상화를 하지 못하게 되고, 로직을 캡슐화하지도 못하게 된다.
    - 그리고 프로그램이 꼭 처리해야 하는 정말 예외적인 비즈니스 로직을 except 블록과 혼합하여 사용하면 상황이 더우 악화될 수 있다.
    - 이렇게 되면 유지보수가 필요한 핵심 논리와 오류를 구별하는 것이 더 어려워진다.
  - 예외 처리는 캡슐화를 악화시키기 때문에 신중하게 사용해야한다.
    - 함수에 예외가 많을수록 호출자는 함수에 대해 더 많은 것을 알아야만 한다.
    - 그리고 함수가 너무 많은 예외를 발생시킨다는 것은 문맥에서 자유롭지 않다는 것을 의미한다.
    - 호출을 할 때마다 발생 가능한 부작용을 염두에 두고 문맥을 유지해야 하기 때문이다.
    - 이는 함수가 응집력이 약하고 너무 많은 책임을 가지고 있다는 사실을 의미할 수도 있다.
    - 만약 함수에서 너무 많은 예외를 발생시켜야 한다면 여러 개의 작은 기능으로 나눌 수 있는지 검토해봐야한다.
  - 엔드 유저에게 traceback을 노출해선 안된다.
    - 이는 보안을 위한 고려사항이다.
    - 특정 문제를 나타내는 예외가 발생한 경우 문제를 효율적으로 해결할 수 있도록 traceback 정보, 메시지 및 기타 수집 가능한 정보를 최대한 로그로 남기는 것이 중요하다.
    - 그러나 이러한 세부 사항은 절대 사용자에게 보여서는 안 된다.
    - Python의 traceback은 매우 풍부하고 유용한 디버깅 정보를 포함하고 있으나, 이 정보는 악의적인 사용자에게도 매우 유용한 정보이다.
    - 사용자에게 문제를 알리려면 "알 수 없는 문제가 발생했습니다" 등의 일반적인 메시지를 사용해야한다.
  - 원본 오류 포함시키기
    - 오류 처리 과정에서 기존 오류와 다른 새로운 오류를 발생시키고 오류 메시지를 변경할 수도 있다.
      - 이런 경우 원래 어떤 오류가 있었는지에 대한 정보를 포함하는 것이 좋다.
      - PEP-3134에서 소개된 `raise <e> from <original_exception>` 구문을 사용하면 여러 예외를 연결할 수 있다.
      - 이렇게 하면 원본 오류의 traceback 정보가 새로운 exception에 포함되고, 원본 오류는 새로운 오류의 원인으로 분류되어 `__cause__` 속성에 할당된다.

  ```py
  def process(data: dict, record_id: str):
      try:
          return data[record_id]
      except KeyError as e:
          raise InternalDataError("Data does not exists") from e

