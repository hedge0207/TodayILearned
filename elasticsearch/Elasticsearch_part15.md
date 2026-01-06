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



- Vector search algorithm
  - Linear search
    - 입력으로 들어온 벡터와 DB에 저장된 모든 벡터를 비교하는 방식이다.
    - 모든 벡터를 비교하기에 항상 가장 유사한 벡터를 찾을 수 있다는 장점이 있다.
    - 소규모 데이터에서는 큰 문제가 없지만, 벡터의 차원과 개수가 커질수록 성능이 급격히 저하된다(시간복잡도는 O(N)).
    - 유사도의 판단 기준은 거리나 각도 등 무엇이든 될 수 있다.
  - K-Dimensional trees(KD tree)
    - 이진 탐색 트리를 k차원 공간으로 일반화한 구조이다.
    - k 차원 공간상에 점들을 저장하여 공간을 점차 더 작은 오른쪽과 왼쪽 서브 트리로 반복적으로 이등분(bisect)하는 방식으로 동작한다.
    - 검색 시에는 알고리즘이 k개의 근접 이웃을 찾을 때 까지 쿼리 벡터 주변의 일부 트리 가지 들만 방문하면 된다.
    - 가장 큰 이점은 쿼리 벡터 주변의 트리 가지들에만 집중하면 된다는 점이다.
    - 반면에 차원의 수가 증가할수록 더 많은 가지를 방문해야 하므로 효율성이 감소하게 되는데 이를 차원의 저주라 부른다.
  - Inverted file index(IVF)
    - 방식은 공간을 분할하는 또 다른 알고리즘으로, 서로 가까운 벡터들을 각 벡터들이 공유하는 중심점에 할당하는 구조이다.
    - 2차원 공간에서 이를 가장 직관적으로 시각화할 수 있는 방법이 보르노이 다이어그램이다.
    - 공간 내의 모든 임베딩 벡터들은 가장 가까운 중심점을 가진 클러스터에 할당된다.
    - 검색할 때는, 알고리즘이 먼저 쿼리 벡터에 가장 가까운 중심점을 찾아 해당 중심점이 속한 클러스터를 우선적으로 탐색하고, 필요하다면 주변 클러스터들도 함께 살펴보면서 가장 근접한 이웃(nearest neighbors) 을 찾아낸다.
    - 이 방식 역시 KD tree와 마찬가지로 차원의 저주로부터 자유롭지 못하다.
    - 차원이 증가할수록 공간의 부피가 차원 수와 함께 급격히 커져서 데이터가 전체적으로 희소(sparse) 해지고, 정확한 결과를 얻기 위해 필요한 데이터의 양이 지수적으로 증가하게 된다.
  - Quantization
    - 압축(compression) 기반의 접근 방식으로, 임베딩 벡터의 정밀도(precision) 를 낮춤으로써 데이터베이스의 전체 크기를 줄이는 방법이다.
    - 이것은 스칼라 양자화(Scalar Quantization, SQ)를 통해 구현할 수 있다.
    - 즉, 부동소수점(floating-point) 형태의 벡터 값을 정수(integer) 값으로 변환한다.
    - 이를 통해 데이터베이스 크기가 약 8배 감소하며, 메모리 사용량도 줄고, 검색 시 벡터 간 거리 계산 속도도 빨라진다.
    - 또 다른 방법으로는 프로덕트 양자화(Product Quantization, PQ)가 있다.
    - 이 방식은 먼저 전체 벡터 공간을 더 낮은 차원의 하위 공간(subspace) 들로 나눈 뒤, 각 하위 공간 안에서 서로 가까운 벡터들을 군집화(clustering) 하는 방법이다(이는 k-means 알고리즘과 유사한 원리로 작동한다).
    - 주의할 점은, 양자화(quantization)는 차원 축소(dimensionality reduction)와는 다르다는 것이다.
    - 차원 축소는 벡터의 차원 수 자체를 줄이는 반면, 양자화는 각 차원의 값의 정밀도(값의 표현 범위)를 줄이는 방식이다.
  - Hierarchical Navigable Small Worlds(HNSW)
    - 매우 효율적이고 널리 사용되는 그래프 기반의 multi layer 구조이다.
    - 가장 상위 계층(top layer)에서는, 서로 가장 멀리 떨어진(즉, 유사성이 가장 낮은) 소수의 벡터들만 연결된 그래프를 볼 수 있다. 즉, 가장 긴 연결(link)을 가진 벡터들 간의 그래프이다.
    - 아래쪽 계층으로 내려갈수록 더 많은 벡터들이 추가되고, 그래프는 점점 조밀해지며, 벡터들 간의 거리가 점점 가까워진다.
    - 가장 하위 계층(lowest layer)에서는 모든 벡터들이 포함되어 있으며, 가장 유사한 벡터들끼리 서로 인접해 있다.
    - 검색 시에는, 알고리즘이 가장 상위 계층의 임의의 시작점(entry point)에서 출발하며, 그곳에서 쿼리 벡터(회색 점)에 가장 가까운 벡터를 찾은 다음, 그 결과를 바탕으로 한 단계 아래 계층으로 이동한다.
    - 이 과정을 반복하며, 이전 계층에서 찾은 벡터를 출발점으로 삼아 다시 가장 가까운 벡터를 탐색한다.
    - 이 과정을 계층을 하나씩 내려가며 반복하다 보면, 마지막으로 가장 하위 계층에 도달했을 때 쿼리 벡터와 가장 가까운 이웃(nearest neighbor)을 찾게 된다.
  - Locality-sensitive hashing (LSH)
    - 앞의 다른 방식들과 마찬가지로, 검색 속도를 높이기 위해 검색 공간을 급격히 줄이는 방식이다. 
    - 이 기법에서는 임베딩 벡터를 해시 값으로 변환하되, 벡터 간의 유사도 정보를 유지하도록 설계된다.
    - 그 결과, 검색 공간은 탐색이 필요한 그래프나 트리가 아니라 단순히 조회할 수 있는 해시 테이블 형태로 바뀐다.
    - 해시 기반 방법의 주요 장점은, 임의의 (매우 많은) 차원을 가진 벡터라도 고정된 크기의 해시로 매핑할 수 있다는 점이며, 이 덕분에 검색 속도를 크게 높이면서도 정밀도를 크게 희생하지 않을 수 있다.
    - 일반적으로 데이터를 해싱하는 방법은 매우 다양하며, 임베딩 벡터를 해싱하는 방식 또한 여러 가지가 있다.
    - 일반적인 해싱 기법은 비슷한 데이터 간의 해시 충돌을 최소화하려고 하지만, LSH의 주요 목표는 그와 정반대이다.
    - 즉, 유사한 데이터들이 높은 확률로 같은 버킷(bucket)에 들어가도록 해시 충돌을 최대화하는 것이 목적이다.
    - 이렇게 하면, 다차원 공간에서 서로 가까운 임베딩 벡터들이 동일한 크기의 해시 값으로 변환되어 같은 버킷에 저장된다.
    - LSH는 이러한 벡터들 간의 근접성을 유지할 수 있게 해주므로, 데이터 클러스터링이나 최근접 이웃 검색(Nearest Neighbor Search)에 매우 유용하게 사용된다.
    - 주요 계산 작업은 인덱싱 단계에서 이루어지는데, 이때 모든 벡터에 대한 해시를 미리 계산해 두고, 검색 단계에서는 쿼리 벡터를 해싱하여 가장 가까운 임베딩 벡터들이 포함된 버킷을 찾기만 하면 된다.
    - 후보 버킷을 찾은 뒤에는, 쿼리 벡터와 실제로 가장 가까운 벡터들을 식별하기 위해 보통 한 번 더 세밀한 비교 과정을 수행한다.



