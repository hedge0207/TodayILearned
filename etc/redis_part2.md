# Data Persistence

- 용어 정리
  - Persistence: 영속성, 데이터를 생성한 프로그램의 실행이 종료되어도 사라지지 않는 데이터의 특성.
  - Durability: 지속성, 성공적으로 수행 된 트랜잭션은 영원히 반영되어야 함.
  - RDB: 특정 시점의 redis 데이터셋을 일정한 간격마다 .rdb 파일로 저장하는 방식
  - AOF(Append Only File): 서버에 입력된 write operation(쓰기 명령)을 파일에 추가해서 저장하는 방식.
    - 서버가 시작될 때마다 입력됐던 명령을 재실행해 dataset을 다시 만들어낸다.
  - fsync: 파일 내부 저장 상태를 장치와 동기화시키는 함수(버퍼에 있는 데이터를 파일로 옮긴다).



- RDB 장단점
  - 장점
    - Redis의 부모 process가 수행해야 할 유일한 작업은 나머지 모든 작업을 수행 할 자식을 포크하는 것 뿐이므로, Redis의 성능을 최대화 한다.
    - 부모 인스턴스는 disk I/O와 같은 작업들을 절대 수행하지 않는다.
    - 데이터 센터나 아마존 S3로 전송할 수 있는 하나의 압축된 파일이므로 재난 복구에 매우 적합하다.
    - RDB는 AOF와 비교해서 큰 데이터 셋으로 더 빠르게 재시작된다.
  - 단점
    - 스냅샷을 생성 한 후 다음 스냅샷이 생성되기 전에 레디스가 작동을 멈춘다면 해당 기간만큼의 데이터는 손실된다.
    - 포크는 데이터 셋이 큰 경우 시간이 많이 소비될 수 있고, 데이터 셋이 크고 CPU의 성능이 좋지 않은 경우 몇 밀리초 또는 1초 동안 레디스의 클라이언트 서비스를 멈추게 할 수 있다.
    - AOF도 포크를 해야 하지만 지속성의 성능 감소 없이 로그를 rewrite 하는 빈도를 수정할 수 있다.



- RDB의 동작 방식
  - SAVE의 경우
    - 메인 프로세스에서 메모리에 있는 데이터를 새 RDB temp 파일에 작성한다.
    - 작성이 완료되면 기존 RDB 파일을 지우고 새 파일로 교체한다.
  - BGSAVE의 경우
    - fork를 통해 자식 프로세스를 생성한다.
    - 자식 프로세스는 임시 RDB 파일에 데이터 세트를 저장하기 시작한다.
    - 자식 프로세스가 새로운 RDB 파일 작성이 끝나면 이전의 것과 변경한다.



- AOF의 장단점
  - 장점
    - RDB에 비해 지속성이 훨씬 뛰어나다.
    - 다양한 fsync 정책을 가질 수 있다.
    - AOF가 너무 커졌을 때 백그라운드에서 자동으로 재작성 할 수 있다(rewrite).
    - 실수로 flushall과 같은 명령어를 입력하여 데이터가 전부 삭제되어도 해당 명령을 제외하고 그 동안 입력되었던 명령들을 재시작하여 데이터를 복구할 수 있다.
  - 단점
    - AOF 파일은 일반적으로 동일한 데이터셋의 RDB 파일보다 크다.
    - 정밀한 fsync 정책에서는 RDB 보다 느릴 수 있다.
    - 과거에는 특정 operation(BRPOPLPUSH 등)에서 버그가 발생해서 AOF가 다시 로드될 때 완벽한 데이터셋을 만들어 내지 못했다.



