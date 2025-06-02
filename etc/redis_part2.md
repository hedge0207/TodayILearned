# Redis Clients

## CLI

- redis-cli 실행

  - 아래 명령어로 redis-cli를 실행시킨다.
    - 기본 port는 6379이다.


  ```bash
  $ redis-cli [-h 호스트 주소] [-p 포트] [-n db 번호] [-s 소켓] [-a 비밀번호] [-u 서버 url]
  ```

  - 비밀번호 입력
    - redis-cli를 실행할 때 `-a`를 입력하지 않았으면 아래 명령어를 통해 인증이 가능하다.

  ```bash
  > auth <password>
  ```




- Redis 정보 확인

  ```bash
  > info
  ```

  - config 정보 확인

  ```bash
  > config get <확인할 정보>
  ```

  - grep 사용하기
    - redis-cli 내부가 아닌 외부에서 아래 명령어 실행

  ```bash
  $ redis-cli [-h 호스트 주소] [-p 포트] [-n db 번호] [-s 소켓] [-a 비밀번호] [-u 서버 url] | grep <찾을 내용>
  ```

  - 도움말

  ```bash
  > help
  ```

  - 모니터링

  ```bash
  > monitor
  ```



- 데이터 삽입

  - 옵션으로ttl(초)을 줄 수 있다.

  ```bash
  > set <key> <value> [ex seconds]
  ```

  - 데이터 여러 개 삽입

  ```bash
  > mset <key1> <value1> <key2> <value2> ...
  ```

  - list 자료형의 맨 앞 부터 삽입 삽입
    - 문자열이라도 `"`는 붙이지 않아도 된다.
    - space 로 구분한다.

  ```bash
  > lpush my_list Hello
  > lpush my_list World
  > lpush my_list Hello World
  > lpush my_list HelloWorld
  ```

  - list 자료형의 맨 뒤 부터 삽입
    - `rpush` 사용
    - 나머지는 `lpush`와 동일

  - 소멸 시간 지정해서 삽입
    - 단위는 초

  ```bash
  > setex <key> <시간> <value>
  ```




- 데이터 조회

  - 찾으려는 키가 없는 경우 `(nil)`을 반환한다.

  ```bash
  > get <key>
  ```

  - 데이터 여러 개 조회

  ```bash
  > mget <key1> <key2> ...
  ```

  - 리스트 데이터 앞에서부터 조회
    - 뒤의 숫자는 몇 번째 부터 몇 번째 까지를 조회할지를 선택하는 것이다.

  ```bash
  lrange my_list 0 -1
  
  # 출력, 마지막에 넣은 것이 가장 먼저 나온다.
  1) "HelloWorld"
  2) "World"
  3) "Hello"
  4) "World"
  5) "Hello"
  ```

  - 모든 key 확인
    - redis는 single thread이다.
    - 이 명령을 처리하기 위해 뒤의 작업들은 멈춰버리므로 가급적 사용을 자제하는 것이 좋다.
    - scan을 사용하는 것이 좋다.

  ```bash
  > keys *
  ```

  - scan
    - cursor 기반으로 key들을 출력한다.
    - 첫 번째 응답(`1)`)으로 다음번 cursor가 오는데 다시 이 것을 cursor애 넣고 명령어를 입력하는 것을 반복하다 0이 나오면 모든 key를 조회했다는 뜻이 된다.
    - 첫 scan에서 아무것도 안 나온다고 결과가 없는 것이 아닐 수도 있다.

  ```bash
  > scan <cursor> [Match pattern] [Count]
  
  # 예시
  > scan 0
  
  #응답
  1) "88"
  2)  1) "key62"
      2) "key71"
      3) "key85"
      4) "key19"
      5) "key92"
      6) "key84"
      7) "key20"
      8) "key40"
      9) "key34"
     10) "key21"
     11) "key2"
  ```




- 데이터 삭제

  - `(integer) 1`은 삭제 성공, `(integer) 0`은 삭제하려는 데이터가 없을 경우 반환된다.

  ```bash
  > del <key>
  ```

  - 모든 데이터 삭제

  ```bash
  > flushall
  ```

  - 리스트형 데이터에서 맨 뒤의 데이터 삭제
    - 맨 앞의 데이터 삭제는 `lpop`

  ```bash
  > rpop my_list
  ```

  - 리스트형 데이터에서 맨 뒤의 데이터 삭제 후, 삭제한 값을 다른 list에 삽입(deprecated)

  ```bash
  > rpop my_list other_list
  ```

  - 리스트형 데이터에서 head나 tail을 삭제하고 이를 다른 list의 head나 tail에 삽입

  ```bash
  # my_list의 tail(right)을 빼서 other_list의 head(left)에 삽입
  > lmove my_list other_list rigth left
  ```

  - 리스트형 데이터에서 일정 범위 제외하고 삭제
    - 삭제할 범위가 아닌 삭제하지 않을 범위를 지정한다.

  ```bash
  > ltrim my_list <시작> <끝>
  ```