- Vector quantization

  - 고차원 벡터를 더 작은 차원으로 압축하여 메모리 사용량을 줄이는 기법이다.

    - 벡터를 손실 압축하므로 정확도는 떨어질 수 있다.

  - Elasticsearch에서는 `dense_vector` type을 저장할 때 quantization 방식을 선택할 수 있다.

    - 이를 통해 보다 적은 메모리를 사용하여 검색이 가능하다.

    - 다만, 정확도가 떨어지는 것은 감수해야하며, float type일 때만 가능하다.
    - 또한 quantized된 벡터와 원본 벡터를 함께 저장하기에 추가적인 디스크 공간을 필요로한다.
    - 예를 들어 40GB의 벡터를 int8로 quantization하여 저장할 경우 40GB의 원본 벡터와 10GB의 quantized된 벡터가 함께 저장되어 50GB의 용량이 필요하다.
    - 그러나 검색에 필요한 메모리는 10GB로 감소한다.
    - 384 차원 이상의 벡터를  float element_type으로 저장할 때는 quantization을 수행하는 것이 강하게 권장된다.

  - Quantization된 vector 값으로 검색하는 방법

    - Query vector가 입력되면 query vector 역시 동일한 방식으로 quantization을 수행한다.
    - 그 후 quantization된 query vector와 기존에 이미 quantization되어 있던 vector들 사이의 유사도를 비교한다.

  - Elasticsearch에서 제공하는 방식은 아래와 같다.

    - `int8`: 각 차원의 vector를 1-byte integer로 quantization한다. 메모리 사용량을 75% 가량 줄일 수 있으며, 약간의 정확도 손실이 있다. 이 방식을 사용하려면 모든 벡터의 차원이 짝수여야 한다.
    - `int4`: 각 차원의 vector를 half-byte interger로 quantization한다. 메모리 사용량을 87% 가량 줄일 수 있으며, 정확도 손실이 있다.
    - `bbq`(Better binary quantization): 각 차원의 vector를 single bit로 Better binary quantization quantization한다. 메모리 사용량을 96%가량 줄일 수 있으며, 큰 정확도 손실이 있다. 검색 단계에서, 더 많은 후보를 가져오거나(oversampling), re-ranking을 통해 정확도 손실을 완호화할 수 있다. 이 방식은 벡터의 차원이 64 이상일 때만 사용할 수 있다.

  - `bbq` 사용시 정확도 손실 보완 방법.

    - 예를 들어 다른 방식에서 top 10개의 후보만 추렸다면,  `bbq`를 통해서는 top 100개의 후보를 추린다.
    - 그 이후 후보군의 원본 vector(quantization을 거치지 않은 vector)를 대상으로  reranking을 수행한다.
    - 즉 낮은 정확도로 빠르게 검색하고, 이후에 높은 정확도를 위해 재평가하는 방식으로 `bbq` 방식의 낮은 정확도를 보완할 수 있다.

  - Quantization과 outlier

    - Quantization을 수행할 때는 outlier들을 적절히 처리해줘야한다.
    - 예를 들어 아래에서 살펴볼 scalar quantization의 경우 전체 범위를 한정된 수의 버킷에 매핑하는 과정을 거친다.
    - 이 때 각 벡터값들이 여러 버킷에 고르게 분포되어야 quantization이 잘 됐다고 할 수 있다.
    - 만약 벡터 값을 int8로 quantization한다고 가정해보자.
    - 약 95%의 값이 [-1, 1]사이에 있는데, 1% 정도의 아주 일부의 값이 -10이나 12 같은 값일 경우 이 상태에서 quantization을 진행할 때 단순히 min/max 기반으로 범위를 잡으면 전체 범위는 [-10, 12]가 되어 범위는 22가 된다.
    - 이를 int8로 표현할 수 있는 256개의 버킷으로 나누면 한 버킷 당 폭은 22/256이 되어 0.086 정도가 된다.
    - 문제는 앞서 말했듯 95%의 데이터는 [-1, 1]구간에 몰려 있는데, 이 범위는 2로 이 안에서 쓸 수 있는 버킷의 수는 2/0.086이 되어 23개 정도 뿐이다.
    - 즉 진짜 중요한 값들이 모여있는 구간에 버킷을 너무 적게 쓰게 되는 문제가 발생한다.
    - 이처럼 outlier를 처리하지 않을 경우 outlier를 커버하기 위해 전체 스케일이 지나치게 크게 잡히는 문제가 발생한다.
    - 만약 가운데 95%만 사용해서 quantization을 진행한다면 범위는 [-1, 1]이 되어 2가 되고, 버킷 당 폿은 2/256이되어 0.078이 된다.
    - 이제 [-1, 1]이 거의 전 범위에 속하므로 이 구간 안에 256개의 버킷을 거의 다 쓸 수 있고, 0.078간격으로 정밀하게 표현 가능해진다.

  - Elasticsearch에서는 outlier처리를 위해  `dense_vector` type의 `index_option`에 `confidence_interval` 옵션을 제공한다.



