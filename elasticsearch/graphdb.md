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

  

