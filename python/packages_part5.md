# Keras

- Keras
  - 딥 러닝을 쉽게 구현 할 수 있게 도와주는 패키지이다.
  - 딥 러닝 구현을 위한 상위 레벨의 인터페이스들을 제공한다.



- 전처리

  - `Tokenizer()`
    - 토큰화와 정수 인코딩을 위해 사용한다.

  ```python
  from tensorflow.keras.preprocessing.text import Tokenizer
  
  tokenizer = Tokenizer()
  train_text = "The earth is an awesome place live"
  
  # 단어 집합 생성
  tokenizer.fit_on_texts([train_text])
  
  # 정수 인코딩
  sub_text = "The earth is an great place live"
  sequences = tokenizer.texts_to_sequences([sub_text])[0]
  
  print("정수 인코딩 : ",sequences)			# 정수 인코딩 :  [1, 2, 3, 4, 6, 7]
  print("단어 집합 : ",tokenizer.word_index)	# 단어 집합 :  {'the': 1, 'earth': 2, 'is': 3, 'an': 4, 'awesome': 5, 'place': 6, 'live': 7}
  ```

  - `pad_sequence()`
    - 모델의 입력으로 사용할 샘플의 길이를 동일하게 맞춰야 하는 경우가 있는데, 자연어 처리에서는 이를 패딩 작업이라 부른다.
    - 보통 숫자 0을 넣어서 샘플들의 길이를 맞춰준다.
    - `pad_sequence()`는 정해준 길이보다 길이가 긴 샘플은 자르고, 짧은 샘플은 값을 0으로 채운다.
    - `padding` parameter에 "pre"를 입력하면 앞에서부터 0을 채우고, "post"를 입력하면 뒤에 0을 채운다.

  ```python
  from tensorflow.keras.preprocessing.sequence import pad_sequences
  
  result = pad_sequences([[1, 2, 3], [1, 2, 3, 4], [1, 2]], maxlen=3, padding='pre')
  print(result)
  
  """
  [[1 2 3]
   [2 3 4]
   [0 1 2]]
  """
  ```



- Word Embedding

  - Word embedding
    - 텍스트 내의 단어들을 밀집 벡터(dense vector)로 만드는 것을 말한다.
    - 임베딩 벡터는 원-핫 벡터에 비해 상대적으로 저차원을 가지며 모든 원소의 값이 실수이다.
    - 단어를 원-핫 벡터로 만드는 과정을 원-핫 인코딩이라고 한다면, 단어를 밀집 벡터로 만드는 과정을 워드 임베딩이라 한다.
    - 밀집 벡터는 워드 임베딩 과정을 통해 나온 결과이므로 임베딩 벡터라고도 한다.
  - `Embedding()`
    - 단어를 밀집 벡터로 만든다.
    - 인공 신경망 용어로는 임베딩 층(embedding layer)을 만드는 역할을 한다.
    - 정수 인코딩이 된 단어들을 받아서 임베딩을 수행한다.
    - (number_of_samples, input_length)인 2D 정수 텐서를 입력 받는데, 각 sample은 정수 인코딩이 된 결과로, 정수의 시퀀스이다.
    - (number_of_samples, input_length, embedding_word_dimensionality)인 3D 텐서를 반환한다.

  ```python
  from tensorflow.keras.layers import Embedding
  import tensorflow as tf
  
  # 토큰화
  tokenized_text = [['Hope', 'to', 'see', 'you', 'soon'], ['Nice', 'to', 'see', 'you', 'again']]
  
  # 각 단어에 대한 정수 인코딩
  encoded_text = [[0, 1, 2, 3, 4],[5, 1, 2, 3, 6]]
  
  # 정수 인코딩 데이터가 아래의 임베딩 층의 입력이 된다.
  vocab_size = 7
  embedding_dim = 2
  embedding = Embedding(vocab_size, embedding_dim)
  
  x = tf.constant(encoded_text)
  
  y = embedding(x)
  print(y.shape)			# (2, 5, 2)
  ```