- Scalar quantization
  - 벡터의 각 차원 값을 더 작은 데이터 타입의 구간으로 나누어 담는 방식이다.
    - 즉 고차원 벡터의 각 스칼라 값(한 요소)을 개별적으로 더 낮은 비트 수로 근사(quantize) 하여 저장 공간을 줄이는 기법이다.
    - 예를 들어 float32로 벡터가 있다고 가정해보자.
    - 1차원 값 한 개의 크기는 4bytes이고, 768차원이라면 하나의 벡터는 3072 bytes(약 3KB)가 되고, 문서가 100만개 있다면 전체 크기는 3GB가 된다.
    - 만약 1차원 값 한 개의 크기를 1bytes로 줄이면 하나의 벡터는 768bytes가 되고, 문서가 100만개 있어도 약 700MB 정도면 된다.
  - 동작 방식
    - 예를 들어  -1에서 1 사이의 값을 갖는 벡터를 -127에서 127까지 표현할 수 있는 int8로 quantization한다고 가정해보자.
    - -127~127까지의 각 구간을 하나의 버킷으로 보고 각 벡터 값을 버킷에 담는 방식으로 quantization을 수행한다.
  - 정보의 손실
    - Quantization을 거치면서 정보의 손실이 발생할 수 있다.
    - 단, 차원의 수가 클수록 quantization으로 인해 발생하는 오차는 줄어들게 된다.
    - 따라서 정확도와 성능 사이에서 적절한 타협을 해야한다.
  - 계산 방식
    - 값을 버킷에 매핑하는 공식은 아래와 같다.
    - $max$는 범위의 최댓값, $min$은 범위의 최솟값, $N$은 버킷의 총 개수(int8의 경우 256)를 의미한다.
    - $bucket(x)= ⌊{x-min \over max-min} \times N⌋$
  - Outlier가 버킷 배분에 미치는 악영향
    - 예를 들어 outlier를 포함한 [-10, -0.9, -0.5, -0.2, 0.1, 0.4, 0.7, 0.9, 5]와 같은 vector를  int8로 quantization하려고 한다.
    - 이 때, 각 값의 버킷은 [0, 9, 10, 10, 10, 11, 11, 11, 15]가 된다.
    - -10, 5 같은 outlier들 때문에 버킷이 고르게 분포되지 못 했다.
    - 만약 outlier를 제한 [-0.9, -0.5, -0.2, 0.1, 0.4, 0.7, 0.9]를 대상으로 int8로 quantization하면 각 값의 버킷은 [0, 3, 6, 8, 12, 13, 15]로 여러 버킷에 고르게 분포되게 된다.



- `dense_vector`의 `similarity`에 설정할 수 있는 값들. 
  - `dot_product`
    - `element_type`이 float일 경우 magnitude가 1(unit-length)로 정규화되어 있어야 색인이 가능하며 query vector 역시 unit-length여야한다.
    - `element_type`이 byte일 경우 모든 벡터의 magnitude가 동일하지 않아도 색인이 가능하지만 결과가 부정확해질 수 있다. 
    - query vector 역시 magnitude가 동일해야 정확한 결과를 얻을 수 있다.
    - `element_type`에 따라 최종 점수를 계산하는 방식이 달라진다.

  - `cosine`
    - 색인을 수행하면서 자동으로 unit-length로 정규화를 수행한다.
    - 이를 통해 내부적으로 dot_product를 사용해 더 효율적인 방식으로 유사도를 계산할 수 있게 된다.
    - 원래의 정규화되지 않은 벡터는 스크립트를 통해 접근이 가능하다.
    - 코사인 유사도는 벡터의 magnitude가 0인 경우 정의되지 않기 때문에 이러한 벡터는 허용되지 않는다.

  - `max_inner_product`
    - MIPS를 위해 사용하는 타입이다.
    - MIPS는 Maximum Inner Product Search의 약자로 inner product가 가장 높은 vector를 찾는 검색 방식이다.
    - Vector 공간에서 가장 가까운 vector를 찾는 것이 아니라 inner product가 가장 높은 vector를 찾는 것이다.
    - `dot_product`와는 달리 벡터가 정규화 되어 있지 않아도 된다.




- 정규화된 상태에서는 dot product값과 cosine값과 max inner product 결과값이 모두 동일하다.
  - 그러나 이는 결과 값이 동일하다는 것이지 Elasticsearch의 검색 결과로 산출 되는 점수가 동일하다는 의미는 아니다.
  - 각 유사도 유형 별로 점수를 계산하는 방식이 다르기 때문이다.
    - 예를 들어 Elasticsearch에서 dot_product는 element_type이 float일 경우 `(1 + dot_product(query, vector)) / 2`와 같이 계산한다.
    - 반면에 `max_inner_product`는 `max_inner_product(query, vector) + 1`와 같이 계산된다.
    - 이 때 정규화된 상태에서는 `max_inner_product(query, vector)`의 값과 `dot_product(query, vector)`의 값이 동일하므로 이를 x로 두면 dot_product는 `(1+x)/2`가 되고, `max_inner_product`의 값은 `1+x`가 되어 Elasticsearch 계산 결과는 항상 `max_inner_product`가 `dot_product`의 두 배가 된다.
    - 마찬가지 이유료 dot_product의 element_type이 float일 경우에는 cosine과 동일한 점수가 나오지만, bytes일 경우에는 점수가 달라지게 된다.



- dense_vector 타입을 색인할 때 `_source.exclude`를 통해 색인은 되게 하되 저장은 되지 않게 하면 크기를 줄일 수 있다.

  - 경우에 따라 다르겠지만 색인시에 `_source.excludes`를 통해 vector 값을 저장하지 않으면 인덱스 크기가 상당히 감소한다.

  - flat일 경우와  hnsw일 경우의 감소 폭이 다르다.
    - hnsw일 경우 상당히 감소하지만, flat일 경우 거의 차이가 없다.



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



