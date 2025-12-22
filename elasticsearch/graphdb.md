# Graph Database

- 개요

  - Graph DB는 데이터를 graph 형태로 저장하는 DB이다.
    - 하나의 entity가 graph의 node가 된다.
    - Node는 다른 node와의 relationship을 가지며, 각 node는 key-value 형태의 property들을 지닌다.
    - 각각의 node는 label로 그룹화 된다.

  ![sample cypher](graphdb.assets/sample-cypher.svg)

  - Node
    - Graphs의 entity이다.
    - Label로 tag가 가능하며, 이를 통해 domain에 따라 각기 다른 역할을 표현한다.
    - Key-value 쌍의 property들을 가질 수 있다.
    - 읽기 성능 향상을 위해 index를 설정하는 것이 가능하며, 제약 조건의 영향을 받는다.

  - Relationship

    - 두 노드 사이의 명명된(named) 연결을 제공한다(위 예시의 경우 `LOVES`).
    - Relationship의 이름을 type이라 부르며, 반드시 하나의 type을 가져야한다.
    - 반드시 start node와 end node가 있어야한다.
    - 반드시 방향이 있어야 한다.
    - Node는 자기 자신을 가리키는 relationship을 가질 수 있다.
    - Node와 마찬가지로 relationship도 property를 가질 수 있다.
    - Node는 다양한 종류의 relationship응 가질 수 있으며, 다양한 종류의 relationship을 가지더라도 성능이 저하되지 않는다.

  - Label

    - Node는 0개 이상의 label을 가질 수 있다.
    - Label은 node들을 그룹화하기 위해 사용된다.
    - Label은 runtime에 추가되거나 삭제될 수 있기 때문에 node의 일시적인 상태를 표현하기 위해 사용하는 것도 가능하다.

  - 순회와 경로

    - 순회(traversal)는 query에 대한 답을 찾기 위해 graph에 질의를 하는 것이다.
    - 그래프를 순회한다는 것은 특정한 규칙에 따라 relationship을 따라가며 node들을 방문하는 것이다.
    - 대부분의 경우에 전체 그래프가 아닌 일부 부분 집합만 방문하게 된다.

    ![graphdb traversal arr](graphdb.assets/graphdb-traversal-arr-20251217110451143.svg)

    - 예를 들어 톰 행크스가 출연한 영화를 찾으려 한다고 가정해보자.
    - 탐색은 `Tom Hanks` 노드에서 시작해서 `ACTED_IN` type의 relationship을 따라 순회한다.
    - `Movie` label이 붙은 `Forrest Gump` node에서 탐색이 종료된다.
    - 가장 짧은 경로의 길이는 0이며, 그래프에 node가 하나 밖에 없을 때가 이에 해당한다.



- Relational Database와의 비교

  - Relational DB는 고도로 구조화된 형태로 데이터를 저장한다.
    - 즉 미리 정의된 column들에 데이터를 저장한다.
    - 이러한 경직성으로 인해 Relational DB는 개발자에게 application에서 사용할 데이터를 엄격하게 구조화하도록 강제한다.
  - Relational DB는 primary key나 foreign key를 사용하여 다른 row 혹은 다른 table의 데이터를 참조한다.
    - JOIN은 연결된 테이블들의 모든 row의 primary key와 foreign key를 매칭시키는 방식으로 실행된다.
    - Query time에 실행되는 이 작업은 memory를 많이 필요로 하는 무거운 작업이다.
  - 예시
    - 예를 들어 한 직원이 여러 부서에 속할 수 있는 조직이 있고, 이 데이터를 relational db로 관리한다고 가정해보자.
    - 이 경우 아래 그림과 같은 table들이 생성될 것이다.
    - Employee 테이블의 직원과 Departments 테이블의 부서를 연결하기 위해, 한 컬럼에는 직원 ID를, 다른 컬럼에는 해당 부서의 ID를 저장하는 Dpt_Members 조인 테이블을 생성해야 한다.

  ![Depiction of a relational database with connecting points in each table](graphdb.assets/relational-model.svg)

  - Relational DB의 이러한 구조는 경우에 따라 비효율적일 수 있다.
    - 위 예시의 경우 어떤 직원이 어떤 부서와 연결되어 있는지를 파악하려면 직원 ID와 부서 ID 값을 알아야 하고(이를 위해 추가적인 조회가 필요하기 때문에), 연결 관계를 이해하는 것이 번거롭다.
    - 또한 이러한 비용이 큰 JOIN 연산들은 필요한 JOIN의 수를 줄이기 위해 데이터를 비정규화하여 처리하는 경우가 많기에, 그 결과 관계형 데이터베이스의 데이터 무결성이 훼손될 수 있다.
  - 반면에 graph DB를 사용하면 동일한 데이터를 보다 단순하게 처리할 수 있다.
    - 예를 들어 Alice가 속한 모든 부서의 이름을 알고 싶다고 하자.
    - Relational DB에서는 Alice의 primary key를 찾기 위해 Employees 테이블을 조회한 후 Dept_Members에서 Alice의 PK에 해당하는 815를 참조하는 모든 행을 조회한 결과를 가지고 Deparments 테이블에서 실제 이름을 찾아야 한다.
    - 반면에 graph DB에서는 Alice의 Employees node를 찾고, 해당 node와 `BELONGS_TO` relationship으로 연결된 노드들을 찾기만 하면 된다.

  ![Representation of tabular data in a relational database and the comparison with the same data structured in a graph](graphdb.assets/relational-as-graph.svg)



