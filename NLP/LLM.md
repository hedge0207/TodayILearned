# 개요

- DSPy

  - 프롬프트를 텍스트가 아닌 프로그램 형태로 작성할 수 있게 해주는 Python package.
    - 다만 핵심은 프롬프트를 프로그램 형태로 작성할 수 있다는 사실 그 자체가 아니다.
    - 프롬프트를 자동으로 최적화해준다는 것이 DSPy의 핵심 기능이다.
  - 설치하기

  ```bash
  $ pip install dspy
  ```

  - 실행해보기

  ```python
  import dspy
  
  
  lm = dspy.LM("ollama_chat/llama3.2:latest",
               api_base="http://localhost:11434",
               temperature=0.0)
  
  dspy.configure(lm=lm)
  
  
  messages = [
      {
          "role": "system",
          "content": "You are a helpful assistant"
      },
      {
          "role": "user",
          "content": "What is the capital of France?"
      }
  ]
  
  print(lm(messages = messages))		# ['The capital of France is Paris.']
  ```



- DSPy를 사용한 프로그램 생성하기

  - DSPy를 사용하여 시를 생성하는 프로그램을 만들 것이다.
    - 아래와 같이 단 몇 줄로 완성할 수 있다.

  ```python
  import dspy
  
  # DSPy가 사용할 LLM을 설정한다.
  lm = dspy.LM("ollama_chat/llama3.2:latest",
               api_base="http://localhost:11434",
               temperature=0.0)
  dspy.configure(lm=lm)
  
  poem_signature = "subject -> poem"
  poem_generator = dspy.Predict(poem_signature)
  result = poem_generator(subject="computer science")
  print(result.poem)
  ```

  - `Signature`
    - `Signature`를 사용하여 수행할 task를 정의한다.
    - `Signature`는 프로그래밍에서 함수의 입력과 출력을 정의하는 시그니처와 유사하다.
    - `Signature`를 정의하는 가장 간단한 방법은 "input -> output" 형태의 문자열을 지정하는 것이다.
    - 이 때, input과 output의 이름은 LLM에게 중요한 정보가 되기에 아무렇게나 설정해선 안 된다.
  - `Predict`
    - `Signature`를 호출 가능한 함수로 전환해주는 모듈이다.
    - `Signature`가 무엇을 할지를 정의한다면, `Predict`는 어떻게 할지를 정의한다.
    - `Predict`는 처리 흐름과 처리할 때 사용할 도구 등 call time의 전략을 정의한다.
  - 위 코드는 아래와 같은 과정을 거쳐 실행된다.
    - 문자열 "subject -> peom"은 `Signature` 클래스로 파싱되고, input과 output field는 기본적으로 str type이 된다.
    - `Signature` instance에 대한 기본적인 instruction string이 생성된다(예시의 경우에는 "Given the fields `subject`, produce the fields `poem`").
    - `Predict` module이 `Signature`를 받아 인스턴스화된다.
    - `poem_generator`는 이제 호출 가능한 함수가 된다.
  - `poem_generator(subject="computer science")`가 호출되면, 아래 과정이 실행된다.
    - `LM`이 설정되었는지 확인한다.
    - `Adapter` module이 `Signature`와 그 input을 rendering하여 `LM`이 사용할 수 있게 만든다(기본적으로 `ChatAdapter`가 수행하지만 JSON, XML 등 다양한 형식에 따라 다양한 `Adapter`가 수행하기도 한다).
    - `ChatAdapter`가 `Signatrue`의 instruction과, input과 output을 설명하는 field schema가 포함된 prompt를 생성하고, 포맷을 형성한다.
    - Message가 `LM`에게 전달되고, 같은 호출을 cache로부터 반환하기 위해 caching이 활성화된다.
    - `LM`으로부터 응답이 반환되고, `ChatAdapter`는 output field를 추출하기 위해 파싱을 실행한다.
    - `Predict` module은 `Prediction` object를 반환한다.
    - 이 호출은 `LM` history에 기록되며, `dspy.inspect_history()`를 통회 조회가 가능하다.
  - 아래와 같이 프롬프트를 확인 가능하다.

  ```python
  dspy.inspect_history(n=1)
  
  """
  [2026-07-01T17:49:39.921004]
  
  System message:
  
  Your input fields are:
  1. `subject` (str):
  Your output fields are:
  1. `poem` (str):
  All interactions will be structured in the following way, with the appropriate values filled in.
  
  Inputs will have the following structure:
  
  [[ ## subject ## ]]
  {subject}
  
  Outputs will be a JSON object with the following fields.
  
  {
    "poem": "{poem}"
  }
  In adhering to this structure, your objective is: 
          Given the fields `subject`, produce the fields `poem`.
  
  
  User message:
  
  [[ ## subject ## ]]
  computer science
  
  Respond with a JSON object in the following order of fields: `poem`.
  
  
  Response:
  
  {
    "poem": "Science of computers, a field so fine,\nA world of code, where logic shines.\nAlgorithms and data, a perfect blend,\nComputer science, where innovation never ends."}
  """
  ```



