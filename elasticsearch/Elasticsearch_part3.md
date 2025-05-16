# Elasticsearch의 자료 구조

- 역색인(Inverted Index)

  - 역색인은 키워드들에 문서들의 PK(또는 주소, 파일명 등)와 같은 값들을 매핑하여 저장하는 기술이다.
    - 역색인 작업의 장점은 매우 빨리 찾을 수 있다는 것이다.
    - 기존보다 추가적인 작업과 메모리가 필요하게 되지만, 검색은 매우 빠른 속도로 처리가 된다.
  - 다음과 같이 dictionary 구조로 저장되어 있는 데이터(문서)가 있다.

  ```python
  data1 = {1:"사과의 효능에 대해서 알아봅시다."}
  data2 = {2:"사과의 효능과 유통기한에 대해서 알아봅시다."}
  data3 = {3:"제가 가장 좋아하는 과일은 사과입니다."}
  ```

  - 위 data들의 value에 형태소 분석을 수행하면 각 value의 형태소들이 추출된다.
    - 일반적으로 검색엔진은 불필요한 품사를 제외한 명사등만 추출한다(명사만 추출된다고 가정).
    - data1: 사과, 효능
    - data2: 사과, 효능, 유통기한
    - data3: 사과, 과일
  - 검색 엔진은 위 결과를 토대로 역색인 작업을 하게 된다.

  | 키워드   | 문서 번호 |
  | -------- | --------- |
  | 사과     | 1, 2, 3   |
  | 효능     | 1, 2      |
  | 유통기한 | 2         |
  | 과일     | 3         |

  - "사과 효능"을 검색했을 경우
    - 사과가 포함된 문서 1,2,3을 가져오고
    - 유통기한이 포함된 문서 1, 2를 가져온다.
    - 마지막으로 둘의 교집합인 1, 2를 출력해준다.

  - AND가 아닌 OR 처리가 필요하다면 다음과 같은 처리를 하면 된다.
    - "사과 효능"을 검색할 경우
    - 1번 문서에는 사과, 효능이 모두 일치한다 - 2개가 일치
    - 2번 문서도 사과, 효능이 모두 일치한다 - 2개가 일치
    - 3번 문서는 사과만 일치한다 - 1개만 일치
    - 이제 문서별로 일치도가 높은 순으로 정렬(1,2,3) 후 사용자에게 출력한다.
  - 문서의 개수가 증가하더라도, 각 keyword에 문서 번호가 추가될 뿐이기에 큰 속도 저하 없이 검색이 가능하다.
  - Inverted index는 Elasticsearch segment로 disk에 저장된다.
    - 검색시에 disk에 저장된 모든 inverted index(segment)를 읽어 검색을 수행한다.
    - 빈번하게 접근하는 inverted index를 cache memory에 저장하여 보다 빠르게 읽을 수 있도록 한다.
  - Segment가 불변성을 가짐으로써 얻는 이점
    - 불변성이 보장된다면, 다수의 스레드가 동시에 접근하여 수정하는 일이 없어지므로 Lock이 필요 없어진다.
    - 불변성이 보장되지 않을 경우 데이터가 변경될 때 마다 system cache를 갱신해야 하는데, 불변성이 보장된다면 system cache가 상대적으로 오랜 기간 동안 변동 없이 유지되므로 system cache를 적극적으로 활용이 가능하다.
    - 역색인을 만드는 과정에서 많은 resource(CPU, memory)를 사용하는데, segment의 변경을 허용할 경우 일부가 수정되더라도 segment file을 다시 작성해야 해서 많은 resource가 소모된다.
    - 따라서 변경 사항이 있을 때 마다 기존의 segment file을 수정하기 보다는 새로운 segment file을 생성하는 것이 낫다.
  - Elasticsearch의 모든 data가 inverted index에 저장되는 것은 아니다.
    - Elasticsearch의 모든 data는 각 field에 최적화된 자료구조에 저장된다.
    - 예를 들어 text field의 data는 inverted index에 저장되지만, numeric이나 geo field는 BKD tree 형태로 저장된다.
  - Term Dictionary
    - 특정 field에 대한 고유한 모든 term을 저장하고 있는 정렬된 skip list이다.
    - Skip list란 linked list와 유사하지만, 하나의 node가 다음 노드를 가리키는 pointer를 여러 개 가지고 있는 자료구조로 바로 다음 node만을 가리키는 pointer만 가지고 있는 linked list와 달리 여러 포인터를 가지고 있기에 중간에 있는 node를 skip할 수 있어 빠른 속도로 검색이 가능하다는 장점있다.
    - 모든 유일한 term을 저장하기 위한 자료구조이다.
    - Term과 해당 term 대한 meta data(예: 문서 빈도, 문서 위치 등)를 매핑하는 역할을 한다.
  - Posting List
    - Term dictionary와 마찬가지로 정렬된 skip list이다.
    - term과 관련된 document id들을 저장하고 있다.
    - 검색된 term들의 document id를 반환하기 위해 사용한다.



- fielddata

  - 집계, 정렬 등은 역인덱스를 통해서는 불가능하다.
    - 역인덱스를 사용하는 검색은 어떤 문서가 특정 키워드를 포함하는지 여부를 보는 것이다.
    - 그러나 집계 정렬 등은 특정 키워드를 포함하는지가 아닌 특정 field의 값이 무엇인지를 알아야 가능하다.
  - 따라서 Elasticsearch는 text 타입 필드를 대상으로 집계, 정렬등을 위해 fielddata라는 자료 구조를 사용한다.
    - fielddata는 key를 document로, value를 field와 field value로 하는 자료구조이다.

  ```json
  [
      {
          "doc_id1": {
              "field1":"foo",
              "field2":"bar"
          }
      },
      {
          "doc_id2": {
              "field1":"baz",
              "field2":"qux"
          }
      }
  ]
  ```

  - text field는 기본적으로 검색이 가능하지만, 집계, 정렬, scripting은 불가능하다.
    - 만약 text field를 대상으로 집계, 정렬, 혹은 script를 사용하여 text field의 값에 접근해야한다면, `fielddata` 값을 true로 변경해줘야한다.
    - 이 경우 분리된 token 별로 집계가 가능하다.

  ```json
  PUT my-index-000001/_mapping
  {
    "properties": {
      "my_field": { 
        "type":     "text",
        "fielddata": true
      }
    }
  }
  ```

  - fielddata는 in-memory 구조로 동작하여 많은 heap memory를 소비한다.
    - 아마 이것이 text type의 fielddata의 기본값으로 false로 설정해둔 이유일 듯 하다.
    - text 필드는 token 단위로 저장을 해야하는데, field value가 조금만 길어져도 무수히 많은 token이 생성될 것이기에 훨씬 많은 heap memory를 소비하게 될 것이다.
    - Elasticsearch는 집계나 정렬 요청이 처음 들어왔을 때, 해당 필드에 대해 fielddata cache(JVM heap memory 영역에 유지된다)를 생성하고 모든 document의 field value를 정렬/집계에 적합한 구조로 구조화하여 그 곳에 저장한다.
    - 예를 들어 `subject`라는 필드를 대상으로 집계를 해야한다고 하면, elasticsearch는 모든 document의 subject field의 값을 전부 구조화하여 heap memory에 올린다.
    - fielddata cache는 더 이상 사용되지 않거나 heap pressure가 높아질 경우 LRU 정책 등에 의해 제거될 수 있다.
    - 추후에 요청이 들어올 경우 다시 생성한다.
  - `fielddata_frequency_filter`
    - 메모리에서 불러오는 term의 수를 빈도를 기준으로 감소시켜 메모리 사용을 줄일 수 있다.
    - `min`, `max` 값을 지정하며 둘 사이의 빈도를 지닌 term들만 메모리에서 불러온다.
    - `min`, `max` 값은 양의 정수 혹은 0~1 사이의 소수로 지정한다.
    - 퍼센트는 세그먼트 내의 모든 문서가 아닌, 해당 field에 값이 있는 문서들만 대상으로 계산된다.
    - `min_segment_size` 옵션을 통해 일정 개수 이상의 문서를 가지지 못한 세그먼트를 제외시킬 수 있다.
  
  ```json
  PUT my-index-000001
  {
    "mappings": {
      "properties": {
        "tag": {
          "type": "text",
          "fielddata": true,
          "fielddata_frequency_filter": {
            "min": 0.001,	# 1%
            "max": 0.1,	# 10%
            "min_segment_size": 500
          }
        }
      }
    }
  }
  ```
  
  - fielddata 모니터링
  
  ```bash
  # 각 index 별로 전체적인 fielddata 상태를 출력
  $ curl -XGET 'localhost:9200/_stats/fielddata?fields=*&pretty'
  
  # 클러스터 내의 각 node 별로 사용되고 있는 fielddata 상태를 출력
  $ curl -XGET 'localhost:9200/_nodes/stats/indices/fielddata?fields=*&pretty'
  ```



