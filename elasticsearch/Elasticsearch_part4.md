# 인덱스 생성하기

- mapping & setting 없이 아래와 같이 인덱스명 만으로 생성이 가능하다.

  ```bash
  $ curl -XPUT 'localhost:9200/<인덱스명>'
  ```



- 인덱스명 규칙
  - 소문자만 사용이 가능하다.
  -  `\`, `/`, `*`, `?`, `"`, `<`, `>`, `|`, ` ` (space character), `,`, `#`는 사용이 불가능하다.
  - `-`, `_`, `+`로 시작할 수 없다.
  - `.` 혹은 `..`으로 생성할 수 없다.
  - 255 bytes 이상으로 생성할 수 없다.
  - `.`으로 시작하는 인덱스는 deprecate 될 예정이다.
    - 오직 hidden index와 plugin에서 관리하는 내부 인덱스에만 허용된다.



## mappings

- Mapping 기본형

  - `properties` 아래에 정의하고자 하는 필드들을 정의한다.

  ```json
  {
      "mappings":{
          "properties":{
              "<field_name>": {
                  "type":"<type>"
              }
          }
      }
  }
  ```



- `date_detection` 옵션

  - 미리 정의하지 않은 string 타입의 필드가 입력될 때, 만일 값이 date 필드의 형식에 부합하면 string이 아닌 date 타입으로 매핑한다.
    - boolean 값을 받으며, 기본값은 true이다.
    - `dynamic_date_formats`에 정의된 pattern과 일치하면 date type으로 본다.
    - `dynamic_date_formats`의 기본 값은 [ [`"strict_date_optional_time"`](https://www.elastic.co/guide/en/elasticsearch/reference/current/mapping-date-format.html#strict-date-time),`"yyyy/MM/dd HH:mm:ss Z||yyyy/MM/dd Z"`]이다.
  - `properties`와 동일한 수준에 정의한다.

  ```json
  // PUT index_name
  {
      "mappings":{
      	"date_detection": true,
          "properties": {
              // ...
          }
      }
  }
  ```
  
  - 예시
  
  ```json
  // PUT date_detection
  {
    "mappings": {
      "date_detection": true
    }
  }
  
  // PUT date_detection/_doc/1
  {
    "today":"2015/09/02"
  }
  ```
  
  - 결과
  
  ```json
  // GET date_detection/_mappings
  
  // response
  {
    "date_detection" : {
      "mappings" : {
        "properties" : {
          "today" : {
            "type" : "date",
            "format" : "yyyy/MM/dd HH:mm:ss||yyyy/MM/dd||epoch_millis"
          }
        }
      }
    }
  }
  ```



- `dynamic_date_formats`

  - `date_detection`이 `true`일 경우 새로운 string field의 값이 들어왔을 때, `date_detection`에 matching되는지 확인하고, matching되면 date type으로 색인한다.
  - 콤마로 구분 된 배열로 설정 가능하며, 배열 내의 하나의 요소는 `||`로 구분 된 string 형식으로 설정이 가능하다.
    - 배열 내에서 일치하는 첫 번째 format이 해당 field의 format이 된다.

  ```json
  // PUT my-index-000001
  {
      "mappings": {
          "dynamic_date_formats": [ "yyyy/MM", "MM/dd/yyyy"]
      }
  }
  
  // PUT my-index-000001/_doc/1
  {
    "create_date": "09/25/2015"
  }
  
  // 결과
  {
      "my-index-000001": {
          "mappings": {
              "dynamic_date_formats": [
                  "yyyy/MM",
                  "MM/dd/yyyy"
              ],
              "properties": {
                  "create_date": {
                      "type": "date",
                      "format": "MM/dd/yyyy"
                  }
              }
          }
      }
  }
  ```

  - `dynamic_date_formats`의 요소를 `||`로 구분 된 string 형식으로 설정할 경우
    - `dynamic_date_formats`의 요소를 `||`로 구분 된 string 형식으로 설정할 경우, string중 일부가 매칭되더라도 해당 string 전체가 format이 된다.
    - 따라서, `||`으로 구분 된 해당 string의 format 중 하나에라도 매칭이 된다면 색인이 가능하다.

  ```json
  // PUT my-index-000001
  {
      "mappings": {
          "dynamic_date_formats": [ "yyyy/MM||MM/dd/yyyy", "yyyy-MM-dd||yyyy-mm-dd"]
      }
  }
  
  
  // PUT my-index-000001/_doc/1
  {
      "create_date": "09/25/2015"
  }
  
  
  // 결과
  {
      "my-index-000001": {
          "mappings": {
              "dynamic_date_formats": [
                  "yyyy/MM||MM/dd/yyyy"
              ],
              "properties": {
                  "create_date": {
                      "type": "date",
                      "format": "yyyy/MM||MM/dd/yyyy"
                  }
              }
          }
      }
  }
  
  // PUT my-index-000001/_doc/1
  {
      // 아래와 같은 형식으로도 색인이 가능하다.
      "create_date": "2015/09"
  }
  ```



- `numeric_detection`

  - 미리 정의하지 않은 string 타입의 필드가 입력될 때, 만일 값이 숫자 형식이라면 string이 아닌 적합한 숫자 타입으로 매핑된다.
    - 기본값은 false이다.
  - `date_detection`과 같이 `properties`와 동일한 수준에 정의한다.

  - 예시

  ```json
  // PUT numeric_detection
  {
    "mappings": {
      "numeric_detection": true
    }
  }
  
  
  // PUT numeric_detection/_doc/1
  {
    "my_float":   "1.0", 
    "my_integer": "1" 
  }
  ```

  - 결과
    - 가장 적합한 숫자형 타입으로 매핑된다.

  ```json
  // GET numeric_detection/_mapping
  
  # response
  {
    "numeric_detection" : {
      "mappings" : {
        "numeric_detection" : true,
        "properties" : {
          "my_float" : {
            "type" : "float"
          },
          "my_integer" : {
            "type" : "long"
          }
        }
      }
    }
  }
  ```



- `dynamic` 옵션

  - mapping을 정의할 때 dynamic mapping을 허용할 것인지 여부를 정할 수 있는데, 설정할 수 있는 값은 아래와 같다.
    - `true`: dynamic mapping을 허용한다.
    - `runtime`: dynamic mapping을 허용한다.
    - `false`: dynamic mapping을 허용하지 않는다.
    - `strict`: dynamic mapping을 허용하지 않는다.
  - `properties`와 동일한 수준에 정의한다.

  - `true`와 `runtime`의 차이
    - 같은 데이터가 들어와도 어떤 타입으로 매핑을 생성할지가 달라진다.

  | JSON data type                              | true                                                 | runtime                                              |
  | ------------------------------------------- | ---------------------------------------------------- | ---------------------------------------------------- |
  | null                                        | No filed added                                       | No filed added                                       |
  | true/false                                  | boolean                                              | boolean                                              |
  | double                                      | float                                                | double                                               |
  | integer                                     | long                                                 | long                                                 |
  | object                                      | object                                               | No field added                                       |
  | array                                       | array 내부의 null이 아닌 첫 번째 값에 따라 달라진다. | array 내부의 null이 아닌 첫 번째 값에 따라 달라진다. |
  | date detection에 걸린 string                | date                                                 | date                                                 |
  | numeric detection에 걸린 string             | float 또는 long                                      | double 또는 long                                     |
  | date/numeric detection에 걸리지 않은 string | keyword를 sub field로 가지는 text 필드               | keyword                                              |

  - `false`
    - `false`는 정적으로 정의되지 않은 필드가 들어올 경우 이를 무시한다.

  ```json
  // false로 줄 경우
  // PUT test_false
  {
    "mappings": {
      "dynamic":"false",
      "properties": {
        "name":{
          "type":"keyword"
        }
      }
    }
  }
  
  // PUT test_false/_doc/1
  {
    "name":"theo",
    "age": 28
  }
  
  // GET test_false/_mapping
  // response
  // 위에서 입력한 age필드가 동적으로 매핑되지 않은 것을 확인 가능하다.
  {
    "test_false" : {
      "mappings" : {
        "dynamic" : "false",
        "properties" : {
          "name" : {
            "type" : "keyword"
          }
        }
      }
    }
  }
  
  ```

  - `strict`

  ```json
  // strict로 줄 경우
  // PUT test_strict
  {
    "mappings": {
      "dynamic":"strict",
      "properties": {
        "name":{
          "type":"keyword"
        }
      }
    }
  }
  
  // 아래와 같이 미리 정의하지 않은 age 필드를 입력하면 error가 발생한다.
  // PUT test_strict/_doc/1
  {
    "name":"theo",
    "age": 28
  }
  
  // response
  {
    "error" : {
      "root_cause" : [
        {
          "type" : "strict_dynamic_mapping_exception",
          "reason" : "mapping set to strict, dynamic introduction of [age] within [_doc] is not allowed"
        }
      ],
      "type" : "strict_dynamic_mapping_exception",
      "reason" : "mapping set to strict, dynamic introduction of [age] within [_doc] is not allowed"
    },
    "status" : 400
  }
  ```



- mapping 작성시 field명에 `.`을 넣으면  object 타입으로 생성된다.

  - 필드명에 `.`을 넣고 생성

  ```bash
  PUT mapping_test
  {
    "mappings": {
      "properties": {
        "user.name":{
          "type":"keyword"
        }
      }
    }
  }
  ```

  - 결과

  ```bash
  GET mapping_test/_mapping
  
  {
    "mapping_test" : {
      "mappings" : {
        "properties" : {
          "user" : {
            "properties" : {
              "name" : {
                "type" : "keyword"
              }
            }
          }
        }
      }
    }
  }
  ```



- `_source` 

  - `_source` 필드에 original data를 저장할지 저장하지 않을지 설정이 가능하다.
    - 아래와 같이 `_source.enabled`를 false로 주면 `_source` 필드를 저장하지 않는다.
    - 저장은 되지 않지만 색인은 되므로 검색은 가능하다.
    - 그러나 get이나 search를 했을 때 response에 `_source`필드, 즉 original data는 반환되지 않는다.
  
  ```json
  // PUT test
  {
    "mappings": {
      "_source": {
        "enabled": false
      }, 
      "properties": {
        "animal":{
          "type": "keyword"
        }
      }
    }
  }
  ```
  
  - 일부 데이터만 저장하거나, 일부 데이터를 저장하지 않는 것도 가능하다.
    - `_source.includes`, `_source.exclude`를 사용한다.
  
  ```json
  // PUT test
  {
    "mappings": {
      "_source": {
        "includes": [
          "*.count",
          "meta.*"
        ],
        "excludes": [
          "meta.description",
          "meta.other.*"
        ]
      }
    }
  }
  ```
  
  - `_source` field에 통으로 저장하지 않고 개별 field를 저장하는 것도 가능하다.
    - `store: true`로 주면 개별 필드를 저장할 수 있다.
    - 아래에서 title field는 field를 저장하고, content filed는 저장하지 않았다.
  
  ```json
  PUT my-index-000001
  {
    "mappings": {
      "properties": {
        "title": {
          "type": "text",
          "store": true 
        },
        "content": {
          "type": "text"
        }
      }
    }
  }
  
  // data 색인
  PUT my-index-000001/_doc/1
  {
    "title":   "Some short title",
    "content": "A very long content field..."
  }
  ```
  
  - 저장된 data를 보고자 하면 아래와 같이 하면 된다.
    - 결과를 보면 `store: true`로 준 title은 정보가 나오지만, content는 나오지 않는다.
  
  ```json
  GET my-index-000001/_search
  {
    "stored_fields": [ "title", "content" ] 
  }
  ```



- `store`

  - `_source` option에서 살펴봤듯, 기본적으로 개별 field들은 저장되지 않고, `_source` field에 일괄적으로 저장된다.
  - `store` option을 true로 주면 개별 field가 저장되게 해준다.

  ```json
  // PUT test_index
  {
      "mappings":{
          "properties":{
              "foo":{
                  "type": "text",
                  "store": true
              },
              "bar":{
                  "type":"text"
              }
          }
      }
  }
  ```

  - 용도
    - 예를 들어 `foo`와 `bar`라는 두 개의 field가 있다고 가정해보자.
    - `foo`는 짧은 text가 주로 저장되고, `bar`에는 매우 긴 text가 주로 저장되는데, 검색 결과에는 `foo`만 포함시켜도 된다.
    - 그렇다면 굳이 긴 `bar` field의 값을 반환할 필요는 없으므로, `foo` field만 반환받도록 하는 것이 효율적이다.
    - 따라서 `_source.exclude`를 통해 `bar` field는 저장하지 않도록 하고, `store`를 통해 foo만 저장하도록 한다..
    - 이를 위해  `foo` field를 저장해두고 검색시에 `stored_fields` 에 `foo` field만 입력하여 `foo` field만 반환받는다.
  
  ```json
  // PUT test_index/_doc/1
  {
      "foo":"foo",
      "bar":"looooooooooooooooooooooooooooooong text"
  }
  
  // GET test_index/_search
  {
      "stored_fields": ["foo"], 
      "query": {
          "match": {
              "foo": "foo"
          }
      }
  }
  
  // response
  {
      "hits": [
          {
              "_index": "test_index",
              "_id": "1",
              "_score": 0.18232156,
              "fields": {
                  "foo": [
                      "foo"
                  ]
              }
          }
      ]
  }
  ```
  
  - `_source`나 `fields`를 사용하는 것과 무엇이 다른가?
    - `_source`와 `fields`와는 달리, 해당 필드가 저장되었다는 것을 명시적으로 표현할 수 있다.
  - `store` 옵션은 Elasticsearch에서 권장하는 방식은 아니다.
    - 현재는 거의 사용하지 않는다.
    - 공식 문서에서는 `stored_fields`보다는 `_source`나 `fields`를 사용하는 것을 권한다.



- `term_vector`

  - Analysis를 거치면 term vector가 생성되는데, 이를 저장할지 말지를 결정하는 옵션이다.
    - 필드 단위로 설정해준다.
    - test 필드에만 설정이 가능하다.
  - term vector에는 아래와 같은 정보들이 포함되어 있다.
    - term들의 list
    - 각 term들의 positin(혹은 order)
    - 각 term들의 start & end offset
    - payload
  - 설정 가능한 값들은 다음과 같다.
    - `no`: Term vectors를 저장하지 않는다.
    - `yes`: Term만 저장한다.
    - `with_positions`: Term과 position만 저장한다.
    - `with_positions_offsets`: Term, postion, offset을 저장한다.
    - `with_positions_payload`: Term, position, payload를 저장한다.
    - `with_positions_offsets_payloads`: Term, postion, offset, payload를 저장한다.

  ```json
  // PUT my-index-000001
  {
    "mappings": {
      "properties": {
        "foo": {
          "type":"text",
          "term_vector": "with_positions_offsets"
        }
      }
    }
  }
  ```

  - 기본값이 `no`이므로 inverted index에 term vector가 전부 포함되어 있지는 않다.
    - inverted index에 어떤 정보까지 저장할지는 `index_options`에 따라 정해지는데, 기본값은 `positions`이다.
    - `postitions`는 doc number, frequency, term position까지 색인한다.
    - 따라서 `term_vector`값이 기본 값인 `no`로 설정되어 있어도 `match_phrase`와 같은 position 기반의 검색이 가능하다.
  - 주의사항
    - `with_positions_offsets`는 해당 필드의 size를 두 배로 늘린다.
    - Fast vector highlighter를 사용하려면 설정을  `with_positions_offsets`로 변경해야 한다.



- `copy_to`

  - 한 필드의 값을 다른 필드에 복사할 수 있는 기능이다.
    - 일반적으로 여러 필드를 한 필드를 통해 검색하기 위해 사용한다.
    - 여러 필드를 검색하는 것 보다 한 필드를 검색하는 것이 검색 성능이 더 뛰어나기 때문이다.

  - mapping 설정하기
    - 아래와 같이 하나의 필드만 설정해도 되고, array 안에 여러 개의 필드도 설정 가능하다.
    - object 형태의 필드에는 적용이 불가능하다.

  ```json
  // PUT test-inex
  {
    "mappings": {
      "properties": {
        "first_name": {
          "type": "text",
          "copy_to": "full_name" 
        },
        "last_name": {
          "type": "text",
          "copy_to": "full_name" 
        },
        "full_name": {
          "type": "text"
        }
      }
    }
  }
  ```

  - 데이터 색인하기

  ```json
  // PUT test-index/_doc/1
  {
    "first_name": "John",
    "last_name": "Smith"
  }
  ```

  - 검색
    - mapping상에는 존재하지만 검색 결과로 표출되지는 않는다.

  ```json
  // GET test-index/_search
  {
    "query": {
      "match": {
        "full_name": { 
          "query": "John Smith",
          "operator": "and"
        }
      }
    }
  }
  
  
  // output
  {
    "took" : 1,
    "timed_out" : false,
    "_shards" : {
      "total" : 1,
      "successful" : 1,
      "skipped" : 0,
      "failed" : 0
    },
    "hits" : {
      "total" : {
        "value" : 1,
        "relation" : "eq"
      },
      "max_score" : 0.5753642,
      "hits" : [
        {
          "_index" : "test-index",
          "_type" : "_doc",
          "_id" : "1",
          "_score" : 0.5753642,
          "_source" : {
            "first_name" : "John",
            "last_name" : "Smith"
          }
        }
      ]
    }
  }
  ```




- `ignore_above`

  - 설정 값 이상의 character를 가진 field는 색인되지 않도록 해준다.
    - 이미 존재하는 필드에도 동적으로 설정이 가능하다.
  - **`keyword`** 필드에만 설정 가능하다.

  - 예시

  ```json
  // PUT my-index-000001
  {
    "mappings": {
      "properties": {
        "message": {
          "type": "keyword",
          "ignore_above": 20	// character의 개수가 20개 이상이면 색인되지 않는다.
        }
      }
    }
  }
  
  // PUT my-index-000001/_doc/1 
  {
    "message": "Syntax error"		// 색인된다.
  }
  
  // PUT my-index-000001/_doc/2 
  {
    "message": "Syntax error with some long stacktrace"	// message field는 색인되지 않는다.
  }
  ```






### Field data types

#### 기본 타입

- ES에서 선언 가능한 문자열 타입에는 text, keyword 두 가지가 있다.
  - 5.0 버전부터는 텍스트 분석의 적용 여부를 text 타입과 keyword 타입으로 구분한다.
    - 2.X 버전 이전에는 string이라는 하나의 타입만 있었고 텍스트 분석 여부, 즉 애널라이저 적용을 할 것인지 아닌지를 구분하는 설정이 있었다.
  - 인덱스를 생성할 때 매핑에 필드를 미리 정의하지 않으면 동적 문자열 필드가 생성될 때 text 필드와 keyword 필드가 다중 필드로 함께 생성된다.



- text
  - 입력된 문자열을 텀 단위로 쪼개어 역 색인 구조를 만든다.
  - 보통은 풀텍스트 검색에 사용할 문자열 필드들을 text 타입으로 지정한다.
  - text 필드에는 아래와 같은 옵션들을 설정 가능하다.
    - `"analyzer" : "애널라이저명"`:  색인에 사용할 애널라이저를 입력하며 디폴트로는 standard 애널라이저를 사용한다. 토크나이저, 토큰필터들을 따로 지정할 수가 없으며, 필요하다면 사용자 정의 애널라이저를 settings에 정의 해 두고 사용한다.
    - `"search_analyzer" : "애널라이저명"`: 기본적으로 text필드는 match 쿼리로 검색을 할 때 색인에 사용한 동일한 애널라이저로 검색 쿼리를 분석하는데, `search_analyzer`를 지정하면 검색시에는 색인에 사용한 애널라이저가 아닌 다른 애널라이저를 사용한다. 보통 NGram 방식으로 색인을 했을 때는 지정 해 주는 것이 바람직하다.
    - `"index" : <true | false>`: 디폴트는 true이고, false로 설정하면 해당 필드는 역 색인을 만들지 않아 검색이 불가능해진다.
    - `"boost" : 숫자 값`: 디폴트는 1이다. 값이 1 보다 높으면 풀텍스트 검색 시 해당 필드 스코어 점수에 가중치를 부여하고, 1 보다 낮은 값을 입력하면 가중치가 내려간다.
    - `"fielddata" : <true | false>`: 디폴트는 false이고, true로 설정하면 해당 필드의 색인된 텀들을 가지고 집계 또는 정렬이 가능하다. 이 설정은 동적으로 변경하는 것이 가능하다.



- keyword

  - 입력된 문자열을 하나의 토큰으로 저장한다.
    - text 타입에 keyword 애널라이저를 적용한 것과 동일하다
  - 보통은 집계 또는 정렬에 사용할 문자열 필드를 keyword 타입으로 지정한다.
  - keyword 필드에는 다음과 같은 옵션들을 설정할 수 있다.
    - `index`, `boost` 옵션은 text와 동일하다.
    - `"doc_values" : <true | false>`: 디폴트는 true이며, keyword 값들은 기본적으로 집계나 정렬에 메모리를 소모하지 않기 위해 값들을 doc_values라고 하는 별도의 열 기반 저장소를 만들어 저장하는데, 이 값을 false로 하면 doc_values에 값을 저장하지 않아 집계나 정렬이 불가능해진다.
    - `"ignore_above" : 자연수`: 디폴트는 2,147,483,647이며 다이나믹 매핑으로 생성되면 256으로 설정된다. 설정된 길이 이상의 문자열은 색인을 하지 않아 검색이나 집계가 불가능하다. `_source`에는 남아있기 때문에 다른 필드 값을 쿼리해서 나온 결과로 가져오는 것은 가능하다.
    - `"normalizer" : 노멀라이저명`: keyword 필드는 애널라이저를 사용하지 않는 대신 노멀라이저의 적용이 가능하다. 노멀라이저는 애널라이저와 유사하게 settings에서 정의하며 토크나이저는 적용할 수 없고 캐릭터 필드와 토큰 필터만 적용해서 사용이 가능하다.
  - text와 keyword의 차이
    - 동적 매핑으로 문자열 필드를 생성하여 아래와 같이 text, keyword가 모두 생성된 경우, `필드`, `필드.keyword`로 모두 검색이 가능하다.
    - 그러나 text, keyword 필드에 검색했을 때 각기 다른 결과가 나오게 된다.
    - 상기했듯 text 필드는 문자열을 텀 단위로 쪼개기에 watching, movie 어느 것을 입력하든 watcing movie라는 문자열을 검색이 가능하다. 
    - 그러나 keyword 필드는 문자열을 하나의 토큰으로 저장하기에 watcing movie로 입력해야만 watcing movie라는 문자열을 검색이 가능하다.

  ```json
  // 데이터 삽입
  // POST test/_doc
  {
    "hobby":"watching movie"
  }
  
  
  // 매핑 확인
  // GET test/_mapping
  // response
  {
    "nation3" : {
      "mappings" : {
        "properties" : {
          "hobby" : {
            "type" : "text",		#  text 필드와
            "fields" : {
              "keyword" : {		# keyword 필드 모두 생성된다.
                "type" : "keyword",
                "ignore_above" : 256
              }
            }
          }
        }
      }
    }
  }
  
  
  // text 필드 검색
  // GET test/_search
  {
    "query":{
      "match": {
        "hobby": "watching"
      }
    }
  }
  // response
  {
    ...
    "hits" : {
      "total" : {
        "value" : 1,
        "relation" : "eq"
      },
      "max_score" : 0.2876821,
      "hits" : [
        {
          "_index" : "nation2",
          "_type" : "_doc",
          "_id" : "1",
          "_score" : 0.2876821,
          "_source" : {
            "hobby" : "watching movie"
          }
        }
      ]
    }
  }
  
  
  // keyword 필드 검색
  // GET "localhost:9200/test/_search
  {
    "query":{
      "match": {
        "hobby.keyword": "watching"
      }
    }
  }
  // 검색 결과가 나오지 않는다.
  
  
  // 아래와 같이 검색해야 결과가 나온다.
  // GET test/_search
  {
    "query":{
      "match": {
        "hobby.keyword": "watching movie"
      }
    }
  }
  ```



- 숫자

  - ES는 JAVA의 숫자 타입들을 지원한다.
    - JAVA에서 사용되는 숫자 타입들: long, integer, short, byte, double, float
    - ES에서만 지원하는 숫자 타입들: half_float, scaled_float
  - 사용 가능한 옵션들
    - `"index"`, `"doc_values"`, `"boost"` 등의 옵션들은 text, keyword 필드의 옵션들과 동일하다.
    - `"cource": <true | false>`: 디폴트는 true이며, 숫자 필드들은 기본적으로 숫자로 이해될 수 잇는 값들은 숫자로 변경해서 저장한다. 이를 false로 설정하면 정확한 타입으로 입력되지 않으면 오류가 발생한다. 
    - `"null_value": 숫자값`: 필드값이 입력되지 않거나 null인 경우 해당 필드의 디폴트 값을 지정한다. 
    - `"ignore_malformed": <true | false>`: 디폴트는 false로, 기본적으로 숫자 필드에 숫자가 아닌 불린 값이 들어오면 ES는 오류를 반환하는데, true로 설정하면 숫자가 아닌 값이 들어와도 도큐먼트를 정상적으로 저장한다. 해당 필드의 값은 `_source`에만 저장되고 겁색이나 집계에는 무시된다.
    - `"scaling_fator": 10의 배수`: scaled_float을 사용하려면 필수로 지정해야 하는 옵션으로 소수점 몇 자리까지 저장할지를 지정한다. 12.345라는 값을 저장하는 경우 `scaling_fator:10`과 같이 설정하면 실제로는 12.3이 저장되고, `scaling_fator:100`과 같이 설정했으면 12.34가 저장된다.
  - 전처리된 데이터가 아니면 항상 `_source`의 값은 변경되지 않는다.
    - `"cource": true`로 "4.5"라는 숫자가 integer 필드에 정상적으로 저장 되어도 `_source`의 값은 그대로 "4.5"이다.
    - `"null_value"`를 설정해도 역시 마찬가지로 `_source`에는 여전히 null로 표시된다.
  - `_source`에 저장된 값과 필드에 저장된 값이 다를 수 있다.
    - 예를 들어 byte 타입을 가진 필드에 4.5를 저장하는 경우에 byte는 오직 정수만을 저장하므로 4가 들어가게 된다.
    - 그러나 `_source`에는 4.5로 값이 들어가 있다.
    - 이 때 집계를 위해 3보다 크고 4.1보다 작은 값을 검색하면 4.5는 4.1보다 큼에도 필드에는 4로 저장되어 있으므로 검색이 되게 된다.
    - 이러한 이유로 숫자 필드를 동적으로 생성하는 것은 매우 위험하다.



- 날짜

  - ES에서 날짜 타입은 ISO8601 형식을 따라 입력한다.
    - "2021-03-18"
    - "2021-03-18T10:08:40"
    - "2021-03-18T10:08:40+09:00"
    - "2021-03-18T10:08:40.428Z"
    - ISO8601 형식을 따르지 않을 경우 text, keyword로 저장된다.
  - ISO8601외에도 long 타입 정수인 epoch_millis 형태의 입력도 가능하다.
    - epoch_millis는 1970-01-01 00:00:00부터의 시간을 밀리초 단위로 카운트 한 값이다.
    - 필드가 date 형으로 정의 된 이후에는 long 타입의 정수를 입력하면 날짜 형태로 저장이 가능하다.
    - epoch_millis외에도 epoch_second의 사용이 가능하다.
    - 사실 날짜 필드는 내부에서는 모두 long 형태의 epoch_millis로 저장한다.
  - 그 외에 사용 가능한 포맷들
    - 매핑의 format 형식만 지정 해 놓으면 지정된 어떤 형식으로도 색인 및 쿼리가 가능하다.
    - basic_date, strict_date_time과 같이 미리 정의 된 포맷들
    - [joda.time.format](https://www.joda.org/joda-time/apidocs/org/joda/time/format/DateTimeFormat.html) 심볼을 사용하여 지정 가능하다.
    - 정의된 포맷들은  [Elastic 홈페이지의 공식 문서](https://www.elastic.co/guide/en/elasticsearch/reference/current/mapping-date-format.html#built-in-date-formats)에서 볼 수 있으며 joda 심볼 기호들은 다음과 같다.

  | 심볼 | 의미                 | 예시) 2019-09-12T17:13:07.428+09.00 |
  | ---- | -------------------- | ----------------------------------- |
  | yyyy | 년도                 | 2019                                |
  | MM   | 월-숫자              | 09                                  |
  | MMM  | 월-문자(3자리)       | Sep                                 |
  | MMMM | 월-문자(전체)        | September                           |
  | dd   | 일                   | 12                                  |
  | a    | 오전/오후            | PM                                  |
  | HH   | 시각(0~23)           | 17                                  |
  | kk   | 시각(01-24)          | 17                                  |
  | hh   | 시각(01-12)          | 05                                  |
  | h    | 시각(1-12)           | 5                                   |
  | mm   | 분(00~59)            | 13                                  |
  | m    | 분(0~59)             | 13                                  |
  | ss   | 초(00~59)            | 07                                  |
  | s    | 초(0~59)             | 7                                   |
  | SSS  | 밀리초               | 428                                 |
  | Z    | 타임존               | +0900/+09:00                        |
  | e    | 요일(숫자 1:월~7:일) | 4                                   |
  | E    | 요일(텍스트)         | Thu                                 |

  - 사용 가능한 옵션들
    - `"doc_values"`, `"index"`, `"null_value"`, `"ignore_malformed"` 옵션들은 문자열, 숫자 필드와 기능이 동일하다.
    - `"format": "문자열 || 문자열..."`: 입력 가능한 날짜 형식을 ||로 구분해서 입력한다.
    - `format`을 설정하지 않을 경우 기본 값은 `"strict_date_optional_time||epoch_millis"`이다.
    - `strict_`가 붙은 format들은 년을 4자리 숫자, 월과 일을 2자리 숫자로 엄격하게 지켜야 한다는 의미이다.
    - 기본 값인 `strict_date_optional_time`는 `yyyy-MM-dd'T'HH:mm:ss.SSSZ` 또는 `yyyy-MM-dd` 형식을 의미한다.



- boolean
  - true, false 두 가지 값을 갖는 필드 타입이다.
  - "true"와 같이 문자열로 입력되어도 boolean으로 해석되어 저장된다.
  - 불리언 필드를 사용할 때는 일반적으로 term 쿼리를 이용해서 검색을 한다.
  - 사용 가능한 옵션들
    - `"doc_values"`, `"index"` 옵션들은 문자열, 숫자 필드와 기능이 동일하다.
    - `"null_value": true|false`: 필드가 존재하지 않거나 값이 null일 때 디폴트 값을 지정한다. 지정하지 않으면 불리언 필드가 없가나 값이 null인 경우 존재하지 않는 것으로 처리되어 true/false 모두 쿼리나 집계에 나타나지 않는다.



- Object

  - JSON에서는 한 필드 안에 하위 필드를 넣는 object, 즉 객체 타입의 값을 사용할 수 있다.
    - 보통은 한 요소가 여러 하위 정보를 가지고 있는 경우 object 타입 형태로 사용한다.
  - object 필드를 선언할 때는 다음과 같이 `"properties"`를 입력하고 그 아래에 하위 필드 이름과 타입을 지정한다.

  ```bash
  $ curl -XPUT 'localhost:9200/movies?pretty' -H 'Content-Type: application/json' -d '{
    "mappings": {
      "properties": {
        "characters": {
          "properties": {
            "name": {
              "type": "text"
            },
            "age": {
              "type": "byte"
            },
            "side": {
              "type": "keyword"
            }
          }
        }
      }
    }
  }
  ```

  - object 필드를 쿼리로 검색하거나 집계를 할 때는 `.`를 이용해서 하위 필드에 접근한다.

  ```bash
  $ curl "http://localhost:9200/movie/_search" -H 'Content-Type: application/json' -d'
  {  
  	"query": {    
  		"match": { 
  			"characters.name": "Iron Man" 
          }
      }
  }'
  ```

  - 역색인 방식
    - 역색인은 필드 별로 생성된다.
    - 즉 object 필드 내부의 값이 각기 따로 따로 역색인 구조를 갖는 것이 아니라 하나의 역색인 구조를 갖게 된다.
    - 아래와 같이 데이터를 입력하고, 검색을 하면 `characters.name`이 Loki 이면서 `characters.side`가 villain인 1번 문서만 검색 될 것 같지만 막상 검색을 해보면 둘 다 검색된다.

  ```json
  // 아래와 같이 2개의 문서를 삽입
  // PUT movies/_doc/1
  {
    "title": "The Avengers",
    "characters": [
      {
        "name": "Iron Man",
        "side": "superhero"
      },
      {
        "name": "Loki",
        "side": "villain"
      }
    ]
  }
  
  // PUT movies/_doc/2
  {
    "title": "Avengers: Infinity War",
    "characters": [
      {
        "name": "Loki",
        "side": "superhero"
      },
      {
        "name": "Thanos",
        "side": "villain"
      }
    ]
  }
  
  // 위에서 삽입한 문서를 검색한다.
  // GET movie/_search
  {
    "query": {
      "bool": {
        "must": [
          {
            "match": {
              "characters.name": "Loki"
            }
          },
          {
            "match": {
              "characters.side": "villain"
            }
          }
        ]
      }
    }
  }
  ```



- Nested

  - 만약에 object 타입 필드에 있는 여러 개의 object 값들이 서로 다른 역 색인 구조를 갖도록 하려면 nested 타입으로 지정해야 한다.
    - nested type으로 색인 된 값들은 각각 하나의 document로 count되어 `_cat` API로 문서의 개수를 확인하면 전체 문서 개수에 함께 집계된다.
    - `_count` API나 `_search`의 total hits를 통해 nested 문서는 합산하지 않은 문서의 개수를 알 수 있다.

  - nested 타입으로 지정하려면 매핑이 다음과 같이 `"type":"nested"`를 명시한다.
    - 다른 부분은 object와 동일하다.
  
  ```json
  curl -XPUT "http://localhost:9200/movie" -H 'Content-Type: application/json' -d'{
  "mappings":{    
    "properties":{      
      "characters":{        
        "type": "nested",        
          "properties": {          
            "name": {            
              "type": "text"          
            },          
            "side": {            
              "type": "keyword"          
            }        
          }      
        }    
      }   
    }
  }'
  ```
  
    - nested 필드를 검색 할 때는 반드시 nested 쿼리를 써야 한다. 
      - nested 쿼리 안에는 path 라는 옵션으로 nested로 정의된 필드를 먼저 명시하고 그 안에 다시 쿼리를 넣어서 입력한다.
      - nested 쿼리로 검색하면 nested 필드의 내부에 있는 값 들을 모두 별개의 도큐먼트로 취급한다.
      - object 필드 값들은 실제로 하나의 도큐먼트 안에 전부 포함되어 있다.
      - nested 필드 값들은 내부적으로 별도의 도큐먼트로 분리되어 저장되며 쿼리 결과에서 상위 도큐먼트와 합쳐져서 보여지게 된다.
  
    - 역색인 방식
      - Object 타입과 달리 필드 내부의 값들이 각각 역색인 된다.
      - 따라서 아래와 같은 검색 쿼리를 보내면 `characters.name`이 Loki 이면서 `characters.side`가 villain인 1번 문서만 검색되게 된다.
  
  ```json
  // 인덱스 생성
  // PUT movies
  {
    "mappings": {
      "properties": {
        "characters": {
          "type": "nested",
          "properties": {
            "name": {
              "type": "text"
            },
            "side": {
              "type": "keyword"
            }
          }
        }
      }
    }
  }
  
  //데이터 삽입
  // PUT movies/_doc/1
  {
    "title": "The Avengers",
    "characters": [
      {
        "name": "Iron Man",
        "side": "superhero"
      },
      {
        "name": "Loki",
        "side": "villain"
      }
    ]
  }
  
  // PUT movies/_doc/2
  {
    "title": "Avengers: Infinity War",
    "characters": [
      {
        "name": "Loki",
        "side": "superhero"
      },
      {
        "name": "Thanos",
        "side": "villain"
      }
    ]
  }'
  
  // nested query를 사용하여 검색
  // GET movie/_search
  {
    "query": {
      "nested": {
        "path": "characters",
        "query": {
          "bool": {
            "must": [
              {
                "match": {
                  "characters.name": "Loki"
                }
              },
              {
                "match": {
                  "characters.side": "villain"
                }
              }
            ]
          }
        }
      }
    }
  }
  ```



- array

  - 공식적으로 지원하는 타입은 아니지만, 사용은 가능하다.
    - 따라서 정적 매핑으로 array type을 지정해주는 것은 불가능하다.
    - 그냥 한 필드에 타입이 동일한 여러 개의 데이터를 넣으면  array 가 된다.
  - 기본적으로 하나의 array 안에는 모두 같은 타입의 데이터가 들어가야 한다.
    - `['hello', 'world']`는 가능하지만 `['hello', 28]`은 불가능하다.
  - 예시

  ```json
  // PUT my-index-000002
  {
    "mappings":{
      "properties":{
        "name":{
          "type":"keyword"
        }
      }
    }
  }
  
  // PUT my-index-000002/_doc/1
  {
    "name":["theo","oeht"]
  }
  
  // GET my-index-000002/_search
  {
    "query": {
      "match_all": {}
    }
  }
  ```

  - 응답

  ```json
  // ...
  {
      "_index" : "my-index-000002",
      "_type" : "_doc",
      "_id" : "1",
      "_score" : 1.0,
      "_source" : {
          "name" : [
              "theo",
              "oeht"
          ]
      }
  }
  // ...
  ```





#### 심화 타입

- histogram
  - 도수분포표를 그림으로 나타낸 것
  - ES에서의 histogram은 사전 집계된 숫자 데이터들을 저장하는 필드이다.
  - 두 쌍의 배열로 정의된다.
    - `values` 배열은 히스토그램의 버킷을 나타내는 `double` 타입의 숫자값들로, 반드시 오름차순으로 정렬되어 있어야 한다.
    - `counts` 배열은 각 버킷에 얼마나 많은 값들이 들어 있는지를 나타내는 `integer` 타입의 숫자값들로, 0 이상의 정수여야 한다.
    - 두 배열의 각 요소는 위치를 기반으로 대응하므로, 두 배열의 길이는 항상 같아야 한다.



- Geo
  - 위치 정보를 저장할 수 있는 Geo Point와 Geo Shape 같은 타입들이 있다.
  - Geo Point
    - 위도(latitude)와 경도(longitude) 두 개의 실수 값을 가지고 지도 위의 한 점을 나타내는 값이다.
  - Geo point는 다양한 방법으로 입력이 가능하다.



- `dense_vector`

  - Float 값으로 된 dense vector들을 저장하기 위한 field이다.
    - 7.3 version에 추가되었다.
    - 원래 최대 dimension 값이 2048이었으나 8.11 version부터는 4096까지 가능해졌다.
    - 이 field는 정렬과 집계의 대상이 될 수 없다.
    - 주로 kNN search에 사용한다.
  - 아래와 같이 색인하면 된다.
    - 별도로 mapping을 설정하지 않은 채 float으로 구성된 128에서 4096 사이의 크기를 가지는 array 값을 색인할 경우 자동으로 `dense_vector` field로 색인된다.
    - 아래에서 설명할 parameter들 중 `dims`를 제외하면 전부 8.0에서 추가됐다.

  ```json
  // PUT my-index
  {
    "mappings": {
      "properties": {
        "my_vector": {
          "type": "dense_vector",
          "dims": 3
        }
      }
    }
  }
  
  // PUT my-index/_doc/1
  {
    "my_vector" : [0.5, 10, 6]
  }
  ```

  - `element_type`
    - Vector를 encoding하기 위한 type을 지정한다.
    - float과 byte를 지원하며, 기본 값은 `float`이다.
    - float은 각 차원 마다 4-byte의 부동소수점으로 색인을 실행하고, byte는 각 차원 마다 1-byte의 integer로 색인을 진행한다.
    - byte로 색인을 하면 정확도는 낮아지지만 index의 크기를 상당히 줄일 수 있다.
    - byte로 색인할 경우 dimension은 -128에서 127 사이의 값이어야한다.
  - `dims`
    - Vector의 dimension을 설정한다.
    - 최대 값은 4096이며, 기본 값은 처음 색인 된 vector의 길이이다.
  - `index`
    - 만약 true일 경우 kNN search API를 사용할 수 있으나 색인 시간이 상당히 증가한다.
    - 기본 값은 true이다.
  - `index_options`
    - HNSW algorithm은 자료 구조가 어떻게 생성될지 결정하는 두 개의 parameter를 가지고 있다.
    - 이들은 결과의 정확성과 검색 속도 사이의 trade-off를 조절한다.
    - `index`가 true일 때만 설정이 가능하다.
    - `m`: HNSW graph에서 각 node가 연결될 이웃의 수를 설정한다(기본값은 16).
    - `ef_construction`: 각 새 노드에 대한 최근접 이웃 목록을 수집하는 동안 추적할 후보 수를 설정한다(기본값은 100).
  - `similarity`
    - kNN search에서 vector 유사도를 판단하는 데 사용할 지표를 설정한다.
    - 각 document의 `_score` 값은 similarity 값으로 대체된다.
    - `index`가 true로 설정되었을 때만 설정이 가능하다.
    - 기본값은 `cosine`이다.



- `dense_vector` field의 `similarity` parameter에 설정할 수 있는 값들
  - `l2_norm`
    - L<sup>2</sup> distance(Euclidean distance)에 기반하여 유사도를 판단한다.
    - `_score`를 계산하는 식은 아래와 같다. 
    - $1/(1 + l2\_norm(query, vector)^2)$
  - `dot_product`
    - Dot product에 기반하여 유사도를 판단한다.
    - Cosine similarity를 계산하기 위한 최적화된 방식을 제공한다.
    - `element_type`의 값에 따라 score를 계산하는 방식이 달라진다.
    - `element_type`이 float일 경우 모든 vector들은 unit length여야 하며, `_score`는 아래와 같이 계산된다.
    - $(1+dot\_product(query,vector))/2$
    - `element_type`이 byte일 경우 모든 vector들이 같은 길이를 가지지 않을 경우 결과가 부정확해 질 수 있으며, `_score`는 아래와 같이 계산된다.
    - $0.5+(dot\_product(query,vector)/(32768*dims))$이고, dims는 각 vector 당 dimension의 수를 의미한다.
  - `cosine`
    - Cosine similarity를 사용하여 유사도를 판단한다.
    - 모든 vector를 unit length로 nornalize한 뒤 `dot_product`를 사용하는 것이 보다 효율적이다.
    - `cosine`은 원본 vector를 보존해야 하거나 normalize를 할 수 없을 때 사용하는 것이 좋다.
    - `_score`는 아래와 같이 계산된다.
    - $(1+cosine(query, vector))/2$
    - Magnitude가 0일 경우 cosine을 정의할 수 없기 때문에, magnitude가 0인 vector를 대상으로는 사용할 수 없다.
  - `max_inner_product`
    - Inner product의 최대값으로 유사도를 판단한다.
    - `dot_product`와 유사하지만 vector를 normalize 할 필요흔 없다.
    - 각 vector의 magnitude가 score에 중대한 영향을 미친다.
    - `_score`는 음수가 나오지 않도록 조정된다.
    - `max_inner_product`가 음수일 경우 `_score`는 아래와 같이 계산하고
    - $1 / (1+-1*max\_inner\_product(query,vector))$
    - 0 이상일 경우 아래와 같이 계산한다.
    - $max\_inner\_product(query,vector)+1$



#### Join field type

- Join field type

  - 한 index 내의 document들 사이에 parent/child 관계를 생성하는 field이다.
  - `relations` section에 `"parent_name":"child_name"` 형태로 parent/child 관계들을 넣는 방식으로 설정할 수 있다.
    - 복수의 관계를 설정하는 것도 가능하다.

  ```json
  // PUT my-index-000001
  {
    "mappings": {
      "properties": {
        // join_field의 이름을 설정하고
        "my_join_field": { 
          "type": "join",
          // "parent_name":"child_name" 형태로 관계를 설정한다.
          "relations": {
            "question": "answer" 
          }
        }
      }
    }
  }
  ```

  - 부모 문서 생성하기
    - 위에서 부모 문서의 이름을 question으로 설정했으므로, 아래와 같이 `name`에 question을 주면 부모 문서가 된다.
    - 아래 두 가지 방식은 모두 사용이 가능하지만, 

  ```json
  // PUT my-index-000001/_doc/1?refresh
  {
    "my_join_field": {
      "name": "question" 
    }
  }
  
  // PUT my-index-000001/_doc/2?refresh
  {
    "my_join_field": "question"
  }
  ```

  - 위와 같이 서로 다른 방식으로 등록한 문서를 검색해보면 아래와 같이 나온다.

  ```json
  // GET my-index-000001/_search
  {
      // ...
      "hits": [
        {
          "_index": "my-index-000001",
          "_id": "1",
          "_score": 1,
          "_source": {
            "my_join_field": {
              "name": "question"
            }
          }
        },
        {
          "_index": "my-index-000001",
          "_id": "2",
          "_score": 1,
          "_source": {
            "my_join_field": "question"
          }
        }
     ]
  }
  ```

  - 자식 문서 생성하기
    - 위에서 자식 문서의 이름을 answer로 설정했으므로, 아래와 같이 `name`에 answer를 주면 자식 문서가 된다.
    - 부모 문서와 달리 `parent`라는 값을 추가로 줘야 하는데, 여기에는 부모 문서의 `_id` 값을 주면 된다. 
    - 자식 문서는 부모 문서와 반드시 같은 shard에 색인되어야 하므로 `routing` 값을 줘야 하며, 부모 document의 `_id`값을 사용한다.
    - `routing`을 주지 않을 경우 기본적으로 document의 `_id` 값을 기반으로 routing하는데, 위에서 부모 document를 생성할 때 `routing` 값을 주지 않았으므로, 부모 document는 routing value로 자신의 `_id` 값을 사용했다.
    - 따라서 자식 문서 생성시 `routing` 값으로 부모 document의 `_id` 값을 주면 같은 shard에 색인되도록 할 수 있다.

  ```json
  // PUT my-index-000001/_doc/3?routing=1&refresh 
  {
    "my_join_field": {
      "name": "answer", 
      "parent": "1"
    }
  }
  ```

  - Depth를 더 줄 수도 있다.

  ```json
  // 하나의 부모와 하나의 자식
  // PUT my-index
  {
    "mappings": {
      "properties": {
        "my_id": {
          "type": "keyword"
        },
        "my_join_field": {
          "type": "join",
          "relations": {
            "question": "answer" # 부모인 question과 자식인 answer로 관계를 정의
          }
        }
      }
    }
  }
  
  // 하나의 부모와 복수의 자식
  // PUT my-index
  {
    "mappings": {
      "properties": {
        "my_id": {
          "type": "keyword"
        },
        "my_join_field": {
          "type": "join",
          "relations": {
            "question": ["answer","comment"]
          }
        }
      }
    }
  }
  
  // 더 높은 레벨의 부모와 자식 관계 설정
  // PUT my-index
  {
    "mappings": {
      "properties": {
        "my_join_field": {
          "type": "join",
          "relations": {
            "question": ["answer", "comment"],  
            "answer": "vote" 
          }
        }
      }
    }
  }'
  
  // 위의 경우 아래와 같은 관계가 설정 된 것이다.
  // question → answer → vote
  //          ↘ comment
  ```



- Join filed 와 성능

  - 관계형 DB의 join과 동일하게 사용하는 것이 아니다.
    - ES에서 성능 향상의 핵심은 데이터를 비정규화하는 것이다.
    - 각각의 join field에서 `has_child`나 `has_parent` 쿼리를 추가하는 것은 쿼리 성능에 상당한 악영향을 미친다.
  - join field를 사용할만한 유일한 경우는 한 entitiy가 다른 entity보다 훨씬 많은 일대 다 관계가 포함된 경우뿐이다.
    - 예를 들어 환자와 병원의 수가 있을 때, 환자의 수가 병원의 수 보다 훨씬 많다면 병원을 parent로, 환자를 child로 설정할 수 있다. 



- Parent-join의 제약사항
  - 한 인덱스에 오직 하나의 join field만 정의해야한다.
  - 이미 존재하는 join field에 새로운 관계를 추가하는 것은 가능하다.
  - Parent인 요소에 child를 추가하는 것은 가능하지만, parent가 아닌 문서에 child를 넣어서 parent/child 관계를 형성하는 것은 불가능하다.
  - 부모 문서와 자식 문서는 반드시 같은 샤드에 색인되어야 한다. 따라서 자식 문서를 색인, 조회, 삭제, 수정시에 같은 routing value가 입력되어야 한다.
  - 각 요소는 여러 자식을 가질 수 있지만, 부모는 오직 하나만 지닐 수 있다.



- Parent-join 검색
  - Parent-join은 document 내의 relation name을 index의 field에 추가한다.
  - 또한 Parent/child 관계 당 field 하나를 생성한다.
    - 이 field의 이름은 `<join_field_name>#<parent_name>` 형식이다.
    - 예를 들어 위에서 색인한 예시의 경우 `my_join_field#question` 형태로 생성된다.
    - 만일 document가 child document라면 이 field에는 document의 parent document의 `_id`값을 저장하고, parent document라면, 자신의 `_id` 값을 저장한다.
  - Join field가 포함된 index를 대상으로 검색할 때 위 두 field는 검색 결과에서 항상 반환된다.



- Global ordinals

  > https://www.elastic.co/guide/en/elasticsearch/reference/current/eager-global-ordinals.html
  
  - join field는 join의 속도 향상을 위해서 global ordinals을 사용한다.
    - global ordinals은 샤드에 변경이 있을 때마다 리빌드된다.
    - 따라서 더 많은 parent id가 샤드에 저장될 수록, 리빌드에도 더 많은 시간이 걸리게 된다.
    - 만약 인덱스가 변경되면,  global ordinals 역시 refresh의 일부로서 리빌드 된다.
    - 이는 refresh 시간에 상당한 영향을 미치게 된다.
    - 그러나 이는 어쩔 수 없다.
  - 만일 join 필드를 자주 사용하지는 않지만, join 필드에 값을 빈번하게 추가해야 할 경우 아래와 같이 `eager_global_ordinals`를 false로 주는 것이 좋다.
  
  ```bash
  $ curl -XPUT "http://localhost:9200/my-index" -H 'Content-Type: application/json' -d'{  "mappings": {    "properties": {      "my_join_field": {        "type": "join",        "relations": {           "question": "answer"        },        "eager_global_ordinals": false      }    }  }}
  ```

  - global ordinals의 heap 사용량을 체크
  
  ```json
  // Per-index
  // GET _stats/fielddata?human&fields=my_join_field#question
  
  // Per-node
  // GET _nodes/stats/indices/fielddata?human&fields=my_join_field#question
  ```



- `has_parent` query

  - 주어진 query에 match되는 parent document들의 child document들을 반환한다.
    - Join을 수행하기에 다른 query와 비교했을 때 느리며 matching되는 parent document의 개수가 증가할 수록 속도가 감소한다.
  - index 생성

  ```json
  // PUT test-index
  {
    "mappings": {
      "properties": {
        "my-join-field": {
          "type": "join",
          "relations": {
            "parent": "child"
          }
        },
        "name":{
          "type": "keyword"
        }
      }
    }
  }
  ```

  - 검색 예시
    - `name`이 foo인 document들의 child document들을 반환한다.

  ```json
  // GET test-index/_search
  {
    "query": {
      "has_parent": {
        "parent_type": "parent",
        "query": {
          "term": {
            "name": "foo"
          }
        }
      }
    }
  }
  ```

  - Parameter
    - `parent_type`: join field에 mapping된 parent의 이름을 입력한다.
    - `query`: child document들을 반환할 parent document들을 검색하기 위한 query를 작성한다.
  - 정렬
    - 일반적인 방식으로는 정렬할 수 없으며, function_score query를 `query` 내부에서 사용하여 parent document의 field로 정렬하는 것은 가능하다.



- `has_child` query

  - 주어진 query에 match되는 child document들의 parent document들을 반환한다.
    - `has_parent` query와 마찬가지 이유로, 다른 query와 비교해서 느리다.
  - 검색 예시

  ```json
  // GET test-index/_search
  {
    "query": {
      "has_child": {
        "type": "child",
        "query": {
          "match_all": {}
        },
        "max_children": 10,
        "min_children": 2,
        "score_mode": "min"
      }
    }
  }
  ```

  - Parameter
    - `type`: join field에 mapping된 child의 이름을 입력한다.
    - `query`: parent document들을 반환할 child document들을 검색하기 위한 query를 작성한다.
    - `max_children`: 쿼리와 일치하는 child documents의 최댓값을 지정, 만일 이 값보다 쿼리와 일치하는 child documents의 수가 많다면, 해당 child documents의 parent document는 반환되지 않는다.
    - `min_children`: 쿼리와 일치하는 child documents의 최솟값을 지정, 만일 이 값보다 쿼리와 일치하는 child documents의 수가 적다면, 해당 child documents의 parent document는 반환되지 않는다.
    - `score_mode`: 쿼리에 매칭된 child documents들의 점수가 어떻게 parent documents들의 관련성 점수에 영향을 줄 것인지를 결정한다. 기본 값은 None으로, avg, sum, min, max 등을 설정 가능하다.
    - `has_child` 쿼리는 일반적인 정렬로는 정렬할 수 없고, function_score를 사용하여 정렬해야 한다.





### Dynamic template


- Elasticsearch `dynamic_templates`에 있는 search_analyzer와 field에 설정된 search_analyzer 중 무엇을 사용하여 검색어를 분석하는가

  - 기본적으로 field에 `search_analyzer`를 따로 설정하지 않을 경우 `analyzer`에 설정된 analyzer를 search analyzer로 사용한다.

  - 그리고 이는 `dynamic_template`을 사용해 기본 mapping을 설정 할 때도 마찬가지로 `analyzer`만 설정할 경우 `analyzer`에 설정된 analyzer가 `search_analyzer`로도 사용된다.
  - 만약 `field`를 생성할 때 `analyzer`만 설정하고 `search_analyzer`는 설정해주지 않을 경우 해당 field는 search analyzer로 `dynamic_template`의 `search_analyzer`를 따르는지, 아니면 field의 `analyzer`를 따르는지 확인해 본다.





  - 테스트

    - 아래와 같이 test용 index를 생성한다.

    ```json
    // PUT test-index
    {
        "settings": {
            "analysis": {
                "analyzer": {
                    "default_analyzer": {
                        "type": "custom",
                        "tokenizer": "standard",
                        "filter":[
                            "lowercase",
                            "snowball"
                        ]
                    },
                    "my_analyzer": {
                        "type": "custom",
                        "tokenizer": "standard"
                    }
                }
            }
        },
        "mappings": {
            "dynamic_templates": [
                {
                    "string": {
                        "match_mapping_type": "string",
                        "mapping": {
                            "analyzer":"default_analyzer",
                            "search_analyzer":"default_analyzer"
                        }
                    }
                }
            ],
            "properties": {
                "foo": {
                    "type": "text",
                    "analyzer": "my_analyzer"
                }
            }
        }
    }
    ```

      - 아래와 같이 document를 색인한다.
        - 두 개의 필드에 완전히 동일한 내용을 색인한다.
        - 다만 `foo` field는 mapping에 설정한대로 `my_analyzer`를 통해 분석될 것이고, `bar` field는 `dynamic_templates`에 설정된대로 `default_analyzer`를 사용하여 분석된다.

    ```json
    // PUT test-index/_doc/1
    {
        "foo":"Quick brown foxes",
        "bar":"Quick brown foxes"
    }
    ```

      - 두 field를 대상으로 검색을 실행한다.
        - `profile`을 통해 검색어가 어떤 analyzer를 통해 분석되었는지 확인할 수 있다.

    ```json
    // GET test-index/_search
    {
        "profile": true, 
        "query": {
            "multi_match": {
                "query": "Quick brown foxes",
                "fields": [
                    "foo",
                    "bar"
                ]
            }
        }
    }
    ```

      - 아래와 같은 내용을 확인할 수 있다.
        - `foo` field는 `my_analyzer`를 search analyzer로 사용하고, `bar` field는 `default_analyzer`를 search analyzer로 사용한다는 것을 볼 수 있다.
        - 어찌보면 당연한 결과인 것이, `dynamic_templates` 자체가 mapping에 미리 지정되지 않은 field들에 대해서만 적용되는 설정이므로 이미 mapping에 정의한 `foo` field는 `dynamic_templates`에 설정한 `analyzer`의 적용을 받지 않는 것이 맞다. 

    ```bash
    "((foo:Quick foo:brown foo:foxes) | (bar:quick bar:brown bar:fox))"
    ```







# Mapping explosion

> https://www.elastic.co/guide/en/elasticsearch/reference/current/mapping-explosion.html

- Mapping explosion
  - Mapping에서 field의 개수가 너무 많거나, 많아질 가능성이 있는 상황을 의미한다.
  - 아래와 같은 증상이 나타날 수 있다.
    - 간단한 검색에도 많은 시간이 걸린다.
    - 색인에 많은 시간이 걸린다.
    - Heap memory 사용량과 CPU 사용량이 증가한다.



- Mapping explosion을 방지하기 위한 방법들

  - Flattened data type을 사용한다.
    - 아래 예시와 같이 object 형식의 data를 색인하면, object 내의 data가 sub field로 생성된다.
    - 즉 foo field의 sub field로 bar, qux field가 생성된다.
    - 그러나 flattened type으로 색인할 경우 sub field를 생성하지 않고 object 내의 모든 data를 단일 field에 색인하여 전체 field의 개수를 줄일 수 있다.

  ```json
  // PUT test
  {
    "mappings": {
      "properties": {
        "foo":{
          "type":"flattened"
        }
      }
    }
  }
  
  // PUT test/_doc/1
  {
    "foo": {
      "bar": "baz",
      "qux": "quxx"
    }
  }
  
  // GET test/_mapping
  /* output
  {
    "test": {
      "mappings": {
        "properties": {
          "foo": {
            "type": "flattened"
          }
        }
      }
    }
  */
  ```

  - Dynamic mapping을 비활성화한다.





## settings

- `auto_expand_replicas`
  - data node의 수에 따라 레플리카 샤드의 수를 자동으로 늘린다.
  - `-`를 사이에 두고 하한값과 상한값을 설정한다(e.g. 0-3). 
    - 상한값은  `all`로 설정이 가능하다(e.g. 0-all)
    - 만일 `all`로 설정할 경우 [shard allocation awareness](https://www.elastic.co/guide/en/elasticsearch/reference/current/modules-cluster.html#shard-allocation-awareness)와 [`cluster.routing.allocation.same_shard.host`](https://www.elastic.co/guide/en/elasticsearch/reference/current/modules-cluster.html#cluster-routing-allocation-same-shard-host)는 무시된다.
  - 기본값은 false로 설정되어 있다.
  - 이 설정을 활성화할 경우 샤드 할당 정책으로 Index-level shard allocation filtering만을 적용 가능하다.



- Field의 최대 개수 설정하기

  - `index.mapping.total_fields.limit` 설정을 통해 field의 최대 개수를 제한할 수 있다.
    - Dynamic 설정으로 index 생성 후에도 변경 가능하다.
    - 기본값은 1000으로 field를 999개 까지 생성할 수 있다.

  ```json
  // PUT test/_settings
  {
    "index": {
      "mapping": {
        "total_fields": {
          "limit": 2
        }
      }
    }
  }
  ```



- `index.highlight.max_analyzed_offset`

  - 많은 양의 text data에 대하서 plain highlighting을 수행하는 것은 상당한 시간과 memory를 필요로한다.
    - 지나치게 많은 시간과 memory가 highlighting에 소모되는 것을 막기 위해서, highlight시에 분석되는 character의 개수를 제한할 필요가 있다.
  - Highlight 요청이 들어왔을 때 최대 몇 개의 character를 분석할지 결정한다.
    - 기본값은 1,000,000이다.
  - 아래와 같이 변경이 가능하다.
    - Dynamic option으로 close/open하지 않고도 바로 적용된다.

  ```json
  // PUT <index_name>/_settings
  {
    "index": {
      "highlight": {
        "max_analyzed_offset": 100000
      }
    }
  }
  ```

  - 이 option은 오직 offset이나 term vector 없이 색인된 text field를 대상으로 highlight해야 할 때만 유효하다.
    - Plain highlither를 사용할 때만 고려해야 할 사항이 아니라 offset이나 term vector 없이 색인된 text field를 대상으로 highlight해야 할 때는 고려해야한다.



