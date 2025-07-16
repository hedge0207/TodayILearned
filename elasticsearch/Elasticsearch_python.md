# Elasticsearch 8

- client

  - elasticsearch package 8.X 이상을 설치해야 한다.
  - 기존과 달리 `ca_certs`, `basic_auth parameter`를 넘겨야 한다.

  ```python
  from elasticsearch import Elasticsearch
  
  
  es_client = Elasticsearch("https://localhost:9210", 
                          ca_certs="./http_ca.crt", 
                          basic_auth=("elastic", "<password>"))
  ```
  
  - Elasticsearch Python client의 버전이 7.x일 경우 `basic_auth`가 아닌 `http_auth` 파라미터를 넣는다.
  
  ```python
  from elasticsearch import Elasticsearch
  
  
  es_client = Elasticsearch("https://localhost:9210", 
                          ca_certs="./ca.crt", 
                          http_auth=("elastic", "<password>"))
  
  # 혹은 아래와 같이 url에 사용자명과 password를 추가한다.
  es_client = Elasticsearch("https://elastic:<password>@localhost:9210", 
                          ca_certs="./ca.crt")
  ```
  
  




# elasticsearch-dsl

- elasticsearch-dsl

  - Python에서 보다 깔끔하게 dsl을 작성하도록 도와주는 library
    - Python에서 ES를 사용하다보면 쿼리 때문에 코드가 지저분해진다.
    - elasticsearch-dsl을 사용하면 보다 깔끔하게 작성이 가능하다.
  - 설치

  ```bash
  $ pip install elasticsearch-dsl
  ```





# Errors

## AsyncElasticsearch 사용시 timeout 발생

- 현상

  - 전체적인 구조를 간소화해서 표현하면 아래와 같다.
    - 아래와 같이 클라이언트로부터 검색 대상 index와 query를 받아 Elasticsearch에 검색 요청을 보내고, 그 결과를 반환하는 FastAPI 애플리케이션이었다.
    - 단기간에 검색 요청이 몰릴 경우 `asyncio.exceptions.TimeoutError`가 발생했다.

  ```python
  import time
  
  from fastapi import FastAPI, HTTPException
  from elasticsearch import AsyncElasticsearch
  from pydantic import BaseModel
  
  
  app = FastAPI()
  
  es_client = AsyncElasticsearch(hosts=["http://localhost:9200"])
  
  class SearchRequest(BaseModel):
      index: str
      query: dict
  
  @app.post("/search")
  async def search_es(request: SearchRequest):
      try:
          search_start = time.time()
          response = await es_client.search(
              index=request.index,
              body={"query": request.query}
          )
          print(f"search_time: {time.time()-search_start}ms")
          return {"hits": response["hits"]["hits"]}
      except Exception as e:
          raise HTTPException(status_code=500, detail=str(e))
  
  @app.on_event("shutdown")
  async def close_es():
      await es_client.close()
  ```

  - 위 에러의 traceback은 아래와 같다.

  ```
  Traceback (most recent call last):
    File "/python3.8/site-packages/elasticsearch/_async/http_aiohttp.py", line 297, in perform_request
      async with self.session.request(
    File "/python3.8/site-packages/aiohttp/client.py", line 1117, in __aenter__
      self._resp = await self._coro
    File "/python3.8/site-packages/aiohttp/client.py", line 619, in _request
      break
    File "/python3.8/site-packages/aiohttp/helpers.py", line 656, in __exit__
      raise asyncio.TimeoutError from None
  asyncio.exceptions.TimeoutError
  ```