- doc_values

  - Elasticsearch는 문서별로 field 값을 조회해야 하는 집계, 정렬 등의 작업을 위해 `doc_values`라는 자료 구조를 사용한다.
    - `doc_values`는 on-disk 자료구조로, document가 색인될 때 생성된다.
    - ` _source`에 저장되는 것과 같은 값을 저장하지만, `_source`와는 달리 집계나 정렬을 효율적으로 하기 위해서 column 형(column-oriented)으로 저장한다.
    - `doc_values`는 거의 대부분의 field type을 지원하지만, text와 annotated_text type은 지원하지 않는다.
  - fielddata와의 차이
    - query time에 생성되는 fielddata와 달리 doc_value는 index time에 생성된다.
    - memory를 사용하는 fielddata와 달리 doc_value는 disk에 저장된다(따라서 속도는 doc_value가 더 느리다).
    - 또한 text type에는 사용할 수 없는 doc_values와 달리 fielddata는 text type에만 사용할 수 있다.
  - 구조 예시
    - key-value 기반이 아닌 column 기반임을 명심해야한다.

  | fieldA | fieldB |
  | ------ | ------ |
  | value1 | value1 |
  | value2 | value2 |
  | value3 | value3 |

  - Doc-value-only-fields
    - numeric, date, boolean, ip, geo_point, keyword type등은 index되지 않아도 doc values가 활성화 되었다면 검색이 가능하다.
    - 단, 검색 속도는 index되었을 때 보다 훨씬 느리다.
    - 그러나 disk 사용량은 훨씬 줄어든다는 장점이 있다.
    - 따라서 거의 검색되지 않는 필드를 대상으로는 아래와 같이 index는 false로 주고 doc value만 활성화(아무 옵션을 주지 않으면 기본적으로 활성화 된다)하여 사용하는 것도 좋은 선택이다.

  ```json
  // PUT my-index-000001
  {
    "mappings": {
      "properties": {
        "session_id": { 
          "type":  "long",
          "index": false
        }
      }
    }
  }
  ```

  - doc values 비활성화하기
    - doc values를 지원하는 모든 field는 기본적으로 doc values가 활성화되어있다.
    - 만일 비활성화 시키고자 한다면 아래와 같이 하면 된다.

  ```json
  // PUT my-index-000001
  {
    "mappings": {
      "properties": {
        "session_id": { 
          "type": "keyword",
          "doc_values": false
        }
      }
    }
  }
  ```

  - `doc_values`를 disable한 상태로 field를 생성하면 aggregation과 sorting이 불가능하다.

  ```json
  // PUT test
  {
    "mappings": {
      "properties": {
        "foo":{
          "type":"keyword",
          "doc_values": false
        }
      }
    }
  }
  
  // GET test/_search
  {
    "aggs": {
      "my_terms": {
        "terms": {
          "field": "foo",
          "size": 10
        }
      }
    }
  }
  
  // GET test/_search
  {
    "sort": [
      {
        "foo": {
          "order": "desc"
        }
      }
    ]
  }
  ```
  
  - doc_values가 등장한 이후에도 fielddata가 유지되는 이유
    - doc_values는 text 필드를 대상으로는 설정할 수 없는데 반해 fielddata는 text field에만 설정할 수 있다.
    - 즉 fielddata는 text field를 대상으로 집계, 정렬을 수행할 수 있게 해주는 유일한 방법이다.



- Global Ordinals

  - keyword field 등의 term 기반 field type들은 doc_values를 내부적으로 ordinal mapping 구조로 저장한다.

    - Mapping은 각 term 마다에 증가하는 intetger 혹은 사전순으로 정렬된 순서(ordinal)를 할당하는 방식으로 이루어진다.
    - 각 document는 ordinal 값만 저장(doc-ordinal mapping)하며, 실제 term은 별도로 term-ordinal mapping에 저장한다.

  - 예시

    - 아래와 같이 각 term에 순서를 매긴다.

    | ordinal | term       |
    | ------- | ---------- |
    | 0       | apple      |
    | 1       | banana     |
    | 2       | watermelon |
    | 3       | kiwi       |

    - 이를 기반으로 각 문서와 ordinal을 매핑한다.

    | doc_id | ordinal |
    | ------ | ------- |
    | doc_1  | 0, 1    |
    | doc_2  | 1, 3    |
    | doc_3  | 0, 3    |
    | doc_4  | 0, 2, 3 |

    - 위 표에서 doc_1은 apple, banana라는 문자열을 저장하는 대신, 0, 1이라는 순서만을 저장했는데, 나중에 field_value가 필요하면 해당 순서에 해당하는 문자열을 가져와서 사용하는 방식이다.

  - 위 두 개의 mapping(term-ordinal, doc-ordinal)을 ordinal mapping이라 부른다.
  
    - Ordinal mapping은 각 shard별로 생성된다.
  
    - 따라서 아래와 같은 문제가 발생할 수 있다.
    
    - Shard A에서는 apple이 0번에 매핑되어 있는데, Shard B에서는 apple이 1번에 매핑되어 있는 경우가 있을 수 있다.
    
    - 이 경우 집계시에 부정확한 결과가 나올 수 있다.
    
    - 이를 해결하기 위한 것이 global ordinal이다.
    
  - global ordinal
  
    - 각 shard는 ordinal mapping을 각자 관리한다.
    - 그러나 정렬, 집계는 전체 shard를 대상으로 이루어져야 하므로, Elasticsearch는 global ordinal이라 부르는 ordinal mapping들의 결합체를 생성한다.
  
  - 아래와 같을 때 사용된다.
    - keyword, ip, falttend field를 대상으로 한 특정 bucket aggregation(terms, composite 등).
    - fielddata가 true로 설정된 text field에 대한 aggregation
    - join field를 대상으로 하는 연산(has_child query 혹은 parent aggregation 등)
    - 정렬



- Stored fields
  
  - Elasticsearch에서 색인과 저장은 다르다.
    - 색인은 들어온 데이터로 역색인 구조를 만드는 것이다.
    - 저장은 들어온 데이터를 그대로 저장하는 것이다.
    - Elasticsearch는 indexing 요청이 들어올 때 모든 필드를 역색인구조로 색인한다.
    - 기본적으로 필드들은 색인되지만, 개별적으로 저장되지 않고 `_source`라 불리는 필드에 일괄적으로 저장된다.
  - Elasticsearch는 index된 데이터에 검색을 실행하고, store된 데이터를 반환한다.
    - 즉, 실제 검색은 `_source` 필드에 행해지는 것이 아니라 역색인 테이블에 행해진다.
  - `_source` 필드는 overhead를 불러올 수 있지만 아래와 같은 이유로 저장이 필요하다.
    - response에 원본 데이터를 함께 반환하기 위해서(response의 `_source` 필드에 담아 보낸다)
    - reindexing, update, update_by_query 등을 사용하기 위해서(당연하게도 원본 데이터가 없다면 reindexing, update 등이 불가능하다).
    - highlight 기능을 사용하기 위해서.
  - Mapping 설정시에 `store`를 true로 준 field는 Lucene의 stored fields형태로 저장된다.
    - 기본값은 false이다.
    - `_id`와 `_source` field의 경우 stored field이다.
    
  - Stored fields는 모든 stored field를 연속적으로 포함한 row 형태로 저장된다.
    - 가장 앞에 있는 field가 `_id` field이며, `_source`가 가장 마지막 field이다.
    - Row 형태로 저장되기에 column 형태로 저장되는 doc_values에 비해 retrieve 속도가 느리다.
    - 아래와 같이 문서 x의 각 field들이 순차적으로 저장되고, 그 이후에 문서 x+1의 각 field들의 field들이 순차적으로 저장되는 형식이다.
    - 따라서 column기반으로 저장되는 doc_values에 비해 탐색 속도가 느릴 수 밖에 없다.
  
  | field1 | field2 | field3 | field4 | field1 | field2 | field3 | field4 | field5 | field6 |
  | ------ | ------ | ------ | ------ | ------ | ------ | ------ | ------ | ------ | ------ |
  
  - Elasticsearch에서 stored fields는 압축된다.
    - 따라서 당연하게도, `_id` field와 `_source` field도 압축된다.
  - 용도
    - _source가 저장되는 것을 비활성화 한 상태에서 일부 필드만 저장되도록 하고 싶을 때 사용할 수 있다.
    - 예를 들어 title, content 필드가 있을 때, 검색 결과에는 title만 반환해도 된 다고 하면 굳이 content를 _source에 저장할 필요는 없다.
    - 이 경우 _source를 비활성화하고, title만 store를 true로 줘서 저장되도록 할 수 있다.





# Elasticsearch의 score 계산 방식

> https://wikidocs.net/31698 참고

