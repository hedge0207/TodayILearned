# Lifespan Events

- Lifespan
  - Application이 실행되기 직전과 application이 종료된 직후에 단 한 번 실행될 로직을 정의 할 수 있는 기능이다.
  - 일반적으로 application 전체에서 사용할 resource들을 setting하기위해 사용한다.
    - DB connection pool이나 machine learning model등이 이에 해당한다.



- 예시

  - Application에서 machine learning model을 사용해야 한다고 가정해보자.
    - 이 model은 많은 endpoint들이 공유한다.
    - 이 model을 loading하기 위해선 많은 data를 disk에서 읽어와야 해서 loading하는 데 많은 시간이 걸리므로, 매 요청마다 loading하는 것이 불가능한 상황이다.
  - Lifespan 기능을 사용하여 model을 loading한다.
    - Model을 loading하기 위한 lifespan 함수를 `asynccontextmanager` decorator와 함께 비동기 함수로 선언한다.
    - Lifespan 함수에는 `yield`가 포함되어 있어야 하며, application이 실행되기 직전에 `yield`문 전까지 실행되고, application이 종료된 직후에 `yield`문 이후의 로직이 실행된다.
    - `FastAPI` 인스턴스 생성시 `lifespan` parameter에 lifespan 함수를 넘겨준다.

  ```python
  from contextlib import asynccontextmanager
  
  from fastapi import FastAPI
  
  
  def fake_answer_to_everything_ml_model(x: float):
      return x * 42
  
  
  ml_models = {}
  
  
  @asynccontextmanager
  async def lifespan(app: FastAPI):
      # ML model을 loading한다.
      ml_models["answer_to_everything"] = fake_answer_to_everything_ml_model
      yield
      # ML model을 정리한다.
      ml_models.clear()
  
  
  app = FastAPI(lifespan=lifespan)
  
  
  @app.get("/predict")
  async def predict(x: float):
      result = ml_models["answer_to_everything"](x)
      return {"result": result}
  ```



- 더 이상 지원하지 않는 방식

  - 기존에는 아래와 같은 방식을 사용했다.
    - `app.on_event`로 이벤트를 등록하는 방식이었다.

  ```python
  from fastapi import FastAPI
  
  app = FastAPI()
  
  items = {}
  
  @app.on_event("startup")
  async def startup_event():
      items["foo"] = {"name": "Fighters"}
      items["bar"] = {"name": "Tenders"}
  
  @app.on_event("shutdown")
  def shutdown_event():
      with open("log.txt", mode="a") as log:
          log.write("Application shutdown")
  ```

  - 더 이상 지원하지 않는 이유
    - Application의 시작 및 종료시에 로직이나 자원을 공유해야하는 경우가 많다.
    - 위 방식의 경우 startp과 shutdown을 서로 다른 함수에서 관리하므로 두 함수가 공유하는 로직이나 자원을 관리하는 것이 쉽지 않다.
    - 반면에, 위에서 살펴본 async context manager를 사용하는 방식은 하나의 함수를 사용하므로 공유가 훨씬 간편하다.

  ```python
  from contextlib import asynccontextmanager
  
  from fastapi import FastAPI
  
  
  @asynccontextmanager
  async def lifespan(app: FastAPI):
      # 하나의 함수에서 관리하면 start up과 shutdown시에 공유하는 자원을 보다 간편하게 사용할 수 있다.
      conn = SomeDB()
      yield
      conn.close()
  ```







# Backround Tasks

- Response를 반환한 뒤 실행할 background task들을 설정하는 것이 가능하다.
  - Request가 들어올 때 실행되어야 하지만, 굳이 그 결과를 client에게 반환할 필요는 없을 때 유용하게 사용할 수 있다.
  - 예를 들어 request가 들어올 때 마다 mail을 전송하여 알림을 보내거나, 시간이 오래 걸리는 작업을 backround에서 처리해야하는 경우에 유용하다.
  - 만일 보다 많은 자원을 필요로 하는 task를 backround에서 실행해야 할 경우 Celery 등의 보다 큰 tool을 사용하는 것이 좋다.