- 다른 NoSQL과 graph DB의 차이

  - 대부분의 NoSQL은 집계 지향적(aggregate oriented)이다.

    - 즉 데이터를 특정한 기준으로 묶고, 이렇게 묶인 데이터를 하나의 단위로 하여 저장하고 조회한다.
    - 예를 들어 Elasticsearch의 경우 여러 개의 field를 하나의 document다른 단위로 묶는다.
    - 이 모델은 단순하고 제한적인 연산만을 제공하며, 데이터에 대한 하나의 특정한 목적을 가진 관점만을 형성한다. 
    - 한 번에 하나의 집계 단위에 집중함으로써, 집계 단위(Elasticsearch를 예로 들면 document)를 기준으로 많은 데이터 조각을 여러 대의 머신으로 쉽게 분산시킬 수 있다.
    - 그러나 이러한 특성으로 인해 데이터를 다른 관점으로 보기 위해서는(즉 다른 형태로 집계하기 위해서는) 데이터를 가공하거나 중복시키는 방식으로 처리해야한다.
    - 예를 들어 인사팀에서 관리하는 직원 정보와, 재무팀 팀에서 관리하는 직원 정보가 있다고 가정해보자.
    - 기본적으로 두 정보는 각기 다른 데이터를 저장하긴 하지만 중복되는 데이터(e.g. 이름, 연락처, 주소 등)도 분명 존재할 것이다.
    - 그러나 NoSQL에서는 이러한 중복되는 정보라 하더라도 집계 기준(인사팀 용, 재무팀 용)에 따라 각 문서에 중복 정보가 들어갈 수 밖에 없다.
    - 즉 중복된 데이터임에도 관점(인사팀, 재무팀)에 따라 어쩔 수 없이 중복 데이터를 저장해야한다.

  - NoSQL 내에 저장된 데이터들은 서로 단절되어 있다.

    - 집계된 각각의 결과들(Elasticsearch의 경우 문서)들은 서로 연결되어 있지 않다.
    - 이러한 저장소에 관계를 추가하기 위해 한 집계의 식별자를 다른 집계에 속한 필드 안에 포함시키는 방식이 사용되기도 하지만 이는 사실상 외래 키를 도입하는 것과 같다.
    - 이러한 방식은 애플리케이션 계층에서 집계들을 직접 조인해야 하므로, 그 비용이 빠르게 증가한다.

  - 다른 NoSQL DB와는 달리 graph DB는 데이터들 사이의 관계를 처리할 수 있다. 

    - 예를 들어 아래와 같은 문서형 NoSQL 데이터는 기본적으로 tree 형태로 이루어져 있으며, 그래프와 달리 상위 노드를 참조하는 것은 불가능하다.

    ![500](graphdb.assets/document_model.jpg)

    - 그러나 위 문서를 아래와 같이 그래프 형태로 표현하면, 각 노드들 사이의 관계를 표현할 수 있으며, 데이터의 중복도 사라지게 된다.

    ![500](graphdb.assets/document_as_graph.jpg)



