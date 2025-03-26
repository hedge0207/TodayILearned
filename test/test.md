# Test

- Test Coverage

  > https://blog.miguelgrinberg.com/post/how-to-write-unit-tests-in-python-part-1-fizz-buzz

  - Fizz Buzz application 작성하기
    - 입력값이 3으로 나누어 떨어지면 "Fizz", 5로 나누어 떨어지면 배수면 "Buzz", 3과 5 모두로 나누어 떨어지면 "FizzBuzz"를 반환하는 간단한 application이다.

  ```python
  # fizzbuzz.py
  
  def fizzbuzz(i):
      if i % 15 == 0:
          return "FizzBuzz"
      elif i % 3 == 0:
          return "Fizz"
      elif i % 5 == 0:
          return "Buzz"
      else:
          return i
  
  
  def main():
      for i in range(1, 101):
          print(fizzbuzz(i))
  
  
  if __name__ == '__main__':
      main()
  ```

  - `unittest` package를 사용하여 test code를 작성한다.

  ```python
  # test_fizzbuzz.py
  
  import unittest
  from fizzbuzz import fizzbuzz
  
  
  class TestFizzBuzz(unittest.TestCase):
      def test_fizz(self):
          for i in [3, 6, 9, 18]:
              print('testing', i)
              assert fizzbuzz(i) == 'Fizz'
  
      def test_buzz(self):
          for i in [5, 10, 50]:
              print('testing', i)
              assert fizzbuzz(i) == 'Buzz'
  
      def test_fizzbuzz(self):
          for i in [15, 30, 75]:
              print('testing', i)
              assert fizzbuzz(i) == 'FizzBuzz'
  ```

  - `pytest`를 사용하여 test를 실행한다.
    - `pytest` command는 `test`가 prefix나 suffix로 들어간 `.py` file들을 찾아 테스트를 실행한다.

  ```bash
  $ pytest
  ```

  - Code coverage
    - 위 예시에서는 3개의 test case를 가지고 test를 실행했는데, 이 test case들이 test 대상 code의 모든 부분을 충분히 test 하고 있는지는 알 수 없다.
    - Code coverage는 interpreter에 의해 실행되는 code들을 추적하여 어떤 line이 실행되고 어떤 line이 실행되지 않는지를 확인하는 기술이다.
    - Code coverage를 사용하면 실행되지 않은 line을 확인하여 test case가 test 대상 code의 모든 부분을 test하고 있는지 확인할 수 있다.
  - `pytest-cov` 설치
    - `pytest`의 plugin인 [pytest-cov](https://pytest-cov.readthedocs.io/en/latest/)를 사용하여 code coverage를 확인할 수 있다.

  ```bash
  $ pip install pytest-cov
  ```

  - Code coverage 확인하기
    - `--cov` option에 code coverage를 확인하고자 하는 module 혹은 package의 이름을 입력하면 된다.

  ```bash
  $ pytest --cov=fizzbug
  
  # output
  Name          Stmts   Miss  Cover
  ---------------------------------
  fizzbuzz.py      13      4    69%
  ---------------------------------
  TOTAL            13      4    69%
  ```

  - 출력 방식 변경하기
    - `pytest-cov`는 다양한 형식으로 report를 작성해주는데, 위에서 출력해준 report는 `term`이라고 불리는 방식으로, terminal에 간략히 요약된 결과를 출력해주는 방식이다.
    - 출력 방식을 변경하고 싶으면 `--cov-report` option으로 다른 값을 설정하면 된다.
    - `term-missing`은 실행되지 않은 line들을 추가적으로 terminal에 출력해준다.

  ```bash
  $ pytest --cov=fizzbuzz --cov-report=term-missing
  
  # output
  Name          Stmts   Miss  Cover   Missing
  -------------------------------------------
  fizzbuzz.py      13      4    69%   9, 13-14, 18
  -------------------------------------------
  TOTAL            13      4    69%
  ```

  - Code coverage 분석하기
    - 13-14번째 line은 `fizzubuzz.py`를 직접 실행했을 때 실행될 `main`함수이므로 test의 범위가 아니다.
    - 또한 18번째 line 역시 `fizzbuzz.py`를 직접 실행했을 때만 실행되므로 test의 범위가 아니다.
    - 그러나 9번째 line은 3이나 5로 나누어 떨어지지 않으면 실행되어야 하는데, 실행되지 않으므로, 이 부분에 대한 test를 추가해야 한다.
  - 분기가 있는 경우의 code coverage
    - `fizzbuzz.py`에는 2, 4, 6, 17번째 line에 `if`이 있다.
    - 이 `if`문이 True로 평가될 때와 False로 평가될 때 모두에 대한 code coverage도 확인을 해야 한다.
    - `pytest-cov`는 `--cov-branch` option을 통해 모든 분기에 대한 code coverage를 볼 수 있는 기능을 제공한다.
    - `--cov-branch`를 주지 않았을 때에 비해 coverage가 하락하여 65%가 된 것을 확인할 수 있다.

  ```bash
  $ pytest --cov=fizzbuzz --cov-report=term-missing --cov-branch
  
  # output
  ---------- coverage: platform win32, python 3.12.0-final-0 -----------
  Name          Stmts   Miss Branch BrPart  Cover   Missing
  ---------------------------------------------------------
  fizzbuzz.py      13      4     10      2    65%   9, 13-14, 18
  ---------------------------------------------------------
  TOTAL            13      4     10      2    65%
  ```

  - Code coverage 높이기
    - 3이나 5로 나누어 떨어지지 않는 경우도 test하기 위해서 아래와 같이 테스를 수정한다.

  ```python
  import unittest
  from fizzbuzz import fizzbuzz
  
  
  class TestFizzBuzz(unittest.TestCase):
      def test_fizz(self):
          for i in [3, 6, 9, 18]:
              print('testing', i)
              assert fizzbuzz(i) == 'Fizz'
  
      def test_buzz(self):
          for i in [5, 10, 50]:
              print('testing', i)
              assert fizzbuzz(i) == 'Buzz'
  
      def test_fizzbuzz(self):
          for i in [15, 30, 75]:
              print('testing', i)
              assert fizzbuzz(i) == 'FizzBuzz'
  
      def test_number(self):
          for i in [2, 4, 88]:
              print('testing', i)
              assert fizzbuzz(i) == i
  ```

  - 다시 code coverage를 확인해본다.
    - Coverage가 기존보다 증가한 것을 확인할 수 있다.
    - 또한 `fizzbuzz()` 함수에 더 이상 실행되지 않는 line이 없으므로 모든 부분을 test한다고 볼 수 있다.

  ```bash
  $ pytest --cov=fizzbuzz --cov-report=term-missing --cov-branch
  
  # output
  Name          Stmts   Miss Branch BrPart  Cover   Missing
  ---------------------------------------------------------
  fizzbuzz.py      13      3     10      1    74%   13-14, 18
  ---------------------------------------------------------
  TOTAL            13      3     10      1    74%
  ```

  - Code coverage에서 제외할 부분 설정하기
    - 위 test code는 검증하고자 하는 함수 `fizzbuzz()`를 모두 cover하지만, `pytest-cov`의 report에서는 coverage가 낮게 나오는 것이 신경쓰인다면, code coverage에서 특정 부분을 제외시킬 수 있다.
    - 아래와 같이 제외시키고자 하는 부분에 `pragma: no cover`를 주석으로 남기면 된다.
    - `pragma: no cover`가 주석으로 남겨진 모든 code block을 coverage 계산에서 제외시킨다.

  ```python
  def fizzbuzz(i):
      if i % 15 == 0:
          return "FizzBuzz"
      elif i % 3 == 0:
          return "Fizz"
      elif i % 5 == 0:
          return "Buzz"
      else:
          return i
  
  
  def main():	# pragma: no cover
      for i in range(1, 101):
          print(fizzbuzz(i))
  
  
  if __name__ == '__main__':  # pragma: no cover
      main()
  ```

  - Code coverage 다시 확인하기
    - Coverage가 100%가 된 것을 확인할 수 있다.

  ```bash
  $ pytest --cov=fizzbuzz --cov-report=term-missing --cov-branch
  
  # output
  Name          Stmts   Miss Branch BrPart  Cover   Missing
  ---------------------------------------------------------
  fizzbuzz.py       8      0      6      0   100%
  ---------------------------------------------------------
  TOTAL             8      0      6      0   100%
  ```