- TF-IDF(Term Frequency-Inverse Document Frequency)

  - Term이 한 문서에서 얼마나 자주 반복되는지와 여러 문서들에서 얼마나 자주 등장하는 term인지가 점수에 영향을 미친다.

    - 하나의 단어가 한 문서 내에서 여러 번 반복되면 점수는 올라가고, 하나의 단어가 여러 문서에서 빈번히 발견될 수록 점수는 낮아진다.

  - 문서 하나를 d, 단어를 t, 문서의 총 개수를 D이라 할 때 TF, DF, IDF는 다음과 같이 정의된다.

    - tf(d, t): 특정한 단어가 문서 내에 얼마나 자주 등장하는지를 나타내는 값.

    - df(t): 특정 단어 t가 전체 문서군 내에서 얼마나 자주 등장하는지를 나타내는 값.
    - idf(d, t): df(t)에 반비례하는 수

  - tf(d, t)의 계산 방식(아래 방식 외에도 무수히 많은 변형이 존재한다)

    - 불린 빈도: t가 d에 한 번이라도 나타나면 1, 아니면 0. 간단하긴 하지만 문서에 단어가 1번 나타나나 1000번 나타나나 차이가 없다는 문제가 있다.
    - 로그 스케일 빈도: `log(f(t,d)+1)`, `f(t,d)`는 문서 d에 t가 등장하는 빈도 수로, 결과값의 크기가 너무 커지지 않게 하기 위해 log를 사용하고, d에 t가 없는 경우 값이 마이너스 무한대로 수렴하는 것을 막기 위해 1을 더해준다. 작은 빈도수 차이일 때는 tf값의 차이가 커지지만, 단어의 빈도가 무수히 늘어날수록 tf의 값의 차이가 크게 벌어지지 않게 된다.
    - 증가빈도: 특정 문서 d에 등장하는 모든 단어의(w) 빈도수를 계산 후 그 최댓값이 분모(문서 내에서 가장 많이 등장한 term의 등장 횟수)가 되고, `f(t,d)`(문서 `d`에서 term `t`의 등장 횟수)는 분자가 된다. 최소 0.5부터 최대 1까지의 값을 가진다. 문서의 길이가 길어 다양한 단어가 나올 때 주로 사용한다.

    $$
    tf(t,d) = 0.5+0.5*\frac{f(t,d)}{max\{f(w,d):w\in d\}} = 0.5+0.5*\frac{f(t,d)}{문서\ 내\ 단어들의\ freq(t,d)중\ 최대값}
    $$

  - idf(d, t)의 계산 방식

    - log로 계산하는 이유는 문서의 개수 D가 커질수록 IDF의 값이 기하급수적으로 커지는 것을 막기 위해서이다.
    - 분모에 1을 더해주는 이유는 특정 단어가 전체 문서에서 아예 등장하지 않을 경우 분모가 0이 되는 것을 방지하기 위해서다.

    $$
    idf(d,t)=log(\frac{D}{1+df(t)})
    $$

  - TF-IDF의 최종 계산식

    - tf(t, d) 값과 idf(t, D) 값을 곱한다.

    $$
    tfidf(t,d,D) = tf(t,d)*idf(t,D)
    $$

  - 예시(조사, 문장부호는 생략)

    - doc1: 식당에서 메뉴를 고민.
    - doc2: 메뉴가 치킨인 식당.
    - doc3: 오늘 저녁 치킨? 치킨!
    - doc4: 점심 식사는 초밥.

  |      | 점심 | 저녁 | 오늘 | 메뉴 | 치킨 | 고민 | 식당 | 초밥 | 식사 |
  | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- |
  | doc1 | 0    | 0    | 0    | 1    | 0    | 1    | 1    | 0    | 0    |
  | doc2 | 0    | 0    | 0    | 1    | 1    | 0    | 1    | 0    | 0    |
  | doc3 | 0    | 1    | 1    | 0    | 2    | 0    | 0    | 0    | 0    |
  | doc4 | 1    | 0    | 0    | 0    | 0    | 0    | 0    | 1    | 1    |

  - TF 계산(증가빈도로 계산)
    - doc1에 등장하는 모든 단어(식당, 메뉴, 고민)는 각 1번씩만 등장하므로 분모는 1이 된다.
    - doc1에서 "메뉴"라는 단어는 문서 내에 1번만 등장하므로 분자는 1이 된다.
    - 따라서 doc1에서 메뉴의 tf(d, t)의 값은 1이다.
    - 나머지도 위 과정에 따라 계산하면 아래와 같이 나오게 된다(공백은 최소값인 0.5)
    - 아래에서 짧은 문장을 증가빈도로 계산했을 때의 문제를 알 수 있는데, doc3에서 두 번 등장한 치킨과 doc2에서 한 번 등장한 치킨이 동일한 TF 값을 갖게 된다.

  |      | 점심 | 저녁 | 오늘 | 메뉴 | 치킨 | 고민 | 식당 | 초밥 | 식사 |
  | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- |
  | doc1 |      |      |      | 1    |      | 1    | 1    |      |      |
  | doc2 |      |      |      | 1    | 1    |      | 1    |      |      |
  | doc3 |      | 0.75 | 0.75 |      | 1    |      |      |      |      |
  | doc4 | 1    |      |      |      |      |      |      | 1    | 1    |

  - IDF계산
    - IDF 계산을 위해 사용하는 로그의 밑은 크기를 조절하는 상수의 역할을 하며, 사용자가 임의로 정한다.
    - TF-IDF를 계산하는 패키지는 언어와 무관하게 자연 로그(밑으로 자연상수(e, 2.7182...)를 사용하는 로그)를 사용하므로 예시에서도 자연 로그를 사용한다.(자연 로그는 보통 log가 아닌 ln으로 표현한다).
    - 문서의 개수는 총 4개이기에 분자는 4가 된다.

  | 단어 | 점심                   | 저녁                   | 오늘                   | 메뉴                   | 치킨                   | 고민                   | 식당                   | 허기                   | 식사                   |
  | ---- | ---------------------- | ---------------------- | ---------------------- | ---------------------- | ---------------------- | ---------------------- | ---------------------- | ---------------------- | ---------------------- |
  | IDF  | ln(4/(1+1)) = 0.693147 | ln(4/(1+1)) = 0.693147 | ln(4/(1+1)) = 0.693147 | ln(4/(2+1)) = 0.287682 | ln(4/(2+1)) = 0.287682 | ln(4/(1+1)) = 0.693147 | ln(4/(2+1)) = 0.287682 | ln(4/(1+1)) = 0.693147 | ln(4/(1+1)) = 0.693147 |

  - 이제 둘을 곱하면 TF-IDF 값을 얻을 수 있다.

  |      | 점심     | 저녁     | 오늘     | 메뉴     | 치킨     | 고민     | 식당     | 허기     | 식사     |
  | ---- | -------- | -------- | -------- | -------- | -------- | -------- | -------- | -------- | -------- |
  | doc1 |          |          |          | 0.287682 |          | 0.693147 | 0.287682 |          |          |
  | doc2 |          |          |          | 0.287682 | 0.287682 |          | 0.287682 |          |          |
  | doc3 |          | 0.519860 | 0.519860 |          | 0.287682 |          |          |          |          |
  | doc4 | 0.693147 |          |          |          |          |          |          | 0.693147 | 0.693147 |