- 선형 탐색 구현하기

  - 아래와 같이 index를 생성하고 데이터를 삽입한다.
    - 선형 탐색은 저장된 모든  vector 값을 탐색하므로 굳이 `index`를 `true`로 설정하여 HNSW graph에 색인되게 할 필요가 없다.
    - 이 방식의 장점은 HNSW graph를 구축할 필요가 없으므로 색인 시간이 급격히 짧아진다는 것이다.
    - 다만, 검색에는 더 긴 시간이 걸릴 수 있다.

  ```json
  // PUT /my-index
  {
    "mappings": {
      "properties": {
        "price": {
          "type": "integer"
        },
        "title_vector": {
          "type": "dense_vector",
          "dims": 3,
          "index": false
        }
      }
    }
  }
  
  
  // POST my-index/_bulk
  { "index": { "_id": "1" } }
  { "title_vector": [2.2, 4.3, 1.8], "price": 23}
  { "index": { "_id": "2" } }
  { "title_vector": [3.1, 0.7, 8.2], "price": 9}
  { "index": { "_id": "3" } }
  { "title_vector": [1.4, 5.6, 3.9], "price": 124}
  { "index": { "_id": "4" } }
  { "title_vector": [1.1, 4.4, 2.9], "price": 1457}
  ```

  - 검색하기
    - 벡터가 HSNW graph 구조로 색인되지 않았으므로, painless script를 사용하여 유사도 계산을 수행해야한다.
    - 아래와 같이 `script_score.query`를 설정하면 유사도를 계산할 문서의 개수를 한정할 수 있어 검색 속도를 올릴 수 있다.

  ```json
  // POST my-index/_search
  {
    "_source": false,
    "fields": [ "price" ],
    "query": {
      "script_score": {
        "query" : {
          "bool" : {
            "filter" : {
              "range" : {
                "price" : {
                  "gte": 100
                }
              }
            }
          }
        },
        "script": {
          "source": "cosineSimilarity(params.queryVector, 'title_vector') + 1.0",
          "params": {
            "queryVector": [0.1, 3.2, 2.1]
          }
        }
      }
    }
  }
  ```