- `BackgroundTasks`

  - `BackgroundTasks` 예시

  ```python
  import time
  # BackroundTasks를 import한다.
  from fastapi import BackgroundTasks, FastAPI
  import uvicorn
  
  
  app = FastAPI()
  
  # backround에서 실행할 function을 정의한다.
  def long_time_task(message: str):
      time.sleep(30)
      print(message)
  
  
  # BackroundTasks를 parameter로 받는다.
  @app.get("/")
  async def send_notification(background_tasks: BackgroundTasks):
      # BackroundTasks instance에 task를 추가한다.
      background_tasks.add_task(long_time_task, "Hello World!")
      return
  
  
  if __name__ == "__main__":
      uvicorn.run(app)
  ```

  - `BackgroundTasks`는 `starlette.background`를 사용하여 구현하였다.
  - 동작 방식
    - `add_task`를 호출하면, 인자로 받은 function(task)를 `BackroundTasks` instance의 `tasks` attribute에 저장한다.
    - 실행시에는 `tasks`를 loop를 돌면서 `tasks`에 저장된 task들을 하나씩 실행한다.

  ```python
  class BackgroundTasks(BackgroundTask):
      def __init__(self, tasks: typing.Optional[typing.Sequence[BackgroundTask]] = None):
          self.tasks = list(tasks) if tasks else []
  
      def add_task(
          self, func: typing.Callable[P, typing.Any], *args: P.args, **kwargs: P.kwargs
      ) -> None:
          # Parameter로 받은 function과 argument들로 task를 생성한 후
          task = BackgroundTask(func, *args, **kwargs)
          # tasks에 넣고
          self.tasks.append(task)
  
      async def __call__(self) -> None:
          # tasks를 순회하면서
          for task in self.tasks:
              # 하나씩 실행시킨다.
              await task()
  ```



- 비동기 함수를 backround에서 실행시키기

  - 동기적으로 동작하는 함수를 실행시키는 것과 동일한 방식으로 `BackroundTasks`를 사용하여 비동기 함수를 backround에서 실행시킬 수 있다.
  - 예시

  ```python
  import asyncio
  
  from fastapi import BackgroundTasks, FastAPI
  import uvicorn
  
  
  app = FastAPI()
  
  
  async def long_time_task(message: str):
      asyncio.sleep(30)
      print(message)
  
  
  @app.get("/")
  async def send_notification(background_tasks: BackgroundTasks):
      background_tasks.add_task(long_time_task, "Hello World!")
      return
  
  
  if __name__ == "__main__":
      uvicorn.run(app)
  ```







# Async

- FastAPI는 async 기능을 제공한다.

  - Path operation function에 `async` keyword만 붙여주면 된다.
    - 10번의 요청을 연속적으로 보내도 비동기로 처리되므로 10초 조금 넘는 시간에 모두 처리된다.

  ```python
  import asyncio
  
  from fastapi import FastAPI
  import uvicorn
  
  app = FastAPI()
  
  
  @app.get("/async")
  async def async_func():
      await asyncio.sleep(10)
      return
  
  if __name__ == "__main__":
      uvicorn.run(app)
  ```

  - async를 붙이지 않았을 경우
    - 놀라운 점은 아래 예시에서 `async` keyword를 붙이지 않은 `/sync`역시 10번의 요청을 보냈을 때 100초 넘는 시간이 걸리는 것이 아니라 10초 좀 넘는 시간 내에 처리가 된다는 것이다.

  ```python
  import asyncio
  import time
  
  from fastapi import FastAPI
  import uvicorn
  
  app = FastAPI()
  
  
  @app.get("/async")
  async def async_func():
      await asyncio.sleep(10)
      return
  
  @app.get("/sync")
  def sync_func():
      time.sleep(10)
      return
  
  
  if __name__ == "__main__":
      uvicorn.run(app)
  ```

  - `async`를 붙이지 않았음에도 `async`처럼 동작하는 이유
    - FastAPI는 `async`가 붙지 않은 일반 path operation의 경우 main thread가 아닌 외부 threadpool에서 실행시킨다.
    - 이는 시간이 오래 걸리는 작업을 main thread에서 실행할 경우 다음 요청을 실행할 수 없기 때문이다.
    - 이러한 이유로 겉으로 보기에는 비동기적으로 보이는 것인데, 실제 내부 동작은 비동기와 전혀 관련이 없다.
  - `async`를 붙인 함수내에서 blocking을 할 경우
    - 이 경우 `async`를 붙이지 않은 일반 함수처럼 외부 threadpool에서 실행되는 것이 아니므로, 10번의 요청을 보냈을 때, 100초가 넘는 시간이 걸리게 된다.

  ```python
  import asyncio
  import time
  
  from fastapi import FastAPI
  import uvicorn
  
  app = FastAPI()
  
  
  @app.get("/async-blocking")
  async def async_blocking():
      time.sleep(10)
      return
  
  
  if __name__ == "__main__":
      uvicorn.run(app)
  ```




