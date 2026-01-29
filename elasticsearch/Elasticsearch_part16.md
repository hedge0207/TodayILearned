# 검색 시스템 평가

- 사전 지식

  - 정밀도(precision)와 재현율(recall)
    - 검색에서의 정밀도와 재현율은 아래와 같이 정의된다.
    - 정밀도: 검색 결과 중 사용자의 쿼리와 실제로 관련있는 결과의 비율.
    - 재현율: 사용자의 쿼리와 관련된 문서 중에서 검색된 비율.
  - 정밀도와 재현율을 표로 나타내면 아래와 같다.
    - 정밀도는 TP / (TP + FP)로 나타낼 수 있다.
    - 재현율은 TP / (TP + FN)으로 나타낼 수 있다.

  |               | relevant | non-relevant |
  | ------------- | -------- | ------------ |
  | retrieved     | TP       | FP           |
  | not retrieved | FN       | TN           |

  - 정밀도와 재현율의 관계
    - 정밀도와 재현율을 트레이드 오프 관계로, 실제 시스템에서 둘을 동시에 높이는 것은 매우 어렵다.
    - 재현율을 높이기 위해서는 검색 범위를 늘려야 하는데, 그럴 경우 정밀도가 낮아지게 된다.
    - 반대로 정밀도를 높이기 위해서는 확실한 것만 보여주면 되는데, 이 경우 재현율이 낮아지게 된다.
    - 일반적으로 의료 시스템과 같이 FN이 위험한 분야에서는 재현율이, 쇼핑과 같이 FP에 민감한 분야에서는 정밀도가 중시된다.
  - F1 score
    - 정밀도와 재현율을 하나의 값으로 통합하여 나타내는 지표이다.
    - 두 지표의 조화평균으로 계산하므로, 한쪽이 낮지면 전체 값이 크게 감소하게 된다.
    - 예를 들어 두 지표의 값이 각각 0.1, 1.0일 경우 산술 평균을 구하면 0.55로 나쁘지 않아보이지만, 조화 평균을 통해 구하면 0.18로 점수의 불균형함을 바로 알 수 있게 되기 때문이다.

  $$
  F1 = 2 \cdot \frac{\text{Precision} \cdot \text{Recall}}{\text{Precision} + \text{Recall}}
  $$



- Hit@K(Hit at K)

  - Top-K개의 검색 결과 중 사용자의 query와 관련된 결과가 하나라도 포함되어 있는지를 나타내는 지표이다.
    - Top-K안에 관련된 결과가 몇 개나 포함되어 있었는지는 고려하지 않는다.

  $$
  \text{Hit@K}(q) = \begin{cases} 1 & \text{if } (\text{Top-K results contain at least one ground truth}) \\ 0 & \text{otherwise} \end{cases}
  $$

  

  - 일반적으로 여러 개의 테스트 쿼리에 대한 평균값을 의미하며, 아래와 같이 계산한다.
    - $q_i$는 i번째 query를 의미한다.

  $$
  \text{Average }Hit@K = \frac{1}{N} \sum_{i=1}^{N} \text{Hit@K}(q_i)
  $$



- R@K(Recall at K)

  - 전체 정답 문서 중 top-k개에 몇 개나 포함되어 있는지를 나타내는 지표이다.
    - `찾은 정답수 / 전체 정답 수`로 구한다.
    - $Relevant(q)$는 query $q$에 대한 전체 정답 수를 의미한다.
    - $|Retrieved_K(q)$는 query $q$의 검색 결과로 나온 상위 K개의 결과를 의미한다.

  $$
  \text{Recall@K}(q) = {|Retrieved_K(q) \cap Relevant(q)|\over |Relevant(q)|}
  $$

  

  - 일반적으로 여러 개의 테스트 쿼리에 대한 평균값을 의미하며, 아래와 같이 계산한다.

  $$
  \text{Average } Recall@K = \frac{1}{N} \sum_{i=1}^{N} \text{Recall@K}(q_i)
  $$

  - 맥락에 따라 Hit@K와 동일한 의미로 사용되기도 한다.

    - 만약 정답이 여러개인 맥락에서 사용될 경우, 검색 결과에 query와 관련된 결과가 몇 개나 포함되어 있는지를 고려한다.
    - 예를 들어 답이 1개 뿐이라거나, RAG에서 사용될 경우 Hit@K와 동일한 의미로 사용되어 정답이 몇 개나 포함되어 있는지는 고려하지 않고 포함 여부만을 고려한다.

    - 사실 이는 정답이 1개일 경우 필연적으로 그렇게 될 수 밖에 없는데, 정답이 1개일 경우 분모에 해당하는 전체 정답 수가 1이 되어 K개 안에 정답이 있으면 1, 없으면 0이 되기 때문이다.

  - R@K의 특징

    - K값을 늘릴수록 분자의 정답 수는 늘어나거나 유지될 뿐 절대 줄어들지 않는다.
    - 분모가 "전체 정답 수"이기 때문에, 아무리 $K$를 늘려도 시스템이 정답을 검색 범위 자체에 포함하지 못하면(색인 누락 등으로 인해) $R@K$는 1.0에 도달할 수 없다.



