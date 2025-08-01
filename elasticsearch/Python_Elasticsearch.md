- `elasticsearch.AsyncElasticsearch.search()` 동작 과정

  - `AsyncElasticsearch.search()`가 호출된다.

  - `AsyncElasticsearch.transport.perform_request()`가 호출된다.

    - `AsynElasticsearch.transport`의 타입은 `AsyncElasticsearch` 생성시에 `transport_class`를 따로 설정하지 않을 경우 기본적으로 `AsyncTransport`가 된다.
    - 결국 `AsyncTransport.perform_request()`가 호출되는 것이다.

  - `AsyncTransport.perform_request()`가 호출된다.

    - `AsyncTransport.perform_request()` 내에서 `AsyncTransport.get_connection()`을 통해 connection을 가져온다.
    - `AsyncTransport.get_connection()` 메서드는 `AsyncTransport.connection_pool.get_connection()` 메서드를 호출한다.
    - `AsyncTransport.connection_pool`은 `ConnectionPool` class의 instance이며, `AsyncTransport`의 부모 클래스인 `Transport` 클래스가 init될 때 instance value로 생성된다. 
    - 초기에는 `ConnectionPool`을 상속 받는 `EmptyConnectionPool`로 먼저 생성되고, 이후에 `Transport.set_connections()`메서드를 통해 `ConnectionPool` class의 instance가 되거나, `DummyConnectionPool` class의 instance가 된다.
    - `ConnectionPool`과 `DummyConnectionPool`중 어느 것의 instance가 될지는 `AsyncElasticsearch`의 인스턴스를 생성할 때 입력한 넘긴 `hosts` 파라미터의 배열 내 elements 개수에 따라 결정되는데, 하나일 경우 `DummyConnectionPool`의 instance가, 둘 이상일 경우 `ConnectionPool`의 instance가 된다.

    - `ConnectionPool.get_connection()`은 `hosts`의 개수 만큼 생성된 `Connection`의 인스턴스 중 하나를 반환하며, `DummyConnectionPool.get_connection()`은 하나뿐인 `Connection`의 인스턴스를 반환한다.

    - `ConnectionPool`에 어떤 `Connection` 클래스의 인스턴스가 저장될지는 어떤 `Transport` class를 사용하는지에 따라 달라지며, `AsyncTransport`의 경우 `DEFAULT_CONNECTION_CLASS`라는 class attribute로 `AIOHttpConnection`가 설정되어 있어 `ConnectionPool`에 `AIOHttpConnection`의 인스턴스가 저장된다.
    - 결국 `AsyncTransport.get_connection()`가 호출되면, `AIOHttpConnection`의 인스턴스가 반환된다.

  - `AIOHttpConnection.perform_request()`가 호출된다.
    - 만약 `AIOHttpConnection.session`이 None이면 `AIOHttpConnection._create_aiohttp_session()` 메서드를 통해 `AIOHttpConnection.session`을 생성한다.
    - `AIOHttpConnection._create_aiohttp_session()`이 호출될 때, `AIOHttpConnection.loop`가 None이면 `asynio.get_running_loop()`를 통해 event loop을 가져와 `AIOHttpConnection.loop`에 할당한다.

    - `AIOHttpConnection.session`의 타입은 `aiohttp.ClientSession`이며, `aiohttp.ClientSession._connector`의 클래스는 `aiohttp.TCPConnector`가 된다.
    - `aiohttp.ClientSession`를 생성할 때 `loop` 파라미터에 `AIOHttpConnection.loop`를 넘겨 event loop도 함께 설정한다.
    - `aiohttp.TCPConnector`를 생성할 때 `limit`을 통해 동시에 통신 가능한 connection의 수를 설정하는데, 이 값은 `AsyncElasticsearch`의 instance를 생성할 때 인자로 넘긴 `maxsize` 값이 설정된다.

  - `AIOHttpConnection.perform_request()`는 실행되면서 `aiohttp.ClientTimeout`의 인스턴스를 생성한다.

    - 이 때 `aiohttp.ClientTimeout`에 설정되는 `total` 값은 전체 작업에 대한 타임아웃 시간을 의미한다.
    - 이 값은 `AIOHttpConnection.perform_request()`의 인자로 넘어온 `timeout` 값(`AsyncElasticsearch.search` 실행 시 `request_timeout`으로 넘긴 값)이 있으면 해당 값이 설정되며, 없을 경우 `AIOHttpConnection.timeout` 값(`AsyncElasticsearch` instance 생성시 `timeout`으로 넘긴 값)이 설정된다.

  ```python
  @attr.s(auto_attribs=True, frozen=True, slots=True)
  class ClientTimeout:
      total: Optional[float] = None			# 전체 작업에 대한 타임아웃 시간
      connect: Optional[float] = None			# 서버와의 연결을 설정하는 데 걸리는 시간 제한
      sock_read: Optional[float] = None		# 소켓으로부터 데이터를 읽는 시간 제한
      sock_connect: Optional[float] = None	# 소켓 연결을 설정하는 시간 제한
  ```

  - `AIOHttpConnection.perform_request()`는 `AIOHttpConnection.session.request()` 메서드를 호출한다.
    - `AIOHttpConnection.session`의 type은 `aiohttp.ClientSession`이므로 결국 `aiohttp.ClientSession.request()`가 호출되는 것이다.
    - `aiohttp.ClientSession.request()` 호출시에 `timeout` 파라미터에 위에서 생성한 `aiohttp.ClientTimeout`의 인스턴스를 넘긴다.
  - `aiohttp.ClientSession.request()`는 `aiohttp._RequestContextManager`를 실행한다.
    - 이 context manager 내에서 `aiohttp.ClientSession._request()` 메서드가 호출된다.
    - `aiohttp.ClientSession._request()` 메서드가 실행되면서 `aiohttp.helpers.TimeoutHandle` instance를 생성하는 데, 이 때 생성자의 `timeout` 파라미터로 앞에서 생성한 `aiohttp.ClientTimeout.total` 값을 넘긴다.
    - `TimeoutHandle.start()` 메서드가 실행되면서 event loop에 특정 시각에 도달할 경우 exception을 발생시키는 event를 등록한다.
    - 이후 `TimeoutHandler.timer()` 메서드를 실행해 `aiohttp.helpers.TimerContext` 인스턴스를 받아온다.

  ```python
  class TimeoutHandle:
      // ...
  
      def start(self) -> Optional[asyncio.Handle]:
          timeout = self._timeout
          if timeout is not None and timeout > 0:
              when = self._loop.time() + timeout		# 현재 시간에서 timeout을 더한 시각
              if timeout >= 5:
                  when = ceil(when)
              return self._loop.call_at(when, self.__call__)		# 이벤트 등록
          else:
              return None
      
      def timer(self) -> "BaseTimerContext":
          if self._timeout is not None and self._timeout > 0:
              timer = TimerContext(self._loop)
              self.register(timer.timeout)
              return timer
          else:
              return TimerNoop()
  
      def __call__(self) -> None:
          for cb, args, kwargs in self._callbacks:
              with suppress(Exception):			# Exception 발생시키도록
                  cb(*args, **kwargs)
  
          self._callbacks.clear()
  ```

  - `TimerContext`는 아래와 같다.
    - 컨텍스트에 진입할 때 `current_task()`를 통해 현재 실행 중인 코루틴(coroutine)을 감싸고 있는 `Task` 객체를 받아온다.
    - `self._cancelled`를 통해 취소되었는지를 확인하며, 취소되었으면 task를 취소하고, TimeoutError를 발생시킨다.
    - 컨텍스트에서 빠져나올 때, `asyncio.CancelledError`가 발생했고, `self._cancelled`가 True면 TimeoutError를 발생시킨다.

  ```python
  # aiohttp.helpers
  
  class TimerContext(BaseTimerContext):
      """ Low resolution timeout context manager """
  
      def __init__(self, loop: asyncio.AbstractEventLoop) -> None:
          self._loop = loop
          self._tasks = []  # type: List[asyncio.Task[Any]]
          self._cancelled = False
  
      def __enter__(self) -> BaseTimerContext:
          task = current_task(loop=self._loop)
  
          if task is None:
              raise RuntimeError(
                  "Timeout context manager should be used " "inside a task"
              )
  
          if self._cancelled:
              task.cancel()
              raise asyncio.TimeoutError from None
  
          self._tasks.append(task)
          return self
  
      def __exit__(
          self,
          exc_type: Optional[Type[BaseException]],
          exc_val: Optional[BaseException],
          exc_tb: Optional[TracebackType],
      ) -> Optional[bool]:
          if self._tasks:
              self._tasks.pop()
  
          if exc_type is asyncio.CancelledError and self._cancelled:
              raise asyncio.TimeoutError from None
          return None
  
      def timeout(self) -> None:
          if not self._cancelled:
              for task in set(self._tasks):
                  task.cancel()
  
              self._cancelled = True
  ```

  - `aiohttp.ClientSession._request()` 메서드는 호출되면서 `aiohttp.ClientSession._connector`(`aiohttp.connector.TCPConnector`)의 `connect()` 메서드를 실행하여 connection을 설정한다.
    - `connect()` 메서드는 `_available_connections()` 메서드를 통해 가용한 연결이 있는지 확인한다.
    - 가용한 연결이 있을 경우 `Connection` 클래스의 인스턴스를 생성하여 반환한다.
    - 가용한 연결이 없을 경우 가용한 연결이 생길 때 까지 대기하기 위해 `Future` 객체를 생성한 후 event loop에서 대기시키다.

  - `aiohttp.ClientSession._request()` 메서드는 호출되면서 `aiohttp.client_reqrep.ClientRequest`의 인스턴스를 생성한다.

    - `aiohttp.client_reqrep.ClientRequest.send()` 메서드를 통해 요청을 전송하는데, 이 때, `connect()` 메서드를 실행하여 받아온 `Connection` 클래스의 인스턴스를 파라미터로 전달한다.

    - `aiohttp.client_reqrep.ClientRequest.send()` 메서드는 `aiohttp.client_reqrep.ClientResponse`의 인스턴스를 반환한다.
    - `aiohttp.client_reqrep.ClientResponse.start()` 메서드를 통해 응답을 받아온다.
    - 여기서 받아온 응답을 호출 스택을 따라 올라가면서 적절히 변형하여 반환하고, 결국 최종 검색 결과를 받게 된다.

  ```python
  # aiohttp.client_reqrep.ClientRequest
  
  class ClientRequest:
      
      async def send(self, conn: "Connection") -> "ClientResponse":
          # ...
          # header 전송
          await writer.write_headers(status_line, self.headers)
          task: Optional["asyncio.Task[None]"]
          if self.body or self._continue is not None or protocol.writing_paused:
              # request body 전송하는 coroutine 생성
              coro = self.write_bytes(writer, conn)
              if sys.version_info >= (3, 12):
                  task = asyncio.Task(coro, loop=self.loop, eager_start=True)
              else:
                  task = self.loop.create_task(coro)
              if task.done():
                  task = None
              else:
                  self._writer = task
          # ...
          # response 생성 및 반환
          self.response = response_class(
              self.method,
              self.original_url,
              writer=task,
              continue100=self._continue,
              timer=self._timer,
              request_info=self.request_info,
              traces=self._traces,
              loop=self.loop,
              session=self._session,
          )
          return self.response
  ```



