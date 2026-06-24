# Generating Diverse Q&A Benchmarks for RAG Evaluation with DataMorgana

> https://arxiv.org/html/2501.12789v1

- 문제
  - 특정 도메인에서의 RAG 평가는 해당 도메인의 고유한 요구사항을 반영하는 벤치마크를 필요로 한다.
    - 가장 이상적인 데이터는 query log틀 통해 얻은 실제 사용자들의 question과 도메인 전문가가 제공한 golden answer 쌍이다.
    - 그러나 실제 데이터는 얻기 힘들기 때문에 일반적으로 LLM을 사용하여 synthetic data를 생성하여 사용한다.
  - 기존 솔루션들은 문서가 주어지면 그에 대한 질문을 생성해 질의응답(Q&A) 쌍을 만드는 범용적인 방식을 취한다.
    - 그러나 생성된 질문들이 개별적으로는 양호할 수 있지만, 실제 최종 사용자가 RAG 시스템과 상호작용하는 다양한 방식을 충분히 포괄할 만큼 다양하지는 않은 것이 일반적이다.



- 최근의 방법론들

  - Q&A pair를 생성하는 일반적인 방식은 생성후 필터링 하는 방식(generate then filter)이다.
    - 문서들의 corpus가 있을 때, 문서들의 subset을 만들고, LLM을 사용하여 각 문서로 답할 수 있는 질문들을 생성한다.
    - 그 다음 LLM이 문서를 기반으로 각 질문에 대해 답변을 생성하게 한다.
    - 마지막으로 생성된 (질문, 답변, 문서) tuple을 여러 기준(golden question과의 의미적 유사도, 다양성 등)에 따라 필터링한다.
  - InPars
    - InPars는 이러한 패러다임을 따르되, 답변 생성 과정은 건너뛰고 질문 생성에 집중한다. 
    - Few-shot 예제를 통해 LLM이 주어진 문서에 대한 질문을 생성하도록 유도한 뒤 각 (질문, 문서) 쌍을 둘 사이의 내적 유사도(inner similarity)에 따라 점수화하고, 높은 점수를 받은 쌍만을 선별한다.
  - Know Your RAG
    - 사용자가 시스템과 상호작용할 수 있는 다양한 방식을 포괄하기 위해 질문 유형의 분류체계(taxonomy)를 정의한다.
    - 질문 생성 과정은 세 단계로 이루어지는데, 먼저 문서를 여러 statement로 분해하고, 질문 유형에 따라 앞서 추출된 statement를 바탕으로 새로운 statement를 생성하며, 그중 하나의 statement를 질문 생성을 위한 기초 정보로 선택한다.
    - 모든 단계는 모두 LLM을 호출하여 수행한다.

  - RAGAs
    - RAG 시스템을 위한 유명한 평가 도구로, synthetic Q&A 벤치마크 생성 기능도 함께 지원한다. 
    - 먼저 코퍼스로부터 엔티티, 토픽, 그리고 이들 간의 관계를 식별해 지식 그래프(KG)를 생성한다. 
    - 이후 테스트 질문들은 이 지식 그래프를 기반으로 LLM 기반 접근법을 통해 생성된다.
    - Know Your RAG와 유사하게, RAGAs 역시 질문 유형(단일 홉 vs 다중 홉, 구체적 vs 추상적)뿐 아니라 사용자 페르소나(시니어, 주니어 등)를 함께 고려하여 생성되는 질문의 유형이 풍부해지고 다양성이 향상된다.

  - 이 밖에도 다양한 방법들이 제안되었다.
    - Promptagator, ARES는 동일한 파이프라인을 따르지만, 생성된 질문을 검색 쿼리로 특정 정보 검색(IR) 시스템에 입력했을 때 해당 질문과 연관된 문서가 결과 목록 상위에 나타나는 경우에만 해당 쌍을 유지한다.
    - Shakeri et al.은 파인튜닝된 인코더-디코더 모델을 사용해 입력 문서로부터 질문과 답변을 생성한 다음, 모델의 perplexity(혼란도) 점수를 기준으로 이를 필터링한다.
    - Yuan et al.은 LLM이 생성한 후보 질문들 중에서 고품질 질문을 선별하기 위한 프롬프트 기반 접근법을 제안했다.



