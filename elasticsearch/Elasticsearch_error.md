- `max virtual memory areas vm.max_map_count [65530] is too low, increase to at least [262144]`

  - 원인
    - 운영체제가 기본으로 사용하는 mmap 사이즈인 65530이 너무 작아서 발생
  - 해결
    - mmap 사이즈를 262144로 증가시킨다.
    - `/etc/sysctl.conf` 파일에 `vm.max_map_count=262144`를 추가 후 재부팅
    - 위 방식은 재부팅이 필요하기에, 라이브 서비스의 경우 아래 명령어를 통해 바로 적용이 가능하다(단, 재부팅시 초기화된다.)

  ```bash
  $ sudo sysctl -w vm.max_map_count=262144
  ```




- GraphTokenStreamFiniteStrings에서 재귀의 깊이가 깊어져 stack overflow가 발생

  > https://github.com/apache/lucene/issues/11218
  >
  > https://github.com/apache/lucene/issues/12359

  - 상세 에러
    - Elasticsearch 7.16.3에서 발생
    - 의어 설정이 적용된 환경에서 사용자가 매우 긴 검색어를 입력했을 때, 검색 엔진 내부 분석 알고리즘이 무한에 가까운 재귀 호출을 일으켜 stack oveflow가 발생.

  ```
  {"type": "server", "timestamp": "2026-07-28T05:24:52,088Z", "level": "ERROR", "component": "o.e.b.ElasticsearchUncaughtExceptionHandler", "cluster.name": "my_cluster", "node.name": "my_node", "message": "fatal error in thread [elasticsearch[my_node][search][T#5]], exiting", "cluster.uuid": "CiWGrQ4hTEabYCTuksNj_Q", "node.id": "4iIhW2iUR0m8jZPc8DZ1UQ" ,
  "stacktrace": ["java.lang.StackOverflowError: null",
  "at org.apache.lucene.util.graph.GraphTokenStreamFiniteStrings.articulationPointsRecurse(GraphTokenStreamFiniteStrings.java:266) ~[lucene-core-8.10.1.jar:8.10.1 2f24e6a49d48a032df1f12e146612f59141727a9 - mayyasharipova - 2021-10-12 15:13:05]",
  "at org.apache.lucene.util.graph.GraphTokenStreamFiniteStrings.articulationPointsRecurse(GraphTokenStreamFiniteStrings.java:277) ~[lucene-core-8.10.1.jar:8.10.1 2f24e6a49d48a032df1f12e146612f59141727a9 - mayyasharipova - 2021-10-12 15:13:05]",
  "at org.apache.lucene.util.graph.GraphTokenStreamFiniteStrings.articulationPointsRecurse(GraphTokenStreamFiniteStrings.java:277) ~[lucene-core-8.10.1.jar:8.10.1 2f24e6a49d48a032df1f12e146612f59141727a9 - mayyasharipova - 2021-10-12 15:13:05]",
  "at org.apache.lucene.util.graph.GraphTokenStreamFiniteStrings.articulationPointsRecurse(GraphTokenStreamFiniteStrings.java:277) ~[lucene-core-8.10.1.jar:8.10.1 2f24e6a49d48a032df1f12e146612f59141727a9 - mayyasharipova - 2021-10-12 15:13:05]",
  "at org.apache.lucene.util.graph.GraphTokenStreamFiniteStrings.articulationPointsRecurse(GraphTokenStreamFiniteStrings.java:277) ~[lucene-core-8.10.1.jar:8.10.1 2f24e6a49d48a032df1f12e146612f59141727a9 - mayyasharipova - 2021-10-12 15:13:05]",
  "at org.apache.lucene.util.graph.GraphTokenStreamFiniteStrings.articulationPointsRecurse(GraphTokenStreamFiniteStrings.java:277) ~[lucene-core-8.10.1.jar:8.10.1 2f24e6a49d48a032df1f12e146612f59141727a9 - mayyasharipova - 2021-10-12 15:13:05]",
  "at org.apache.lucene.util.graph.GraphTokenStreamFiniteStrings.articulationPointsRecurse(GraphTokenStreamFiniteStrings.java:277) ~[lucene-core-8.10.1.jar:8.10.1 2f24e6a49d48a032df1f12e146612f59141727a9 - mayyasharipova - 2021-10-12 15:13:05]",
  # 같은 로그 반복
  ```

  - 재현
    - `synonym_graph` token filter를 사용하는 analyzer로 synonym이 다수 포함된 매우 긴 text를 분석하면 위 에러가 발생하는 것을 확인할 수 있다.

  ```json
  // PUT reproduce_sofe
  {
    "settings": {
      "index": {
        "number_of_shards": 1,
        "analysis": {
          "analyzer": {
            "heavy_synonym_analyzer": {
              "tokenizer": "nori_tokenizer",
              "filter": [
                "custom_graph_synonyms"
              ]
            }
          },
          "filter": {
            "custom_graph_synonyms": {
              "type": "synonym_graph",
              "synonyms": [
                "가나다, 가나, 나다, 가, 나, 다",
                "사아자, 사아, 아자, 사, 아, 자",
                "차카타, 차카, 카타, 차, 카, 타",
                "파하가, 파하, 하가, 파, 하, 가",
                "123, 12, 23, 1, 2, 3",
                "456, 45, 56, 4, 5, 6",
                "789, 78, 89, 7, 8, 9"
              ]
            }
          }
        }
      }
    },
    "mappings": {
      "properties": {
        "content": {
          "type": "text",
          "analyzer": "heavy_synonym_analyzer"
        }
      }
    }
  }
  
  GET reproduce_sofe/_search
  {
    "query": {
      "match": {
        "content": "가나다 라마바 사아자 차카타 파하가 123 456 789 가나다 라마바 사아자 차카타 파하가 123 456 789 가나다 라마바 사아자 차카타 파하가 123 456 789 가나다 라마바 사아자 차카타 파하가 123 456 789 가na다 라마바 사아자 차카타 파하가 123 456 789 가나다 라마바 사아자 차카타 파하가 123 456 789 가나다 라마바 사아자 차카타 파하가 123 456 789 가나다 라마바 사아자 차카타 파하가 123 456 789 가나다 라마바 사아자 차카타 파하가 123 456 789 가나다 라마바 사아자 차카타 파하가 123 456 789 가나다 라마바 사아자 차카타 파하가 123 456 789 가나다 라마바 사아자 차카타 파하가 123 456 789 가나다 라마바 사아자 차카타 파하가 123 456 789 가나다 라마바 사아자 차카타 파하가 123 456 789 가나다 라마바 사아자 차카타 파하가 123 456 789 가나다 라마바 사아자 차카타 파하가 123 456 789 가나다 라마바 사아자 차카타 파하가 123 456 789 가나다 라마바 사아자 차카타 파하가 123 456 789 가나다 라마바 사아자 차카타 파하가 123 456 789 가나다 라마바 사아자 차카타 파하가 123 456 789 가나다 라마바 사아자 차카타 파하가 123 456 789 가나다 라마바 사아자 차카타 파하가 123 456 789 가na다 라마바 사아자 차카타 파하가 123 456 789 가나다 라마바 사아자 차카타 파하가 123 456 789 가나다 라마바 사아자 차카타 파하가 123 456 789 가나다 라마바 사아자 차카타 파하가 123 456 789 가나다 라마바 사아자 차카타 파하가 123 456 789 가나다 라마바 사아자 차카타 파하가 123 456 789 가나다 라마바 사아자 차카타 파하가 123 456 789 가나다 라마바 사아자 차카타 파하가 123 456 789 가나다 라마바 사아자 차카타 파하가 123 456 789 가나다 라마바 사아자 차카타 파하가 123 456 789 가나다 라마바 사아자 차카타 파하가 123 456 789 가나다 라마바 사아자 차카타 파하가 123 456 789 가나다 라마바 사아자 차카타 파하가 123 456 789 가나다 라마바 사아자 차카타 파하가 123 456 789 가나다 라마바 사아자 차카타 파하가 123 456 789 가나다 라마바 사아자 차카타 파하가 123 456 789 가나다 라마바 사아자 차카타 파하가 123 456 789 가나다 라마바 사아자 차카타 파하가 123 456 789 가na다 라마바 사아자 차카타 파하가 123 456 789 가나다 라마바 사아자 차카타 파하가 123 456 789 가나다 라마바 사아자 차카타 파하가 123 456 789 가나다 라마바 사아자 차카타 파하가 123 456 789 가나다 라마바 사아자 차카타 파하가 123 456 789 가나다 라마바 사아자 차카타 파하가 123 456 789 가나다 라마바 사아자 차카타 파하가 123 456 789 가나다 라마바 사아자 차카타 파하가 123 456 789 가나다 라마바 사아자 차카타 파하가 123 456 789 가나다 라마바 사아자 차카타 파하가 123 456 789 가나다 라마바 사아자 차카타 파하가 123 456 789 가나다 라마바 사아자 차카타 파하가 123 456 789 가나다 라마바 사아자 차카타 파하가 123 456 789 가나다 라마바 사아자 차카타 파하가 123 456 789 가나다 라마바 사아자 차카타 파하가 123 456 789 가나다 라마바 사아자 차카타 파하가 123 456 789 가나다 라마바 사아자 차카타 파하가 123 456 789 가나다 라마바 사아자 차카타 파하가 123 456 789 가na다 라마바 사아자 차카타 파하가 123 456 789 가나다 라마바 사아자 차카타 파하가 123 456 789 가나다 라마바 사아자 차카타 파하가 123 456 789 가나다 라마바 사아자 차카타 파하가 123 456 789 가나다 라마바 사아자 차카타 파하가 123 456 789 가나다 라마바 사아자 차카타 파하가 123 456 789 가나다 라마바 사아자 차카타 파하가 123 456 789 가나다 라마바 사아자 차카타 파하가 123 456 789 가나다 라마바 사아자 차카타 파하가 123 456 789 가나다 라마바 사아자 차카타 파하가 123 456 789 가나다 라마바 사아자 차카타 파하가 123 456 789 가나다 라마바 사아자 차카타 파하가 123 456 789 가나다 라마바 사아자 차카타 파하가 123 456 789 가나다 라마바 사아자 차카타 파하가 123 456 789 가나다 라마바 사아자 차카타 파하가 123 456 789 가나다 라마바 사아자 차카타 파하가 123 456 789 가나다 라마바 사아자 차카타 파하가 123 456 789 가나다 라마바 사아자 차카타 파하가 123 456 789 가na다 라마바 사아자 차카타 파하가 123 456 789 가나다 라마바 사아자 차카타 파하가 123 456 789 가나다 라마바 사아자 차카타 파하가 123 456 789 가나다 라마바 사아자 차카타 파하가 123 456 789 가나다 라마바 사아자 차카타 파하가 123 456 789 가나다 라마바 사아자 차카타 파하가 123 456 789 가나다 라마바 사아자 차카타 파하가 123 456 789 가나다 라마바 사아자 차카타 파하가 123 456 789 가나다 라마바 사아자 차카타 파하가 123 456 789 가나다 라마바 사아자 차카타 파하가 123 456 789 가나다 라마바 사아자 차카타 파하가 123 456 789 가나다 라마바 사아자 차카타 파하가 123 456 789 가나다 라마바 사아자 차카타 파하가 123 456 789 가나다 라마바 사아자 차카타 파하가 123 456 789"
      }
    }
  }
  ```
  
  - 상세 원인
    - Elasticsearch는 검색어가 들어오면 설정된 분석기(Analyzer)를 통해 token으로 쪼갠다.
    - synonym_graph의 경우 분해된 token에 동의어 관계를 덮어씌워 여러 갈래의 대체 경로(Path)를 만든다.
    - 이 과정을 통해 검색어는 단순한 단어 리스트가 아니라, 수많은 갈래가 복잡하게 얽힌 형태의 '토큰 그래프(DAG)'로 변환되는데, 검색어가 길어질수록 이 그래프의 길이와 복잡도는 기하급수적으로 증가한다.
    - Lucene은 이렇게 만들어진 복잡한 토큰 그래프를 해석하기 위해 내부적으로 `articulationPointsRecurse`라는 그래프 분석 함수를 실행하는데, 이 함수는 그래프의 주요 관절점(Node)을 찾기 위해 DFS를 사용한다.
    - JVM은 각 스레드마다 메서드 호출 정보를 저장하는 스택 메모리 공간(`-Xss`, 기본 1MB)을 할당하는데, 동의어가 많이 포함된 긴 쿼리가 들어오면서 그래프 탐색 재귀 깊이가 수천 단계 이상 깊어졌고, 스택 메모리 용량을 초과하면서 `StackOverflowError`가 발생하게 된다.
  - 기본적으로 standard tokenizer에 비해 더 많은 token을 생성하는 nori, ngram, edge-ngram 등에서 발생할 가능성이 높다.
  
  - 해결
  
    > https://github.com/apache/lucene/pull/12249/changes
  
    - Lucene 9.7(Elasticsearch 8.9) 버전에서 수정되었다.
    - 재귀의 깊이를 제한하는 방식으로 에러 발생을 방지했다.
  - 만약 버전을 올릴 수 없을 경우 snonym_graph를 사용하지 않거나, synonym이 다수 포함된 긴 text를 분석하지 못하도록 막아두는 것이 최선으로 보인다.