- 원인 파악 과정

  - API 또는 Elasticsearch 중 하나가 병목인 상황에서 처음에는 Elasticsearch를 의심했다.
    - Elasticsearch가 병목이라면 thread pool을 확인했을 때, queue에 검색 요청이 쌓이거나 reject된 요청이 있어야 한다.
    - 그러나 부하가 한참 가해지고 있을 때도 reject 된 요청은 커녕 queue에도 검색 요청이 쌓이지 않았다.
    - 또한 위 API코드를 보면 알 수 있듯 `AsyncElasticsearch.search()` 메서드의 실행 시간을 기록하고 있었는데, 만약 Elasticsearch가 병목이라면 부하가 지속될수록 시간이 선형적으로 증가하다 timeout이 발생하는 30초 전후의 시간이 걸릴 때 부터 위 에러가 발생해야 했다.
    - 그러나 `AsyncElasticsearch.search()`의 호출 시간을 기록한 로그를 확인해보니 어느 정도 선형적으로 증가하긴 하지만 메서드 호출이 완료되기까지 30초에 가깝게 걸린 경우는 없었으며, 대부분이 8~11초 에 머물러 있었다.
    - 이는 모종의 이유로 API에서 Elasticsearch로 아예 검색 요청을 보내고 있지 않고 있으며, 정상적으로 보내진 요청들만 로그에 남겨지고 있다는 것을 의미했다. 

  ```
  search time: 3134.384124422314ms
  search time: 3264.124124545352ms
  search time: 8234.346236326231ms
  search time: 8422.983573802837ms
  Traceback (most recent call last):
    File "/python3.8/site-packages/elasticsearch/_async/http_aiohttp.py", line 297, in perform_request
      async with self.session.request(
    File "/python3.8/site-packages/aiohttp/client.py", line 1117, in __aenter__
      self._resp = await self._coro
    File "/python3.8/site-packages/aiohttp/client.py", line 619, in _request
      break
    File "/python3.8/site-packages/aiohttp/helpers.py", line 656, in __exit__
      raise asyncio.TimeoutError from None
  asyncio.exceptions.TimeoutError
  ```

  - 위 과정을 거쳐 API를 확인해보게 됐다.



- 원인
  - AsyncElasticsearch 패키지는 내부적으로 `aiohttp` 패키지를 사용한다.
    - `aiohttp.helpers.TimerContext.timeout()` 메서드는 `aiohttp.helpers.TimeoutHandler.timer()` 메서드를 통해 callback에 추가된다.
    - Event loop에 등록된 task가 시간 내에 완료되지 못하면 callback에 등록된 ``aiohttp.helpers.TimerContext.timeout()` 메서드가 호출되어 `asyncio.exceptions.TimeoutError`를 발생시킨다.
  - `AsyncElasticsarch` 패키지는 HTTP 요청을 보내기 위해 `aiohttp.connector.TCPConnector`를 사용한다.
    - `aiohttp.connector.TCPConnector`는 동시에 유지하는 최대 세션의 개수를 `limit`이라는 파라미터로 받아서 조정한다.
    - `AsyncElasticsearch` 인스턴스를 생성할 때, `maxsize` 파라미터를 넘길 수 있는데, 이 값이 `AsyncElasticsearch` 내에서 `aiohttp.connector.TCPConnector`의 인스턴스를 생성할 때 `limit`으로 넘어간다.
    - `aiohttp.connector.TCPConnector`의 생성자 함수에 설정된 `limit`의 기본 값은 100인데, `AsyncElasticsearch`의 생성자 함수에 설정된 `maxsize`의 기본값은 10이다.
    - 결국 `AsyncElasticsearch`를 사용할 경우 Elasticsearch에 동시에 보낼 수 있는 최대 요청의 개수(유지할 수 있는 최대 세션의 개수)는 10개로 제한되고, event loop에 등록된 task들은 사용 가능한 세션이 생길때 까지 대기하게 된다.
    - 사용 가능한 세션이 생길때 까지 대기하다 타임아웃으로 설정된 시간이 초과되면 `asyncio.exceptions.TimeoutError`가 발생하게 되는 것이다.



- 해결

  - `AsyncElasticsearch`의 인스턴스를 생성할 때 `maxsize`를 기본값인 10보다 크게 설정한다.

  ```python
  es_client = AsyncElasticsearch(hosts=["http://localhost:9200"], maxsize=30)
  ```









# ETC

- readtimeout error

  - 아래와 같이 ES 클라이언트를 생성후 ES에 요청을 보냈을 때,  readtimeout 에러가 뜨는 경우가 있다.

  ```python
  from elasticsearch import Elasticsearch
  
  es = Elasticsearch('10.11.12.103:9200')
  ```

  - 이럴 때는 아래와 같이 다양한 옵션을 통해 timeout 관련 설정을 해줄 수 있다.
    - timeout은 기본값은 10으로, timeout error가 발생하기 까지의 시간(초)을 의미한다.
    - max_retries는 error가 발생할 경우 최대 몇 번까지 재시도를 할지 설정하는 것이다.
    - retry_on_timeout는 timeout이 발생했을 때, 재시도를 할지 여부를 설정하는 것이다.

  ```python
  from elasticsearch import Elasticsearch
  
  es = Elasticsearch('10.11.12.103:9200', timeout=30, max_retries=5,retry_on_timeout=True)
  ```




- CSV 파일 bulk

  ```python
  import csv
  # helpers import
  from elasticsearch import helpers, Elasticsearch 
  
  
  es = Elasticsearch(['localhost:9200'])
  
  with open('/test.csv') as f: 
      reader = csv.DictReader(f) 
      helpers.bulk(es, reader, index="my_index")
  ```