- P@K(Precision at K)

  - Top-K개의 검색 결과 중 정답인 문서가 몇 개나 포함되어 있는지를 나타내는 지표이다.

  $$
  Precision@K(q) = \frac{|Retrieved_K(q) \cap Relevant(q)|}{|Retrieved_K(q)|}
  $$

  - 일반적으로 여러 개의 테스트 쿼리에 대한 평균값을 의미하며, 아래와 같이 계산한다.

  $$
  $$\text{Average } P@K = \frac{1}{N} \sum_{i=1}^{N} P@K(q_i)$$
  $$

  - P@K의 특징
    - R@K와 반대로 K가 증가할수록 대체로 감소하는 경향을 보인다.
    - 또한 서비스 이탈률과의 높은 상관관계를 보이는데, 대부분의 사용자는 첫 페이지(top-20)를 보고 서비스를 판단하기 때문이다.
    - $P@K$가 높다는 것은 사용자가 검색 결과를 보고 정확하다고 느낄 확률이 높다는 의미이므로 검색 만족도를 즉각적으로 반영하는 지표이다.



- MRR(Mean Reciprocal Rank)

  - 사용자가 원하는 정답이 얼마나 높은 순위에 위치해 있는가를 평가하는 지표이다.
    - 각 쿼리의 역수 순위의 평균으로 구한다.
    - $rank_i$는 $i$번째 쿼리에 대해 시스템이 찾아낸 첫 번째 정답의 순위를 나타낸다.
    - 만약 정답을 못 찾았다면 $rank_i$는 무한대가 되어 $1 \over rank_i$은 0으로 처리된다.
    - 일반적으로 정답이 하나인 경우에 많이 사용된다.

  $$
  MRR = \frac{1}{N} \sum_{i=1}^{N} \frac{1}{rank_i}
  $$

  - 특징
    - 정답이 리스트에 여러 개 있어도, 오직 가장 상단에 있는 정답만을 점수에 반영한다.
    - 상위권 순위에 매우 민감한데, 1위와 2위의 차이는 $0.5$점($1.0 \rightarrow 0.5$)이나 나지만, 10위와 11위의 차이는 약 $0.009$점($0.1 \rightarrow 0.091$)에 불과하다.



- nDCG(Normalized Discounted Cumulative Gain)

  - 관련성이 높은 문서가 상단에 올수록 가중치를 주되, 정답이 여러 개인 경우의 순서까지 평가하기 위한 지표이다.
    - 정답이 여러 개인 경우에 많이 사용된다.
  - DCG
    - $rel_i$는 i번째 결과의 관련도 점수를 의미한다.
    - $log_2(i+1)$로 나눠 뒤에 나올수록 점수를 크게 깎는다(discount).
    - 일반적으로 아래 두 공식 중 하나의 방식으로 계산한다.

  $$
  DCG_K = \sum_{i=1}^{K} \frac{rel_i}{\log_2(i+1)}\\
  DCG_K = \sum_{i=1}^{K} {2^{rel_i}-1 \over \log_2(i+1)}
  $$

  - nDCG
    - nDCG는 아래와 같이 계산한다.
    - IDCG(Ideal DCG)는 실제 정답들을 가장 이상적인 순서(점수 높은 순)로 배치했을 때의 DCG 값이다.
    - 결국 완벽한 정답 배치 대비 현재 검색 시스템은 몇 점인가를 나타낸다.

  $$
  nDCG = \frac{DCG}{IDCG}
  $$

  - IDCG를 구하는 예시
    - 예를 들어, 특정 쿼리에 대해 데이터베이스에 [3, 1, 2]와 같은 관련성 점수를 가진 정답들이 있다고 가정해 보자(점수: 3점=매우 관련, 2점=관련, 1점=약간 관련).
    - 가장 점수가 높은 순서대로 $\{3, 2, 1\}$ 순서로 배치했을 때가 IDCG이므로 IDCG 값은 아래와 같다.

  $$
  IDCG = \frac{3}{\log_2(2)} + \frac{2}{\log_2(3)} + \frac{1}{\log_2(4)} \approx 3 + 1.26 + 0.5 = 4.76
  $$