- Modeling

  - `Sequential()`
    - 인공 신경망 챕터에서 살펴본 입력층, 은닉층, 출력층 같은 층을 구성하기 위해 사용한다.
    - `Sequential`을 model로 선언한 뒤에 `model.add()`라는 코드를 통해 층을 단계적으로 추가한다.

  ```python
  from tensorflow.keras.models import Sequential
  
  model = Sequential()
  # 세 개의 층을 추가한다.
  model.add(...)
  model.add(...)
  model.add(...)
  ```

  - 아래는 전결합층(fully-connected layer)를 추가하는 예시이다.
    - `Dense`는 첫 번째 인자로 출력 뉴런의 수를 받는다.
    - `activation`에 활성화 함수를 입력한다.
    - 아래 예시에서는 3개의 입력층 뉴런과 1개의 출력층 뉴런을 만들었다.

  ```python
  from tensorflow.keras.models import Sequential
  from tensorflow.keras.layers import Dense
  
  model = Sequential()
  model.add(Dense(1, input_dim=3, activation='relu'))
  ```

  - 위와 같이 추가를 한 후 아래와 같이 추가가 가능하다.
    - 이 경우에 `input_dim`을 전달 받지 않는데, 이는 이전 층의 출력 뉴런의 수가 1개임을 알고 있기 때문이다.

  ```python
  model.add(Dense(1, activation='relu'))
  ```

  - `Sequential.summary()`를 통해 모델 정보를 요약해서 볼 수 있다.

  ```python
  model.summary()
  
  """
  Model: "sequential"
  ┏━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┳━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┳━━━━━━━━━━━━━━━━━┓
  ┃ Layer (type)                         ┃ Output Shape                ┃         Param # ┃
  ┡━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━╇━━━━━━━━━━━━━━━━━━━━━━━━━━━━━╇━━━━━━━━━━━━━━━━━┩
  │ dense (Dense)                        │ (None, 1)                   │               4 │
  ├──────────────────────────────────────┼─────────────────────────────┼─────────────────┤
  │ dense_1 (Dense)                      │ (None, 1)                   │               2 │
  └──────────────────────────────────────┴─────────────────────────────┴─────────────────┘
   Total params: 6 (24.00 B)
   Trainable params: 6 (24.00 B)
   Non-trainable params: 0 (0.00 B)
  """
  ```



- Compile & Training

  - `compile()`
    - 모델을 기계가 이해할 수 있도록 컴파일한다.
    - 손실 함수와 옵티마이저, 메트릭 함수를 선택한다.
    - 문제 유형에 따라 대표적으로 사용되는 손실 함수와 활성화 함수의 조합이 있다.

  ```python
  from tensorflow.keras.layers import SimpleRNN, Embedding, Dense
  from tensorflow.keras.models import Sequential
  
  vocab_size = 10000
  embedding_dim = 32
  hidden_units = 32
  
  model = Sequential()
  model.add(Embedding(vocab_size, embedding_dim))
  model.add(SimpleRNN(hidden_units))
  model.add(Dense(1, activation='sigmoid'))
  model.compile(optimizer='rmsprop', loss='binary_crossentropy', metrics=['acc'])
  ```

  - `fit()`
    - 모델의 훈련을 시작하는 함수이다.
    - 첫 번째 인자로 훈련 데이터, 두 번째 인자로 지도 학습에서 레이블 데이터를 받는다.
    - 만약 미니 배치 경사 하강법을 사용하고 싶지 않을 경우 `batch_size`를 None으로 넘기면 된다.

  ```python
  # 위에서 이어지는 코드이다.
  model.fit(X_train, y_train, epochs=10, batch_size=32)
  ```



- Evaluation & Prediction

  - `evaluate()`
    - 테스트 데이터를 통해 학습한 모델에 대한 정확도를 평가하는 데 사용한다.
    - 첫 번째 인자로 테스트 데이터, 두 번째 인자로 지도 학습에서 레이블 테스트 데이터를 받는다.

  ```python
  # 위에서 이어지는 코드이다.
  model.evaluate(X_test, y_test, batch_size=32)
  ```

  - `predict()`
    - 임의의 입력에 대한 모델의 출력값을 확인한다.
    - 첫 번째 인자로 예측하고자 하는 데이터를 받는다.

  ```python
  model.predict(X_input, batch_size=32)
  ```



- 모델의 저장과 로드

  - `save()`
    - 인공 신경망 모델을 hdf5 파일에 저장한다.

  ```python
  model.save("/path/to/model.h5")
  ```

  - `load_model()`
    - 저장한 모델을 불러온다.

  ```python
  from tensorflow.keras.models import load_model
  
  
  model = load_model("/path/to/model.h5")
  ```