- Neo4j

  - 대표적인 graph DB이다.
    - 2007년부터 [오픈소스](https://github.com/neo4j/neo4j)로 개발되었다.
    - Cypher라는 전용 query language를 사용한다.
    - Java, Python, Go 등에서 사용할 수 있는 라이브러리를 지원한다.
  - Naming conventions

  | Graph entity      | Style                | Example      |
  | ----------------- | -------------------- | ------------ |
  | Node label        | PascalCase           | VehicleOwner |
  | Relationship type | Screaming snake_case | OWNS_VEHICLE |
  | Property          | camelCase            | firstName    |




- Graph RAG
  - RAG는 일반적으로 데이터를 저장하는데 vector DB만 사용한다.
    - 검색하고자 하는 문서를 embeddinga하여 vector DB에 저장하고 이를 검색시에 활용한다.
  - Graph RAG는 vector DB뿐 아니라 graph DB도 함께 사용하여 데이터를 저장한다.
  - 일반적인 흐름은 아래와 같다.
    - 사용자 질의를 embedding하여 Vector DB에서 관련 entity 후보를 검색한다.
    - 검색된 entity를 graph DB의 seed node로 매핑한다.
    - Seed node를 기준으로 k-hop 또는 조건 기반 그래프 탐색을 수행해 관련 sub graph를 구성한다.
    - Sub graph를 요약·구조화하여 prompt의 context로 LLM에 전달한다.
    - LLM이 해당 context를 근거로 답변을 생성한다.
  - Graph RAG가 일반적인 RAG에 비해 지니는 강점은 아래와 같다.
    - 일반적인 RAG는 검색 단계에서 관계 정보를 명시적으로 다루지 않으며, 관계에 대한 해석과 추론을 LLM에 의존한다.
    - 즉, A 문서에 원인, B 문서에 결과, C 문서에 배경과 같이 정보가 분산되어 있을 경우, RAG는 이 세 문서를 찾을 수는 있어도, 세 문서가 어떻게 연관되어 있는지는 알 수 없고, 이 추론 과정은 LLM에 전가한다.
    - RAG는 단순히 질문과 관련된 문서들만을 추출할 뿐이며, 해당 문장들의 관계 탐색 및 제공은 LLM이 실행한다.
    - 반면에 graph RAG는 entity들 사이의 relationship을 형성함으로써 관계 추론이 가능하다.
    - 즉 grahph RAG는 LLM이 추론하던 관계 연결을 검색 단계로 끌어내린 RAG라고 할 수 있다.
  
  - Graph RAG가 항상 일반 RAG보다 나은 것은 아니며, 아래와 같은 유형의 질문들은 일반 RAG가 더 나을 수 있다.
    - 단순 사실에 대한 질문.
    - FAQ.
    - 최신 뉴스 검색.
  - Graph RAG는 아래와 같은 데이터에 강점이 있다.
    - 관계가 중요한 도메인.
    - “왜 / 어떻게 / 영향”과 같은 질문이 많은 경우.
  - 예를 들어 아래와 같은 세 문장이 있다고 하자.
    - A는 오늘 점심에 B를 먹었다.
    - A가 가장 좋아하는 음식은 B이다.
    - A는 오늘 가장 좋아하는 음식을 먹기로 했다.
  - 이 때, "A가 오늘 점심에 B를 먹은 이유가 무엇인가?"라는 질문이 들어온다면, 이는 RAG나 graph RAG 모두 잘 처리할 수 있을 것이다.
    - 세 문장은 모두 임베딩 공간에서 가깝게 위치할 확률이 높다.
    - 그러므로 top-k를 검색하는 과정에서 세 문장이 모두 함께 들어올 확률이 크다.
    - 그럼 LLM은 세 문장을 받아서 "A는 점심으로 가장 좋아하는 음식을 먹기로 했고, 그 음식이 B이므로 B를 먹었다."는 추론 보다는 사실상 요약에 가까운 추론을 하게 된다.
    - 이 예시의 경우에는 graph RAG가 필요 없는 조건을 거의 완벽하게 만족한다.
  - Graph RAG는 아래와 같은 경우 굳이 도입할 필요가 없다.
    - 문서의 수가 적은 경우.
    - 정보가 한 덩어리로 밀집되어 있는 경우.
    - 관계가 단순한 경우.
    - 누락에 취약하지 않은 경우.
    - 장거리 추론이 불필요한 경우.
  - 만약 위 문장들이 아래와 같이 변경된다면 RAG로는 추론이 어려워질 수 있다.
    - B는 A가 반복적으로 선택해 온 음식 중 하나이다.
    - 점심 메뉴는 개인의 선호와 무관하지 않으며, 특히 강한 선호가 있을 경우 해당 선택이 우선된다.
    - 오늘 A의 점심 선택은 최근의 개인적 선호 패턴을 반영한 결과였다.
  - 위 문장들은 기존 문장들과 아래와 같은 점에서 차이가 있다.
    - A가 무엇을 좋아하는 지에 대한 직접적인 언급이 없다.
    - A가 점심에 무엇을 먹었는지에 대한 언급이 없다.
    - 원인과 결과가 서술적으로 분산되어 있다.
  - 그리고, 위 문장들에 추가적으로 매우 유사한 문장들 다수가 추가되었다고 가정해보자.
    - 이 경우 위 문장들이 top-k에서 누락될 가능성이 생기게 된다.
  - 다시 동일한 질문인 "A가 오늘 점심에 B를 먹은 이유가 무엇인가?"가 들어왔다고 가정해보자.
    - 만약 너무 많은 문장이 임베딩 공간에 몰려 있어 위 세 문장중 한 문장이 top-k에서 누락되었다고 가정해보자.
    - 세 문장 중 한 문장만 누락되어도 LLM 입장에서는 인과 추론이 쉽지 않아진다.
    - 만약 세 문장 다 누락되어 있지 않더라도 명시적인 서술이 없으므로 LLM은 불명확한 추론을 할 수 밖에 없어지고, hallucination 가능성이 높아진다.
  - Graph RAG는 위 문장을 아래와 같이 처리한다.
  
  ```
  Entity:
  - A (Person)
  - B (Food)
  - Lunch Selection (Event)
  - Preference Pattern (Concept)
  
  Relations:
  - A ── frequently_selects ──▶ B
  - Lunch Selection ── influenced_by ──▶ Preference Pattern
  - Preference Pattern ── belongs_to ──▶ A
  ```
  
  - 동일한 질문이 들어왔을 때, graph를 아래와 같은 순서로 탐색하게 된다.
    - 선택의 이유가 경로로 존재한다.
    - 따라서 LLM은 더 이상 관계를 만들어 낼 필요가 없으며 graph를 탐색하면서 얻어진 맥락을 바탕으로 문장만 만들어 내면 된다.
  
  ```
  A
   → Preference Pattern
   → Lunch Selection
   → B
  ```



- Multihop RAG

  - 질문에 답하기 위해 필요한 근거를 여러 단계(hop)로 나눠 순차적으로 수집·결합하는 RAG 설계 패턴이다.
    - 단순히 RAG를 여러 번 사용하는 것이 아니다.
    - 한 번의 검색으로는 답을 만들 수 없을 때, 중간 근거를 얻고 이를 바탕으로 다음 검색을 반복하여 최종 답을 구성하는 RAG 방식이다.
  - Multihop RAG는 주로 아래와 같은 질문들에 사용된다.
    - 왜 / 어떻게 / 영향 / 원인
    - A와 B의 관계는?
    - 이 결정이 어디까지 전파됐나?
  - 아래는 multihop RAG의 전형적인 동작 방식이다.

  ```
  Q0 (원 질문)
   → 검색 → 근거1
   → (LLM) 중간 질문 Q1 생성
   → 검색 → 근거2
   → (LLM) 중간 질문 Q2 생성
   → 검색 → 근거3
   → 모든 근거 결합 → 최종 답
  ```

  - Graph RAG와 유사하지만 같은 개념은 아니다.
    - 둘 다 multi hop에 대응하기 위한 방식이라는 점에서는 동일하다.
    - 그러나 multi-hop RAG는 실시간 추적형인 반면 GraphRAG는 사전 구조화형 multi-hop 대응 방식이다.
    - 관계가 자주 재사용되고 안정적이면 GraphRAG를 사용하는 것이 적절하고, 관계가 질문마다 달라지고 즉석 탐색이 필요하면 Multi-hop RAG를 사용하는 것이 적절하다.



- Graph RAG 구성하기

  > https://neo4j.com/blog/developer/graphrag-agent-neo4j-milvus/ 참고
  >
  > 전체 코드는 https://github.com/milvus-io/bootcamp/blob/master/bootcamp/RAG/advanced_rag/langgraph-graphrag-agent-local.ipynb에서 확인 가능하다.

  - 본 예시에서 구성할 agent는 아래 세 가지 주요 개념을 가지고 있다.
    - Routing: Query에 기반하여 vector DB를 사용할지, graph DB를 사용할지, 혹은 둘 다 사용할지를 결정하는 mechanism이다.
    - Fallback: 첫 검색이 충분하지 않을 경우 agent는 Tavily를 사용하여 다시 web 검색을 수행한다.
    - Self-correction: Agent는 자신이 생성한 답변을 평가하여 hallucination이나 부정확한 내용을 교정한다.
  - 위 세 가지 주요 개념 외에 아래와 같은 component들도 포함된다.
    - Retrieval: 데이터를 저장하고, 사용자의 query에 대해 문서를 검색하기 위해 Milvus를 vector DB로 사용한다.
    - Graph enhancement: 탐색된 문서로부터 knowledge graph를 생성하고, 관계와 엔티티 정보를 통해 문맥을 더욱 풍부하게 만든다.
    - LLMs integration: 로컬 LLM인 Llama 3.1 8B를 사용해 답변을 생성하고 검색된 정보의 관련성과 정확성을 평가한다. Neo4j의 질의 언어인 Cypher 생성에는 GPT-4o를 사용한다.
  - Graph RAG 구조
    - Question routing:  Agent는 우선 질문을 분석하여 vector search와 graph search 혹을 둘 다 사용하는 것 중 뭐가 가장 적절한지를 결정한다.
    - Retrieval: Routing 결과에 따라 질문과 과련된 문서들이 Milvus 또는 Neo4j graph에서 수집된다.
    - Generation: LLM은 수집된 context를 사용하여 답변을 생성한다.
    - Evaluation: Agent는 LLM이 생성한 답변의 관련성, 정확성, hallucination 가능성에 대해 평가한다.
    - Refinement: 만약 답변이 만족스럽지 않다면, agent는 error를 교정하고 검색 결과를 정제한다.

  ![langgraph_adaptive_rag.png](graphdb.assets/RAG_Agent_langGraph.png)

  - Graph 생성
    - `cypher_llm`는 LLM instance를 입력하며, 이 instance를 통해 사용자의 질의를 기반으로 graph에서 연관된 정보를 추울하기 위한 Cypher query를 생성한다.

  ```python
  # Cypher query 생성을 위한 llm instance를 생성한다.
  llm = ChatOllama(model=local_llm, temperature=0)
  
  # Chain
  graph_rag_chain = GraphCypherQAChain.from_llm(
          cypher_llm=llm,
          qa_llm=llm,
          validate_cypher=True,
          graph=graph,
          verbose=True,
          return_intermediate_steps=True,
          return_direct=True,
      )
  
  # 실행한다.
  question = "agent memory"
  generation = graph_rag_chain.invoke({"query": question})
  ```

  - Graph DB를 사용하여 검색하기

  ```python
  # Composite Vector + Graph Generations
  cypher_prompt = PromptTemplate(
      template="""You are an expert at generating Cypher queries for Neo4j.
      Use the following schema to generate a Cypher query that answers the given question.
      Make the query flexible by using case-insensitive matching and partial string matching where appropriate.
      Focus on searching paper titles as they contain the most relevant information.
      
      Schema:
      {schema}
      
      Question: {question}
      
      Cypher Query:""",
      input_variables=["schema", "question"],
  )
  
  # QA prompt
  qa_prompt = PromptTemplate(
      template="""You are an assistant for question-answering tasks. 
      Use the following Cypher query results to answer the question. If you don't know the answer, just say that you don't know. 
      Use three sentences maximum and keep the answer concise. If topic information is not available, focus on the paper titles.
      
      Question: {question} 
      Cypher Query: {query}
      Query Results: {context} 
      
      Answer:""",
      input_variables=["question", "query", "context"],
  )
  
  llm = ChatOpenAI(model="gpt-4o", temperature=0)
  
  # Chain
  graph_rag_chain = GraphCypherQAChain.from_llm(
      cypher_llm=llm,
      qa_llm=llm,
      validate_cypher=True,
      graph=graph,
      verbose=True,
      return_intermediate_steps=True,
      return_direct=True,
      cypher_prompt=cypher_prompt,
      qa_prompt=qa_prompt,
  )
  
  
  # Example input data
  question = "What paper talks about Multi-Agent?"
  generation = graph_rag_chain.invoke({"query": question})
  print(generation)
  ```

  - Vector DB를 사용하여 검색하기

  ```python
  # Example input data
  question = "What paper talks about Multi-Agent?"
  
  # Get vector + graph answers
  docs = retriever.invoke(question)
  vector_context = rag_chain.invoke({"context": docs, "question": question})
  ```

  - Graph DB의 결과와 vector DB의 결과를 결합하기

  ```py
  composite_chain = prompt | llm | StrOutputParser()
  answer = composite_chain.invoke({"question": question, "context": vector_context, "graph_context": graph_context})
  
  print(answer)
  ```




- Graph RAG를 테스트하기에 적절한 dataset들
  - https://github.com/Alab-NII/2wikimultihop
  - https://github.com/yixuantt/MultiHop-RAG



