- BM25(Okapi BM25)

  > https://blog.naver.com/shino1025/222240603533
  >
  > https://inyl.github.io/search_engine/2017/04/18/bm25_2.html
  >
  > https://www.elastic.co/blog/practical-bm25-part-2-the-bm25-algorithm-and-its-variables
  
  - TF-IDF의 문제점
    - 한 단어가 반복적으로 등장하면 TF의 값이 지속적으로 증가한다.
    - 문서 길이가 길어질 수록 다양한 단어가 등장할 수 밖에 없으므로, 검색될 확률이 높아진다.
  - TF-IDF의 변형이다.
    - Elasticsearch 5.0부터 사용중인 계산 방법이다.
    - 원래 Lucene과 Elasticsearch 모두 TF-IDF 방식을 사용했으나 Lucene이 BM25로 계산 방식을 변경하면서 Elasticsearch도 같이 변경했다.
    - 보정 파리미터들을 추가해서 성능을 개선한 알고리즘이다.
  - 계산식
    - 토큰 q<sub>1</sub>, q<sub>2</sub>, ... q<sub>n</sub>을 포함하는 쿼리 Q가 주어질 때, 단일 문서 D에 대한 BM25 점수는 다음과 같이 구한다.
  
  $$
  score(D, Q) = \sum_{i=1}^nIDF(q_i) * \frac{f(q_i,D)*(k1+1)}{f(q_i,D)+k1*(1-b+b*\frac{|D|}{avgdl})}
  $$
  
  - Elasticsearch의 BM25 계산
    - BM25에 boost를 더한다.
    - 즉, `boost * IDF * TF`로 계산한다.
  - Elasticsearch의 IDF 계산
    - `N`은 전체 문서의 개수, $n(q_i)$는 $q_i$라는 term을 포함하고 있는 문서의 개수이다.
  
  $$
  IDF(q_i) = ln(1+\frac{N-n(q_i)+0.5}{n(q_i)+0.5})
  $$
  
  - Elasticsearch의 TF 계산
    - f(q<sub>i</sub>, D) 부분은  단일 문서 D에서 토큰 q<sub>i</sub>의 term frequency를 계산하는 것이다.
    - k1은 특정 term이 문서에 여러 번 등장했을 때 점수의 증가율을 제어한다(기본값은 1.2).
    - k1이 클수록 전체 점수가 TF에 더 민감하게 반응(점수가 더 많이 증가)하며, k1이 작을수록 TF가 커도 전체 점수 증가가 제한된다.
    - b는 TF를 측정할 때, 문서의 길이에 따른 보정 정도를 조절하는 파리미터이다(기본값은 0.7).
    - b가 높으면, 문서의 길이(정확히는 필드 값의 길이)가 길어질수록 점수에 패털티를 준다.
    - dl은 단일 문서의 필드 길이를 뜻하고, avgdl은 전체 문서의 필드의 평균 길이를 뜻한다.
  
  $$
  TF(q_i)=\frac{f(q_i,D)*(k1+1)}{f(q_i,D)+k1*(1-b+b*\frac{dl}{avgdl})}
  $$
  
  - 요약
    - 특정 단어가 문서에 많이 등장했다면 높은 점수를 준다(TF).
    - 만일 해당 필드가 너무 길다면 점수를 약간 깎는다(b).
    - 같은 단어가 단일 문서에 여러 번 등장한다면 점수의 증가폭이 작아진다(k1).
    - 만일 해당 단어가 여러 문서에 흔하게 등장한다면 점수를 많이 깎는다(IDF).



- Elasticsearch의 점수 계산 방식 변경하기

  > https://www.elastic.co/guide/en/elasticsearch/reference/current/index-modules-similarity.html

  - Index 생성시에 각 field 별로 다른 similarity 계산을 사용하도록 변경할 수 있다.
  - 아래와 같이 settings에 similarity 계산 방식을 정의하고, mapping에서 field를 설정할 때 settings에서 정의한 계산 방식을 설정해주면 된다.

  ```json
  // PUT my-index
  {
      "settings": {
          "index": {
              "similarity": {
                  "my_similarity": {
                      "type": "DFR",
                      "basic_model": "g",
                      "after_effect": "l",
                      "normalization": "h2",
                      "normalization.h2.c": "3.0"
                  }
              }
          }
      },
      "mappings":{
          "properties" : {
              "title" : { 
                  "type" : "text", 
                  "similarity" : "my_similarity" 
              }
          }
      }
  }
  ```

  - Elasticsearch는 아래와 같은 similarity module을 지원한다.
    - BM25
    - DFR
    - DFI
    - IB
    - LM Jlinek Mercer

  - 혹은 script를 사용하여 직접 점수 계산 방식을 구현하는 것도 가능하다.
    - 아래 script는 TF-IDF를 script를 사용하여 구현한 것이다.
    - 주의할 점은 반환하는 score는 반드시 양수여야 한다는 것이다.

  ```json
  // PUT my-index
  {
      "settings": {
          "number_of_shards": 1,
          "similarity": {
              "scripted_tfidf": {
                  "type": "scripted",
                  "script": {
                      "source": "double tf = Math.sqrt(doc.freq); double idf = Math.log((field.docCount+1.0)/(term.docFreq+1.0)) + 1.0; double norm = 1/Math.sqrt(doc.length); return query.boost * tf * idf * norm;"
                  }
              }
          }
      }
  }
  ```

  - `weight_script`를 통해 가중치를 주는 script를 별도로 작성하는 것도 가능하다.
    - `weight_script`를 사용하지 않을 경우 가중치는 1로 고정된다.

  ```json
  {
      "settings": {
          "similarity": {
              "scripted_tfidf": {
                  "type": "scripted",
                  "weight_script": {
                      "source": "double idf = Math.log((field.docCount+1.0)/(term.docFreq+1.0)) + 1.0; return query.boost * idf;"
                  },
                  "script": {
                      "source": "double tf = Math.sqrt(doc.freq); double norm = 1/Math.sqrt(doc.length); return weight * tf * norm;"
                  }
              }
          }
      }
  }
  ```