- Signature 확장하기

  - 추가적인 input 정의하기
    - location, mood라는 복수의 input을 사용한다.

  ```python
  poem_bot = dspy.Predict("location, mood -> poem")
  result = poem_bot(location="a quiet library", mood="mysterious")
  print(result.poem)
  
  """
  Silent halls where shadows play,
  A mysterious world, far away.
  In this quiet library, I roam,
  Where secrets hide, and tales are home.
  """
  ```

  - 추가적인 output 정의하기

  ```python
  poem_bot = dspy.Predict("location, mood -> poem, poem_title")
  result = poem_bot(location="a quiet library", mood="mysterious")
  print(result.poem_title)
  print("- - -")
  print(result.poem)
  
  """
  Whispers in the Stacks
  - - -
  Silent halls where shadows play,
  A mysterious world in disarray.
  The books stand tall, a sentinel row,
  Guarding secrets that few may know.
  """
  ```

  - Field에 type 지정하기

  ```py
  poem_bot = dspy.Predict("location, mood, contains_pun: bool -> poem")
  result = poem_bot(location="a quiet library", mood="mysterious", contains_pun=True)
  print(result.poem)
  
  """
  Silent halls where shadows play,
  A mysterious library's hush of day.
  Where books hold secrets, old and new,
  And whispers hide the truth from you.
  """
  ```



- Class 기반 signature

  - Class 기반 signature를 사용하면 뉘앙스를 전달할 수 있다.
    - class 내에 작성한 docstring은 prompt를 작성할 때 task instructions이 된다.
    - 또한 각 필드에 `desc` 파라미터로 description을 전달해 뉘앙스를 추가할 수 있다.
    - 필드명으로 충분한 맥락 정보를 줄 수 없을 것 같을 때, `desc`를 통해 추가적인 정보를 전달할 수 있다.
    - 그러나 이미 field를 통해 유추할 수 있는 정보를 추가적으로 주지는 않도록 해야한다.

  ```python
  import dspy
  
  
  class PoemBot(dspy.Signature):
      """
      Write a classical poem given the provided inputs.
      """
      location: str = dspy.InputField(desc="The setting of the poem")
      mood: str = dspy.InputField()
      poem: str = dspy.OutputField()
  ```

  - 호출 방식은 앞서 살펴본 string 기반 방식과 유사하다 

  ```python
  poem_bot = dspy.Predict(PoemBot)
  result = poem_bot(location="a quiet library", mood="mysterious")
  print(result.poem)
  ```

  - DSPy가 LM에 전달하는 메시지는 아래와 같다.
    - String 기반 방식과 달리 class 내에 작성한 docstring과 각 field의 description이 추가된 것을 확인할 수 있다.

  ```python
  dspy.inspect_history(n=1)
  
  """
  System message:
  
  Your input fields are:
  1. `location` (str): The setting of the poem
  2. `mood` (str):
  Your output fields are:
  1. `poem` (str):
  All interactions will be structured in the following way, with the appropriate values filled in.
  
  [[ ## location ## ]]
  {location}
  
  [[ ## mood ## ]]
  {mood}
  
  [[ ## poem ## ]]
  {poem}
  
  [[ ## completed ## ]]
  In adhering to this structure, your objective is: 
          Write a classical poem given the provided inputs.
  
  ...
  """
  ```

  - 값을 제한하기
    - `typinh.Literal`을 통해 특정 타입에 올 수 있는 값을 제한하는 것이 가능하다.

  ```python
  from typing import Literal
  
  
  Season = Literal[
      "spring", "summer", "autumn", "winter",
  ]
  
  class PoemBot(dspy.Signature):
      """
      Write a classical poem given the provided inputs.
      """
      location: str = dspy.InputField()
      mood: str = dspy.InputField()
      season: Season = dspy.InputField()
      poem: str = dspy.OutputField()
  
  poem_bot = dspy.Predict(PoemBot)
  result = poem_bot(location="Bodega Bay", mood="mysterious", season="autumn")
  print(result.poem)
  ```

  - 단, 아래와 같이 제한된 값을 주지 않더라도 warning만 발생할 뿐 예외나 에러가 발생하지는 않는다.

  ```python
  poem_bot = dspy.Predict(PoemBot)
  result = poem_bot(location="Bodega Bay", mood="mysterious", season="ABC")
  print(result.poem)
  
  # WARNING dspy.predict.predict: Type mismatch for field 'season': expected Literal['spring', 'summer', 'autumn', 'winter'] based on given Signature, but the provided value is incompatible: ABC.
  ```



