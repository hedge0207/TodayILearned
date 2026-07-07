# DSPy

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

  