- DataMorgana
  - DataMorgana는 주로 RAG 시스템, 그리고 Q&A 벤치마크를 필요로하는 그 외 시스템들의 학습 및 테스트를 위한 합성 벤치마크를 생성하기 위해 고안되었다.
    - 다른 도구들과 차별화되는 점은, 높은 다양성을 지닌 벤치마크를 손쉽게 생성할 수 있는 설정(configuration) 기능을 제공한다는 것이다.
    - DataMorgana는 사용자 카테고리와 질문 카테고리에 대한 세밀한 설정이 가능하며, 벤치마크 내에서 이들의 분포를 직접 제어하는 기능도 지원한다.
    - 가볍고 2단계 프로세스를 사용함으로써 효율적이고 빠른 반복 작업을 보장하는 동시에, 예상되는 실제 트래픽을 반영한 벤치마크를 생성한다.
    - Q&A 쌍이 생성되는 방식은 자연어 설명을 통해 설정하며, 이를 통해 비기술적인 사용자도 손쉽게 커스터마이징할 수 있다.
    - 다양한 생성 방식과 관련해서는, 최종 사용자와 질문 양측 모두에 대해 사전에 정의된 옵션에 제한받지 않고 다양한 범주(categorization)와 그 분포를 자유롭게 정의할 수 있도록 지원한다.
    - 이러한 범주들은 함께 결합되어 Q&A 쌍을 정의하는 조합적으로 매우 많은 수의 가능성을 만들어내며, 이를 통해 매우 다양한 벤치마크를 생성할 수 있다.
  - DataMorgana는 두 단계로 동작한다.
    - 첫 번째는 설정 단계로, DataMorgana 관리자(admin)가 자신이 원하는 요구사항을 명시하는 단계이다.
    - 두 번째는 생성 단계로, 입력된 설정 정보를 바탕으로 LLM의 도움을 받아 원하는 벤치마크를 실제로 생성하는 단계이다.



- 설정 단계

  - 설정(configuration) 단계에서는 질문과 최종 사용자(end-user) 양측 모두에 대해 세부적인 범주화(categorization)와 그에 따른 카테고리를 정의한다.
    - 이를 통해 RAG 애플리케이션에서 예상되는 트래픽에 대한 높은 수준(high-level)의 정보를 제공할 수 있다.
    - 질문과 사용자에 대한 범주화는 필요한 만큼 얼마든지 정의할 수 있다.
    - 단, 하나의 범주화 내에 속한 카테고리들은 상호 배타적(mutually exclusive)이어야 한다.
  - 이러한 설정은 원하는 대로 생성된 벤치마크를 커스터마이징하는 데 필요한 모든 정보를 담은 JSON 파일로 정의한다. 
    - 예를 들어, 사용자가 사실형(factoid) 질문과 비사실형(non-factoid) "경험(experience)" 질문을 생성하고자 한다면 아래와 같이 설정하면 된다.
    - `description`에는 자연어로 각 카테고리에 대한 설명을 설정한다.
    - `probability` 속성으로 각 카테고리의 분포 확률을 설정할 수 있다.

  ```json
  {
      "categories": [
          {
              "name": "factoid",
              "probability": 0.25,
              "description": "A question seeking a specific, concise piece of information or a short fact about a particular subject, such as a name, date, or number."
          },
          {
              "name": "non-factoid-experience",
              "probability": 0.75,
              "description": "A question to get advice or recommendations on a particular topic."
          }
      ]
  }
  ```

  - 최종 사용자(end-user)에 대한 범주화도 질문 범주화와 동일한 방식으로 정의한다. 
    - 아래 예시는 사용자의 전문성(expertise)을 정의하는 최종 사용자 범주화를 지정하는 방식이다.

  ```json
  {
      "categories": [
          {
              "name": "expert",
              "probability": 0.50,
              "description": "a specialized user with deep understanding of the corpus."
          },
          {
              "name": "novice",
              "probability": 0.50,
              "description": "a regular user with no understanding of specialized terms."
          }
      ]
  }
  ```



- 생성 단계

  - Benchmark는 Q&A 쌍을 한 번에 한 쌍씩 점진적으로 생성한다.
    - 각 쌍은 DataMorgana가 설정 파일을 기반으로 자동으로 생성한 프롬프트를 LLM에 전달하여 생성된다.
    - 설정 파일 내 구조화된 부분(e.g. 각 카테고리의 `name`, `probability`)은 내부적으로 DataMorgana가 구성하는 프롬프트를 생성하는 데 활용된다.
    - 이 때 `description` 값은 프롬프트에 그대로(as is) 삽입된다는 점에 주의해야 한다.
    - 이를 통해 DataMorgana 사용자는 상당한 자유도를 갖게 되며, 벤치마크를 생성하는 과정에서 필요에 따라 카테고리에 대한 설명을 자유롭게 반복적으로 수정(iterate) 할 수 있게 된다.
  - 하나의 Q&A 쌍은 아래 4단계를 거쳐 생성된다.
    - 먼저 사용자 카테고리 $u_i$와 질의 카테고리 $c_j$가 설정 파일에 명시된 분포 확률에 따라 각 범주별로 선택된다. 즉 ($u_1, c_1, ..., c_4$)와 같은 tuple이 생성된다. 이 튜플과 각 카테고리에 설정된 `description`이 함께 프롬프트 템플릿을 인스턴스화하는 데 사용된다. 이렇게 모든 카테고리 조합을 허용함으로써, 조합론적으로 매우 많은 수의 선택지가 가능해지며, 그 결과 매우 다양한 벤치마크가 만들어진다.
    - 문서 $d_i$가 RAG corpu에서 샘플링되어 프롬프트에 추가된다.
    - LLM이 instance화된 프롬프트를 전달 받아 문서 $d_i$에 대한 k개의 Q&A 후보 쌍을 생성한다.
    - 마지막으로 생성된 후보들이 프롬프트에 작성된 제약 조건을 충족하는지, 사용자 카테고리와 질의 카테고리에 속하는지, 문서에 충실한지(faithful)를 확인한다. 만약 조건을 모두 만족하는 쌍이 여러 개 있을 경우, 그중 하나가 샘플링된다.

  - 프롬프트 예시

  ```
  You are a user simulator that should generate [num_questions]
  candidate questions for starting a conversation.
  
  The [num_questions] questions must be about facts discussed in
  the documents you will now receive. When generating the questions,
  assume that the real users you must simulate, as well as the readers
  of the questions, do not have access to these documents. Therefore,
  never refer to the author of the documents or the documents
  themselves. Also, assume that whoever reads the questions will read
  each question independently. The [num_questions] questions must
  be diverse and different from each other. Return only the questions
  without any preamble. Write each pair in a new line, in the following
  JSON format: ’{"question": <question>, "answer": <answer>}.’
  
  ### The generated questions should be about facts from the
  following document:
  [document (d_i)]
  
  ### Each of the generated questions must reflect a user with
  the following characteristics:
      - They must be [description of user category 1 (u_1)]
      - They must be [description of user category 2 (u_2)]
      …
  ### Each of the generated questions must have the following
  characteristics:
      - It must be [description of question category 1 (c_1)]
      - It must be [description of question category 2 (c_2)]
      …
  
  ```