- Approximate k-NN search 구현하기

  - 이 방식이 Elasticsearch를 통해 vector search를 구현할 때 가장 많이 사용되는 방식이다.
    - Lucene이 vector 값들로 HNSW graph를 구현해야 해서 색인 시간이 조금 더 오래 걸린다.
    - 또한 검색시에도 HNSW graph를 메모리에 올려야 해서 조금 더 많은 메모리가 필요하다.
    - 그러나 선형 탐색에 비해 더 빠른 검색 속도를 기대할 수 있다.
  - 아래와 같이 index를 생성한다.
    - `index`를 true로 설정해야 knn 검색이 가능하다.
    - `index_options.type`을 `*hnsw`로 설정하여 index를 생성하고, vector값들을 색인하면, vector들은 scalar-quantized HNSW graph에 색인된다.

  ```json
  // PUT /my-index
  {
    "mappings": {
      "properties": {
        "price": {
          "type": "integer"
        },
        "title_vector": {
          "type": "dense_vector",
          "dims": 3,
          "index": true,                // 기본값
          "similarity": "cosine",       // 기본값
          "index_options": {
            "type": "int8_hnsw",        // 기본값
            "ef_construction": 128,
            "m": 24    
          }
        }
      }
    }
  }
  
  // POST my-index/_bulk
  { "index": { "_id": "1" } }
  { "title_vector": [2.2, 4.3, 1.8], "price": 23}
  { "index": { "_id": "2" } }
  { "title_vector": [3.1, 0.7, 8.2], "price": 9}
  { "index": { "_id": "3" } }
  { "title_vector": [1.4, 5.6, 3.9], "price": 124}
  { "index": { "_id": "4" } }
  { "title_vector": [1.1, 4.4, 2.9], "price": 1457}
  ```

  - 8.11까지는 아래와 같이 `query`와 같은 레벨에 `knn`을 지정하여 검색해야 했다.

  ```json
  // POST my-index/_search
  {
    "_source": false,
    "fields": [ "price" ],
    "knn": {
      "field": "title_vector",
      "query_vector": [0.1, 3.2, 2.1],
      "k": 2,
      "num_candidates": 100
    }
  }
  ```

  - 8.12부터는 hybrid 검색을 지원하기 위하여 아래와 같이 `query` 내부에서 사용할 수 있게 되었다.
    - 그러나, 만약 `query`를 굳이 사용해야 할 이유가 없을 경우에는 여전히 `query`와 같은 레벨에서 `knn`을 사용할 수 있다.
    - 주의할 점은 아래 방식의 경우 기존과 달리 `k` 매개 변수 대신 `size`를 사용한다는 점이다.
    - 또한 이 방식은 bool 쿼리(bool query)와 결합하여 하나 이상의 필터(filter)를 knn 검색 쿼리와 함께 사용함으로써 k-NN 검색 결과에 후처리(post-filtering) 를 적용할 수 있다.
    - 즉 아래 query는 먼저 `size`에 정의된 대로 3개의 가장 가까운 vector들을 찾은 후에 price가 100 미만인 값들을 제외시킨다.
    - 이러한 동작 방식은 일반적인 bool full text query와는 다른데, 일반적으로 filter가 먼저 실행되어 score를 계산해야 하는 문서 수를 줄이기 때문이다.

  ```json
  // POST my-index/_search
  {
    "size": 3,
    "_source": false,
    "fields": [
      "price"
    ],
    "query": {
      "bool": {
        "must": {
          "knn": {
            "field": "title_vector",
            "query_vector": [
              0.1,
              3.2,
              2.1
            ],
            "num_candidates": 100
          }
        },
        "filter": {
          "range": {
            "price": {
              "gte": 100
            }
          }
        }
      }
    }
  }
  ```

  - `num_candidates` parameter
    - 가까운 이웃을 찾을 가능성을 조정하는 parameter이다.
    - 큰 값을 줄 수록 검색 속도는 감소하지만 정말로 가까운 이웃을 찾을 가능성이 증가한다.
    - 각 샤드(shard)에서는 num_candidates 개수만큼의 벡터가 고려되며, 그중 상위 k개의 벡터가 coordinator node로 전달된다.
    - 코디네이터 노드는 각 샤드에서 전달된 로컬 결과들을 병합한 뒤 그 중에서 다시 전역(global) 결과 기준으로 상위 k개의 벡터를 선택하여 반환한다.
  - 여러 개의 field를 대상으로  knn 검색을 수행하는 것도 가능하다.
    - 아래와 같이 `knn`의 값을 배열 형태로 설정하면 된다.
    - 최종 점수는 각 점수의 평균으로 계산된다.

  ```json
  // POST my-index/_search
  {
    "_source": false,
    "fields": [ "price" ],
    "knn": [
      {
        "field": "title_vector",
        "query_vector": [0.1, 3.2, 2.1],
        "k": 2,
        "num_candidates": 100,
        "boost": 0.4
      },
      {
        "field": "content_vector",
        "query_vector": [0.1, 3.2, 2.1],
        "k": 5,
        "num_candidates": 100,
        "boost": 0.6
      }
    ]
  }
  ```

  - Filter 설정하기
    - Script score를 사용하는 방식과 마찬가지로 knn 검색 역시 탐색해야 하는 vector의 범위를 줄이기 위해 filter를 주는 것이 가능하다.
    - `knn.filter`는 pre-filter로 적용되고, 그 외에 `query`에 작성한 내용들은 모두 post-filter로 적용된다.
    - `knn.filter`가 지나치게 엄격할 경우(조건이 많을 경우) knn 검색이 지나치게 희소한 벡터 공간에서 실행되어 정확한 결과를 얻기 어렵다.

  ```json
  // POST my-index/_search
  {
    "_source": false,
    "fields": [ "price" ],
    "knn": {
      "field": "title_vector",
      "query_vector": [0.1, 3.2, 2.1],
      "k": 2,
      "num_candidates": 100,
      "filter" : {
        "range" : {
          "price" : {
            "gte": 100
          }
        }
      }
    }
  }
  ```

  - `similarity` parameter
    - 앞서 살펴본 것 처럼 filter를 적용할 경우 검색 속도를 증가시킬 수 있지만, 희소한 vector 차원에서 검색해야 한다는 문제가 있다.
    - 따라서 관련성이 없는 문서들이 검색되는 문제가 발생할 수 있는데, 이는  `similarity` parameter를 통해 해결할 수 있다.
    - `similarity`에는 최소 유사도 값을 설정하여, 이 이상의 유사도를 가진 vector들만 검색되도록 한다.
    - 아래 query는  filter에 해당하지 않거나, `similarity` 미만의 유사도를 가지는 문서들을 제외하고  k개의 근접 이웃을 찾는 방식으로 동작한다.
    - 만약 filter로 너무 많이 걸러지거나  `similarity`가 너무 높게 설정되어 k개의 이웃을 찾을 수 없는 경우 brute-force search를 실행한다.
    - 어떤 값을 설정해야 하는지는 vector field mapping에 어떤 유사도 지표를 사용했는지에 달려있다.
    - `l2_norm`을 사용할 경우 최대 거리를, `dot_product`, `cosine`을 사용할 경우 최소 유사도를 설정하면 된다. 

  ```json
  // POST my-index/_search
  {
    "_source": false,
    "fields": [ "price" ],
    "knn": {
      "field": "title_vector",
      "query_vector": [0.1, 3.2, 2.1],
      "k": 2,
      "num_candidates": 100,
      "similarity": 0.975,
      "filter" : {
        "range" : {
          "price" : {
            "gte": 100
          }
        }
      }
    }
  }
  ```

  -  k-NN의 제약 사항
     - 8.11 버전까지는 nested documents 내부에 위치한 vector field에 k-NN 검색을 실행할 수 없으나, 8.12 버전부터는 이 제한이 해제되었다. 다만, 이러한 중첩 k-NN 쿼리는 필터 조건을 지정하는 기능을 지원하지 않는다.
     - search_type은 항상 dfs_query_then_fetch로 설정되며, 이를 동적으로 변경하는 것은 불가능하다.
     - 교차 클러스터 검색(cross-cluster search)으로 여러 클러스터에 걸쳐 검색할 때는 `ccs_minimize_roundtrips` 옵션을 지원하지 않는다.
     - Lucene이 사용하는 HNSW 알고리즘(그리고 다른 근사 최근접 탐색 알고리즘들 역시)의 특성상 “근사(approximate)”라는 의미는 반환되는 k개의 최근접 이웃이 항상 실제 최근접 이웃과 일치하지 않을 수 있음을 뜻한다.






- Exact kNN search

  - 정확한 결과를 보장하지만, 데이터의 크기가 커질수록 정확도가 떨어질 수 있다.

  - vector data 색인하기
    - `dense_vector` type으로 색인한다.
    - `dims` 옵션에 검색할 query와 같은 차원을 입력한다(4096을 초과해서 설정할 수 없다).
    - 만일 approximate kNN을 사용할 것이 아니라면 `index` 옵션을 false로 설정하면 된다(기본값은 true).

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



