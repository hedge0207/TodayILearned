# Elasticsearch Vector DB

- Lexical search의 한계

  - Lexical search의 경우 비구조화된 데이터(사진, 음향 등)를 대상으로 검색을 할 수 없다.
  - Lexical search는 입력된 검색어 그 자체 혹은 그 변형(stemmping이나 유의어 등)과 일치하는 문서를 찾는다.
    - 검색어나 문서의 의미에는 집중하지 않고 검색어와 문서를 구성하는 단어들에만 집중한다.
  - 따라서 아래와 같이 의미적으로는 전혀 연관이 없지만, 일치하는 단어들이 포함된 문서들이 검색되게 된다.
    - 검색 된 결과 중 실제 사용자가 얻고자 했던 정보와 관련된 정보는 하나도 없다.
    - 또한 다의어나 동음이의어가 있을 경우에도 전혀 상관 없는 문서들이 검색되는 문제가 발생할 수 있다.

  ```json
  // PUT lexical/_bulk
  {"index":{}}
  {"text":"A german shepherd watches the house"}
  {"index":{}}
  {"text":"I'm a English teacher"}
  {"index":{}}
  {"text":"We are going on holidays to Nice"}
  
  
  // GET lexical/_search
  {
      "query": {
          "match": {
            "text": "nice german teacher"
          }
      }
  }
  
  // output
  [
      {
          "_index": "lexical",
          "_id": "H_R_dpoBQQRAfducKgqk",
          "_score": 1.1149852,
          "_source": {
              "text": "I'm a English teacher"
          }
      },
      {
          "_index": "lexical",
          "_id": "HvR_dpoBQQRAfducKgqk",
          "_score": 0.95778096,
          "_source": {
              "text": "A german shepherd watches the house"
          }
      },
      {
          "_index": "lexical",
          "_id": "IPR_dpoBQQRAfducKgqk",
          "_score": 0.89470756,
          "_source": {
              "text": "We are going on holidays to Nice"
          }
      }
  ]
  ```

  - Lexical mismatch
    - 의미는 같지만 단어가 달라서 검색이 실패하는 문제를 의미한다.
    - 동의어, 형태소의 변화, 언어 표현의 다양성 등으로 인해 발생하는 문제이다.
    - 예를 들어 "휴대폰 수리"와 "핸드폰 고치기"는 같은 의미이지만 서로 다른 단어로 구성되어 있다. 따라서 lexical search로 검색하려면 추가적인 작업이 필요하다.



- 거리 기반 유사도
  - L1 distance(Manhattan distance)
    - 아래와 같이 두 벡터 사이의 거리를 계산한다.
    - $d(x,y)=∑^n_{i=1}∣x_i−y_i∣$
    - 예를 들어 두 벡터 x, y가 각각 (1, 2), (4, 3)이라면 두 벡터의 L1 distance는 $|1-4|+|2-3|$으로 계산하여 4가 된다.
  - L2 distance(Euclidean distance)
    - L1 distance와는 달리 두 벡터 사이의 직선 거리를 아래와 같이 계산한다.
    - $d(x,y)=\sqrt{∑^n_{i=1}(x_i−y_i)^2}$
    - 예를 들어 두 벡터 x, y가 각각 (1, 2), (4, 3)이라면 두 벡터의 L2 distance는 $\sqrt{(1−4)^2+(2−3)^2}$가 되어 3.16이 된다.
  - Linf distance(L-infinity distance)
    - 두 벡터의 차이 중 큰 값을 거리로 계산한다.
    - $d(x,y)=max_i|x_i−y_i|$
    - 예를 들어 두 벡터 x, y가 각각 (1, 2), (4, 3)이라면 두 벡터의  L infinity distance는 $max(|1-4|, [2-3])$으로 3이 된다.



- 각도 기반 유사도
  - Cosine similarity
    - 두 벡터 사이의 각도를 계산한다.
    - $s_{cos}(x,y)={x⋅y \over |x|\times|y|}$
    - Cosine similarity는 항상 -1과 1을 포함하는 그 사이의 값을 가지며, -1에 가까울수록 유사하지 않음을, 0이면 유사도가 없음을, 1에 가까울수록 유사함을 나타낸다.
    - 예를 들어 두 벡터 x, y가 각각 (1, 2), (4, 3)이라면 두 벡터의 cosine similarity는 $(1⋅4)+(2⋅3)\over (1^2+2^2)^{1/2}+(4^2+3^2)^{1/2}$이 되어 0.894427가 된다.
    - 이를 각도로 표현하면 약 $26^\circ$이다.
  - Dot product similarity(scalar or inner product)
    - Cosine similarity는 두 벡터 사이의 각도만 계산할 뿐 magnitude(length, 원점으로부터 벡터까지의 거리)를 고려하지 않는다는 문제가 있다.
    - 즉 두 벡터 (1, 1)과 (10000, 10000)은 다른 magnitude를 갖고 있는데, 각도가 완전히 같으므로 유사하다고 판단하게 된다.
    - 반면 dot product는 각도와 함께 magnitude도 계산에 포함시킨다.
    - $s_{dot}=|x|\times|y|\times cos\ a$
    - 예를 들어 두 벡터 x, y가 각각 (1, 2), (4, 3)이라면 두 벡터의 dot product similarity는 $((1^2+2^2)^{1/2}+(4^2+3^2)^{1/2})⋅cos(26^\circ)$가 되어 약 10이다.
    - 주의할 점은 모든 벡터를 정규화(각 벡터의 길이를 1로)한다면 dot product similarity와 cosine similarity는 완전히 동일해진다는 것이다($|x| |y| = 1$이기 때문).
    - 벡터를 정규화하면 벡터의 magnitude는 무의미해지고 각도만으로 유사도 계산이 가능해지며 이를 통해 색인이나 검색 속도를 향상시킬 수 있다.
    - 최신 embedding model들은 대부분 정규화를 전제로 설계되어 출력 자체를  L2-normalized 형태로 갖게 한다.



- Magnitude와 normalization
  - Magnitude는 원점과 vector 사이의 거리를 의미한다.
    - 예를 들어 5차원 벡터 A가 (2,1,3,6,2)일 때 magnitude는 아래와 같이 계산한다.
    - $∥A∥=\sqrt{2^2+1^2+3^2+6^2+2^2}=\sqrt{4+1+9+36+4}=\sqrt{54}≈7.348$
  - 정규화는 벡터의 magnitude를 1로 만드는 과정을 의미한다.
    - 벡터의 각 요소들을 벡터의  magnitude로 나누면 된다.
    - $Anormalized={A\over∥A∥}=({2\over7.348},{1\over7.348},{3\over7.348},{6\over7.348},{2\over7.348})≈(0.272,0.136,0.408,0.817,0.272)$
    - 이 정규화된 벡터의 길이를 다시 계산하면 아래와 같이 1이 된다.
    - $\sqrt{0.272^2+0.136^2+0.408^2+0.817^2+0.272^2}=0.999...≈1$







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