- key 이름 변경

  - rename의 경우 변경하려는 이름의 key가 이미 존재할 경우 덮어 쓴다.
  - renamenx의 경우 변경하려는 이름의 key가 있을 경우 `(integer) 0`을 반환한다.

  ```bash
  > rename <key>
  
  > renamenx <key>
  ```

  - 리스트형 데이터 변경

  ```bash
  # my_list의 value인 리스트에서, 첫 번째 인자를 Bye로 변경
  LSET my_list 0 "Bye"
  ```




- 기타

  - 타임 아웃까지 남은 시간 확인
    - `(integer) -1`은 기한이 없는 경우, `(integer) -2`는 키 값이 없거나 소멸된 경우 반환된다.

  ```bash
  # 초 단위로 반환
  > ttl <key>
  
  # 밀리 초 단위로 반환
  > pttl <key>
  ```

  - 리스트 길이 확인

  ```bash
  > llen my_list
  ```

  - ttl(time to live) 제거하기
    - 특정 키를 삭제하지 않고 redis에 계속 저장하고 싶을 때 사용한다.

  ```python
  > persist <key>
  ```

  - ttl(time to live) 설정하기
    - 기본적으로 expire 명령을 통해 타임아웃을 설정해주지 않으면 -1(무기한)로 설정된다.

  ```bash
  > expire <key> <seconds>
  ```





## Python

- Python에서 사용하기

  - Redis 패키지 설치

  ```bash
  $ pip install redis
  ```

  - 테스트
    - 호스트와 포트를 지정해주고 `ping()` 메서드를 통해 테스트 해본다.
    - 그 밖에도 password, decode_response 등의 옵션을 줄 수 있다.
    - `Redis`, 혹은 `RedisStrict`를 사용할 수 있는데 둘 사이의 차이는 없다.
    - `RedisStrict`가 구버전에서 사용하던 것이다.

  ```python
  import redis
  
  
  r = redis.Redis(host="localhost",port=6379)
  # 비밀번호를 서정한 경우
  r = redis.Redis(host="localhost",port=6379,password=1234)
  print(r.ping())	# True
  ```

  - 데이터 삽입
    - list, set, dictionary 등은 삽입이 불가능하다.
    - json 패키지를 통해 dumps, loads를 사용하면 삽입, 조회가 가능하다.

  ```bash
  import redis
  
  
  r = redis.Redis(host="localhost", port=6379)
  r.set("key", "value")
  
  my_dict = {"a":1,"B":2}
  r.set("my_dict", jsonDataDict)
  result_data = r.get("my_dict").decode('utf-8')
  result = dict(json.loads(result_data))
  ```

  - 데이터 여러 개 삽입

  ```python
  import redis
  
  
  r = redis.Redis(host="localhost", port=6379)
  r.mset({"key":"value","key2":"value2"})
  ```

  - 데이터 조회

  ```bash
  import redis
  
  
  r = redis.Redis(host="localhost", port=6379)
  print(r.get("key")) 	# b'value'
  print(r.get("key").decode("utf-8"))		# value
  ```

  - scan을 통한 데이터 조회

  ```python
  import redis
  
  
  r = redis.StrictRedis('localhost', port=6379)
  init = 0 # cursor 값 0으로 스캔 시작
  
  while(True):
      ret = r.scan(init)
      print(init)
      init = ret[0]
      print(ret[1])
      if (init == 0): # 반환된 cursor 값이 0이면 스캔 종료
          break
  ```

  - 데이터 삭제

  ```bash
  import redis
  
  
  r = redis.Redis(host="localhost",port=6379)
  r.delete("key")
  ```

  - 데이터 전체 삭제

  ```python
  import redis
  
  
  r = redis.Redis(host="localhost",port=6379)
  r.flushdb()
  ```







# Redis 사용 전략

- Look Aside(==Lazy Loading)
  - 필요할 때만 데이터를 캐시에 로드하는 캐싱 전략
  - 캐시는 DB와 어플리케이션 사이에 위치한다.
    - 어플리케이션에서 data를 가져올 때 redis를 먼저 찔러보고, redis에 data가 있으면 data를 반환한다.
    - 없을 경우 어플리케이션은 DB에 data를 요청하고, 어플리케이션이 이 데이터를 받아 레디스에 저장한다.
  - 장점
    - 실제로 사용되는 데이터만 캐시할 수 있다.
    - 레디스의 장애가 어플리케이션에 치명적인 영향을 주지 않는다.
  - 단점
    - 캐시(redis)에 없는 데이터를 쿼리할 때 오랜 시간이 걸린다.
    - 캐시가 최신 데이터를 가지고 있다는 것을 보장하지 못한다(redis에 해당 key 값이 없을 때만 캐시에 대한 업데이트가 일어나기 때문).



- Write-Through
  - DB에서 데이터를 작성할 때마다 캐시에 데이터를 추가하거나 업데이트한다.
  - 장점
    - 캐시의 데이터는 항상 최신 상태로 유지된다.
  - 단점
    - 데이터 입력 시 두 번의 과정을 거쳐야 하기 때문에 지연 시간이 증가한다.

