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



- Conway's Game of Life

  > https://blog.miguelgrinberg.com/post/how-to-write-unit-tests-in-python-part-2-game-of-life

  - Conway's Game of Life 개요
    - Game of Life는 2차원 격자에서 실행된다.
    - 격자 내부의 각 cell은 살아 있거나 죽어 있는 두 가지 상태 중 하나의 상태를 가진다.
    - 각 time step마다 모든 cell의 상태가 변경되며, 상태 변경에는 cell의 현재 상태와, 8개의 이웃 cell이 영향을 미친다.
  - Game of Life의 규칙
    - 생존 규칙: 만약 cell이 alive 상태이고, 2개 또는 3개의 이웃 cell이 alive상태이면, 해당 cell은 alive 상태를 유지하고, 아니면 죽는다.

    - 탄생 규칙: 만약 cell이 dead 상태이고, 3개의 이웃 cell이 alive 상태이면, 해당 cell은 alive 상태가 되고, 아니면 dead 상태를 유지한다.

  - 구현하기
    - Cell들의 정보를 담고 있는 `CellList` class와 simulation logic을 구현할 `Life` class를 구현한다.
    - Game of Life와 같은 cellular automata를 저장하기 위한 format인 Life 1.05와 Life 1.06을 모두 지원하는 code이다.
    - 화면으로 출력하는 부분 없이, engine 부분만 구현한 것이다.
    - `Life.advance()` method를 통해 cell들이 갱신된다.

  ```python
  class CellList:
      """Maintain a list of (x, y) cells."""
  
      def __init__(self):
          self.cells = {}
  
      def has(self, x, y):
          """Check if a cell exists in this list."""
          return x in self.cells.get(y, [])
  
      def set(self, x, y, value=None):
          """Add, remove or toggle a cell in this list."""
          if value is None:
              value = not self.has(x, y)
          if value:
              row = self.cells.setdefault(y, set())
              if x not in row:
                  row.add(x)
          else:
              try:
                  self.cells[y].remove(x)
              except KeyError:
                  pass
              else:
                  if not self.cells[y]:
                      del self.cells[y]
  
      def __iter__(self):
          """Iterator over the cells in this list."""
          for y in self.cells:
              for x in self.cells[y]:
                  yield (x, y)
  
  
  class Life:
      """Game of Life simulation."""
  
      def __init__(self, survival=[2, 3], birth=[3]):
          self.survival = survival
          self.birth = birth
          self.alive = CellList()
  
      def rules_str(self):
          """Return the rules of the game as a printable string."""
          survival_rule = "".join([str(n) for n in self.survival])
          birth_rule = "".join([str(n) for n in self.birth])
          return f'{survival_rule}/{birth_rule}'
  
      def load(self, filename):
          """Load a pattern from a file into the game grid."""
          with open(filename, "rt") as f:
              header = f.readline()
              if header == '#Life 1.05\n':
                  x = y = 0
                  for line in f.readlines():
                      if line.startswith('#D'):
                          continue
                      elif line.startswith('#N'):
                          self.survival = [2, 3]
                          self.birth = [3]
                      elif line.startswith('#R'):
                          self.survival, self.birth = [
                              [int(n) for n in i]
                              for i in line[2:].strip().split('/', 1)]
                      elif line.startswith('#P'):
                          x, y = [int(i) for i in line[2:].strip().split(' ', 1)]
                      else:
                          i = line.find('*')
                          while i != -1:
                              self.alive.set(x + i, y, True)
                              i = line.find('*', i + 1)
                          y += 1
              elif header == '#Life 1.06\n':
                  for line in f.readlines():
                      if not line.startswith('#'):
                          x, y = [int(i) for i in line.strip().split(' ', 1)]
                          self.alive.set(x, y, True)
              else:
                  raise RuntimeError('Unknown file format')
  
      def toggle(self, x, y):
          """Toggle a cell in the grid."""
          self.alive.set(x, y)
  
      def living_cells(self):
          """Iterate over the living cells."""
          return self.alive.__iter__()
  
      def bounding_box(self):
          """Return the bounding box that includes all living cells."""
          minx = miny = maxx = maxy = None
          for cell in self.living_cells():
              x = cell[0]
              y = cell[1]
              if minx is None or x < minx:
                  minx = x
              if miny is None or y < miny:
                  miny = y
              if maxx is None or x > maxx:
                  maxx = x
              if maxy is None or y > maxy:
                  maxy = y
          return (minx or 0, miny or 0, maxx or 0, maxy or 0)
  
      def advance(self):
          """Advance the simulation by one time unit."""
          processed = CellList()
          new_alive = CellList()
          for cell in self.living_cells():
              x = cell[0]
              y = cell[1]
              for i in range(-1, 2):
                  for j in range(-1, 2):
                      if (x + i, y + j) in processed:
                          continue
                      processed.set(x + i, y + j, True)
                      if self._advance_cell(x + i, y + j):
                          new_alive.set(x + i, y + j, True)
          self.alive = new_alive
  
      def _advance_cell(self, x, y):
          """Calculate the new state of a cell."""
          neighbors = 0
          for i in range(-1, 2):
              for j in range(-1, 2):
                  if i != 0 or j != 0:
                      neighbors += 1 if self.alive.has(x + i, y + j) else 0
  
          if self.alive.has(x, y):
              return neighbors in self.survival
          else:
              return neighbors in self.birth
  ```

  - GUI 작성하기
    - `pygame` pacakge를 사용한다.
    - GUI는 test 대상이 아니다.

  ```python
  # life_gui.py
  import sys
  import pygame
  from life import Life
  
  SCREEN_SIZE = 500
  FPS = 5
  
  life = Life()
  
  
  def initialize_game(pattern_file=None):
      pygame.init()
      screen = pygame.display.set_mode([SCREEN_SIZE, SCREEN_SIZE])
  
      if pattern_file:
          life.load(pattern_file)
  
      pygame.display.set_caption(f'Game of Life [{life.rules_str()}]')
      return screen
  
  
  def center(scale):
      cell_count = SCREEN_SIZE // scale
      minx, miny, maxx, maxy = life.bounding_box()
  
      basex = minx - (cell_count - (maxx - minx + 1)) // 2
      basey = miny - (cell_count - (maxy - miny + 1)) // 2
      return basex, basey
  
  
  def game_loop(screen):
      running = True
      paused = False
      scale = 20
      basex, basey = center(scale)
      interval = 1000 // FPS
  
      while running:
          start_time = pygame.time.get_ticks()
  
          screen.fill((255, 255, 255))
          for i in range(0, SCREEN_SIZE, scale):
              pygame.draw.line(screen, (0, 0, 0), (i, 0), (i, SCREEN_SIZE))
              pygame.draw.line(screen, (0, 0, 0), (0, i), (SCREEN_SIZE, i))
          for cell in life.alive:
              x = cell[0]
              y = cell[1]
              pygame.draw.rect(screen, (80, 80, 192),
                               ((x - basex) * scale + 2, (y - basey) * scale + 2,
                                scale - 3, scale - 3))
  
          pygame.display.flip()
          if not paused:
              life.advance()
  
          wait_time = 1
          while wait_time > 0:
              event = pygame.event.wait(timeout=wait_time)
              while event:
                  if event.type == pygame.QUIT:
                      running = False
                      break
                  elif event.type == pygame.KEYDOWN:
                      if event.key == pygame.K_ESCAPE:
                          running = False
                      elif event.key == pygame.K_LEFT:
                          basex -= 2
                      elif event.key == pygame.K_RIGHT:
                          basex += 2
                      elif event.key == pygame.K_UP:
                          basey -= 2
                      elif event.key == pygame.K_DOWN:
                          basey += 2
                      elif event.unicode == ' ':
                          paused = not paused
                          if paused:
                              pygame.display.set_caption('Game of Life (paused)')
                          else:
                              pygame.display.set_caption(
                                  f'Game of Life [{life.rules_str()}]')
                      elif event.unicode == '+':
                          if scale < 50:
                              scale += 5
                      elif event.unicode == '-':
                          if scale > 10:
                              scale -= 5
                      elif event.unicode == 'c':
                          basex, basey = center(scale)
                      break
                  elif event.type == pygame.MOUSEBUTTONUP:
                      mx, my = pygame.mouse.get_pos()
                      x = mx // scale + basex
                      y = my // scale + basey
                      life.toggle(x, y)
                      break
                  event = pygame.event.poll()
              if event:
                  break
  
              current_time = pygame.time.get_ticks()
              wait_time = interval - (current_time - start_time)
  
  
  if __name__ == '__main__':
      pattern_file = sys.argv[1] if len(sys.argv) > 1 else None
      screen = initialize_game(pattern_file)
      print('''
  Press: Arrows to scroll
         Space to pause/resume the simulation
         +/- to zoom in/out
         c to re-center
         mouse click to toggle the state of a cell
         Esc to exit''')
      game_loop(screen)
      pygame.quit()
  ```

  - `pygame` package 설치

  ```bash
  $ pip install pygame
  ```

  - Pattern 파일 작성

  ```
  #Life 1.05
  #N
  .***.
  .....
  *...*
  *...*
  .....
  .***.
  .....
  .....
  .***.
  .....
  *...*
  *...*
  .....
  .***.
  ```

  - 실행
    - Pattern file을 입력하지 않아도 실행 가능하다.

  ```bash
  $ python life_gui.py [pattern_file]
  ```