- 결과

  - 질적 다양성
    - Q&A 셋 생성을 위한 다른 방법론들과 비교했을 때 보다 다양한 유형의 질문들이 나오는 것을 확인할 수 있다.
    - 다양성의 지표로 생성된 질문의 품사 구조를 사용했다.
    - DataMorgana로 생성한 질문들에서는 가장 흔한 품사 구조(의문사 동사 관사 형용사 복수명사)로 생성된 질문들이 top 3안에 드는 비율이 5.8%밖에 되지 않을 정도로 다양한 형태로 생성되는 것을 확인할 수 있다.

  ![image-20260622134158950](rag_part2.assets/image-20260622134158950.png)

  - 양적 다양성

    - N-Gram Diversity (NDG) Score: 1~4-gram에 대해 "고유 n-gram 수 / 전체 n-gram 수"를 합산하여 같은 단어 조합이 반복될수록 값이 낮아진다.

    - Self-Repetition Score (SRS): 4-gram을 기준으로, 다른 질문과 겹치는 4-gram을 하나라도 가진 질문의 비율을 계산하는데, 값이 높을수록 비슷한 표현이 여러 질문에서 재사용된다는 의미이다.
    - Compress Ratio (CR): 압축률 기반 다양성으로, 압축률이 높다는 것은 반복되는 패턴이 많다는 의미이다.
    - Homogenization Score (HS): 벤치마크 내 모든 질문 쌍을 비교해 평균 유사도를 계산한다.
    - 위 지표들로 비교한 결과 DataMorgana가 다른 방법들에 비해 더 다양한 질문들을 생성한 것을 확인할 수 있다.

  - 한계

    - 위에서 사용한 지표들은 전문가들이 사용할만한 어려운 전문 용어를 선호하는 편향이 있다.
    - 즉 전문 용어는 드물게 등장하기 때문에 고유 단어/구문 비율을 높여서 다양성 점수를 높게 받는 반면, 쉬운 용어를 쓰는 질문은 반복되는 단어가 많아서 오히려 다양성 점수가 낮게 나온다.
    - DataMorgana는 일부러 "환자(patient)"나 "비전문가(non-expert)" 같은 사용자 카테고리를 써서 쉬운 질문과 어려운 질문을 골고루 섞으려고 했는데, 이로 인해 현재의 측정 지표로는 오히려 손해(penalize)를 본다.
    - 새로운 다양성 지표 개발이 필요하다.



- 요약

  - DataMorgana의 핵심은 사용자 카테고리와 질문 카테고리를 조합하여 다양한 Q&A 쌍을 생성하는 것이다.
  - 예를 들어 아래와 같이 카테고리를 나눴다고 가정해보자.

  ```
  [사용자 카테고리] - 전문성
  - 초보자 (확률 50%)
  - 전문가 (확률 50%)
  
  [질문 카테고리 1] - 사실형/경험형
  - 사실형 질문 (확률 70%)
  - 경험형 질문 (확률 30%)
  
  [질문 카테고리 2] - 단일/복합
  - 단순 질문 (확률 60%)
  - 복합 질문 (확률 40%)
  ```

  - 질문을 하나 만들 때마다, 이 카테고리들에서 확률에 의해 하나씩 뽑아 tuple을 생성한 후 이를 자연어로 변환한다.

  ```
  조합 예시 1: 초보자 + 사실형 + 단순 질문
  → "아이폰13의 배터리 용량이 얼마예요?"
  
  조합 예시 2: 전문가 + 경험형 + 복합 질문  
  → "아이폰13과 14의 발열 관리 방식 차이가 실사용에서 체감되나요?
  ```