- Functional API

  - 위에서 살펴본 설계 방식은 Sequential API를 사용한 것이다.
    - 그러나 Sequential API는 여러 층을 공유하거나 다양한 종류의 입력과 출력을 사용하는 등의 복잡한 모델을 만드는 일에는 한계가 있다.
    - 아래는 Sequential API로 모델을 만드는 예시이다.

  ```python
  from tensorflow.keras.models import Sequential
  from tensorflow.keras.layers import Dense
  
  model = Sequential()
  model.add(Dense(3, input_dim=4, activation='softmax'))
  ```

  - Functional API를 사용하면 보다 복잡한 모델을 생성할 수 있다.
    - Functional API는 각 층을 일종의 함수로 정의한다.
    - 그리고 각 함수를 조합하기 위한 연산자들을 제공하는데, 이를 이용하여 신경망을 설계한다.
    - 입력의 크기(shape)를 명시한 입력층(input layer)을 모델의 앞단에 정의해야한다.
  - 전결합 피드 포워드 신경망(Fully-connected FFNN)을 Functional API로 만들어보기
    - `Input()`에 입력의 크기를 정의한다.
    - 이전층을 다음층 함수의 입력으로 사용하고, 변수에 할당한다.
    - `Model()`에 입력과 출력을 정의한다.
    - 이를 model로 저장하면 Sequential API를 사용할 때와 마찬가지로 `model.compile`, `model.fit` 등을 사용 가능하다.

  ```python
  from tensorflow.keras.layers import Input, Dense
  from tensorflow.keras.models import Model
  
  
  inputs = Input(shape=(10,))
  hidden1 = Dense(64, activation='relu')(inputs)
  hidden2 = Dense(64, activation='relu')(hidden1)
  output = Dense(1, activation='sigmoid')(hidden2)
  model = Model(inputs=inputs, outputs=output)
  ```



- 다중 입력을 받는 모델

  - Functional API를 사용하면 다중 입력과 다중 출력을 갖는 모델도 만들 수 있다.

  ```python
  model = Model(inputs=[a1, a2], outputs=[b1, b2, b3])
  ```

  - 다중 입력을 받는 모델을 입력층부터 출력층까지 구현하면 아래와 같다.
    - 두 개의 입력층에서 분기되어 진행되다 마지막에 하나의 출력을 예측하는 모델이다.

  ```python
  from tensorflow.keras.layers import Input, Dense, concatenate
  from tensorflow.keras.models import Model
  
  # 두 개의 입력층
  inputA = Input(shape=(64,))
  inputB = Input(shape=(128,))
  
  # 첫번째 입력층으로부터 분기되어 진행되는 인공 신경망
  x = Dense(16, activation="relu")(inputA)
  x = Dense(8, activation="relu")(x)
  x = Model(inputs=inputA, outputs=x)
  
  # 두번째 입력층으로부터 분기되어 진행되는 인공 신경망
  y = Dense(64, activation="relu")(inputB)
  y = Dense(32, activation="relu")(y)
  y = Dense(8, activation="relu")(y)
  y = Model(inputs=inputB, outputs=y)
  
  # 두개의 인공 신경망의 출력을 연결
  result = concatenate([x.output, y.output])
  
  z = Dense(2, activation="relu")(result)
  z = Dense(1, activation="linear")(z)
  
  model = Model(inputs=[x.input, y.input], outputs=z)
  ```



- Keras Subclassing API

  - Sequential API, Functional API 외에도 Subclassing API라는 구현 방식이 존재한다.

  - 서브 클래싱 API로 구현한 선형 회귀
    - 클래스 형태의 모델은 `tf.keras.Model`을 상속받는다.
    - `__init__()`에서 모델의 구조와 동작을 정의하는 생성자를 정의한다.

  ```python
  import tensorflow as tf
  
  
  class LinearRegression(tf.keras.Model):
    def __init__(self):
      super(LinearRegression, self).__init__()
      self.linear_layer = tf.keras.layers.Dense(1, input_dim=1, activation='linear')
  
    def call(self, x):
      y_pred = self.linear_layer(x)
      return y_pred
  
  model = LinearRegression()
  X = [1, 2, 3, 4, 5, 6, 7, 8, 9] # 공부 시간
  y = [11, 22, 33, 44, 53, 66, 77, 87, 95] # 각 공부 시간에 맵핑될 성적
  sgd = tf.keras.optimizers.SGD(lr=0.01)
  model.compile(optimizer=sgd, loss='mse', metrics=['mse'])
  model.fit(X, y, epochs=300)
  ```

  - 서브 클래싱 API를 사용해야하는 경우
    - 대부분의 딥 러닝 모델은 Functional API 수준에서도 전부 구현이 가능하다.
    - 그러나 새로운 수준의 아키텍처를 구현해야 하는 실험적 연구를 해야 할 경우 서브 클래싱 API를 사용해야한다.



- 세 가지 구현 방식의 비교
  - Sequential API
    - 단순히 층을 쌓는 방식으로 사용이 간편한다.
    - 다만 다수의 입력, 다수의 출력을 가진 모델 또는 층 간의 연결이나 덧셈과 같은 연결을 하는 모델을 구현하기에는 적합하지 않다.
  - Functional API
    - Sequntial API로는 구현하기 어려운 복잡한 모델을 구현할 수 있다.
    - 입력의 크기(shape)을 명시한 입력층(input layer)을 모델의 앞단에 정의해야한다.
  - Subclassing API
    - Functional API로도 구현할 수 없는 모델들을 구현할 수 있다.
    - OOP에 익숙해야 하므로 구현이 가장 까다롭다.