- kNN search와 kNN query의 차이

  - 최상위 계층에 `knn`을 사용하여 검색하는 것을 kNN search라하고, `query`내부에 `knn`을 추가하는 것을 kNN query라 한다.

  ```json
  // knn search
  {
      "_source": false,
      "fields": [
          "price"
      ],
      "query":{
          "range":{
              "gte": 100
          }
      },
      "knn": {
          "field": "title_vector",
          "query_vector": [0.1, 3.2, 2.1],
          "k": 3,
          "num_candidates": 100,
          "filter": {
              "range": {
                  "price": {
                      "gte": 100
                  }
              }
          }
      }
  }
  
  // knn query
  {
      "size": 3,
      "_source": false,
      "fields": [
          "price"
      ],
      "query": {
          "bool": {
              "must": {
                  "knn": {
                      "field": "title_vector",
                      "query_vector": [0.1, 3.2, 2.1],
                      "num_candidates": 100
                  }
              },
              "filter": {
                  "range": {
                      "price": {
                          "gte": 100
                      }
                  }
              }
          }
      }
  }
  ```

  - kNN search는 아래 과정을 거쳐 동작한다.
    - 사용자가 Elasticsearch로 검색 요청을 보낸다.
    - Coordinator node는 data node들에 kNN search 요청을 전달한다.
    - 각 data node는 kNN search를 수행하고, 각자 top-k개의 결과를 coordinator node로 전송한다.
    - Coordinator node는 각 data node가 전송한 local top-k개를 병합하여 global top-k개의 결과를 생성한다.
    - 이후 coordinator node는 병합된 global top-k개의 결과를 `query`에 작성된 query와 함께 다시 각 data node에 전송한다.
    - Data node들은 query를 검색한 결과를 global top-k개의 결과에 추가하여 coordinator에 반환한다.
    - Coordinator는 이 결과를 병합하여 사용자에게 반환한다.
    - 따라서 위 예시의 경우 `knn.k`는 3으로 설정했으므로, `query`와 일치하는 문서의 개수가 3개 미만이라도 3개의 문서를 반환하게 된다.
    - 반대로 `knn.k`를 3으로 설정했더라도, `query`와 일치하는 문서는 전부 반환하므로 3개 이상의 문서가 반환될 수 있다.
    - 이렇게 구현된 이유는 사용자가 최소한 `k`개의 문서는 반환받을 수 있도록 하기 위함이다.
  - kNN query는 아래 과정을 거쳐 동작한다.
    - kNN query는  kNN search와 달리 `k` parameter의 기본값이  `size`로 설정된다. 일반적인 query와 마찬가지로  `size`를 통해 반환 받을 문서의 개수를 결정한다.
    - 사용자가 Elasticsearch로 검색 요청을 보낸다.
    - Coordinator node는 data node들에 query(kNN + others)를 전달한다.
    - 각 data node는 query에 따라 검색을 수행하고 size만큼의 결과를 coordinator node로 전송한다.
    - 결과를 받은 coordinator node는 각 data node가 전송한 결과를 병합하여 size만큼의 문서를 사용자에게 반환한다.
  - 예시 index 생성 및 데이터 색인

  ```json
  // PUT products
  {
      "mappings": {
          "dynamic": "strict",
          "properties": {
              "department": {
                  "type": "keyword"
              },
              "brand": {
                  "type": "keyword"
              },
              "description": {
                  "type": "text"
              },
              "embedding": {
                  "type": "dense_vector",
                  "index": true,
                  "similarity": "l2_norm"
              },
              "price": {
                  "type": "float"
              }
          }
      }
  }
  
  // POST products/_bulk?refresh=true
  {"index":{"_id":1}}
  {"department":"women","brand": "Levi's", "description":"high-rise red jeans","embedding":[1,1,1,1],"price":100}
  {"index":{"_id":2}}
  {"department":"women","brand": "Calvin Klein","description":"high-rise beautiful jeans","embedding":[1,1,1,1],"price":250}
  {"index":{"_id":3}}
  {"department":"women","brand": "Gap","description":"every day jeans","embedding":[1,1,1,1],"price":50}
  {"index":{"_id":4}}
  {"department":"women","brand": "Levi's","description":"jeans","embedding":[2,2,2,0],"price":75}
  {"index":{"_id":5}}
  {"department":"women","brand": "Levi's","description":"luxury jeans","embedding":[2,2,2,0],"price":150}
  {"index":{"_id":6}}
  {"department":"men","brand": "Levi's", "description":"jeans","embedding":[2,2,2,0],"price":50}
  {"index":{"_id":7}}
  {"department":"women","brand": "Levi's", "description":"jeans 2023","embedding":[2,2,2,0],"price":150}
  ```

  - kNN query 내부에도 filter 사용이 가능하다.

  ```json
  // GET products/_search
  {
    "size": 3,
    "query": {
      "knn": {
        "field": "embedding",
        "query_vector": [2, 2, 2, 0],
        "num_candidates": 10,
        "filter": {
          "term": {
            "department": "women"
          }
        }
      }
    }
  }
  ```

  - kNN query는 kNN search에 비해 보다 다양한 결과를 얻을 수 있다.
    - 그리고 이를 collapse나 aggregation에 사용할 수 있다.

  ```json
  // GET products/_search
  {
    "size": 3,
    "query": {
      "knn": {
        "field": "embedding",
        "query_vector": [2, 2, 2, 0],
        "num_candidates": 10,
        "filter": {
          "term": {
            "department": "women"
          }
        }
      }
    },
    "collapse": {
      "field": "brand"
    }
  }
  ```

  - 반면 위와 유사한 검색을  kNN search로 수행하면, 다른 결과가 나오게 된다.

  ```json
  // GET products/_search?size=3
  {
    "knn": {
      "field": "embedding",
      "query_vector": [2, 2, 2, 0],
      "k": 3,
      "num_candidates": 10,
      "filter": {
        "term": {
          "department": "women"
        }
      }
    },
    "collapse": {
      "field": "brand"
    }
  }
  ```

  - 이는  aggregation에서도 마찬가지다.
    - 두 검색의  aggregation 결과는 다르다.

  ```json
  // GET products/_search
  {
    "size": 0,
    "query": {
      "knn": {
        "field": "embedding",
        "query_vector": [2, 2, 2, 0],
        "num_candidates": 10,
        "filter": {
          "term": {
            "department": "women"
          }
        }
      }
    },
    "aggs": {
      "brands": {
        "terms": {
          "field": "brand"
        }
      }
    }
  }
  
  // GET products/_search
  {
    "size": 0,
    "knn": {
      "field": "embedding",
      "query_vector": [2, 2, 2, 0],
      "k": 3,
      "num_candidates": 10,
      "filter": {
        "term": {
          "department": "women"
        }
      }
    },
    "aggs": {
      "brands": {
        "terms": {
          "field": "brand"
        }
      }
    }
  }
  ```