- `CellList` class 테스트하기

  - `CellList.set()` method의 `value` parameter가 True일 경우 새로운 cell을 추가하고, False일 경우 cell을 제거하며, None일 경우 toggle을 의미한다.

  ```python
  import unittest
  from life import CellList, Life
  
  
  class TestCellList(unittest.TestCase):
      def test_empty(self):
          c = CellList()
          assert list(c) == []
  
      def test_set_true(self):
          c = CellList()
          c.set(1, 2, True)
          assert c.has(1, 2)
          assert list(c) == [(1, 2)]
          c.set(500, 600, True)
          assert c.has(1, 2) and c.has(500, 600)
          assert list(c) == [(1, 2), (500, 600)]
          c.set(1, 2, True)  # make sure a cell can be set to True twice
          assert c.has(1, 2) and c.has(500, 600)
          assert list(c) == [(1, 2), (500, 600)]
          
      def test_set_false(self):
          c = CellList()
          c.set(1, 2, False)
          assert not c.has(1, 2)
          assert list(c) == []
          c.set(1, 2, True)
          c.set(1, 2, False)
          assert not c.has(1, 2)
          assert list(c) == []
          c.set(1, 2, True)
          c.set(3, 2, True)
          c.set(1, 2, False)
          assert not c.has(1, 2)
          assert c.has(3, 2)
          assert list(c) == [(3, 2)]
      
      def test_set_default(self):
          c = CellList()
          c.set(1, 2)
          assert c.has(1, 2)
          assert list(c) == [(1, 2)]
          c.set(1, 2)
          assert not c.has(1, 2)
          assert list(c) == []
  ```