- BM25 similarity 사용시 TF값의 계산 요소 중 term_frequency의 가중치 낮추기

  - BM25 similarity 사용시 term_frequency 값의 가중치를 낮춰야 한다면 중복 token을 제거하는 것도 고려해 볼 수 있다.
  - 예를 들어 아래와 같은 문서가 있다고 가정해보자.

  ```json
  // PUT test-index/_doc/1
  {
      "text":"brown fox playing with fox"
  }
  ```

  - 위 문서는 standard tokenizer 기준으로 아래와 같이 tokenizing된다.
    - "fox" 라는 token이 두 번 나왔으므로 `term_freq` 값은 2가 된다.

  ```json
  // GET test-index/_termvectors/1?fields=text
  {
    // ...
    "term_vectors": {
      "text": {
        // ...
          "fox": {
            "term_freq": 2,
            "tokens": [
              {
                "position": 1,
                "start_offset": 6,
                "end_offset": 9
              },
              {
                "position": 4,
                "start_offset": 23,
                "end_offset": 26
              }
            ]
          },
          // ...
        }
      }
    }
  }
  ```

  - 아래와 같이 추가 문서를 색인하고

  ```json
  // PUT test-index/_doc/2
  {
      "text":"brown fox playing with box"
  }
  ```

  - "fox"라는 검색어로 검색을 하면

  ```json
  // GET test-index/_search
  {
      "explain": true, 
      "query": {
          "match": {
              "text": "fox"
          }
      }
  }
  ```

  - "fox"라는 token을 두 개 가진 1번 문서의 tf 값이 더 크므로 1번 문서의 점수가 더 높게 나오는 것을 확인할 수 있다.
    - TF 값 계산에 포함되는 요소 중 `freq`를 제외한 다른 모든 값들은 동일한 것을 볼 수 있다.

  ```json
  {
    // ...
    "hits": {
      // ...
      "hits": [
        {
          // ...
          "_id": "1",
          "_score": 0.25069216,
          "_source": {
            "text": "brown fox playing with fox"
          },
          "_explanation": {
            "value": 0.25069216,
            "description": "weight(text:fox in 0) [PerFieldSimilarity], result of:",
            "details": [
              {
                "value": 0.25069216,
                "description": "score(freq=2.0), computed as boost * idf * tf from:",
                "details": [
                  // ...
                  {
                    "value": 0.625,
                    "description": "tf, computed as freq / (freq + k1 * (1 - b + b * dl / avgdl)) from:",
                    "details": [
                      {
                        "value": 2,
                        "description": "freq, occurrences of term within document",
                        "details": []
                      },
                      {
                        "value": 1.2,
                        "description": "k1, term saturation parameter",
                        "details": []
                      },
                      {
                        "value": 0.75,
                        "description": "b, length normalization parameter",
                        "details": []
                      },
                      {
                        "value": 5,
                        "description": "dl, length of field",
                        "details": []
                      },
                      {
                        "value": 5,
                        "description": "avgdl, average length of field",
                        "details": []
                      }
                    ]
                  }
                ]
              }
            ]
          }
        },
        {
          // ...
          "_id": "2",
          "_score": 0.18232156,
          "_source": {
            "text": "brown fox playing with box"
          },
          "_explanation": {
            "value": 0.18232156,
            "description": "weight(text:fox in 0) [PerFieldSimilarity], result of:",
            "details": [
              {
                "value": 0.18232156,
                "description": "score(freq=1.0), computed as boost * idf * tf from:",
                "details": [
                  // ...
                  {
                    "value": 0.45454544,
                    "description": "tf, computed as freq / (freq + k1 * (1 - b + b * dl / avgdl)) from:",
                    "details": [
                      {
                        "value": 1,
                        "description": "freq, occurrences of term within document",
                        "details": []
                      },
                      {
                        "value": 1.2,
                        "description": "k1, term saturation parameter",
                        "details": []
                      },
                      {
                        "value": 0.75,
                        "description": "b, length normalization parameter",
                        "details": []
                      },
                      {
                        "value": 5,
                        "description": "dl, length of field",
                        "details": []
                      },
                      {
                        "value": 5,
                        "description": "avgdl, average length of field",
                        "details": []
                      }
                    ]
                  }
                ]
              }
            ]
          }
        }
      ]
    }
  }
  ```

  - 이제 term_frequency의 가중치를 낮추기 위해 아래와 같이 `unique` token filter를 사용하여 중복 token을 제거한다.

  ```json
  // PUT unique-token
  {
      "settings": {
          "analysis": {
              "analyzer": {
                  "unique_analyzer": {
                      "type":"custom",
                      "tokenizer":"standard",
                      "filter": [
                          "unique"
                      ]
                  }
              }
          }
      },
      "mappings": {
          "properties": {
              "text": {
                  "type": "text",
                  "analyzer": "unique_analyzer"
              }
          }
      }
  }
  ```

  - 그 후 다시 2개의 문서를 색인한다.
    - 위와 동일하게 하나의 문서에만 fox가 2번 등장한다.

  ```json
  // PUT unique-token/_doc/1
  {
      "text":"brown fox playing with fox"
  }
  
  // PUT unique-token/_doc/2
  {
      "text":"brown fox playing with box"
  }
  ```

  - 1번 문서의 token을 확인해보면 아래와 같다.
    - 중복 token을 모두 제거했으므로 fox token의 `term_freq` 값이 1인 것을 볼 수 있다.

  ```json
  // GET unique-token/_termvectors/1?fields=text
  {
    // ...
    "term_vectors": {
      "text": {
        // ...
        "terms": {
          // ...
          "fox": {
            "term_freq": 1,
            "tokens": [
              {
                "position": 1,
                "start_offset": 6,
                "end_offset": 9
              }
            ]
          },
          // ...
        }
      }
    }
  }
  ```

  - 이제 다시 동일한 query로 검색을 해보면

  ```json
  // GET unique-token/_search
  {
      "explain": true, 
      "query": {
          "match": {
              "text": "fox"
          }
      }
  }
  ```

  - `term_freq` 값이 1이 된 것을 확인할 수 있다.

  ```json
  {
      // ...
      "hits": {
          // ...
          "max_score": 0.19100355,
          "hits": [
              {
                  // ...
                  "_id": "1",
                  "_score": 0.19100355,
                  // ...
                  "_explanation": {
                      "value": 0.19100355,
                      "description": "weight(text:fox in 0) [PerFieldSimilarity], result of:",
                      "details": [
                          {
                              "value": 0.19100355,
                              "description": "score(freq=1.0), computed as boost * idf * tf from:",
                              "details": [
                                  // ...
                                  {
                                      "value": 0.47619045,
                                      "description": "tf, computed as freq / (freq + k1 * (1 - b + b * dl / avgdl)) from:",
                                      "details": [
                                          {
                                              "value": 1,
                                              "description": "freq, occurrences of term within document",
                                              "details": []
                                          },
                                          {
                                              "value": 1.2,
                                              "description": "k1, term saturation parameter",
                                              "details": []
                                          },
                                          {
                                              "value": 0.75,
                                              "description": "b, length normalization parameter",
                                              "details": []
                                          },
                                          {
                                              "value": 4,
                                              "description": "dl, length of field",
                                              "details": []
                                          },
                                          {
                                              "value": 4.5,
                                              "description": "avgdl, average length of field",
                                              "details": []
                                          }
                                      ]
                                  }
                              ]
                          }
                      ]
                  }
              },
              {
                  // ...
                  "_id": "2",
                  "_score": 0.17439456,
                  // ...
                  "_explanation": {
                      "value": 0.17439456,
                      "description": "weight(text:fox in 0) [PerFieldSimilarity], result of:",
                      "details": [
                          {
                              "value": 0.17439456,
                              "description": "score(freq=1.0), computed as boost * idf * tf from:",
                              "details": [
                                  // ...
                                  {
                                      "value": 0.43478262,
                                      "description": "tf, computed as freq / (freq + k1 * (1 - b + b * dl / avgdl)) from:",
                                      "details": [
                                          {
                                              "value": 1,
                                              "description": "freq, occurrences of term within document",
                                              "details": []
                                          },
                                          {
                                              "value": 1.2,
                                              "description": "k1, term saturation parameter",
                                              "details": []
                                          },
                                          {
                                              "value": 0.75,
                                              "description": "b, length normalization parameter",
                                              "details": []
                                          },
                                          {
                                              "value": 5,
                                              "description": "dl, length of field",
                                              "details": []
                                          },
                                          {
                                              "value": 4.5,
                                              "description": "avgdl, average length of field",
                                              "details": []
                                          }
                                      ]
                                  }
                              ]
                          }
                      ]
                  }
              }
          ]
      }
  }
  ```

  - 주의 사항
    - 위 내용을 봤을 때 문서 1과 문서 2에서 token fox에 대한 `freq` 값이 1로 같으므로 두 문서의 점수가 같아야 할 것 같지만, 그렇지 않다.
    - 이는 field의 길이를 나타내는 `dl` 때문이다.
    - 문서 1의 token 들 중 중복된 fox token 하나가 날아가면서 문서 1의 `dl`이 5에서 4로 1 감소했다.
    - 이로 인해 TF 값은 올라가게 된다.
    - 따라서 위 방식을 사용할 때는, 사라진 token들로 인해 `dl` 값이 감소하여 TF값을 증가시킬 수 있다는 것을 염두에 둬야한다.




- Shard의 개수와 score의 관계

  - Shard의 개수에 따라 score가 달라질 수 있다.
  - 예를 들어 아래와 같이 shard가 1개인 index가 있다고 가정해보자.

  ```json
  // PUT test
  {
    "settings": {
      "number_of_shards": 1,
      "number_of_replicas": 0
    }
  }
  ```

  - 위 index에 문서를 색인한다.

  ```json
  // POST _bulk
  {"index":{"_index":"test"}}
  {"title":"Hello"}
  {"index":{"_index":"test"}}
  {"title":"Hello World"}
  {"create":{"_index":"test"}}
  {"title":"Hello Tom"}
  {"create":{"_index":"test"}}
  {"title":"Hello John"}
  ```

  - 그 후 검색을 실행하면

  ```json
  // GET test/_search
  {
    "query": {
      "match": {
        "title": "hello"
      }
    }
  }
  ```

  - 아래와 같은 결과가 나온다.

  ```json
  {
      "hits" : [
        {
          "_index" : "test",
          "_type" : "_doc",
          "_id" : "SoiHwZIB02CLrgcgzYKI",
          "_score" : 0.12776,
          "_source" : {
            "title" : "Hello"
          }
        },
        {
          "_index" : "test",
          "_type" : "_doc",
          "_id" : "S4iHwZIB02CLrgcgzYKI",
          "_score" : 0.099543065,
          "_source" : {
            "title" : "Hello World"
          }
        },
        {
          "_index" : "test",
          "_type" : "_doc",
          "_id" : "TIiHwZIB02CLrgcgzYKI",
          "_score" : 0.099543065,
          "_source" : {
            "title" : "Hello Tom"
          }
        },
        {
          "_index" : "test",
          "_type" : "_doc",
          "_id" : "TYiHwZIB02CLrgcgzYKI",
          "_score" : 0.099543065,
          "_source" : {
            "title" : "Hello John"
          }
        }
      ]
  }
  ```

  - 이번에는 shard의 개수를 3개로 변경해서 index를 다시 생성하고

  ```json
  // PUT test
  {
    "settings": {
      "number_of_shards": 3,
      "number_of_replicas": 0
    }
  }
  ```

  - 이전과 동일한 문서를 색인한 뒤

  ```json
  // POST _bulk
  {"index":{"_index":"test"}}
  {"title":"Hello"}
  {"index":{"_index":"test"}}
  {"title":"Hello World"}
  {"create":{"_index":"test"}}
  {"title":"Hello Tom"}
  {"create":{"_index":"test"}}
  {"title":"Hello John"}
  ```

  - 동일한 query로 다시 검색을 해보면

  ```json
  // GET test/_search
  {
    "query": {
      "match": {
        "title": "hello"
      }
    }
  }
  ```

  - 점수가 달라진 것을 확인할 수 있다.
    - 점수가 달라지면서 순위도 달라졌다.

  ```json
  {
      "hits" : [
        {
          "_index" : "test",
          "_type" : "_doc",
          "_id" : "VIiKwZIB02CLrgcgJIL0",
          "_score" : 0.2876821,
          "_source" : {
            "title" : "Hello Tom"
          }
        },
        {
          "_index" : "test",
          "_type" : "_doc",
          "_id" : "UoiKwZIB02CLrgcgJIL0",
          "_score" : 0.15965708,
          "_source" : {
            "title" : "Hello"
          }
        },
        {
          "_index" : "test",
          "_type" : "_doc",
          "_id" : "U4iKwZIB02CLrgcgJIL0",
          "_score" : 0.12343237,
          "_source" : {
            "title" : "Hello World"
          }
        },
        {
          "_index" : "test",
          "_type" : "_doc",
          "_id" : "VYiKwZIB02CLrgcgJIL0",
          "_score" : 0.12343237,
          "_source" : {
            "title" : "Hello John"
          }
        }
      ]
  }
  ```

  - 위와 같은 결과가 나오게 된 원인은 아래와 같다.
    - Elasticsearch는 문서를 여러 개의 shard에 분산해서 저장한다.
    - Elasticsearch는 문서의 점수를 매길 때 BM25를 사용하는데, 이는 전체 문서에서 term이 얼마나 빈번하게 등장하는지, 특정 문서에 term이 얼마나 빈번하게 등장하는지를 가지고 점수를 계산한다.
    - 검색은 각 shard별로(Lucene index별로) 실행되므로, 점수 역시 각 shard별로 매겨진다.
    - 이 때, 전체 문서에서 term이 얼마나 빈번하게 등장하는지, 특정 문서에 term이 얼마나 빈번하게 등장하는지는 index 전체에 색인된 문서들을 대상으로 계산되는 것이 아니라, 각 shard에 색인된 문서들을 대상으로 계산된다.
    - 따라서 shard의 개수가 달라지면(정확히는 문서가 shard에 분산되어 저장되면) 점수가 달라질 수 있다.
  - 핵심은 문서가 shard에 분산되어 저장될 때 점수가 달라질 수 있다는 것이다.
    - 문서의 분산 없는 shard 개수의 증가 만으로는 점수가 달라지지 않는다.
    - 예를 들어 아래와 같이 모두 하나의 shard에 색인되도록 한 뒤 검색을 하면, shard가 1개일 때와 점수가 같다.

  ```json
  // PUT test
  {
    "settings": {
      "number_of_shards": 3,
      "number_of_replicas": 0
    }
  }
  
  // POST _bulk
  {"index":{"_index":"test", "routing":"1"}}
  {"title":"Hello"}
  {"index":{"_index":"test", "routing":"1"}}
  {"title":"Hello World"}
  {"create":{"_index":"test", "routing":"1"}}
  {"title":"Hello Tom"}
  {"create":{"_index":"test", "routing":"1"}}
  {"title":"Hello John"}
  
  // GET _cat/shards/test
  test 2 p STARTED 4 4.3kb 10.0.0.73 es
  test 1 p STARTED 0  226b 10.0.0.73 es
  test 0 p STARTED 0  226b 10.0.0.73 es
  ```

  - 검색 API에서 `search_type` query parameter에 `dfs_query_then_fetch` 값을 주면, 점수를 shard별이 아닌 전체 index를 대상으로 계산하도록 할 수 있다.
    - `search_type`의 기본값은 `query_then_fetch`로 shard별로 점수를 계산한다.
    - 단, 기본값(`query_then_fetch`)을 사용하는 것이 권장되는데, 전체 index를 대상으로 점수를 계산하는 것은 검색 속도를 느리게 할 수 있기 때문이다.

  ```json
  // GET test/_search?search_type=dfs_query_then_fetch
  {
    "query": {
      "match": {
        "title": "hello"
      }
    }
  }
  ```

  - 각 shard에 문서가 분산 저장되어 점수가 달라지는 현상 자체는 실제 서비스를 운영할 때 크게 문제가 되지는 않는다.
    - 색인되는 문서의 개수가 많아질수록 문서의 shard별 분산 저장이 점수에 미치는 영향은 작아진다.
    - 만약 문서의 개수가 적어 분산 저장이 점수에 미치는 영향이 클 것으로 예상되는 경우에는 shard 수를 줄이는 것을 고려할 수 있다.