- Module을 변경하여 inference 전략 변경하기

  - 아래와 같이 module을 변경하여 inference 전략을 바꿀 수 있다.
    - 이전까지 사용했던 `Predict` 대신 `ChainOfThought`를 사용하는데, 이는 LM이 최종 답변을 내놓기 전에 추론하도록 유도하는 모듈이다.
    - 코드의 나머지 부분은 모두 동일하다.

  ```python
  poem_bot = dspy.ChainOfThought(PoemBot)
  result = poem_bot(location="Bodega Bay", mood="mysterious", season="spring")
  print(result.poem)
  ```

  - `ChainOfThought`를 사용할 경우 DSPy는 모델이 최종적으로 시를 만들어내기 전에 추론하도록 유도하기 위해 시그니처를 수정한다. 
    - 새로 추가된 시그니처 출력 필드인 `reasoning`에는 모델의 추론 근거가 담기게 된다.

  ```python
  print(result.reasoning)
  ```



- ReAct

  - DSPy tool은 type-hinted parameter들과 docstring이 있는 Python function이다. 
    - DSPy는 함수의 이름, parameter들 그리고 docstring을 읽은 뒤 이를 종합하여 LM에 전달한다.
    - 예를 들어 아래와 같인 `wikipedia` library를 사용하여 Wikipedia에서 web search를 수행하는 tool을 Python 함수로 정의할 수 있다.
    - Signarture를 정의하는 것과 마찬가지로, tool을 정의할 때도 이름이 중요하다.

  ```python
  import wikipedia
  
  def wikipedia_search(query: str) -> list[str]:
      """Search Wikipedia for the given query and return a list of page titles."""
      return wikipedia.search(query)
  ```

  - `dspy.ReAct` module은 위 함수를 아래와 같에 LM에 전달한다.
    - 아래 tool들 중 `finish`는 DSPy가 내부적으로 정의해서 사용하는 특수 tool로, 모델이 작업을 마쳤을 때 호출한다.

  ```python
  """
  When selecting the next_tool_name and its next_tool_args, the tool must be one of:
  
  (1) wikipedia_search, whose description is <desc>Search Wikipedia for the given query and return a list of page titles.</desc>. It takes arguments {'query': {'type': 'string'}}.
  (2) finish, whose description is <desc>Marks the task as complete. That is, signals that all information for producing the outputs, i.e. `poem`, are now available to be extracted.</desc>. It takes arguments {}.
  """
  ```

  - Wikipedia에서 title에 해당하는 페이지를 가져오는 tool을 추가로 정의한다.

  ```python
  def get_wikipedia_page(title: str) -> str:
      """Get the content of a Wikipedia page given its title."""
      return wikipedia.page(title).content
  ```

  - 위에서 정의한 tool들을 `ReAct.tools`에 넘긴다.
    - 코드가 실행되면 `wikipedia_search`를 통해 Wikipedia에서 "Camp Meeker"와 관련된 페이지들을 찾는다.
    - 찾은 페이지들을 `get_wikipedia_page`을 통해 가져온다.
    - `finish`를 호출하고 결과를 종합한다.

  ```python
  poem_bot = dspy.ReAct(PoemBot, tools=[wikipedia_search, get_wikipedia_page])
  result = poem_bot(location="Camp Meeker", mood="pensive", season="summer")
  print(result.poem)
  
  """
  A summer's day at Camp Meeker,
  Where memories of George Winslow seep.
  The stories of its past unfold,
  Of Klondike Gold Rush tales to be told.
  
  In the heart of California's land,
  A place where history takes a stand.
  George Winslow's name is etched in time,
  A figure of the past, sublime.
  
  The creek that flows, Green Valley's stream,
  Echoes whispers of a bygone dream.
  Dutch Bill Creek's waters, pure and bright,
  Reflect the beauty of a summer's night.
  
  As the sun sets, casting shadows long,
  The memories of Camp Meeker linger strong.
  A poem begins to take shape, weaved with care,
  A tribute to George Winslow, and the history that's shared.
  """
  ```
  
  - `ReAct`는 agentic loop를 관리한다.
    - ReAct는 추론 시점에 작동하는 루프 전략이다. 
    - 모델에게 일련의 도구와 과제를 부여하면, `dspy.ReAct` 모듈은 모델이 도구를 사용해 추론한 뒤 행동하도록 지시한다.
    - 모델이 `finish`를 호출하면, DSPy는 루프를 중단하고 마지막으로 종합(synthesis) 과정을 실행하여 선언된 출력 필드를 생성한다.
    - Model이 몇 번의 loop를 실행할지를 결정하지만, 아래와 같이 `max_iters`를 통해 상한을 제한할 수 있다.
  
  ```python
  poem_bot = dspy.ReAct(PoemBot, tools=[wikipedia_search, get_wikipedia_page], max_iters=4)
  ```
  
  - ReAct가 반환하는 `Prediction` 인스턴스에는 `trajectory` 필드가 포함되어 있다.
    - 이는 각각의 사고(thought), 도구 호출, 그리고 관찰 결과(observation, 도구가 반환한 값)를 순서대로 기록한 딕셔너리이다. 
    - 에이전트가 예상치 못한 행동을 했을 때, 가장 먼저 확인해야 한다.
  
  ```python
  for step, value in result.trajectory.items():
      print(f"{step}: {value}")
  ```