- kNN query의 유연성

  - kNN query는 `query` 내부에 작성할 수 있다는 특성 덕분에, 매우 유연하게 사용할 수 있다.
    - 예를 들어 아래와 같이 bool query 내부에서 사용이 가능하다.

  ```json
  // GET products/_search?include_named_queries_score
  {
    "size": 3,
    "query": {
      "bool": {
        "should": [
          {
            "knn": {
              "field": "embedding",
              "query_vector": [2, 2, 2, 0],
              "num_candidates": 10,
              "_name": "knn_query"
            }
          },
          {
            "match": {
              "description": {
                "query": "luxury",
                "_name": "bm25query"
              }
            }
          }
        ]
      }
    }
  }
  ```

  - `function_score` 같은 복잡한 query 내부에서 사용하는 것도 가능하다.

  ```json
  // GET products/_search
  {
    "size": 3,
    "query": {
      "function_score": {
        "query": {
          "knn": {
            "field": "embedding",
            "query_vector": [2, 2, 2, 0],
            "num_candidates": 10,
            "_name": "knn_query"
          }
        },
        "functions": [
          {
            "filter": {
              "match": {
                "department": "men"
              }
            },
            "weight": 100
          },
          {
            "filter": {
              "match": {
                "department": "women"
              }
            },
            "weight": 50
          }
        ]
      }
    }
  }
  ```



- Lexical search의 점수와  semantic search의 점수를 결합하기
  - 두 방식은 최종 점수의 범위에 차이가 있다.
    - 일반적으로  BM25 혹은  TF-IDF로 계산하는  lexical search의 경우 점수에 상한이 없다.
    - 그러나  semantic search의 경우 일반적으로 점수에 상한이 있다(cosine 유사도의 경우 0~2)
    
  - 따라서 두 방식을 결합하기 위해서는 두 방식이 계산한 점수를 적절히 결합하여 최종 점수를 계산할 방법이 있어야한다.
    - 최종 점수를 계산하는 다양한 방식이 있지만 아래 두 방식이 가장 대표적이다.
    - Convex Combination(CC, Linear Combination)
    - Reciprocal Rank Fusion(RRF)
    
  - Convex combination
    - 정규화된 lexical search의 결과 점수와 semantic search의 결과 점수에 각각 가중치 α, β(0≤α,β)를 주는 방식이다.
    - $score_{convex}(doc)=\alpha\times score_{lex}(doc)+\beta \times score_{sem}(doc)$
    - CC는 렉시컬 점수와 시맨틱 점수의 가중 평균으로 볼 수 있다.
    - 가중치가 0과 1 사이이면 해당 쿼리의 영향을 감소시키는 역할을 하며, 가중치가 1보다 크면 해당 쿼리의 영향을 강화시키는 데 사용된다.
    
  - RRF(Reciprocal Rank Fusion)
    
    - 여러 정보 검색(IR) 시스템의 문서 순위를 결합하는 방법이다.
    - RRF는 CC와 달리 점수 보정이나 정규화를 전혀 필요로 하지 않는다.
    - 개별 시스템 중 어느 하나보다도 일관되게 더 뛰어난 결과를 제공하며 기존의 표준 방식인 Condorcet Fuse보다도 더 좋은 성능을 보인다.
    - 매우 단순한 점수 계산 공식을 사용하여 문서들을 정렬한다.
    - 단순히 결과 집합에서의 순위(rank)에 따라 문서에 점수를 부여한다.
    - 순위를 매겨야 하는 문서들의 집합 D와 각 문서에 대해 1..|D| 범위의 순위를 매기는 여러 개의 순위 목록 집합 R이 주어졌을 때, 아래와 같이 계산한다.
    - 이때, k는 일반적으로 60을 사용하며, 특정 시스템이 비정상적으로 높은 순위를 부여했을 때 그 영향이 과도하게 커지는 것을 완화하는 역할을 한다.
    
    $$
    RRFscore(d \in D) = \sum_{r \in R}{1\over k+(d)}
    $$
    
  - Elasticsearch는 위 두 가지 방식을 모두 지원한다.



- 각 방식의 점수 계산 예시

  - 예를 들어 세 개의 문서를 대상으로 아래와 같은 점수와 순위가 나왔다고 가정해보자.

  | 문서 | lexical score | lexical rank | semantic score | semantic rank |
  | ---- | ------------- | ------------ | -------------- | ------------- |
  | A    | 0.9           | 1            | 0.2            | 3             |
  | B    | 0.4           | 3            | 0.8            | 1             |
  | C    | 0.6           | 2            | 0.6            | 2             |

  - CC

    - α=0.5, β=0.5라고 가정한다.
    - 문서 A의 최종 점수는 $0.5×0.9 + 0.5×0.2 = 0.55$
    - 문서 B의 최종 점수는 $0.5×0.4 + 0.5×0.8 = 0.60$
    - 문서 C의 최종 점수는 $0.5×0.6 + 0.5×0.6 = 0.60$

  - RRF

    - k=60이라고 가정한다.

    - 문서 A의 최종 점수는 $1/(60+1) + 1/(60+3) ≈ 0.01639 + 0.01626 = 0.03265$
    - 문서 B의 최종 점수는 $1/(60+3) + 1/(60+1) ≈ 0.01626 + 0.01639 = 0.03265$
    - 문서  C의 최종 점수는 $1/(60+2) + 1/(60+2) ≈ 0.01639 + 0.01639 = 0.03278$



- `retriever`

  - 여러 단계에 걸쳐서 Elasticsearch에 요청해야 했던 검색을 일종의 파이프라인처럼 구성하여 한 번의 요청으로 처리할 수 있게 해주는 기능이다.
    - Elasticsearch 8.14부터 추가되어 8.16부터 일반적으로 사용이 가능한 기능이다.
    - Lexical search와 semantic search를 결합하기 위해서도 사용할 수 있다.
    - linear, rule, RRF, text similarity re-ranker는 commercial license(Enterprise)가 필요하다.
  - Dense vector를 대상으로 hybrid 검색 실행하기
    - 아래와 같이 lexical query(예시의 경우  `match`)를 `knn` query와 함께 사용하면 된다.
    - `rrf` retriever를 사용하여 표준  retriever로 실행한 lexical 검색 결과 점수와 semantic 검색 결과 점수를 결합한다.
    - 아래 query는 먼저 전역 범위에서 vector 기반 상위 5개 문서를 먼저 검색하고, 그 결과를 lexical 결과와 결합한 최종적으로 가장 적합한 상위 10개 문서를 반환한다.

  ```json
  // POST my-index/_search
  {
    "size": 10,
    "_source": false,
    "fields": [ "price" ],
    "retriever": {
      "rrf": { 
        "retrievers": [ 
          {
            "standard": { 
              "query": {
                "match": {
                  "text-field": "fox"
                }
              }
            }
          },
          {
            "knn": { 
              "field": "title_vector",
              "query_vector": [0.1, 3.2, 2.1],
              "k": 5,
              "num_candidates": 100
            }
          }
        ]
      }
    }
  }
  ```

  - RRF의 랭킹에 영향을 미치는 두 개의 parameter를 설정 가능하다.
    - `rank_constant`,  `rank_window_size`(기본값은 `size`)
    - 아래 쿼리는 앞선 예와 동일한 방식으로 동작하지만, 다른 점은 vector 쿼리와 lexical 쿼리에서 가져오는 문서 수가 단순히 10개가 아니라 `rank_window_size` 값만큼 더 많이 조회된다는 점이다.
    - 이렇게 조회된 문서들을 RRF 방식으로 랭킹한 뒤, 최종적으로 1위부터 `size`까지의 상위 문서들이 결과로 반환된다.

  ```json
  // POST my-index/_search
  {
    "size": 10,
    "_source": false,
    "fields": [ "price" ],
    "retriever": {
      "rrf": { 
        "retrievers": [ 
          // ...
        ],
        "rank_constant": 60, 
        "rank_window_size": 100
      }
    }
  }
  ```