- Elasticsearch에서 BM25 점수를 0~1사이로 변환하는 방법

  - `script_score` query를 아래와 같이 작성한다.
    - Sigmoid 함수를 사용하는 방식이다.

  ```json
  // GET my-index/_search
  {
    "query": {
      "script_score": {
        "script": {
          "source": "1 / (Math.exp(-_score) + 1)"
        },
        "query": {
          "match": {
            "text": "ipsum"
          }
        }
      }
    }
  }
  ```







# 데이터 색인

## 색인 처리 과정

- Lucene의 색인 처리 과정
  -  In-memory buffer에 데이터 적재
     - 문서의 색인을 요청해도, Lucene은 바로 segment를 생성하지 않는다.
     - segment 생성이란 곧 데이터를 디스크에 쓰는 것을 의미하는데, 이는 비용이 많이 드는 작업이기 때문이다.
     - 따라서 Lucene은 먼저 작업 속도가 더 빠른 in-memory buffer에 data를 쌓는다.
     - 아직 segment를 생성하지는 않았으므로, 검색은 불가능하다.
  -  Flush
     - 물리 디스크가 아닌 시스템 캐시에 segment를 생성하고 data를 저장한다.
     - segment가 생성되었으므로 검색이 가능해진다.
     - 그러나, cache에 저장한 것이므로 데이터가 유실 될 위험이 있다.
  -  Commit
     - 물리 디스크에 segment를 생성하고 data를 저장한다.
     - 리소스가 많이 필요하지만 물리 디스크에 저장이 되었으므로 data의 유실로부터 보다 안전한 상태가 된다.



- Elasticsearch와 Lucene의 차이점

  - 유사한 동작을 두고 서로 지칭하는 용어가 다르다.

  | Lucene | Elasticsearch |
  | ------ | ------------- |
  | flush  | refresh       |
  | commit | flush         |

  - Lucene에는 존재하지 않는 translog(transactional log)를 사용한다.



- Translog(Transactional log)
  - Lucene의 변경사항은 commit이 발생해야만 disk에 저장되면서 영구적으로 반영된다.
    - 이는 상대적으로 비용이 많이 드는 작업이므로 모든 색인, 삭제 작업이 발생할 때마다 실행할 수는 없다.
    - Commit과 다음 commit 사이에 발생한 작업들은 process가 종료되거나 hardware에 문제가 생기면 유실될 수 있다.
  - Commit이 비용이 많이 드는 작업이므로 각 shard는 translog라 불리는 transaction log에 작업들을 저장해둔다.
    - 모든 shard들은 translog를 하나씩 가지고있다.
    - 색인되고 삭제되는 모든 작업은 translog에 기록된다.
    - Commit과 commit 사이에 문제가 생겨서 시스템이 내려가더라도, translog를 보고 복구가 가능하다.



- Elasticsearch의 색인 처리 과정
  - Elasticsearch는 Lucene 기반으로 만들어져 색인 방식도 유사하다.
  - In-memory buffer와 translog에 data를 적재한다.
    - translog는 memory가 아닌 disk에 저장된다.
    - Lucene의 경우 flush와 commit 사이에 뭔가 문제가 생겨 memory에서 disk로 data를 옮기지 못하면, 해당 data는 유실되게 된다.
    - Elasticsearch는 이러한 문제를 해결하기 위해 요청 정보와 data가 담긴 translog를 작성하고, 문제가 생겼을 경우 이를 복구에 사용한다.
  - Refresh
    - Lucene의 Flush와 마찬가지로 memory에 segment를 생성하고, data를 저장한다.
    - segment에 저장되었으므로 검색이 가능해진다.
  - Flush
    - in-memory로 저장된 segment를 disk에 쓰는 작업을 한다.
    - 메모리에 저장된 여러 개의 segment들을 하나로 병합하여 disk에 저장한다.
    - 동시에 translog의 작성을 멈추고, 새로운 blank translog를 생성한다.
    - Flush가 발생하는 주기가 따로 있는 것은 아니다.
    - 아직 flush 되지 않은 translog와 각 flush를 수행하는 비용의 trade off를 고려하는 heuristic으로 flush 실행 여부를 결정한다.
    - 이러한 결정에는 translog에 얼마나 많은 transaction log가 적재되었는지, data의 크기, 마지막 flush 시점 등이 영향을 미친다.



- Lucene index
  - Lucene index는 IndexWriter와 IndexSearcher를 가지고 색인과 검색을 동시에 제공하는 Lucene instance이다.
    - 하나의 Lucene index는 자체적으로 데이터를 색인하고 검색할 수 있는 가장 작은 크기의 단일 검색엔진이라고도 할 수 있다.

  - 하나의 Elasticsearch shard는 하나의 Lucene index이다.
    - 그러나 shard에는 Elasticsearch에서 추가한 다양한 기능이 포함되어 있기는 하다.
    - 그러나 shard의 본질이 Lucene index라는 사실에는 변함이 없다.
    - Lucene index가 자기 자신이 가지고 있는 세그먼트 내에서만 검색이 가능한 것과 달리 샤드는 모든 샤드가 가지고 있는 세그먼트들을 논리적으로 통합해서 검색할 수 있다.
    - Lucene의 세그먼트는 확장이 불가능하다는 문제가 있었는데, Elasticsearch는 shard를 사용하여 여러 segment를 통합하여 검색할 수 있게 함으로써 이러한 한계를 극복하고 확장 가능한 시스템을 만들었다.