# Elasticsearch as GraphDB

- Elasticsearch를 GraphDB로 사용하기

  > https://www.elastic.co/search-labs/blog/rag-graph-traversal#iii)-storing-graphs-with-elastic:-how-to?

  - Elasticsearch를 제한적이지만 GraphDB로 사용할 수 있다.
    - 이는 아래와 같은 관찰에 근거한다.
    - 첫째, 사용자는 전체 그래프에 관심이 있는 것이 아니라, 자신의 질문과 관련된 작은 서브 그래프에만 관심이 있다.
    - 둘째, 헝가리 과학자 Frigyes Karinthy의 Six Degrees Principle에 따르면 모든 사람은 다른 사람과 최대 6개의 관계로 이어져 있는데, 제한된 환경에서는 훨씬 적은 수의 관계로도 모든 사람들이 연결될 수 있다.
    - 이와 같은 관찰에 따르면, 사용자의 질문과 관련된 서브 그래프로 탐색 범위를 줄일 수 있다.

  - KG를 LLM에 전달하는 4 단계
    - 사용자의 query에서 node 추출.
    - Elasticsearch를 사용하여 사용자의 질문과 관련된 subgraph 탐색.
    - Graph pruning
    - Graph를 텍스트 형식으로 선형화.




- Elasticsearch로 GraphDB 구성하기

  - Index구성
    - Node index: 각각의 node를 하나의 문서로 저장하기 위한 index.
    - Relationship index: node 사이의 관계를 하나의 문서로 저장하기 위한 index.
  - Node index 생성

  ```json
  // PUT kg_node
  {
      "mappings": {
          "properties":{
              "label": {
                  "type": "keyword"
              },
              "name": {
                  "type": "keyword"
              }
          }
      }
  }
  ```

  - Relationship index 생성

  ```json
  // PUT kg_relationship
  {
      "mappings":{
          "properties":{
              "source": {
                  "type": "keyword"
              },
              "destination": {
                  "type": "keyword"
              },
              "relationship": {
                  "type": "keyword"
              }
          }
      }
  }
  ```

  - 데이터 삽입
    - 테스트 데이터는 [MetaQA](https://github.com/yuyuz/MetaQA) 데이터셋을 사용한다.
    - 중복 node가 생성되지 않도록 node의 value를 _id 값으로 사용한다.
    - `relationship.source`에는 영화가 고정적으로 들어간다.

  ```python
  from enum import Enum
  
  from elasticsearch import Elasticsearch
  from elasticsearch.helpers import bulk
  
  
  class NodeLabel(Enum):
      YEAR = "Year"
      GENRE = "Genre"
      RATING = "Rating"
      WRITER = "Writer"
      TAG = "Tag"
      LANGUAGE = "Language"
      VOTE = "Vote"
      ACTOR = "Actor"
      DIRECTOR = "Director"
  
  
  class Relationship(Enum):
      RELEASE_YEAR = "RELEASE_YEAR"
      HAS_GENRE = "HAS_GENRE"
      HAS_IMDB_RATING = "HAS_IMDB_RATING"
      WRITTEN_BY = "WRITTEN_BY"
      HAS_TAGS = "HAS_TAGS"
      IN_LANGUAGE = "IN_LANGUAGE"
      HAS_IMDB_VOTES = "HAS_IMDB_VOTES"
      STARRED_ACTORS = "STARRED_ACTORS"
      DIRECTED_BY = "DIRECTED_BY"
  
      @property
      def target_label(self) -> NodeLabel:
          mapping = {
              Relationship.RELEASE_YEAR: NodeLabel.YEAR,
              Relationship.HAS_GENRE: NodeLabel.GENRE,
              Relationship.HAS_IMDB_RATING: NodeLabel.RATING,
              Relationship.WRITTEN_BY: NodeLabel.WRITER,
              Relationship.HAS_TAGS: NodeLabel.TAG,
              Relationship.IN_LANGUAGE: NodeLabel.LANGUAGE,
              Relationship.HAS_IMDB_VOTES: NodeLabel.VOTE,
              Relationship.STARRED_ACTORS: NodeLabel.ACTOR,
              Relationship.DIRECTED_BY: NodeLabel.DIRECTOR,
          }
          return mapping[self]
  
  
  es_client = Elasticsearch("http://localhost:9200")
  relation_index_name = "kg_relationship"
  node_index_name = "kg_node"
  
  def set_up():
      relation_index_mapping = {
          "properties":{
              "source": {
                  "type": "keyword"
              },
              "destination": {
                  "type": "keyword"
              },
              "relationship": {
                  "type": "keyword"
              }
          }
      }
  
      node_index_mapping = {
          "properties":{
              "label": {
                  "type": "keyword"
              },
              "name": {
                  "type": "keyword"
              }
          }
      }
  
      if not es_client.indices.exists(index=relation_index_name):
          es_client.indices.create(index=relation_index_name, mappings=relation_index_mapping)
      
      if not es_client.indices.exists(index=node_index_name):
          es_client.indices.create(index=node_index_name, mappings=node_index_mapping)
  
  def index():
      node_bulk = []
      relationship_bulk = []
      with open("./metaqa/kb.txt", "r") as f:
          for line in f.readlines():
              movie, rel, entity = line.rstrip().split("|")
              try:
                  rel_enum = Relationship(rel.upper())
                  target_label = rel_enum.target_label.value
              except ValueError:
                  target_label = "Entity"
              node_bulk += [
                  {
                      "_index": node_index_name,
                      "_id": movie.replace(" ", "_"),
                      "_source":{
                          "label": "Movie",
                          "name": movie
                      }
                      
                  },
                  {
                      "_index": node_index_name,
                      "_id": entity.replace(" ", "_"),
                      "_source":{
                          "label": target_label,
                          "name": entity
                      }
                  }
              ]
              relationship_bulk.append({
                  "_index": relation_index_name,
                  "_source":{
                      "source": movie,
                      "destination": entity,
                      "relationship": rel_enum.value
                  }
              })
      bulk(es_client, node_bulk)
      bulk(es_client, relationship_bulk)
  
  
  if __name__ == "__main__":
      set_up()
      index()
  ```



- 검색 테스트

  - 두 엔티티 사이의 관계를 묻는 질문은 아래 과정을 통해 처리가 가능하다.
    - 만약 query에 두 개의 entity가 있고, 두 entity 사이에 직접적인 relationship이 있으면, 탐색을 종료한다.
    - 만약 직접적인 relationship이 없으면 두 entity 각각을 탐색하면서 서로 공유하는 node가 있는지 찾는다.
    - 공유하는 node가 없을 경우 두 node와 연결된 node들의 모든 이웃을 대상으로 같은 과정을 반복한다.
    - 공유하는 node를 발견하면 탐색을 종료한다.
    - 두 개체가 6hop을 초과해 연결될 경우에는 관련성이 약하다고 보기에 반복 횟수는 3번으로 제한한다.
    - 다만, 이 과정은 두 entity 사이의 관계를 찾는 질문일 경우에만 유효한 것 처럼 보인다(?).
  - 몇 번의 검색을 반복하다보면 탐색 대상 노드의 수가 폭발적으로 증가할 것 같지만 KG의 topology로 인해, 그런 상황은 거의 발생하지 않는다.
    - KG는 일반적으로 다수의 노드와 연결된 소수의 허브 노드가 존재하고, 대부분의 노드는 소수의 이웃 노드만을 가진다.
    - 탐색 과정에서 많은 이웃 노드와 연결된 노드가 일부 탐색되더라도, 탐색의 크기(size)를 줄여 탐색하면 된다.
    - 또한 허브 노드들은 일반적으로 다른 허브 노드들과 연결되어 있기 때문에 허브와 연결된 개체들 사이의 경로를 찾는 경우 경로가 매우 짧아(즉, 홉 수가 적어) 이웃 수의 지수적 증가가 거의 발생하지 않는다.
    - 반대로 멀리 떨어져 있는(홉 수가 큰) 노드들끼리 연결해야 하는 경우에는 대부분의 경로에 cardinality가 적은 노드들이 위치하게 되어 마찬가지로 이웃 수가 지수적으로 증가하지는 않는다.
  - 질문1:  "Who directed the movie The Terminal?"
    - source에 해당하는 영화 이름과, relatiohsip이 주어진 경우.

  ```json
  // GET kg_relationship/_search
  {
    "query": {
      "bool": {
        "filter": [
          {
            "term": {
              "source": "The Terminal"
            }
          },
          {
            "term": {
              "relationship": "DIRECTED_BY"
            }
          }
        ]
      }
    }
  }
  ```

  - 질문2: "Was The Terminal directed by Steven Spielberg?"
    - source에 해당하는 영화 이름과 destination에 해당하는 감독 이름, relatiohsip이 주어진 경우.
    - 질문1에서 사용한 query를 그대로 사용해도 찾을 수 있다.

  ```json
  // GET kg_relationship/_search
  {
    "query": {
      "bool": {
        "filter": [
          {
            "term": {
              "source": "The Terminal"
            }
          },
          {
            "term": {
              "destination": "Steven Spielberg"
            }
          }
        ]
      }
    }
  }
  ```

  - 질문3. "Are there any films directed by Steven Spielberg that Tom Hanks has appeared in?"
    - 두 개의 destination, relationship이 주어지고, destination사이의 연결을 찾아야 하는 경우.

  ```json
  // GET kg_relationship/_search
  {
    "query": {
      "bool": {
        "filter": [
          {
            "term":{
              "destination": "Steven Spielberg"
            }
          },
          {
            "term": {
              "relationship": "DIRECTED_BY"
            }
          }
        ]
      }
    }
  }
  
  // response
  {
    // ...
      "hits": [
        {
          "_index": "kg_relationship",
          "_id": "3xxx-ZsBDUSVc-wAvMOq",
          "_score": 0,
          "_source": {
            "source": "1941",
            "destination": "Steven Spielberg",
            "relationship": "DIRECTED_BY"
          }
        },
        {
          "_index": "kg_relationship",
          "_id": "8Bxx-ZsBDUSVc-wAvc64",
          "_score": 0,
          "_source": {
            "source": "Catch Me If You Can",
            "destination": "Steven Spielberg",
            "relationship": "DIRECTED_BY"
          }
        },
        {
          "_index": "kg_relationship",
          "_id": "jhxx-ZsBDUSVc-wAvtqi",
          "_score": 0,
          "_source": {
            "source": "The Terminal",
            "destination": "Steven Spielberg",
            "relationship": "DIRECTED_BY"
          }
        },
        // ...
      ]
    }
  }
  
  
  // 위 응답에서 나온 결과를 source로 다시 검색을 실행한다.
  // GET kg_relationship/_search
  {
    "size": 100,
    "query": {
      "bool": {
        "filter": [
          {
            "terms": {
              "source": [
                "1941",
                "Catch Me If You Can",
                "The Terminal"
              ]
            }
          },
          {
            "term": {
              "destination": "Tom Hanks"
            }
          }
        ]
      }
    }
  }
  ```



- 3-hop: The films that share actors with the [The Harvey Girls] film were in what genres.

  - movie-actor

  ```json
  // POST kg_relationship/_search
  {
    "query": {
      "bool": {
        "filter": [
          {
            "term": {
              "relationship": "STARRED_ACTORS"
            }
          },
          {
            "term": {
              "source": "The Harvey Girls"
            }
          }
        ]
      }
    }
  }
  
  
  // response
  {
    // ...
    "hits": {
      "total": {
        "value": 4,
        "relation": "eq"
      },
      "max_score": 0,
      "hits": [
        {
          "_index": "kg_relationship",
          "_id": "qR1x-ZsBDUSVc-wA4ObS",
          "_score": 0,
          "_source": {
            "source": "The Harvey Girls",
            "destination": "Judy Garland",
            "relationship": "STARRED_ACTORS"
          }
        },
        {
          "_index": "kg_relationship",
          "_id": "qh1x-ZsBDUSVc-wA4ObS",
          "_score": 0,
          "_source": {
            "source": "The Harvey Girls",
            "destination": "Ray Bolger",
            "relationship": "STARRED_ACTORS"
          }
        },
        {
          "_index": "kg_relationship",
          "_id": "qx1x-ZsBDUSVc-wA4ObS",
          "_score": 0,
          "_source": {
            "source": "The Harvey Girls",
            "destination": "Angela Lansbury",
            "relationship": "STARRED_ACTORS"
          }
        },
        {
          "_index": "kg_relationship",
          "_id": "rB1x-ZsBDUSVc-wA4ObS",
          "_score": 0,
          "_source": {
            "source": "The Harvey Girls",
            "destination": "John Hodiak",
            "relationship": "STARRED_ACTORS"
          }
        }
      ]
    }
  }
  ```

  - actor-movie

  ```json
  // POST kg_relationship/_search
  {
    "size": 100,
    "query": {
      "bool": {
        "filter": [
          {
            "term": {
              "relationship": "STARRED_ACTORS"
            }
          },
          {
            "terms": {
              "destination": [
                "Judy Garland",
                "Ray Bolger",
                "Angela Lansbury",
                "John Hodiak"
              ]
            }
          }
        ]
      }
    }
  }
  
  // response
  {
    // ...
    "hits": {
      "total": {
        "value": 40,
        "relation": "eq"
      },
      "max_score": 0,
      "hits": [
        {
          "_index": "kg_relationship",
          "_id": "rxxx-ZsBDUSVc-wAwfRO",
          "_score": 0,
          "_source": {
            "source": "Thoroughbreds Don't Cry",
            "destination": "Judy Garland",
            "relationship": "STARRED_ACTORS"
          }
        },
        {
          "_index": "kg_relationship",
          "_id": "zxxx-ZsBDUSVc-wAwfd9",
          "_score": 0,
          "_source": {
            "source": "Babes in Toyland",
            "destination": "Ray Bolger",
            "relationship": "STARRED_ACTORS"
          }
        },
        // ...
        {
          "_index": "kg_relationship",
          "_id": "7h5x-ZsBDUSVc-wA87h_",
          "_score": 0,
          "_source": {
            "source": "The Mirror Crack'd",
            "destination": "Angela Lansbury",
            "relationship": "STARRED_ACTORS"
          }
        },
        {
          "_index": "kg_relationship",
          "_id": "bR5x-ZsBDUSVc-wA9ccm",
          "_score": 0,
          "_source": {
            "source": "The Court Jester",
            "destination": "Angela Lansbury",
            "relationship": "STARRED_ACTORS"
          }
        }
      ]
    }
  }
  ```

  - movie-genre

  ```json
  // POST kg_relationship/_search
  {
    "size": 0,
    "query": {
      "bool": {
        "filter": [
          {
            "term": {
              "relationship": "HAS_GENRE"
            }
          },
          {
            "terms": {
              "source": [
                "Thoroughbreds Don't Cry",
                "Babes in Toyland",
                "Kind Lady",
                "Easter Parade",
                // ...
              ]
            }
          }
        ]
      }
    },
    "aggs":{
      "genre_aggs":{
        "terms":{
          "field":"destination"
        }
      }
    }
  }
  
  // response
  {
    // ...
    "aggregations": {
      "genre_aggs": {
        "doc_count_error_upper_bound": 0,
        "sum_other_doc_count": 0,
        "buckets": [
          {
            "key": "Drama",
            "doc_count": 2
          },
          {
            "key": "Musical",
            "doc_count": 2
          },
          // ...
        ]
      }
    }
  }
  ```