- `Life` class 테스트하기

  - `Life` class의 생성자와 `rules_str` method를 test하기 위한 코드를 작성한다.

  ```python
  import unittest
  from life import CellList, Life
  
  
  class TestLife(unittest.TestCase):
      def test_new(self):
          life = Life()
          assert life.survival == [2, 3]
          assert life.birth == [3]
          assert list(life.living_cells()) == []
  	
      def test_new_custom(self):
          life = Life([3, 4], [4, 7, 8])
          assert life.survival == [3, 4]
          assert life.birth == [4, 7, 8]
          assert list(life.living_cells()) == []
  ```

  - `Life` class의 `load` method를 test하기 위한 txt 파일을 작성한다.

  ```
  pattern1.txt
  
  #Life 1.05
  #D test pattern 1
  #N
  #P 10 10
  *.
  .*
  #P 15 10
  *.*
  
  
  pattern2.txt
  
  #Life 1.06
  # test pattern 2
  10 10
  11 11
  15 10
  17 10
  ```

  - `Life` class의 `load`와 `bounding_box` method를 test하기 위한 코드를 작성한다.
    - 아래 두 코드는 `load` method의 argument로 넘기는 file명을 제외하고 전부 동일하다.

  ```python
  class TestLife(unittest.TestCase):
      # ...
  
      def test_load_life_1_05(self):
          life = Life()
          life.load('pattern1.txt')
          assert set(life.living_cells()) == {
              (10, 10), (11, 11), (15, 10), (17, 10)}
          assert life.bounding_box() == (10, 10, 17, 11)
  
      def test_load_life_1_06(self):
          life = Life()
          life.load('pattern2.txt')
          assert set(life.living_cells()) == {
              (10, 10), (11, 11), (15, 10), (17, 10)}
          assert life.bounding_box() == (10, 10, 17, 11)
  ```

  - Parameter 활용하기
    - 위에서 `load` method를 test하는 두 개의 unit test는 argument로 넘기는 file명을 제외하고 전부 동일하다.
    - 이를 parameterization하면 중복 코드를 줄일 수 있다.
    - `pytest` 기반의 unit test에서는 `@pytest.mark.parametrize` decorator를 사용하면 되지만, `unittest` 기반의 unit test에서는 별도의 package를 설치해야한다.

  ```bash
  $ pip install parameterized
  ```

  - `parameterized`를 활용하도록 test 코드 수정하기
    - `parameterized.expand`를 사용한다.

  ```python
  from parameterized import parameterized
  
  class TestLife(unittest.TestCase):
      @parameterized.expand([('pattern1.txt'), ('pattern2.txt')])
      def test_load(self, pattern_file):
          life = Life()
          life.load(pattern_file)
          assert set(life.living_cells()) == {
              (10, 10), (11, 11), (15, 10), (17, 10)}
          assert life.bounding_box() == (10, 10, 17, 11)
  ```

  - Coverage 확인

  ```bash
  $ pytest --cov=life --cov-report=term-missing --cov-branch
  
  # output
  ---------- coverage: platform win32, python 3.12.0-final-0 -----------
  Name      Stmts   Miss Branch BrPart  Cover   Missing
  -----------------------------------------------------
  life.py      98     24     60      2    72%   62, 79, 83, 107-119, 123-132
  -----------------------------------------------------
  TOTAL        98     24     60      2    72%
  ```

  - Coverage를 높이기 위해 새로운 pattern file을 추가한다.
    - `pattern1.txt`와 `pattern2.txt`에는 `#R`을 사용하는 부분이 없어 62번째 줄이 실행되지 않는다.
    - 또한, 모두 유효한 foramt으로 작성되어 79번째 줄도 실행되지 않는다.
    - 따라서 두 경우를 모두 포함하기 위한 새로운 pattern file을 작성한다.

  ```
  pattern3.txt
  
  #Life 1.05
  #D test pattern 3
  #R 34/45
  #P 10 10
  *.
  .*
  *.
  
  
  pattern4.txt
  
  this is not a life file
  ```

  - 위 두 경우를 test하기 위한 코드를 추가한다.
    - `assert`로는 error가 raise되는 경우를 검사할 수 없다.
    - `pytest.raises`를 사용하여 의도한 대로 error가 raise되는지 검사할 수 있다.

  ```python
  import pytest
  
  class TestLife(unittest.TestCase):
      # ...
      
      def test_load_life_custom_rules(self):
          life = Life()
          life.load('pattern3.txt')
          assert life.survival == [3, 4]
          assert life.birth == [4, 5]
          assert list(life.living_cells()) == [(10, 10), (11, 11), (10, 12)]
          assert life.bounding_box() == (10, 10, 11, 12)
          
      def test_load_invalid(self):
          life = Life()
          with pytest.raises(RuntimeError):
              life.load('pattern4.txt')
  ```

  - `Life.toggle` method를 test하기 위한 코드를 작성한다.
    - 기존에 test가 cover하지 못 했던 83번째 줄을 cover한다.

  ```python
  class TestLife(unittest.TestCase):
      # ...
  
      def test_toggle(self):
          life = Life()
          life.toggle(5, 5)
          assert list(life.living_cells()) == [(5, 5)]
          life.toggle(5, 6)
          life.toggle(5, 5)
          assert list(life.living_cells()) == [(5, 6)]
  ```

  - 여기까지 작성 후 다시 coverage를 확인한다.
    - Coverage가 상승한 것을 확인할 수 있다.

  ```bash
  $ pytest --cov=life --cov-report=term-missing --cov-branch
  
  # output
  ---------- coverage: platform win32, python 3.12.0-final-0 -----------
  Name      Stmts   Miss Branch BrPart  Cover   Missing
  -----------------------------------------------------
  life.py      98     21     60      0    75%   107-119, 123-132
  -----------------------------------------------------
  TOTAL        98     21     60      0    75%
  ```



