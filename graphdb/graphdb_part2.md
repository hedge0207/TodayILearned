- 비구조화된 데이터로 knowledge graph를 만드는 과정
  
  > 예시 데이터는 Wikipedia의 Neo4j 문서이다.
  
  - 데이터 수집.
    - Knowledge graph에 포함시키고자 하는 데이터를 수집하는 단계이다.
    - 이 단계에서 데이터의 종류에 따라 데이터를 LLM이 처리할 수 있는 형태(일반적으로 text)로 변환하기도 한다.
  
  ```
  Neo4j is a graph database management system (GDBMS) developed by
  Neo4j Inc.
  The data elements Neo4j stores are nodes, edges connecting them
  and attributes of nodes and edges. Described by its developers
  as an ACID-compliant transactional database with native graph
  storage and processing...
  ```
  
  - 데이터 분할(chunking)
    - 데이터를 적절한 크기로 분할하는 단계이다.
    - 분할의 크기는 사용하는 LLM 혹은 데이터의 복잡도 따라 달라지며, 경우에 따라 분할하지 않기도 한다.
  
  ```
  Neo4j is a graph database management system (GDBMS) developed
  by Neo4j Inc.
  
  The data elements Neo4j stores are nodes, edges connecting them
  and attributes of nodes and edges.
  
  Described by its developers as an ACID-compliant transactional
  database with native graph storage and processing...
  ```
  
  - 데이터 임베딩
    - 데이터의 vector embedding을 생성하는 단계이다.
    - 비슷한 내용의 청크들을 모아서 LLM에게 전달해 중복된 노드 생성을 방지하기 위함이다.
    - 이 때 저장한 vector를 후에 vector RAG를 사용할 때 이용할 수도 있다.
    - 데이터의 수가 많지 않거나, vector RAG를 사용하지 않을 경우 건너뛸 수도 있다.
  
  ```
  [0.21972137987, 0.12345678901, 0.98765432109, ...]
  
  [0.34567890123, 0.23456789012, 0.87654321098, ...]
  
  [0.45678901234, 0.34567890123, 0.76543210987, ...]
  ```
  
  - Node와 relationship을 추출하기 위해 데이터를 LLM에 전달.
    - Text 데이터를 LLM에 전달하여 node와 relationship을 추출한다.
    - 이 때, LLM에게 적절한 prompt를 제공하는 것이 중요하다.
  
  ```json
  /* prmpt
  Your task is to identify the entities and relations requested
  with the user prompt from a given text. You must generate the
  output in a JSON format containing a list with JSON objects.
  Text:
  {text} 
  */
  
  // output
  [
      {
          "text": (
              "Neo4j is a graph database management system (GDBMS) developed by Neo4j Inc."
          ),
          "head": "Neo4j",
          "head_type": "GraphDatabase",
          "relation": "DEVELOPED_BY",
          "tail": "Neo4j Inc",
          "tail_type": "Company",
      },
      {
          "text": (
              "Neo4j is implemented in Java"
          ),
          "head": "Neo4j",
          "head_type": "Graph Database",
          "relation": "IMPLEMENTED_IN",
          "tail": "Java",
          "tail_type": "ProgrammingLanguage",
      },
      ...
  ]
  ```
  
  - LLM의 output으로 knowledge graph 생성.
  
  ```cypher
  MERGE (neo4jInc:Company {id: 'Neo4j Inc'})
  MERGE (neo4j:GraphDatabase {id: 'Neo4j'})
  MERGE (java:ProgrammingLanguage {id: 'Java'})
  MERGE (neo4j)-[:DEVELOPED_BY]->(neo4jInc)
  MERGE (neo4j)-[:IMPLEMENTED_IN]->(java)
  ```

