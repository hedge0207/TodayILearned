# LangChain

> https://github.com/langchain-ai

- LangChain
  - LLM을 활용한 애플리케이션을 쉽게 만들 수 있도록 도와주는 프레임워크.
    - LLM을 사용한 custom agents를 보다 간편하게 구축하기 위한 여러 가지 기능을 제공한다.
    - MIT 라이선스로 공개된 open source이다.
  - LangChain AI 에서 개발하며, LangChain외에도 아래와 같은 프레임워크들을 제공한다.
    - LangGraph: LangChain보다 유연하게 custom agent를 구축할 수 있는 툴들을 제공한다.
    - LangSmith: 모니터링을 위한 다양한 툴을 제공하며, 제한적으로 무료로 사용이 가능하다.



- LangChain 사용해보기

  > Local LLM 사용을 위해 Ollama를 실행한다.

  - LangChain 설치
    - Python 3.10 이상이 필요하다.

  ```bash
  $ pip install -U langchain
  ```

  - Ollama integration 설치

  ```bash
  $ pip install langchain-ollama
  ```

  - 사용해보기
    - `create_agent()`메서드의 `tools` 파라미터는 LLM이 호출할 수 있는 함수들을 받는데, LLM은 상황에 맞게 이 tool들을 활용하게 된다. 
    - `agent.invoke()`메서드가 반환하는 dictionary의 `messages`에는 대화의 전체 히스토리가 담겨 있다.
    - 테스트에 사용한 LLM에 따라 다른 답변이 나올 수 있으며, 같은 LLM이라도 호출할 때 마다 다른 답변이 나올 수 있다.
    - `HumanMessage`는 사용자의 메시지, `ToolMessage`는 tool 실행 후 반환된 결과, `AIMessage`는 LLM의 메시지이다.

  ```python
  from langchain.agents import create_agent
  
  def get_weather(city: str) -> str:
      """Get weather for a given city."""
      return f"It's always sunny in {city}!"
  
  agent = create_agent(
      model="ollama:llama3.2",
      tools=[get_weather],
      system_prompt="You are a helpful assistant",
  )
  
  result = agent.invoke(
      {"messages": [{"role": "user", "content": "What's the weather in Seoul?"}]}
  )
  # 최종 응답 확인
  print(result["messages"][-1].content_blocks)
  
  # 히스토리 확인
  for msg in result["messages"]:
      print(type(msg).__name__, ":", msg)
      print("-"*100)
  ```



- vLLM으로 실행한 model에 연결할 경우.

  - vLLM은 OpenAI 호환 API를 제공하기 때문에 `langchain-openai`를 활용하면 된다.
    - 단, vLLM 서버를 띄울 때 아래 옵션을 꼭 추가해야 tool calling이 가능하다.

  ```bash
  $ vllm serve <model> \
      --host 0.0.0.0 \
      --port 8000 \
      --enable-auto-tool-choice \        # tool calling 활성화
      --tool-call-parser <call_parser>          # model에 맞는 call parser 지정
  ```

  - 아래와 같이 연결한다.
    - 정확한 모델명은 `http://your-server-ip:port/v1/models`에서 model의 id를 확인하면 된다.

  ```python
  from langchain_openai import ChatOpenAI
  from langchain.agents import create_agent
  
  
  llm = ChatOpenAI(
      model="your_model_name",
      base_url="http://your-server-ip:port/v1",
      api_key="EMPTY",
      temperature=0
  )
  
  def get_weather(city: str) -> str:
      """Get weather for a given city."""
      return f"It's always sunny in {city}!"
  
  agent = create_agent(
      model=llm,
      tools=[get_weather],
      system_prompt="You are a helpful assistant",
  )
  
  result = agent.invoke(
      {"messages": [{"role": "user", "content": "What's the weather in Seoul?"}]}
  )
  print(result["messages"][-1].content)
  ```







## Agent

- Agent

  - LLM을 tool과 결합하여 task를 해결할 수 있게 해주는 component이다.
    - Input이 들어오면 model은 가지고 있는 tool들을 활용하여 답변을 생성한다.
    - 이 과정은 답변이 완성되거나 설정된 반복 횟수에 도달할 때 까지 반복적으로 수행된다. 

  - `create_agent()`를 통해 성생할 수 있다.
    - 생성시에 name을 설정할 수 있는데, 이는 주로 multi-agent system을 구축할 때 사용한다.

  ```python
  agent = create_agent(
      model,
      tools,
      name="research_assistant"
  )
  ```

  - 아래와 같은 component들을 가진다.
    - Model
    - Tools
    - System prompt
    - Name