- `async`를 붙이지 않을 경우 외부 thread pool에서 테스크를 실행한다면, 굳이 `async`를 붙이지 않아도 빠른 처리가 가능한 것 아닌가?

  - 예를 들어 아래와 같은 API가 있을 때
    - 현재 thread pool 크기는 40이다.

  ```python
  from contextlib import asynccontextmanager
  import anyio
  import time
  import asyncio
  
  from fastapi import FastAPI
  
  @asynccontextmanager
  async def lifespan(app: FastAPI):
      # Load the ML model
      limiter = anyio.to_thread.current_default_thread_limiter()
      print("current thread pool size:", limiter.total_tokens)	# 40
      yield
  
  app = FastAPI(lifespan=lifespan)
  
  @app.get("/sync")
  def sync_handler():
      time.sleep(1)  # 동기 블로킹 (가짜 I/O)
      return {"message": "sync done"}
  
  @app.get("/async")
  async def async_handler():
      await asyncio.sleep(1)  # 비동기 non-blocking (가짜 I/O)
      return {"message": "async done"}
  
  
  if __name__ == "__main__":
      import uvicorn
      uvicorn.run(app, host="0.0.0.0", port=8000)
  ```

  - 아래와 같이 테스트를 하면
    - Thread pool의 크기인 40개의 요청을 동시에 보낸다.

  ```python
  import asyncio
  import httpx
  import time
  
  API_URL_SYNC = "http://localhost:8000/sync"
  API_URL_ASYNC = "http://localhost:8000/async"
  
  async def fetch(client, url):
      response = await client.get(url)
      return response.json()
  
  async def benchmark(url, total_requests=40):
      async with httpx.AsyncClient() as client:
          tasks = [fetch(client, url) for _ in range(total_requests)]
          start = time.perf_counter()
          results = await asyncio.gather(*tasks)
          elapsed = time.perf_counter() - start
          return elapsed, results
  
  async def main():
      print("Benchmarking sync handler")
      elapsed_sync, _ = await benchmark(API_URL_SYNC)
      print(f"sync handler: {elapsed_sync:.2f} seconds")
  
      print("Benchmarking async handler")
      elapsed_async, _ = await benchmark(API_URL_ASYNC)
      print(f"async handler: {elapsed_async:.2f} seconds")
  
  if __name__ == "__main__":
      asyncio.run(main())
  ```

  - 위 결과를 확인해보면 두 방식 사이의 속도 차이는 거의 나지 않는다.
    - 만약 위와 같이 thread pool이 고갈되지 않을 만큼의 요청만 들어온다면, 실제 둘의 실행 속도 차이는 나지 않는다.
    - 그러나 만약 thread pool 크기 이상의 요청이 들어간다면, 그 때부터는 동기 방식이 비동기 방식에 비해 느려지기 시작한다.
  - 아래와 같이 thread pool 이상의 요청을 전송한다.
    - 겨우 1건 늘렸을 뿐인데, 두 방식 사이의 처리 시간이 약 2배 정도 차이가 나게 된다.
    - 만약 요청 건수를 81로 설정하여 thread pool 크기의 2배를 초과하는 요청을 보내게 되면, 3배 정도로 차이가 벌어진다.

  ```python
  import asyncio
  import httpx
  import time
  
  API_URL_SYNC = "http://localhost:8000/sync"
  API_URL_ASYNC = "http://localhost:8000/async"
  
  async def fetch(client, url):
      response = await client.get(url)
      return response.json()
  
  async def benchmark(url, total_requests=41):
      async with httpx.AsyncClient() as client:
          tasks = [fetch(client, url) for _ in range(total_requests)]
          start = time.perf_counter()
          results = await asyncio.gather(*tasks)
          elapsed = time.perf_counter() - start
          return elapsed, results
  
  async def main():
      print("Benchmarking sync handler")
      elapsed_sync, _ = await benchmark(API_URL_SYNC)
      print(f"sync handler: {elapsed_sync:.2f} seconds")
  
      print("Benchmarking async handler")
      elapsed_async, _ = await benchmark(API_URL_ASYNC)
      print(f"async handler: {elapsed_async:.2f} seconds")
  
  if __name__ == "__main__":
      asyncio.run(main())
  ```

  - 즉 I/O bound 작업이라면, thread pool 이상의 요청이 들어올 경우 비동기 방식이 동기 방식에 비해 압도적으로 빠른 속도를 보여준다.