- AOF 관련 이슈
  - 내구성
    - 디스크의 데이터에 fsync를 얼마나 수행할지를 설정할 수 있다.
    - ` appendfsync alway`: 매번 새로운 command가 AOF에 추가될 때마다 fsync를 수행한다. 매우 느리지만 매우 안전하다.
    - ` appendfsync everysec`: fsync 매 초마다 수행한다. 스냅샷 만큼 빠르며, 재난 발생시 1초의 데이터를 잃을 수 있다(권장).
    - `appendfsync no`: 디스크에 쓰는 시점을 OS에 맡긴다(리눅스는 일반적으로 30초 간격으로 내려 쓴다).
  - AOF가 잘리는 경우
    - AOF 파일이 작성되는 동안 서버가 꺼지거나 AOF 파일 작성 시점에 저장 공간이 가득 찬 경우에 발생할 수 있다.
    - 잘못된 형식의 명령이 존재하는 경우 이를 무시하고 계속 진행하는 것이 기본값으로 설정되어 있다.
    - 그러나 설정을 변경하여 레디스를 강제 중단하도록 할 수 도 있다.
    - AOF 파일의 백업 파일을 생성하고 redis에서 제공하는 redis-check-aof 툴을 사용하면, 형식에 맞지 않는 명령을 수정할 수 있다.
  - AOF 파일이 손상되는 경우
    - 단순히 잘린 것이 아니고 손상된 경우에는 보다 까다롭다.
    - `--fix` 옵션 없이 redis-check-aof 툴을 실행 한 다음 문제점을 확인하고 파일 안에 해당하는 오프셋으로 이동해서 파일을 수동으로 복구할 수 있는지 확인하는 것이다.
    - 파일을 edis-check-aof 툴로 `--fix`을 넣어서 수정하도록 할 수 있지만 이렇게 하면 AOF의 유효하지 않은 부분에서 파일 끝까지의 부분이 제거될 수 있으므로 주의해야 한다.
  - rewrite
    - AOF는 파일의 크기가 너무 커지게 되면 자동으로 rewrite를 수행할 수 있다.
    - AOF의 목적은 모든 연산을 기록하는 것이 아닌, 현재 데이터셋을 생성할 수 있는 연산을 기록하는 것이다.
    - 예를 들어 현재 redis 내부에는 `{'some_key':1000}`이라는 하나의 데이터만이 존재하는데, 이 데이터는 `{'some_key':0}`부터 시작해서  value를 `+1`  씩 올리는 write 연산을 통해 총 1000번의 write 연산 결과 쓰여진 것이라고 가정한다.
    - 일반적으로 AOF 파일의 크기가 RDB 파일의 크기보다 큰 것은 이때문이다.
    - 이와 같이 AOF 파일의 크기가 지나치게 커질 경우 AOF 파일은 rewrtie를 1000번 진행하는 연산이 아닌 `set some_key 1000`이라는 연산을 작성하게 된다.
  - log rewrite 동작 방식
    - fork를 통해서 자식 프로세스를 생성한다.
    - 자식 프로세스는 임시 파일에 새로운 AOF를 작성한다.
    - 부모 프로세스는 인 메모리 버퍼에 새로운 변경 사항을 축적한다.
    - 자식 프로세스의 rewriting 파일 작성이 끝나면 부모 클래스는 신호를 받고 자식 프로세스가 생성한 AOF 파일 끝에 인 메모리 버퍼에 축적한 내용을 추가한다.
    - 레디스는 원자적으로 새로운 파일의 이름을 이전 파일의 이름으로 변경하고 새로운 파일에 새로운 데이터를 추가한다.



- 무엇을 사용해야 하는가?
  - 레디스를 캐시 용도로 사용할 때에는 RDB와 AOF를 사용하지 않는 것이 권장된다.
    - persistence 기능으로 인한 장애 발생 가능성이 크다.
    - fork로 인한 COW(Copy-On-Write)로 전체 성능 저하가 발생할 수 있다.
  - 두 가지 방식을 모두 사용하는 것을 권장한다.
  - Snapshotting
    - dump.rdb라 불리는 바이너리 파일에 데이터 셋을 저장한다.
    - 데이터에 N초마다 M개 이상의 변경 사항이 있는 경우저장하도록 설정하거나 SAVE, BGSAVE 명령을 수동으로 호출할 수 있다.
  - Log rewrting
    - 쓰기 작업이 수행될 수록 AOF의 크기가 커진다.
    - 클라이언트에 대한 서비스 중단 없이 백그라운드에서 AOF를 재구성할 수 있다.
    - BGREWRITEOF를 레디스에 요청할 때마다 메모리에서 현재의 dataset을 재구성하는 데 필요한 최소한의 명령 시퀀스를 작성한다.







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