- Timeout 관련 설정들

  - AsyncElasticsearch Python client의 timeout 설정은 크게 두 개로 나눌 수 있다.
    - Elasticsearch Python client 객체를 생성할 때 설정하는 `request_timeout`과 `timeout`.
    - 단, `timeout`은 이전 버전에서 사용하던 파라미터 이름으로, 최신 버전에서는 deprecate 되었으며, 최신 버전에서는 `request_timeout`을 사용해야 한다.
    - 버전에 따라 둘 다 지원하는 경우도 있는데, 내부적으로 `timeout`만 설정할 경우 그 값을 `request_timeout`에 할당해서 사용하며, 둘 다 설정할 경우 ValueError가 발생하므로, `request_timeout`만 사용해야 한다.
    - Elasticsearch Python client 객체를 통해 메서드를 실행할 때 argument로 전달하는 `timeout`과 `request_timeout`.

  ```python
  import asyncio
  
  from elasticsearch import AsyncElasticsearch
  
  
  async def main():
      client = AsyncElasticsearch(
          hosts=["http://localhost:9200"],
          request_timeout=5
      )
      await client.search(index="test", timeout="1s", request_timeout=3)
  
  
  if __name__ == "__main__":
      asyncio.run(main())
  ```

  - 메서드의 argument로 전달하는 `request_timeout`은 Elasticsearch Python client 객체를 생성할 때 설정하는 `request_timeout`과 동일하다.
    - 즉 메서드를 실행할 때, `request_timeout` argument로 넘기지 않으면, client 생성시에 설정 된 `request_timeout`에 설정한 값이 기본값으로 설정된다.
    - 만약 메서드를 실행할 때, `request_timeout` argument를 넘기면, client 생성시에 설정 된 `request_timeout` 값을 덮어쓴다.
  - `request_timeout`
    - 이 값은 HTTP 연결 전체 과정의 timeout을 설정하는 값이다.
    - `aiohttp.ClientSession.request()`가 실행되면서 `TimeoutHandler.start()`가 실행되는 순간부터 카운트를 시작한다.
    - 즉, Elasticsearch에 실제 요청을 전송하기 전에 카운트가 시작된다.
  - 메서드의 argument로 전달하는 `timeout`은 Elasticsearch 내부에서 각 shard로 부터 응답을 대기할 시간을 설정하는 것이다.
    - 즉, shard가 이 시간 내에 응답을 반환하지 못 할 경우 timeout이 발생하게 된다.
    - Elasticseasrch의 `_search` endpoint로 요청을 보낼 때 설정하는 `timeout`과 완전히 동일한 설정이다.

  ```json
  // GET my_index/_search
  {
      "timeout": "1s",
      "query": {
          "match_all": {}
      }
  }
  ```

  - Timeout관련 error가 발생할 수 있는 경우.
    - 상기했듯 `request_timeout`은 실제 Elasticsearch로 요청을 보내기 전에 카운트가 시작된다.
    - Connection pool에서 connection을 얻지 못 한 task들은 가용한 connection이 생길 때 까지 대기한다.
    - 따라서 `request_timeout`내에 가용한 connection을 얻지 못하면 실제 요청을 보내지도 못 한 상태로 TimeoutError가 발생하게 된다.
    - 또한 기다린 끝에 connection을 얻어 요청을 보낸다 하더라도 응답을 기다리는 중에 `request_timeout`만큼의 시간이 지나게 되면 역시 TimeoutError가 발생하게 된다.
    - 따라서 실제 Elasticsearch에서 응답이 오래 걸리는 것이 아니라, 가용한 connection을 기다리다 Timeout이 발생하고 있다면 connection pool size를 늘릴 필요가 있다.
  - Connection pool 사이즈 설정하기
    - 아래와 같이 Elasticsearch client 객체를 생성할 때 `maxsize`를 통해 connection pool size를 늘릴 수 있다.
    - 이 값은 `AIOHttpConnection._limit` 값으로 설정되고, 최종적으로 `aiohttp.TCPConnector.limit` 값이 되어 동시에 연결할 수 있는 최대 connection의 개수가 된다.
    - `aiohttp.TCPConnector.limit`의 기본 값은 100인데, `AIOHttpConnection._limit`의 기본값이 10으로 설정되어 있어 `AsyncElasticsearch`를 생성할 때 별도로 설정하지 않을 경우 기본 값은 10이 된다.

  ```python
  client = AsyncElasticsearch(
      hosts=["http://localhost:9200"],
      request_timeout=5,
      maxsize=30
  )
  ```



- 일부 field만 update하기

  - 아래와 같이 문서 전체를 교체하는 방식이 아니라 일부 field만 수정할 수 있다.

  ```python
  from elasticsearch import Elasticsearch
  
  
  es_client = Elasticsearch("http://localhost:9200")
  INDEX_NAME = "my_index"
  
  es_client.update(index=INDEX_NAME, id="1", body={"doc":{"foo":"bar"}})
  ```