- 동기 방식과 비동기 방식의 동작 방식 차이는 아래와 같다.

  - 동기 방식
    - Thread pool의 크기가 3이라고 가정한다.
    - 하나의 요청을 처리하는데 2만큼의 시간이 걸린다.
    - 편의를 위해 요청1,2,3, 요청4,5,6은 모두 동시에 시작해서 동시에 종료된다고 가정한다.

  ```
  [t1]
  요청1이 들어온다.
  thread pool에서 하나의 thread가 해당 요청을 받아 요청 처리를 시작한다.
  요청2가 들어온다.
  thread pool에서 하나의 thread가 해당 요청을 받아 요청 처리를 시작한다.
  요청3이 들어온다.
  thread pool에서 하나의 thread가 해당 요청을 받아 요청 처리를 시작한다.
  
  [t2]
  요청4가 들어온다.
  thread pool에 더 이상 남은 thread가 없으므로 가용한 thread가 생길 때까지 대기한다.
  요청5가 들어온다.
  thread pool에 더 이상 남은 thread가 없으므로 가용한 thread가 생길 때까지 대기한다.
  요청6이 들어온다.
  thread pool에 더 이상 남은 thread가 없으므로 가용한 thread가 생길 때까지 대기한다.
  
  [t3]
  요청1에 대한 응답을 반환한다.
  thread pool에 가용한 thread가 생겼으므로 요청4의 처리가 시작된다.
  요청2에 대한 응답을 반환한다.
  thread pool에 가용한 thread가 생겼으므로 요청5의 처리가 시작된다.
  요청3에 대한 응답을 반환한다.
  thread pool에 가용한 thread가 생겼으므로 요청6의 처리가 시작된다.
  
  [t5]
  요청4에 대한 응답을 반환한다.
  요청5에 대한 응답을 반환한다.
  요청6에 대한 응답을 반환한다.
  ```

  - 비동기 방식
    - 비동기 방식은 동기 방식과 달리 외부 스레드 풀이 아닌 메인 스레드 풀에서 이벤트 루프를 기반으로 동작한다.

  ```python
  [t1]
  요청1이 들어온다.
  요청1에 대한 테스크를 이벤트 루프에 등록한다.
  이벤트 루프가 다음 스케줄링 타이밍에 테스크를 실행한다.
  이벤트 루프는 테스크를 처리하다 await 문을 만나게 되고 일시 중단 된다.
  요청2가 들어온다.
  요청2에 대한 테스크를 이벤트 루프에 등록한다.
  이벤트 루프가 다음 스케줄링 타이밍에 테스크를 실행한다.
  이벤트 루프는 테스크를 처리하다 await 문을 만나게 되고 일시 중단 된다.
  요청3이 들어온다.
  요청3에 대한 테스크를 이벤트 루프에 등록한다.
  이벤트 루프가 다음 스케줄링 타이밍에 테스크를 실행한다.
  이벤트 루프는 테스크를 처리하다 await 문을 만나게 되고 일시 중단 된다.
  
  [t2]
  요청4가 들어온다.
  요청4에 대한 테스크를 이벤트 루프에 등록한다.
  이벤트 루프가 다음 스케줄링 타이밍에 테스크를 실행한다.
  이벤트 루프는 테스크를 처리하다 await 문을 만나게 되고 일시 중단 된다.
  요청5가 들어온다.
  요청5에 대한 테스크를 이벤트 루프에 등록한다.
  이벤트 루프가 다음 스케줄링 타이밍에 테스크를 실행한다.
  이벤트 루프는 테스크를 처리하다 await 문을 만나게 되고 일시 중단 된다.
  요청6이 들어온다.
  요청6에 대한 테스크를 이벤트 루프에 등록한다.
  이벤트 루프가 다음 스케줄링 타이밍에 테스크를 실행한다.
  이벤트 루프는 테스크를 처리하다 await 문을 만나게 되고 일시 중단 된다.
  
  [t3]
  요청1에 대한 응답을 반환한다.
  요청2에 대한 응답을 반환한다.
  요청3에 대한 응답을 반환한다.
  
  [t4]
  요청4에 대한 응답을 반환한다.
  요청5에 대한 응답을 반환한다.
  요청6에 대한 응답을 반환한다.
  ```



