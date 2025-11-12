# kNN search

- Elasticsearch의 유사도 검색

  - Elasticsearch는 7.3부터 `dense_vector` field를 지원한다.

    - 마찬가지로 7.3부터 `script_score` query에서 `cosine_similarity` 함수를 지원한다.

  - 8.0부터 `_search` API에서 `kNN` search를 지원한다.

  - 8.0부터 ANN(Approximate Nearest Neighbor) 검색을 지원한다.

    - 저차원 vector에서 kNN에 활용하기 위한 KD-trees라는 자료구조가 있다.
    - Elasticsearch에는 지리 data나 숫자 data를 처리하기 위해 이미 KD-trees가 내장되어 있다.

    - 그러나, 현대의 text 또는 image embedding model들은 대부분 100~1000 사이, 혹은 그 이상의 고차원 vector를 생성한다.
    - 이렇게 큰 차원에서는 가까운 이웃을 찾는 것이 쉽지 않다.

  - ANN
    - Approximate Nearest Neighbor algorithm은 Nearest Neighbor algorithm에서 정확도를 희생하여 속도를 개선시킨 algorithm이다.
    - 항상 정확한 k개의 가장 가까운 vector를 반환하지는 않는다.
    - 그러나 효율적으로 동작하며, 성능을 유지하면서 거대한 data를 scaling할 수 있게 해준다.
    - ANN algorithm은 학술적으로 매우 활발히 연구되고 있어 많은 algorithm들이 나와 있다.
    - 이 algorithm들은 일반적으로 검색 속도와 구현의 복잡성 그리고 색인 속도에서 서로 다른 trade-off를 가지고 있다.
    - Elasticsearch에서는 HNSW(Hierarchical Navigable Small World graphs) algorithm을 사용한다.
    - HNSW는 여러 ANN algorithm들을 대상으로 한 benchmark에서 뛰어는 검색 성능을 보여 줬으며, 이미 많은 업계에서 널리 사용되고 있다.



- kNN
  - query로 들어온 vector값과 가장 가까운 k개의 문서를 찾아준다.
    - Elasticsearch에서 vector화 시키는 기능을 제공하지는 않기에 vector화는 다른 방법을 이용하여 진행한 뒤, vector 값을 색인해야 한다.
    - 검색 대상이 되는 vector와 검색할 vector 값은 같은 차원이어야 한다.

  - Elasticsearch는 2 종류의 kNN search method를 제공한다.
    - Exact, brute-force kNN
    - Approximate kNN(ES 8.4 기준으로 아직 preview 상태)

  - Approximate kNN의 경우 아래와 같은 장단점이 있다.
    - 검색 속도가 Exact kNN에 비해 빠르다.
    - 인덱싱 속도는 Exact kNN에 비해 느리다.
    - 정확도가 떨어질 수 있다.




- Exact kNN search

  - 정확한 결과를 보장하지만, 데이터의 크기가 커질수록 정확도가 떨어질 수 있다.

  - vector data 색인하기
    - `dense_vector` type으로 색인한다.
    - `dims` 옵션에 검색할 query와 같은 차원을 입력한다(`index` 옵션을 true로 줬을 경우 1024를 넘을 수 없고, false로 주더라도 2048을 넘을 수 없다).
    - 만일 approximate kNN을 사용할 것이 아니라면 `index` 옵션을 false로 주거나 아예 빼버리면 된다(text type과 달리 dense_vector의 index 옵션의 기본값은 false이다).

  ```json
  // PUT product-index
  {
    "mappings": {
      "properties": {
        "product-vector": {
          "type": "dense_vector",
          "dims": 5,
          "index": false
        },
        "price": {
          "type": "long"
        }
      }
    }
  }
  ```

  - vector 값을 색인한다.

  ```json
  // POST product-index/_bulk?refresh=true
  { "index": { "_id": "1" } }
  { "product-vector": [230.0, 300.33, -34.8988, 15.555, -200.0], "price": 1599 }
  { "index": { "_id": "2" } }
  { "product-vector": [-0.5, 100.0, -13.0, 14.8, -156.0], "price": 799 }
  { "index": { "_id": "3" } }
  { "product-vector": [0.5, 111.3, -13.0, 14.8, -156.0], "price": 1099 }
  ```

  - `script_score` query를 사용하여 검색한다.

  ```json
  POST product-index/_search
  {
    "query": {
      "script_score": {
        "query" : {
          "bool" : {
            "filter" : {
              "range" : {
                "price" : {
                  "gte": 1000
                }
              }
            }
          }
        },
        "script": {
          "source": "cosineSimilarity(params.queryVector, 'product-vector') + 1.0",
          "params": {
            "queryVector": [-0.5, 90.0, -10, 14.8, -156.0]
          }
        }
      }
    }
  }
  ```



- Approximate kNN
  - `dense_vector` 필드를 대상으로 kNN search를 수행한다.
    - 색인시에 `dense_vector` 필드의  `index` prameter를 true로 줘야 한다(ES 8부터 지원).
    - 또한 `similarity` parameter에도 값을 설정해야 한다.
  - `similarity` parameter
    - 유사도 판별의 기준을 설정하는 것으로, `l2_norm`, `dot_product`, `cosine` 중 하나를 선택해야한다.
    - 공식문서에서는 `cosine` 보다 `dot_product`를 사용하는 것을 권장한다.
    - `dot_product`를 사용하려면, 모든 vector의 길이가 1로 정규화되어야 한다.
    - 따라서 `dot_product`를 사용할 경우 vector의 길이를 계산하는 연산을 수행할 필요가 없기에, 성능상 `cosine` 보다 낫다.