- Code coverage의 맹점

  - `Life` class의 핵심 method는 `advance`와 `_advance_cell` method이다.
    - 이 두 method가 위에서 아직 테스트 범위에 포함되지 않은 107-119, 123-132 line에 해당한다.
    - `_advance_cell` method는 Game of Life의 핵심 logic을 구현한 method로, cell의 x와 y 좌표를 받아 다음 time step에 cell의 생사 여부를 boolean 값으로 반환한다.
    - `_advance_cell` method는 살아있는 이웃 cell들의 개수를 계산하여 현재 cell의 다음 상태를 결정한다.
    - `advance` method는 모든 cell들을 순회하면서 `_advance_cell` method를 호출하여 각 cell의 상태를 갱신한다.
  - Code coverage의 맹점
    - `_advance_cell` method가 특정 cell이 alive 상태이고, 그 이웃 cell 중 3개가 alive 상태면 True를 반환하는지를 test한다.
    - Cell (0, 0)과 그 이웃 cell들을 가지고 test를 진행하며, `random`을 통해 매 test마다 이웃 cell을 무선적으로 선택한다.
    - 그 후 code coverage를 확인해보면, `_advance_cell` method 중 132번째 line을 제외한 모든 line이 실행되어, code coverage가 85%까지 상승한 것을 확인할 수 있다.
    - 그러나, 이 code coverage의 상승에는 맹점이 있다.
    - 실행 되지 않은 132번째 line은 탄생 규칙과 관련된 line으로, 아래 예시에서는 이미 생존 상태인 cell로 test를 진행했으므로, 실행되지 않는다.
    - 또한 생존 규칙으로만 한정하여 생각하더라도, 생존 규칙은 총 9가지 경우의 수가 있는데(이웃 cell 중 생존한 cell의 개수가 1~9인 경우), 아래 예시는 생존한 cell의 수가 3개일 때만을 test한다.
    - 또한, `Life` class를 생성할 때 생존 규칙을 custom하게 설정할 수 있는데(기본 값은 이웃 cell 중 2개 또는 3개가 alive 상태면 생존), 아래 예시는 custom하게 설정된 경우를 test하지 못 한다.
    - 이처럼 code coverage가 수치상으로 상승했다고 해서 test의 품질이 향상됐다고 볼 수는 없다.

  ```python
  import random
  
  # ...
  
  class TestLife(unittest.TestCase):
      # ...
      
      def test_advance_cell(self):
          life = Life()
          life.toggle(0, 0)
          # cell (0, 0)을 기준으로 한 이웃 cell들
          neighbors = [(-1, -1), (0, -1), (1, -1),
                       (-1, 0), (1, 0),
                       (-1, 1), (0, 1), (1, 1)]
          for _ in range(3):
              n = random.choice(neighbors)
              neighbors.remove(n)
              life.toggle(*n)
  
          assert life._advance_cell(0, 0)
  ```

  - 모든 경우의 이웃 cell의 개수에 대해 test하도록 변경한다.
    - 이전 test에 비해서 수치상의 code coverage는 상승하지 않았지만, test의 품질은 향상됐다.

  ```python
  import random
  
  # ...
  
  class TestLife(unittest.TestCase):
      # ...
      
      @parameterized.expand([(n,) for n in range(9)])
      def test_advance_cell(self, num_neighbors):
          life = Life()
          life.toggle(0, 0)
          neighbors = [(-1, -1), (0, -1), (1, -1),
                       (-1, 0), (1, 0),
                       (-1, 1), (0, 1), (1, 1)]
          
          for _ in range(num_neighbors):
              n = random.choice(neighbors)
              neighbors.remove(n)
              life.toggle(*n)
  		
          new_state = life._advance_cell(0, 0)
          if num_neighbors in [2, 3]:
              assert new_state is True
          else:
              assert new_state is False
  ```

  - 이제 생존 규칙뿐만 아니라 탄생 규칙도 test하도록 변경한다.
    - Test 대상 cell이 생존 상태일 때의 모든 생존한 이웃 cell의 개수와, test 대상 cell이 사망 상태일 때의 모든 생존한 이웃 cell의 개수에 대해 test하도록 변경한다(즉 2*9=18개의 경우에 대해 test를 실행한다).
    - 아래와 같이 변경함으로써 탄생 규칙도 test하게 되어 모든 `_advance_cell()` method의 모든 부분이 실행되게 된다.

  ```python
  import itertools
  
  # ...
  
  class TestLife(unittest.TestCase):
      # ...
  
      @parameterized.expand(itertools.product([True, False], range(9)))
      def test_advance_cell(self, alive, num_neighbors):
          life = Life()
          if alive:
              life.toggle(0, 0)
          neighbors = [(-1, -1), (0, -1), (1, -1),
                       (-1, 0), (1, 0),
                       (-1, 1), (0, 1), (1, 1)]
          
          for _ in range(num_neighbors):
              n = random.choice(neighbors)
              neighbors.remove(n)
              life.toggle(*n)
  
          new_state = life._advance_cell(0, 0)
          if alive:
              # survival rule
              if num_neighbors in [2, 3]:
                  assert new_state is True
              else:
                  assert new_state is False
          else:
              # birth rule
              if num_neighbors in [3]:
                  assert new_state is True
              else:
                  assert new_state is False
  ```

  - 마지막으로 보다 다양한 생존 규칙과 탄생 규칙에 대해서도 test할 수 있게 변경한다.
    - 위 예시에서는 생존 규칙은 생존한 이웃의 개수가 2개 또는 3개일 경우, 탄생 규칙은 생존한 이웃의 개수가 3개일 때만을 대상으로 test했다.
    - 아래와 같이 다양한 생존, 탄생 규칙에 대해 test할 수 있도록 변경한다.

  ```python
  class TestLife(unittest.TestCase):
      # ...
  
      @parameterized.expand(itertools.product(
          [[2, 3], [4]],  # two different survival rules
          [[3], [3, 4]],  # two different birth rules
          [True, False],  # two possible states for the cell
          range(0, 9),    # nine number of possible neighbors
      ))
      def test_advance_cell(self, survival, birth, alive, num_neighbors):
          life = Life(survival, birth)
          if alive:
              life.toggle(0, 0)
          neighbors = [(-1, -1), (0, -1), (1, -1),
                       (-1, 0), (1, 0),
                       (-1, 1), (0, 1), (1, 1)]
          for _ in range(num_neighbors):
              n = random.choice(neighbors)
              neighbors.remove(n)
              life.toggle(*n)
  
          new_state = life._advance_cell(0, 0)
          if alive:
              # survival rule
              if num_neighbors in survival:
                  assert new_state is True
              else:
                  assert new_state is False
          else:
              # birth rule
              if num_neighbors in birth:
                  assert new_state is True
              else:
                  assert new_state is False
  ```

  - Test 결과는 아래와 같다.
    - Code coverage는 크게 달라지지 않았으나, 실제로는 더 많은 경우를 test하게 되었다.

  ```bash
  $ pytest --cov=life --cov-report=term-missing --cov-branch
  
  # output
  Name      Stmts   Miss Branch BrPart  Cover   Missing
  -----------------------------------------------------
  life.py      98     13     60      0    85%   107-119
  -----------------------------------------------------
  TOTAL        98     13     60      0    85%
  ```