- Lucene의 flush
  - Lucene은 효율적인 색인 작업을 위해 내부적으로 일정 크기의 버퍼를 가지고 있다.
    - 이러한 내부 버퍼를 In-memory buffer라 부른다.
    - 색인 작업이 요청되면 전달된 데이터는 일단 in-memory buffer에 순서대로 쌓인다.
    - 그 후 정책에 따라 내부 버퍼에 일정 크기 이상의 데이터가 쌓이거나 일정 시간이 흐를 경우 버퍼에 쌓은 데이터를 모아 한꺼번에 처리한다.
    - 버퍼를 일종의 큐로 사용하는 것이다.
  - 버퍼에 모여 한 번에 처리된 데이터는 segment 형태로 생성되고 이후에 디스크에 동기화된다.
    - 새로운 segment가 생성되고 디스크에 동기화하는 과정까지 거쳐야만 검색이 가능해진다.
  - Lucene은 무거운 fsync 방식을 이용해 디스크를 동기화하는 대신 상대적으로 가벼운 write 방식을 이용해 쓰기 과정을 수행한다.
    - 디스크에 물리적으로 동기화하는 과정은 OS 입장에서는 매우 비용이 큰 연산이기 때문에 segment가 생성될 때 마다 동기화를 할 경우 성능이 급격히 나빠질 수 있다.
    - 따라서 Lucene은 무거운 `fsync()` 함수가 아니라 상대적으로 가벼운 `write()` 함수를 사용한다.
    - `write()` 함수는 일단 캐시에만 기록되고 리턴되며, 실제 데이터는 특정한 주기에 따라 물리적인 디스크로 기록된다.
    - 물리적인 디스크 쓰기 작업을 수행하지 않기에 빠른 처리가 가능한 반면, 시스템이 비정상 종료될 경우 데이터 유실이 일어날 수도 있다.
    - 이를 통해 쓰기 성능을 높이고 이후 일정한 주기에 따라 물리적인 디스크 동기화 작업을 수행한다.
  - 이처럼 in-memory buffer 기반의 처리 과정을 Lucene에서는 flush라고 부른다.
    - 일단 flush 처리에 의해 segment가 생성되면 커널 시스템 캐시에 segment가 캐시되어 읽기가 가능해진다.
    - 컨널 시스템 캐시에 캐시가 생성되면 Lucene의 `openIfChanged()`함수를 이용하여 `IndexSearcher`도 읽을 수 있는 상태가 된다.
    - 즉 flush 이후부터 검색이 가능해진다.
  - `openIfChanged()` 함수
    - Lucene의 `IndexSearcher`는 일단 생성되고 나면 이후 변경된 사항들을 인지하지 못한다.
    - 이를 가능하게 해주는 것이 `openIfChanged()` 함수로 일정 주기마다 실행되어 변경 사항을 반영할 수 있게 해준다.



- Lucene의 commit
  - Lucene이 디스크 동기화를 위해 `write()` 함수를 이용하기 때문에 물리적으로 기록되는 것이 100% 보장되지는 않는다.
    - 디스크에 물리적으로 동기화되기 위해서는 결과적으로 커널 레벨에서 제공하는 `fsync()` 함수가 언젠가는 반드시 실행되어야 한다.
    - 일반적인 쓰기 함수인 `write()` 함수는 커널에 쓰기 작업을 위임하기 때문에 최악의 경우 시스템에 문제가 생기면 데이터가 유실될 수 있다.
  - Lucene에서는 물리적으로 디스크에 기록을 수행하는 `fsync()` 함수를 호출하는 작업을 commit이라 한다.
    - 매번 commit을 수행할 필요는 없지만 일정 주기로 commit을 통해 물리적인 디스크로 기록 작업을 수행해야한다.



- Lucene의 merge
  - Merge는 여러 개의 segment를 하나로 합치는 작업이다.
    - Segment 불변성에 따라 시간이 지날수록 segment의 개수는 증가할 수 밖에 없다.
    - 따라서 늘어난 segment를 하나로 합치는 작업이 필요해진다.
    - Lucene은 특정 주기 마다 자동으로 merge를 수행한다.
  - Merge의 장점
    - 검색 요청이 들어오면 모든 segment를 검색해야 하는데 segment의 개수가 줄어들면 검색 횟수도 줄어들어 검색 성능이 좋아진다.
    - 삭제된 문서가 merge와 함께 디스크에서 삭제되므로 디스크 용량을 확보할 수 있다.
  - Merge 작업은 반드시 commit 작업을 동반해야 한다.
    - Commit 작업은 매우 비용이 많이 드는 연산으로 실행하는 데 많은 리소스가 필요하다/
    - 따라서 적절한 주기를 설정해서 진행해야 한다.



- Elasticsearch의 refresh, flush, optimize API
  - Lucene의 flush는 Elasticsesarch의 refresh에 해당한다.
    - Elasticsearch는 refresh 주기를 수동으로 조절할 수 있는 API를 제공한다.
    - 특별한 경우가 아니면 refresh 주기를 임의로 변경하지 않는 것이 좋다.
  - Lucene의 commit은 Elasticsearch의 flush에 해당한다.
    - Elasticsearch의 flush는 Lucene의 commit 작업을 수행하고 새로운 translog를 시작한다는 의미이다.
    - Translog는 Elasticsearch에만 존재하는 개념으로 고가용성과 매우 밀접한 관련이 있다.
    - Translog는 샤드의 장애 복구를 위해 제공되는 특수한 파일로, 샤드는 자신에게 일어나는 모든 변경사항을 translog에 먼저 기록한 후 내부에 존재하는 Lucene을 호출한다.
    - 시간이 흐를수록 translog의 크기는 계속 증가하고 shard는 1초마다 refresh 작업을 수행해서 실시간에 가까운 검색을 제공한다.
    - 지속적인 refresh 작업에 의해 검색은 가능해지지만 아직은 디스크에 물리적인 동기화가 되지 않은 상태이기 때문에 주기적으로 Lucene commit을 수행해야 한다.
    - 정책에 의해 lucene commit이 정상적으로 수행되면 변경사항이 디스크에 물리적으로 기록되고 translog 파일에서 commit이 정상적으로 일어난 시점까지의 내역이 삭제된다.
  - Elasticsearch에서는 강제로 Lucene의 merge를 실행할 수 있는 API를 제공한다.
    - 세그먼트를 병합하여 검색 성능을 올리는 것이 가능하다.



- Translog
  - Elasticsearch는 고가용성을 제공하기 위해 내부적으로 translog라는 파일을 유지하고 관리한다.
    - 장애 복구를 위한 백업 데이터 및 데이터 유실 방지를 위한 저장소로 사용한다.
  - Translog의 동작 순서
    - 샤드에 어떤 변경사항이 생길 경우 Translog 파일에 먼저 해당 내역을 기록한 후 내부에 존재하는 Lucene index로 데이터를 전달한다.
    - Lucene에 전달된 데이터는 인메모리 버퍼로 저장되고 주기적으로 처리되어 결과적으로 segment가 된다.
    - Elasticsearch는 기본적으로 1초에 한번씩 refresh 작업이 수행되는데, 이를 통해 추가된 세그먼트의 내용을 읽을 수 있게 되고 검색이 가능해진다.
    - 그러나 refresh 이후에도 translog에 기록된 내용은 삭제되지 않고 계속 유지된다.
    - Elasticsearch의 flush가 실행되어 내부적으로 `fsync` 함수를 통해 실제 물리적인 디스크에 변경 내용이 성공적으로 기록돼야 translog 파일의 내용이 비로소 삭제된다.
  - Translog가 필요한 이유
    - Refresh가 실행되면 검색은 가능해지지만 아직은 불안정한 상태라고 볼 수 있다.
    - 장애 발생시 완벽한 복구를 위해서는 물리적인 디스크로 동기화하는 flush가 언젠가는 이뤄져야한다.
    - 만약 refresh와 flush 사이에 장애가 발생할 경우 flush를 통해 물리적인 디스크에 기록되지 못 한 데이터는 유실되게 된다.
    - 그러나 translog가 있다면, refresh와 flush 사이에 장애가 발생하더라도 translog를 통해 복구가 가능하다.
    - 또한 Elasticsearch는 shard crash가 발생할 경우에도 translog를 사용한다.
    - Translog에 기록된 내용을 사용하면 shard를 완벽하게 복구할 수 있다.