- 주의할 점은 비동기 방식이 동기 방식에 비해 무조건 더 빠른 것이 아니라는 점이다.

  - 처리해야 하는 작업이 non-blocking 작업이고, 많은 요청이 지속적으로 들어올 경우에 비동기 방식이 동기 방식에 비해 빠른 성능을 보인다.

  - 예를 들어 아래와 같이 동일한 작업을 처리하는 path operation을 하나는 동기로, 다른 하나는 비동기로 선언했다고 가정해보자.

  ```python
  from contextlib import asynccontextmanager
  import anyio
  import time
  import asyncio
  
  from fastapi import FastAPI
  
  @asynccontextmanager
  async def lifespan(app: FastAPI):
      # Load the ML model
      limiter = anyio.to_thread.current_default_thread_limiter()
      print("current thread pool size:", limiter.total_tokens)
      yield
  
  app = FastAPI(lifespan=lifespan)
  
  @app.get("/sync")
  def sync_handler():
      time.sleep(1)
      return {"message": "sync done"}
  
  @app.get("/async")
  async def async_handler():
      time.sleep(1)
      return {"message": "async done"}
  
  
  if __name__ == "__main__":
      import uvicorn
      uvicorn.run(app, host="0.0.0.0", port=8000)
  ```

  - 이제 아래와 같이 테스트를 실행하면
    - 동기 path operation의 경우 약 1초, 비동기 path operation의 경우 약 10초가 소요된다.

  ```python
  import asyncio
  import httpx
  import time
  
  API_URL_SYNC = "http://localhost:8000/sync"
  API_URL_ASYNC = "http://localhost:8000/async"
  
  async def fetch(client, url):
      response = await client.get(url)
      return response.json()
  
  async def benchmark(url, total_requests=10):
      async with httpx.AsyncClient(timeout=15) as client:
          tasks = [fetch(client, url) for _ in range(total_requests)]
          start = time.perf_counter()
          results = await asyncio.gather(*tasks)
          elapsed = time.perf_counter() - start
          return elapsed, results
  
  async def main():
      print("Benchmarking sync handler")
      elapsed_sync, _ = await benchmark(API_URL_SYNC)
      print(f"sync handler: {elapsed_sync:.2f} seconds")
  
      print("Benchmarking async handler")
      elapsed_async, _ = await benchmark(API_URL_ASYNC)
      print(f"async handler: {elapsed_async:.2f} seconds")
  
  if __name__ == "__main__":
      asyncio.run(main())
  ```

  - 위와 같은 차이가 발생하는 원인은 아래와 같다.
    - 동기 path opeation의 경우 외부 thread pool에서 실행되어 10개의 요청이 외부 thread pool에서 동시에 실행된다.
    - 반면에 비동기 path operation의 경우 단일 thread에서 실행되는데, 처리해야 하는 작업이 blocking 작업이라 작업을 처리하는 동안 이벤트 루프에서 다른 작업을 수행할 수 없다.
    - 결국 비동기 방식의 경우 10개의 요청을 단일 스레드에서 순차적으로 처리해야 해서 약 10초의 시간이 걸리게 된다.



- FastAPI에서 모든 요청은 event loop에 등록된다.
  - 동기 path operation이든 비동기 path operation이든 모든 요청은 먼저 event loop에 task로 등록된다.
  - 만약 동기 path operation일 경우 이벤트 루프는 함수의 실행을 외부 스레드풀에 위임한다.
    - `anyio.to_thread.run_sync` 또는 `run_in_executor`를 통해 `ThreadPoolExecutor`에 offload한다.
    - 따라서 이벤트 루프 외부의 스레드에서 함수가 실행된다.
  - 반면 비동기 path operation일 경우 이벤트 루프가 함수를 직접 실행한다.