- Mocking

  - 마지막으로 남은 `advance()` method를 test할 것이다.
    - `advance()` method는 `_advance_cell()` method를 호출한다.
    - 이 둘의 logic을 분리하는 것은 쉽지 않다.
  - Mocking의 필요성
    - `advance()`는 `_advance_cell()`에 의존하므로 아래와 같은 test가 필요하다. 
    - 먼저 `_advance_cell()`의 반환 결과에 따라 `advance()` method가 적절히 실행되는지 test해야한다.
    - 또한 `advance()` method가 `_advance_cell()` method를 적절히 호출하는지(하나의 cell에 대해 `_advance_cell()` method를 여러번 호출하지는 않는지, 상태가 변경될 가능성이 없는 cell에 대해 호출하지는 않는지)도 test해야한다.
    - 그런데, `_advance_cell()` method는 이미 철저하게 test가 끝났으므로, 추가적인 test를 할 필요는 없다.
    - 따라서 `_advance_cell()`을 대체할 mock을 사용하여 test하면 보다 편리하게 test가 가능하다.
  - `unittest.mock` 사용하기
    - 아래와 같이 `MagicMock`의 instance에 `return_value`를 지정하면, 어떤 argument로 해당 instance를 호출하던 `return_value`를 반환하게 된다.
    - 또한 `MagucMock` instance는 호출 횟수와 호출시에 사용된 argument 정보를 계속 가지고 있는다.

  ```python
  from unittest import mock
  
  fake = mock.MagicMock()
  fake.return_value = 42
  print(fake())					# 42
  print(fake("Hello"))			# 42
  print(fake("World", num=1))		# 42
  
  # 호출 횟수
  print(fake.call_count)			# 3
  # 호출 argument
  print(fake.call_args_list)		# [call(), call('Hello'), call('World', num=1)]
  ```

  - `_advance_cell`이 False를 반환하는 경우에 대해 test하기
    - 아래와 같이 `mock.patch.object()` decorator를 통해 `MagicMock` instance를 주입할 수 있으며, 아래 예시는 `Life._advance_cell()` method를 patch하겠다는 것을 의미한다.
    - 이를 통해 `_advance_cell()` method가 호출되더라도 실제 `Life._advance_cell()` 이 실행되는 것이 아닌, mock object가 실행된다.

  ```python
  from unittest import mock
  
  
  class TestLife(unittest.TestCase):
      # ...
      
      @mock.patch.object(Life, '_advance_cell')
      def test_advance_false(self, mock_advance_cell):
          mock_advance_cell.return_value = False
          life = Life()
          life.toggle(10, 10)
          life.toggle(12, 10)
          life.toggle(20, 20)
          life.advance()
  
          # _advance_cell method를 정확히 24번 호출돼야 한다.
          # (10, 10) cell일 때 9번
          # (12, 10) cell일 때 6번(3번은 (10, 10) 일 때 이미 호출되었으므로 호출할 필요 없다)
          # (20, 20) cell일 때 9번
          assert mock_advance_cell.call_count == 24
          assert list(life.living_cells()) == []
  ```

  - `_advance_cell`이 True를 반환하는 경우에 대해 test하기
    - Check 대상인 모든 cell이 alive 상태가 되므로, 최종적으로 21개의 cell이 alive상태가 된다.

  ```python
  class TestLife(unittest.TestCase):
      # ...
  
      @mock.patch.object(Life, '_advance_cell')
      def test_advance_true(self, mock_advance_cell):
          mock_advance_cell.return_value = True
          life = Life()
          life.toggle(10, 10)
          life.toggle(11, 10)
          life.toggle(20, 20)
          life.advance()
  
          # _advance_cell method를 정확히 21번 호출돼야 한다.
          # (10, 10) cell일 때 9번
          # (12, 10) cell일 때 3번(6번은 (10, 10) 일 때 이미 호출되었으므로 호출할 필요 없다)
          # (20, 20) cell일 때 9번
          assert mock_advance_cell.call_count == 21
  
          # 최종적으로 21개의 cell이 alive 상태가 된다.
          assert set(life.living_cells()) == {
              (9, 9), (10, 9), (11, 9), (12, 9),
              (9, 10), (10, 10), (11, 10), (12, 10),
              (9, 11), (10, 11), (11, 11), (12, 11),
              (19, 19), (20, 19), (21, 19),
              (19, 20), (20, 20), (21, 20),
              (19, 21), (20, 21), (21, 21),
          }
  ```

  - 테스트를 실행한다.

  ```bash
  $ pytest --cov=life --cov-report=term-missing --cov-branch
  
  # output
  Name      Stmts   Miss Branch BrPart  Cover   Missing
  -----------------------------------------------------
  life.py      98      0     70      0   100%
  -----------------------------------------------------
  TOTAL        98      0     70      0   100%
  ```