- 운영 중에 샤드의 개수를 변경할 수 없는 이유
  - Elasticsearch의 shard는 Lucene index의 확장이다.
    - 각 shard 내부에는 독립적인 Lucene library를 가지고 있으며 Lucene은 단일 머신 위에서 동작(stand alone)하는 검색 엔진이다.
    - 이러한 특성 때문에 shard 내부의 Lucene 입장에서는 함께 index를 구성하는 다른 shard의 존재를 전혀 눈치채지 못한다.
    - Lucene 입장에서 Elasticsearch index를 구성하는 전체 데이터가 별도로 존재하고 자신은 그 중 일부만 가지고 있다는 사실을 알 수가 없다.
  - Primary shard의 개수를 변경한다는 말의 의미는 각각의 독립적인 Lucene이 가지고 있는 data들을 모두 재조정한다는 의미와 같다.
    - 만약 Lucene의 개수가 5개에서 10개로 늘어난다면 Lucene 내부에 가지고 있는 segment가 잘게 쪼개져 일부 segment들은 새롭게 추가된 Lucene 쪽으로 전송되어야 할 것이다.
    - 그리고 새로 추가된 Lucene에서는 여기저기에서 전송된 segment 조각들을 다시 합치는 작업을 수행해야 할 것이다.
    - 그러나 segment는 segment 불변성에 의해 변경이 불가능하다.
    - 따라서 primary shard의 개수를 변경하는 것은 불가능하다.





## Reindex

- `_reindex`

  - 한 index로 부터 다른 index로 문서들을 복제할 때 사용하는 API이다.
    - Reindex가 실행되는 순간의 source index의 snapshot을 생성하여 이 snapshot의 문서들을 복제하는 방식이다.
    - 오직 document만 복사하며, mapping, setting은 복제되지 않는다.

  - 기본적인 예시

  ```json
  // POST _reindex
  {
    "source": {
      "index": "my-index"
    },
    "dest": {
      "index": "my-new-index"
    }
  }
  ```

  - 여러 source에서 reindexing하는 것도 가능하다.

  ```json
  // POST _reindex
  {
    "source": {
      "index": ["my-index-1", "my-index-2"]
    },
    "dest": {
      "index": "my-new-index"
    }
  }
  ```



- `source`의 옵션들
  - `_source`
    - 복사할 field들을 입력한다.
  - `query`
    - Source index에서 query에 matching되는 document들만 복사할 수 있다.



- `dest`의 옵션들

  - `op_type`
    - `index`와 `create` 중 하나를 받는다.
    - `index`: dest index에 동일한 id를 가진 문서가 있을 경우 덮어쓴다.
    - `create`: dest index에 동일한 id를 가진 문서가 있을 경우 덮어쓰지 않는다. 
    - data stream의 경우 append-only이기 때문에 data stream에 reindexing 할 경우 반드시 `create`로 줘야한다.

  - `version_type`
    - `internal`, `external`, `external_gt`, `external_gte` 중 하나의 값을 받는다.
    - `internal`: 문서의 version과 무관하게 무조건 덮어쓴다.
    - `external`, `external_gt`: source index의 document version이 dest index의 document version보다 클 경우에만 기존 dest index의 document를 덮어쓰며, dest index의 document의 version이 source index에 있는 document의 version으로 변경된다.
    - `external_gte`: source index의 document version이 dest index의 document version보다 크거나 같을 경우에만 기존 dest index의 document를 덮어쓰며, dest index의 document의 version이 source index에 있는 document의 version으로 변경된다.
  - `pipeline`
    - Pipeline의 이름을 설정한다.



- 다른 cluster로부터 reindexing하기

  - 다른 cluster로부터 reindexing하기 위해선 dest cluster의 `elasticsearch.yml`에 `reindex.remote.whitelist` 설정이 되어 있어야한다.
    - Source cluster의 host:port를 입력하면 된다.

  ```yaml
  reindex.remote.whitelist: "otherhost:9200, another:9200, 127.0.10.*:9200, localhost:*"
  ```

  - Reindex 예시
    - 만약 source cluster의 secuirty가 disable 되어 있다면 username, password는 입력하지 않아도 된다.

  ```json
  POST _reindex
  {
    "source": {
      "remote": {
        "host": "http://otherhost:9200",
        "username": "user",
        "password": "pass"
      },
      "index": "my-index",
    },
    "dest": {
      "index": "my-new-index"
    }
  }
  ```





## Split index

- 이미 존재하는 index를 shard를 증가시켜 새로운 index로 생성한다.

  - Index의 primary shard를 늘리기 위해 사용한다.
    - `num_of_shards`는 static한 setting으로 index를 생성한 이후에는 변경할 수 없다.
    - 따라서 primary shard를 늘리기 위해서는 primary shard를 늘린 index를 생성 후 해당 index에 재색인 해야 하는데, split index API를 사용하여 상대적으로 간편하게 실행이 가능하다.
  - 기본 예시

  ```json
  // POST my-index/_split/split-my-index
  {
    "settings": {
      "index.number_of_shards": 2
    }
  }
  ```



- 기존 index의 shard를 split하여 새로운 index의 shard를 생성한다.
  - Index가 분할될 수 있는 횟수와 원본 shard가 분할 될 수 있는 회수는 `index.number_of_routing_shards`에 의해 결정된다.
    - 이 값은 documents들을 여러 shard에 고정된 hashing을 가지고 분배하기 위해 사용하는 hashing space를 설정한다.
    - 예를 들어 `number_of_routing_shard`가 30이고, index의 shard가 5이라고 할 때, 30은 `5*2*3`이므로, 5개의 shard를 가진 shard는 2 또는 3을 기준으로 분할 될 수 있다.
    - 즉 `5 → 10 → 30`(2로 분할 후 3으로 분할) 또는 `5 → 15 → 30`(3으로 분할 후 2로 분할) 또는 `5 → 30`(6으로 분할)와 같이 분할될 수 있다.
  - 아래 과정을 거쳐 split이 일어난다.
    - 먼저 source index와 동일한 definition을 가진 dest index를 생성한다.
    - Source index의 segment를 target index에 hard-link시킨다(만약 file system이 hard link를 지원하지 않을 경우, segment를 복제는데, 이 경우 시간이 더 오래 걸린다).
    - 모든 문서를 hashing하여 다른 shard에 속한 문서들을 삭제한다.
    - Target index를 recorver한다(close 상태의 index를 open 상태로 전환).



- Elasticsearch가 primary shard를 동적으로 증가시키지 못하게 막아놓은 이유.
  - 많은 key-value 저장소들은 incremental resharding을 지원한다.
    - 즉 shard의 개수를 동적으로 늘리는 기능을 지원한다.
  - shard의 크기가 일정 이상이 되면 새로운 shard를 생성하고, 새로운 shard에 새로운 data를 추가하는 방식은 좋지 않을 수 있다.
    - 하나의 shard에만 색인이 이루어지게 되므로 병목이 발생하기 쉽다.
    - Document들이 아무 기준 없이 색인되는 순서대로 shard에 들어가게 되므로 찾는데 시간이 오래 걸린다.
    - 따라서 이 방식을 사용했을 경우 일정한 hashing scheme에 따라 document들을 각 shard에 재배치하는 작업이 필요하다.
  - 다른 key-value 저장소들은 incremental sharding을 위해 일반적으로 consistent hashing을 사용한다.
    - 이 방식은 shard의 개수가 N에서 N+1로 증가했을 때 오직 1/N 번째 key만 재배치하면 된다. 
  - Elasticsearch의 저장 단위는 검색에 적합한 data 구조를 가진 Lucene index를 기반으로 만들어졌다.
    - 따라서 다른 key-value 저장소에 비해 매우 많은 비용이 든다.



# 데이터 삭제

- delete api를 사용하여 삭제가 가능하다.

  - 기본형

  ```bash
  DELETE /<index>/_doc/<document_id>
  ```



- ES에서 문서가 삭제되는 과정
  - delete api를 통해 삭제 요청을 보낸다.
  - 삭제 요청이 들어온 문서에 삭제 했다는 표시를 하고 검색시에 검색하지 않는다.
  - 세그먼트 병합이 일어날 때 삭제 표시가 되어 있는 문서를 실제로 삭제한다.



- query parameter
  - `if_seq_no`: document가 이 파라미터에 설정해준 sequence number를 가지고 있을 때만 삭제가 수행된다.
  - `if_primary_term`: document가 이 파라미터에 설정해준 primary term을 가지고 있을 때만 삭제가 수행된다.
  - `refresh`
    - `true`로 설정할 경우 delete의 변경사항을 즉각 반영(검색이 가능하게)한다. 
    - `wait_for`로 설정할 경우 refresh를 기다리다 refresh가 발생하면 변경 사항이 반영(검색이 가능하게)된다.
    - `false`로 설정하면 



- `delete_by_query`를 통해 특정 쿼리와 일치하는 문서를 삭제하는 것도 가능하다.

  - 삭제할 문서들을 지정해줄 query를 작성한다.

  ```json
  POST /test-index/_delete_by_query
  {
      "query":{
          "match":{
              "name":"test"
          }
      }
  }
  ```

  

  