- kNN 검색에 필요한 메모리
  - kNN 검색을 수행할 경우, data node에 충분한 메모리가 필요하다.
    - HNSW 검색을 수행하기 위해서는 vector와 HNSW 그래프가 메모레이 올라가야 한다.
    - 필요한 메모리의 양은, vector의 개수, vector의 차원, quantization 여부 `index_options.m`값 등에 의해 결정된다.
    - 주의할 점은 여기서 말하는 memory는 java heap memory와는 분리된 filesystem cache에 사용되는 메모리라는 점이다.
  - Vector를 위해 필요한 메모리의 양은 아래와 같이 구할 수 있다.
    - `element_type: float`: `num_vectors * num_dimensions * 4`
    - `element_type: float` with `quantization: int8`: `num_vectors * (num_dimensions + 4)`
    - `element_type: float` with `quantization: int4`: `num_vectors * (num_dimensions/2 + 4)`
    - `element_type: float` with `quantization: bbq`: `num_vectors * (num_dimensions/8 + 12)`
    - `element_type: byte`: `num_vectors * num_dimensions`
    - `element_type: bit`: `num_vectors * (num_dimensions/8)`
  - HNSW 그래프를 위해 필요한 메모리의 양은 아래와 같이 구할 수 있다.
    - `num_vectors * 4 * HNSW.m`. 
    - `HNSW.m` 의 기본값은 16이므로, 기본적으로 `num_vectors * 4 * 16` 만큼의 memory가 필요하다.
  - Vector와 HNSW 그래프 외에도 RAM이 필요한 경우가 있으므로 어느 정도의 buffer도 확보해야 한다.
    - 예를 들어 text, numerics field들의 경우 file system cache를 사용하므로, 이들을 위한 공간도 남겨둬야한다.



- Warm up filesystem cache
  - Elasticsearch를 실행하던 장치가 재실행될 경우, filesystem cache가 비워지게 된다.
    - 이 경우 index의 데이터 다시 메모리에 올려 검색 속도를 증가시키기 까지 일정 시간이 소요된다.
  - Elasticsearch에서는 `index.store.preload`설정을 통해 특정 확장자를 가진 파일들을 보다 빠르게 memory에 load할 수 있는 기능을 제공한다.
    - `elasticsearch.yml`에 설정하여 전체 index에 적용되도록 할 수도 있고, 개별 index 마다 적용되도록 할 수도 있다.
    - 다만, 너무 많은 파일을 읽게 될 경우 filesystem cache가 모든 데이터를 hold하지 못하게 되어 오히려 검색 속도가 하락할 수 있으므로 주의해야한다.
  - 빠른 kNN 검색을 위해, 설정해야 하는 파일 확장자는 아래와 같다.
    - `vex`: HNSW graph를 저장한다.
    - `vec`: 모든 type(`float`, `byte`, `bit`)의 양자화되지 않은 vector 값들을 저장한다.
    - `veq`: `int4` 또는 `int8`로 양자화된 vector 값들을 저장한다.
    - `veb` : `bbq`로 양자화된 binary vector 값들을 저장한다.
    - `vem`, `vemf`, `vemq`, `vemb`: metadata를 저장한다. 일반적으로 매우 적은 양이며 굳이 pre loading을 하지 않아도 된다.
  - Dynamic setting으로 index 생성 이후에도, close만 하면 변경이 가능하다.
  
  ```json
  POST my_index/_close
  
  PUT my_index/_settings
  {
    "index.store.preload": ["vex", "vec", "veq", "veb", "vem"]
  }
  
  POST my_index/_open
  ```



- kNN search와 segment 개수의 관계
  - 각각의 segment들은 고유한 HNSW graph를 가진다.
    - 따라서 segment의 개수가 적을수록, 탐색해야 하는 graph의 개수가 적어지므로 보다 빠른 검색이 가능하다.
    - 그러나 결과의 정확도는 떨어질 수 있다. 
  - Segment size의 최대치를 늘리기
    - `index.merge.policy.max_merged_segment` 설정을 통해 segment의 최대 크기를 조정할 수 있다.
    - 기본값은 5GB이지만, vector의 차원이 큰 경우 10GB나 20GB로 늘리는 변경하는 것을 고려해 볼만하다.
  - Bulk indexing 중에 큰 segment 생성하기
    - 만약 색인 중에 준 실시간 검색이 요구되지 않는다면, `index.refresh_interval`을 -1로 설정함으로써 refresh가 발생하지 않도록 하고, 추가적인 segement가 생성되는 것을 방지할 수 있다.
    - Flushing 전에 보다 많은 document들을 받기 위해서 indexing buffer를 충분히 활용하는 것도 방법이 될 수 있다.
    - `indices.memory.index_buffer_size`의 기본 값은 heap size의 10%인데, 32GB와 같이 충분한 heap size가 확보되었을 경우, 이 설정을 변경할 필요는 거의 없다.
    - 다만, `index.translog.flush_threshold_size`를 늘려 indexing buffer를 최대한 활용하도록 조정할 수 있다.