- Model

  - Agent가 사용하는 추론 engine이다.
    - Static하게 설정할 수도 있고 dynamic하게 모델을 선택하도록 설정할 수도 있다.
  - Static model
    - Agent를 처음 생성할 때 설정하고, 실행까지 변경하지 않는 방식이다.
    - 가장 일반적이고 직관적인 방식이다.

  ```python
  from langchain.agents import create_agent
  
  agent = create_agent("openai:gpt-5.4", tools=tools)
  ```

  - Provider package에서 제공하는 model class의 인스턴스를 생성하여 각 model의 설정을 변경할 수 있다.
    - 각 provider별 설정은 [링크](https://docs.langchain.com/oss/python/integrations/providers/all_providers#providers)에서 확인할 수 있다.

  ```python
  from langchain.agents import create_agent
  from langchain_openai import ChatOpenAI
  
  model = ChatOpenAI(
      model="gpt-5.4",
      temperature=0.1,
      max_tokens=1000,
      timeout=30
      # ... (other params)
  )
  agent = create_agent(model, tools=tools)
  ```

  - Dynamic model
    - 현자 상태와 맥락에 따라 runtime에 model을 선택하는 방식이다.
    - 이는 정교한 routing고 비용 최적화를 가능하게 한다.
    - Dynamic model 설정을 위해서는 `@wrap_model_call` decorator를 사용하여 middleware를 생성해야한다.

  ```python
  from langchain_openai import ChatOpenAI
  from langchain.agents import create_agent
  from langchain.agents.middleware import wrap_model_call, ModelRequest, ModelResponse
  
  
  basic_model = ChatOpenAI(model="gpt-5.4-mini")
  advanced_model = ChatOpenAI(model="gpt-5.4")
  
  @wrap_model_call
  def dynamic_model_selection(request: ModelRequest, handler) -> ModelResponse:
      """Choose model based on conversation complexity."""
      message_count = len(request.state["messages"])
  
      if message_count > 10:
          # Use an advanced model for longer conversations
          model = advanced_model
      else:
          model = basic_model
  
      return handler(request.override(model=model))
  
  agent = create_agent(
      model=basic_model,  # Default model
      tools=tools,
      middleware=[dynamic_model_selection]
  )
  ```



- Tools

  - Agent가 취할 수 있는 행동들을 제공하는 기능으로, 아래와 같은 것들이 가능하다.
    - 여러 tool을 순차적으로 호출.
    - 여러 tool을 병렬 호출 호출.
    - 이전 결과를 기반으로 한 동적 tool 선택.
    - Tool 재시도 로직 및 오류 처리.
    - Tool 호출 간 상태 지속 유지.
  - Static tools
    - Agent가 생성될 때 설정되어 실행시까지 변경되지 않는 방식이다.
    - 가장 일반적이고 직관적인 방식이다.

  ```python
  from langchain.tools import tool
  from langchain.agents import create_agent
  
  
  @tool
  def search(query: str) -> str:
      """Search for information."""
      return f"Results for: {query}"
  
  @tool
  def get_weather(location: str) -> str:
      """Get weather information for a location."""
      return f"Weather in {location}: Sunny, 72°F"
  
  agent = create_agent(model, tools=[search, get_weather])
  ```

  - Dynamic tools
    - Runtime에 tools들을 변경하는 방식이다.
    - 모든 tool 모든 상황에 적합한 것은 아니며, tool이 너무 많으면 모델에 과부하가 걸려 오류가 증가할 수 있고, tool 너무 적으면 기능이 제한될 수 있다.
    - Tool을 통적으로 선택하게 함으로써 인증 상태, 사용자 권한, 기능 플래그 또는 대화 단계에 따라 사용 가능한 tool들을 조정할 수 있다.
    - 아래 예시는 tool들을 runtime에 추가하는 예시이며, agent 생성 시점에 이미 등록한 tool들을 변경하는 예시는 하단에서 확인 가능하다.
    - `wrap_model_call`를 통해 request에 tool을 추가하고, `wrap_tool_call`를 통해 동적으로 추가된 tool의 실행을 통제한다.

  ```python
  from langchain.tools import tool
  from langchain.agents import create_agent
  from langchain.agents.middleware import AgentMiddleware, ModelRequest, ToolCallRequest
  
  # A tool that will be added dynamically at runtime
  @tool
  def calculate_tip(bill_amount: float, tip_percentage: float = 20.0) -> str:
      """Calculate the tip amount for a bill."""
      tip = bill_amount * (tip_percentage / 100)
      return f"Tip: ${tip:.2f}, Total: ${bill_amount + tip:.2f}"
  
  class DynamicToolMiddleware(AgentMiddleware):
      """Middleware that registers and handles dynamic tools."""
  
      def wrap_model_call(self, request: ModelRequest, handler):
          # Add dynamic tool to the request
          # This could be loaded from an MCP server, database, etc.
          updated = request.override(tools=[*request.tools, calculate_tip])
          return handler(updated)
  
      def wrap_tool_call(self, request: ToolCallRequest, handler):
          # Handle execution of the dynamic tool
          if request.tool_call["name"] == "calculate_tip":
              return handler(request.override(tool=calculate_tip))
          return handler(request)
  
  agent = create_agent(
      model="gpt-4o",
      tools=[get_weather],  # Only static tools registered here
      middleware=[DynamicToolMiddleware()],
  )
  
  # The agent can now use both get_weather AND calculate_tip
  result = agent.invoke({
      "messages": [{"role": "user", "content": "Calculate a 20% tip on $85"}]
  })
  ```

  - Tool error handling
    - `@wrap_tool_call` decorator를 사용하여 tool 호출 중 발생한 에러를 처리할 수 있다. 

  ```python
  from langchain.agents import create_agent
  from langchain.agents.middleware import wrap_tool_call
  from langchain.messages import ToolMessage
  
  
  @wrap_tool_call
  def handle_tool_errors(request, handler):
      """Handle tool execution errors with custom messages."""
      try:
          return handler(request)
      except Exception as e:
          # Return a custom error message to the model
          return ToolMessage(
              content=f"Tool error: Please check your input and try again. ({str(e)})",
              tool_call_id=request.tool_call["id"]
          )
  
  agent = create_agent(
      model="gpt-5.4",
      tools=[search, get_weather],
      middleware=[handle_tool_errors]
  )
  ```

  - ReAct 루프에서의 도구 사용
    - Agent는 ReAct("추론 + 행동") 패턴을 따른다. 
    - 이 패턴은 간략한 추론 단계와 목적에 맞는 tool 호출을 번갈아 수행하고, 그 결과로 얻은 관찰 내용을 이후 의사결정에 반영하는 방식으로, 최종 답변을 도출할 때까지 이 과정을 반복한다.

  ```
  Prompt: Identify the current most popular wireless headphones and verify availability.
  
  ================================ Human Message =================================
  
  Find the most popular wireless headphones right now and check if they're in stock
  
  Reasoning: “Popularity is time-sensitive, I need to use the provided search tool.”
  Acting: Call search_products("wireless headphones")
  
  
  ================================== Ai Message ==================================
  Tool Calls:
    search_products (call_abc123)
   Call ID: call_abc123
    Args:
      query: wireless headphones
  
  
  ================================= Tool Message =================================
  
  Found 5 products matching "wireless headphones". Top 5 results: WH-1000XM5, ...
  
  Reasoning: “I need to confirm availability for the top-ranked item before answering.”
  Acting: Call check_inventory("WH-1000XM5")
  
  ================================== Ai Message ==================================
  Tool Calls:
    check_inventory (call_def456)
   Call ID: call_def456
    Args:
      product_id: WH-1000XM5
  
  ================================= Tool Message =================================
  
  Product WH-1000XM5: 10 units in stock
  
  Reasoning: “I have the most popular model and its stock status. I can now answer the user’s question.”
  Acting: Produce final answer
  
  ================================== Ai Message ==================================
  
  I found wireless headphones (model WH-1000XM5) with 10 units in stock...
  ```



- Tool들이 미리 정의된 경우의 dynamic tools 방식

  - 아래와 같이 agent가 생성될 때 tool들이 이미 모두 정의된 상태라면, agent 생성시에 tool들을 미리 정의해두고, 추후에 변경하는 것이 가능하다.

  ```python
  from langchain.agents import create_agent
  from langchain.agents.middleware import wrap_model_call, ModelRequest, ModelResponse
  from typing import Callable
  
  @wrap_model_call
  def state_based_tools(
      request: ModelRequest,
      handler: Callable[[ModelRequest], ModelResponse]
  ) -> ModelResponse:
      """Filter tools based on conversation State."""
      # Read from State: check if user has authenticated
      state = request.state
      is_authenticated = state.get("authenticated", False)
      message_count = len(state["messages"])
  
      # Only enable sensitive tools after authentication
      if not is_authenticated:
          tools = [t for t in request.tools if t.name.startswith("public_")]
          request = request.override(tools=tools)
      elif message_count < 5:
          # Limit tools early in conversation
          tools = [t for t in request.tools if t.name != "advanced_search"]
          request = request.override(tools=tools)
  
      return handler(request)
  
  agent = create_agent(
      model="gpt-5.4",
      tools=[public_search, private_search, advanced_search],
      middleware=[state_based_tools]
  )
  ```

  - 또한 아래와 같이 Store에 저장된 정보를 통해 판단할 수도 있다.

  ```python
  from dataclasses import dataclass
  from langchain.agents import create_agent
  from langchain.agents.middleware import wrap_model_call, ModelRequest, ModelResponse
  from typing import Callable
  from langgraph.store.memory import InMemoryStore
  
  @dataclass
  class Context:
      user_id: str
  
  @wrap_model_call
  def store_based_tools(
      request: ModelRequest,
      handler: Callable[[ModelRequest], ModelResponse]
  ) -> ModelResponse:
      """Filter tools based on Store preferences."""
      user_id = request.runtime.context.user_id
  
      # Read from Store: get user's enabled features
      store = request.runtime.store
      feature_flags = store.get(("features",), user_id)
  
      if feature_flags:
          enabled_features = feature_flags.value.get("enabled_tools", [])
          # Only include tools that are enabled for this user
          tools = [t for t in request.tools if t.name in enabled_features]
          request = request.override(tools=tools)
  
      return handler(request)
  
  agent = create_agent(
      model="gpt-5.4",
      tools=[search_tool, analysis_tool, export_tool],
      middleware=[store_based_tools],
      context_schema=Context,
      store=InMemoryStore()
  )
  ```

  - 혹은 아래와 같인 runtime context를 통해 판단할 수도 있다.

  ```python
  from dataclasses import dataclass
  from langchain.agents import create_agent
  from langchain.agents.middleware import wrap_model_call, ModelRequest, ModelResponse
  from typing import Callable
  
  @dataclass
  class Context:
      user_role: str
  
  @wrap_model_call
  def context_based_tools(
      request: ModelRequest,
      handler: Callable[[ModelRequest], ModelResponse]
  ) -> ModelResponse:
      """Filter tools based on Runtime Context permissions."""
      # Read from Runtime Context: get user role
      if request.runtime is None or request.runtime.context is None:
          # If no context provided, default to viewer (most restrictive)
          user_role = "viewer"
      else:
          user_role = request.runtime.context.user_role
  
      if user_role == "admin":
          # Admins get all tools
          pass
      elif user_role == "editor":
          # Editors can't delete
          tools = [t for t in request.tools if t.name != "delete_data"]
          request = request.override(tools=tools)
      else:
          # Viewers get read-only tools
          tools = [t for t in request.tools if t.name.startswith("read_")]
          request = request.override(tools=tools)
  
      return handler(request)
  
  agent = create_agent(
      model="gpt-5.4",
      tools=[read_data, write_data, delete_data],
      middleware=[context_based_tools],
      context_schema=Context
  )
  ```



- System prompt

  - Prompt를 제공하여 agent가 task에 접근하는 방식을 설계할 수 있다.
    - `system_prompt` parameter는 `str` 또는 `SystemMessage`의 인스턴스 중 하나를 받는다.

  ```python
  agent = create_agent(
      model,
      tools,
      system_prompt="You are a helpful assistant. Be concise and accurate."
  )
  ```

  - `SystemMessage`를 사용할 경우 prompt를 보다 구조적으로 작성할 수 있다.

  ```python
  from langchain.agents import create_agent
  from langchain.messages import SystemMessage, HumanMessage
  
  literary_agent = create_agent(
      model="google_genai:gemini-3.1-pro-preview",
      system_prompt=SystemMessage(
          content=[
              {
                  "type": "text",
                  "text": "You are an AI assistant tasked with analyzing literary works.",
              },
              {
                  "type": "text",
                  "text": "<the entire contents of 'Pride and Prejudice'>",
                  "cache_control": {"type": "ephemeral"}
              }
          ]
      )
  )
  
  result = literary_agent.invoke(
      {"messages": [HumanMessage("Analyze the major themes in 'Pride and Prejudice'.")]}
  )
  ```

  - Dynamic system prompting
    - Runtime에 prompt를 변경하는 것도 가능하다.
    - `@dynamic_prompt`를 사용하면 된다.

  ```python
  from typing import TypedDict
  
  from langchain.agents import create_agent
  from langchain.agents.middleware import dynamic_prompt, ModelRequest
  
  
  class Context(TypedDict):
      user_role: str
  
  @dynamic_prompt
  def user_role_prompt(request: ModelRequest) -> str:
      """Generate system prompt based on user role."""
      user_role = request.runtime.context.get("user_role", "user")
      base_prompt = "You are a helpful assistant."
  
      if user_role == "expert":
          return f"{base_prompt} Provide detailed technical responses."
      elif user_role == "beginner":
          return f"{base_prompt} Explain concepts simply and avoid jargon."
  
      return base_prompt
  
  agent = create_agent(
      model="gpt-5.4",
      tools=[web_search],
      middleware=[user_role_prompt],
      context_schema=Context
  )
  
  # The system prompt will be set dynamically based on context
  result = agent.invoke(
      {"messages": [{"role": "user", "content": "Explain machine learning"}]},
      context={"user_role": "expert"}
  )
  ```

  







