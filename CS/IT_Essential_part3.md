# CORS

> https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS
>
> https://evan-moon.github.io/2020/05/21/about-cors/
>
> https://ko.javascript.info/fetch-crossorigin

- CORS(Cross-Origin Resource Sharing, 교차 출처 리소스 공유)

  - Origin
    - scheme(protocol), domain(hostname), port을 묶어서 origin이라 부른다.
    - 예를 들어 `http://example.com:80/foo`라는 URL이 있다고 할 때 `http://example.com:80`까지가 origin이다.

  - 의미
    - 서로 다른 origin들 사이의 자원을 공유하는 것을 의미한다.
    - 예를 들어 `http://domain-a.com`이라는 front-end origin에서 `http://domain-b.com`이라는 back-end origin에 정보를 요청하는 것이 CORS다.
  - SOP(Same Origin Policy)
    - 같은 origin에서만 리소스를 공유할 수 있게 하는 정책.
    - 현실적으로 웹에서 같은 origin에 있는 리소스만을 가져오는 것은 불가능했다.
    - 따라서 예외 조항을 두었는데, 그 중 하나가 CORS 정책을 지킨 리소스 요청이다.
    - 이와 같은 제약을 둔 것은 XSS(Cross Site Script)나 CSRF(Cross Site Request Forgery) 같은 공격을 막기 위해서이다.
  - CORS는 브라우저의 정책이다.
    - 즉, 브라우저가 자신이 보낸 요청 및 서버로부터 받은 응답이 CORS 정책을 지키는지 검사한다.
    - 따라서 이를 지키지 않은 요청은 보내지 않고, 이를 지키지 않은 응답은 받아서 버린다.



- 안전한 요청

  - 크로스 오리진 요청은 크게 안전한 요청과 그 외의 요청의 두 가지 종류로 구분된다.
  - 안전한 요청은 아래의 두 가지 조건을 모두 충족하는 요청을 말한다.
    - 안전한 메서드 사용(GET, POST, HEAD).
    - 안전한 헤더를 사용(`Accept`, `Accept-Language`, `Content-Language`, 값이 `multipart/form-data`, `test/plain`, `application/x-www-from-urlencoded` 중 하나인 `Content-type`)

  - 그 외의 요청은 당연히 둘 중 하나라도 충족하지 못한 요청을 말한다.



- CORS와 안전한 요청(MDN 문서에서는 Simple Request라고 표현)

  - 크로스 오리진 요청을 보낼 경우 브라우저는 항상 `Origin`이라는 헤더를 요청에 추가한다.
  - 예시
    - `http://A.com/product`에서 `httpL//B.com/request`로 요청을 보낸다고 가정했을 때 헤더는 다음과 같다.
    - `Origin`에는 요청이 이루어지는 페이지 경로(`/product`)가 아닌 origin 정보가 담기게 된다.

  ```text
  GET /request
  Host: B.com
  Origin: http://B.com
  ```

  - 서버는 요청 헤더에 있는 `Origin`을 검사하고, 이 origin에서 요청을 받기로 설정이 되어 있다면, 헤더에 `Access-Control-Allow-Origin`을 추가해 응답을 보낸다.
    - 이 헤더에는 허가된 origin(예시의 경우 `http://A.com`)에 대한 정보나 `*`가 명시된다.



- 응답 헤더
  - 크로스 오리진 요청이 이루어진 경우, JavaScript는 기본적으로 안전한 응답 헤더로 분류되는 헤더에만 접근할 수 있다.
    - 그 외의 헤더에 접근하면 에러가 발생한다.
  - 안전한 응답 헤더는 다음과 같다.
    - `Cache-Control`
    - `Content-Language`
    - `Content-Type`
    - `Expires`
    - `Last-Modified`
    - `Pragma`
  - 만일 안전하지 않은 헤더에 에러 없이 접근하려면 서버에서 `Access-Control-Expose-Headers`라는 헤더를 보내줘야 한다.
    - 이 헤더에는 JavaScript에 접근을 허용하지만 안전하지는 않은 header들의 콤마로 구분된 목록이 담겨 있다.



- CORS와 안전하지 않은 요청
  - 안전하지 않은 요청이 이루어지는 경우, 서버에 바로 요청을 보내지 않고 preflight 요청이라는 사전 요청을 서버에 보내 권한이 있는지를 확인한다.
    - 클라이언트에서 preflight 요청을 보내는 로직을 추가해야 되는 것은 아니고, 브라우저가 알아서 보낸다.
  - preflight 요청은 `OPTIONS` 메서드를 사용하고 아래의 두 헤더가 함께 들어가며, body는 비어 있다.
    - `Access-Control-Request-Method`: 안전하지 않은 요청에서 사용하는 메서드 정보가 담겨 있다.
    - `Access-Control-Request-Headers`: 안전하지 않은 요청에서 사용하는 헤더 목록이 담겨있다.
  - 서버가 preflight 요청을 받으면 빈 body와 아래 헤더들을 status code 200으로 브라우저로 보낸다.
    - `Access-Control-Allow-Origin`
    - `Access-Control-Allow-Methods`: 허용된 메서드 정보가 담겨 있다.
    - `Access-Control-Allow-Headers`: 허용된 헤더 정보가 담겨 있다.
    - `Access-Control-Max-Age`: 퍼미션 체크 여부를 몇 초간 캐싱해 놓을지가 담겨 있다. 이 기간 동안은 브라우저가 preflight를 보내지 않고 바로 본 요청을 보낸다.



- Credentialed Request
  - 쿠키 정보나 인증과 관련된 정보를 담을 수 있게 해주는 옵션이다.
  - 요청시에 아래와 같은 3가지 옵션이 있다.
    - `omit`: cookie(혹은 cookie를 비롯한 사용자 credentials)를 보내거나 받지 않는다.
    - `same-origin`(기본값): origin이 같을 경우에만 cookie를 받는다.
    - `include`: origin이 다르더라도 cookie를 보낸다.
  - Credentialed Request를 사용할 때는 서버의 응답 헤더에 두 가지 헤더가 추가되어야 한다.
    - `Access-Control-Allow-Credentials`의 값이 true여야 한다.
    - `Access-Control-Allow-Origin`의 값이 `*`여선 안 된다.



- nginx 등의 web server와 함께 사용할 경우

  - web server와 backend server 중 한 곳에만 설정하면 되는 듯 하다(확인 필요)
    - 아래 예시에서 fastapi에 `allow_origins` 옵션을 추가하거나 nginx에 `add_header 'Access-Control-Allow-Origin' '*'`를 추가하면 정상적으로 동작한다.
  - `SendReq.vue`

  ```vue
  <template>
    <div class="hello">
      <h1>{{ msg }}</h1>
      <p>
        <button v-on:click="sendReq">send_req</button>
      </p>
    </div>
  </template>
  
  <script>
  import axios from 'axios'
  export default {
    name: 'HelloWorld',
    props: {
      msg: String
    },
    methods:{
      sendReq(){
        axios.delete("http://<nginx_host>:<nginx_port>/user", null, {withCredentials: true}).then((res)=>{
          console.log(res.headers)
        }).catch((err) => {console.log(err)})
      }
    }
  }
  </script>
  ```

  - `fastapi`

  ```python
  import uvicorn
  from fastapi import FastAPI, Request
  from fastapi.middleware.cors import CORSMiddleware
  
  app = FastAPI()
  
  origins = [
      "*",
  ]
  
  app.add_middleware(
      CORSMiddleware,
      allow_origins=origins,
      allow_credentials=True,
      allow_methods=["*"],
      allow_headers=["*"],
      expose_headers=["*"]
  )
  
  @app.delete("/user")
  async def qux(request: Request):
      print(request.headers)
      return "User Deleted!"
  
  if __name__ == '__main__':
      uvicorn.run(app, host='0.0.0.0', port=8002)
  ```

  - `nginx.conf`

  ```nginx
  ...
  http {
      include       /etc/nginx/mime.types;
      server {
        server_name <fastapi_host>;
        location / {
              proxy_pass http://<fastapi_host>:<fastapi_port>;
              # add_header 'Access-Control-Allow-Origin' '*';
          }
      }
      ...
  }
  ```





# 왜 숫자는 0부터 세는가?

> https://www.cs.utexas.edu/users/EWD/transcriptions/EWD08xx/EWD831.html

- 자연수의 부분수열 표기
  - 자연수의 부분수열(`[2, 3, ..., 12]`)을 표기할 때 중략을 표현하는 점 3개(`...`) 없이 표현하는 방법에는 다음과 같은 것들이 있다.
    - a: `2 <= i < 13`
    - b: `1 < i <= 12`
    - c: `2 <= i <= 12`
    - d: `1 < i < 13`
  - 위의 4가지 방법들 중 더 효율적인 표기법은 분명히 존재한다.
  - 한 쪽에만 equal or bigger than 표기(<=)가 있으면 아래와 같은 점에서 효율적이다.
    - a, b 방식의 경우 양 경계값의 차이가 부분수열의 길이와 같다. 따라서 부분수열의 길이를 보다 직관적으로 알 수 있다.
    - 또한 두 부분 수열이 맞닿아 있을 때 한 쪽의 상한 수가 다른 쪽의 하한수와 같아진다.
    - 예를 들어 a 표기법의 경우 `2 <= i < 13`과 같이 표기할 경우 이전 수열은 `1 <= 1 < 2`과 같이 상한수와 하한수(이 경우 `2`)의 변경 없이 표기가 가능하고, 다음 수열인 `13 <= i < 15` 역시 상한수(이 경우 `13`)와 하한수의 변경 없이 표기가 가능하며, b 표기법 역시 마찬가지다.
  - b와 d 처럼 하한수(예시의 경우 `2`)를 제외하면 가장 작은 자연수 0(0을 자연수라고 할 때 0이 가장 작은 자연수이다)으로 시작하는 부분 수열을 표기할 때 하한수를 비자연수에 있는 숫자에서 선택하도록 강제된다.
    - 만일 `[0, 1, 2, 3]`과 같은 부분수열을 표기하려면 `-1 < i <= 3`(b 방식), `-1 < i < 4`(d 방식)와 같이 표기해야한다.
    - 이는 직관적이지도 않고 부자연스럽다.
  - 따라서 a 방식이 가장 효율적인 표현 방법이다.



- 만약 인덱스가 1부터 시작할 경우
  - 위에서 a 방식이 부분 수열을 표기하는 가장 효율적인 방식이라는 것을 확인했다.
    - 따라서 a 방식으로 인덱스의 범위를 표기할 때, 인덱스를 1부터 시작하는 것과, 0부터 시작하는 것의 차이는 아래와 같다.
  - 길이가 N인 수열의 인덱스가 1부터 시작할 경우(a 표기법에 따른) 인덱스의 범위는 `1 <= i < N+1`이 된다.
  - 반면 0부터 시작할 경우(a 표기법에 따른) 인덱스의 범위는 보다 깔끔한 `0 <= N < N`이 된다.





# Data processing

- Data processing이란

  - Data를 수집하고, 조작하고, 수집된 데이터를 의도한 목적대로 사용하는 것을 의미한다.

  - 일반적으로 수집-집계-분석의 과정으로 이루어진다.

  - 아래와 같이 다양한 방식이 존재한다.
    - Batch processing
    - Real-time/Stream processing
    - Online Processing
    - MultiProcessing
    - Time-Sharing



- Batch processing

  - 대량의 데이터를 모아서 정해진 기간 동안 한 번에 처리하는 data processing 기법이다.
    - 여러 개의 데이터를 처리하는 것 뿐 아니라, 대용량의 데이터를 한 번에 처리한다면 batch processing이라 할 수 있다.
    - 즉 핵심은 몇 개의 데이터를 한 번에 처리한다는 것이 아니라 큰 데이터를 한 번에 처리한다는 것이다.

  - 아래와 같은 경우에 주로 사용한다.
    - Data size가 정해져 있거나, 예측할 수 있는 경우.
    - Data가 일정 기간동안 축적되고, 비슷한 data들을 그룹화 할 수 있는 경우.
    - 데이터의 생성과 사용 사이에 시간적 여유가 있는 경우.
  - 사용 예시
    - 월 단위 결제



- Stream processing

  - 데이터가 모일 때 까지 기다리지 않고 data가 생성된 즉시 바로 처리하는 data processing 기법이다.
    - data를 real-time 혹은 near real-time으로 처리한다.
    - 이전에는 real-time processing이라는 명칭을 더 많이 사용했으나 요즘은 stream processing이 더 널리 사용된다.

  - 아래와 같은 경우에 주로 사용한다.
    - Data size가 무한하거나, 예측할 수 없는 경우.
    - input이 들어오는 속도만큼 output을 산출하는 속도를 낼 수 있는 경우.
    - 데이터가 연속적으로 들어오면서, 데이터를 즉각적으로 사용해야 할 경우.
  - 사용 예시
    - 실시간 로그 분석



- Batch processing과 Stream processing의 차이

| Batch Processing                                             | Stream Processing                                            |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| 많은 양의 data를 한 번에 처리                                | streaming data를 실시간으로 처리                             |
| 여러 단계를 거쳐 data를 처리                                 | 적은 단계를 거쳐 data를 처리                                 |
| Data를 처리하는데 분~일 단위의 시간이 소요                   | 초 또는 밀리초 단위로data를 처리                             |
| input이 안정적                                               | input이 유동적                                               |
| Data 처리가 모두 끝난 후 응답이 제공된다.                    | 응답이 바로 제공된다.                                        |
| 대량의 batch data 처리를 위해 storage와 processing resource가 필요하다 | Batch에 비해 storage는 덜 필요하지만 processing resources는 필요하다. |





# Monorepo Vs Polyrepo

> https://medium.com/hcleedev/dev-monorepo-%EA%B0%9C%EB%85%90-%EC%95%8C%EC%95%84%EB%B3%B4%EA%B8%B0-33fd3ce2b767

- Monorepo
  - 하나의 project를 하나의 repository에서 관리하는 것을 의미한다.
    - 예로 Front-end 코드와 Back-end 코드를 한 repo에서 관리하는 것을 들 수 있다.
  - one-repo, uni-repo라고도 부른다.



- Polyrepo
  - 하나의 project를 하나의 repository에서 관리하는 것을 말한다.
    - 예로 Front-end 코드와 Back-end 코드를 각기 다른 repo에서 관리하는 것을 들 수 있다.
  - many-repo, multi-repo라고도 부른다.



- Monorepo의 장점
  - Project의 version을 일괄적으로 관리하기가 쉽다.
    - 여러 모듈의 버전을 일일이 맞춰주지 않아도 된다.
  - 코드의 재사용성이 증가한다.
    - 만일 하나의 project를 구성하는 2개의 각기 다른 repo에서 공통된 code를 사용해야 할 경우, 공유가 불가능하므로 중복된 code가 들어갈 수 밖에 없다.
    - 그러나 Monorepo의 경우 여러 모듈을 하나의 repo에서 관리하므로 코드의 재사용이 가능하다.
  - 의존성 관리가 쉬워진다.
    - 의존성을 하나의 repo에서만 관리하면 되므로 의존성 관리가 간편해진다.
  - 변경 사항을 보다 원자적으로 관리할 수 있다.
    - 만일 여러 repo에서 공통적으로 쓰이는 코드가 변경되었을 경우, 모든 repo를 돌면서 변경 사항을 적용해줘야한다.
    - 그러나 monorepo의 경우 한번의 변경으로 적용이 가능하다.
  - Team 간의 경계와 code ownership이 유연해진다.
    - 이를 통해 보다 원활한 협업이 가능해진다.



- Monorepo의 단점

  - Project가 거대해질 수록 monorepo를 관리하는데 소요되는 비용이 증가한다.
  - 의존성을 추가하는 데 부담이 없어, 불필요한 의존성이 증가한다.

  - Code Ownership에 위배된다.
    - Code의 소유권은 단일 팀에 속해야 한다고 생각하는 개발자들도 있다.
    - Code에 대한 책임이 모호해져 코드 관리가 더 힘들어질 수 있다.





# 보안

- Software 보안의 3대 요소
  - 기밀성(confidentiality): 인가되지 않은 접근 차단.
  - 무결성(integrity): 인가 받지 않은 사용자는 데이터의 변경이 불가능.
  - 가용성(availability): 권한을 가진 사용자는 서비스를 지속해서 사용할 수 있음.



- 용어
  - 자산(assets)
    - 조직이 가치를 부여한 대상
    - 데이터, 하드웨어 등
  - 위협(threat)
    - 자산에 악영향을 끼칠 수 있는 사건이나 행위
    - 해킹, 데이터 변조 등
  - 취약점(vulnerability)
    - 위협이 발생하기 위한 사전 조건.
    - 비밀번호 공유, 입력값 미 검증 등
  - 위험(risk)
    - 위협이 취약점을 이용하여 조직에 피해를 끼칠 가능성.



- 접근 통제 기법

  - 접근 통제 용어
    - 주체(subject): 시스템에 접근을 요청하는 능동적인 개채
    - 객체(object): 접근 대상이 되는 개채
    - 접근(access): 주체의 활동
  - 식별(identification)
    - 자신의 정보를 시스템에 밝히는 행위
    - 주체가 객체에게 자신의 정보를 제공.
  - 인증(authentication)
    - 주체의 신원을 검증하는 활동
    - 주체의 신원을 객체가 인증.
  - 인가(authorization)
    - 인증된 주체에게 접근을 허용하는 활동
  - 책임추적(accountability)
    - 주체의 접근을 추적하고 행동을 기록하는 활동

  - 3A
    - authentication
    - authorization
    - accounting



# Data의 단위

- Bit(b)
  - binary
    - 컴퓨터에 입력되는 정보는 구체적이고 명확해야한다.
    - 이를 위해 컴퓨터에 입력되는 모든 정보를 숫자로 표현했다.
    - 모든 정보는 숫자로 enocoding되어 컴퓨터에 입력된다.
    - 컴퓨터 내의 모든 정보는 구별의 편의성을 위해 0과 1로 이루어진 이진수로 표현된다.
    - 1은 신호가 있음(혹은 높은 신호)을 의미하고, 0은 신호가 없음(혹은 낮은 신호)을 의미한다.
  - bit는 0과 1로 이루어진 가장 작은 data의 단위이다.



- 더 큰 단위들

  - Byte(B)
    - 최초의 컴퓨터는 8 bits의 정보를 동시에 처리할 수 있었다.
    - 따라서 8 bits를 1byte로 묶어 표현하기 시작했다.
  - byte 이상의 데이터들은 크게 2가지 system으로 측정된다.
    - 10진수 기반 system: 10의 거듭 제곱으로 각 단위의 값을 정한다(국제 단위계에서 사용).
    - 2진수 기반 system: 2의 거듭 제곱으로 각 단위의 값을 정한다(컴퓨터의 memory를 설명하기 위해 사용).
    - 기존에는 두 방식 모두 동일한 단위(Kilobyte, Megabyte 등)를 사용했으나, 국제 전기 기술 위원회(IEC, International Electrotechnical Commission)의 권고로 2진수 기반은 중간에 bi를 넣어 kibibyte, mebibyte 등으로 부르기 시작했다.
  - 10진수 기반 system의 예시

  | 단위         | 값    | bytes             |
  | ------------ | ----- | ----------------- |
  | Byte(B)      | 1     | 1                 |
  | Kilobyte(KB) | 10^3  | 1,000             |
  | Megabyte(MB) | 10^6  | 1,000,000         |
  | Gigabyte(GB) | 10^9  | 1,000,000,000     |
  | Terabyte(TB) | 10^12 | 1,000,000,000,000 |

  - 2진수 기반 system의 예시
    - 공통적으로 들어가 있는 bi는 binary의 약자로, 2진수 기반이라는 것을 표현하기 위해 사용한다.

  | 단위           | 값   | bytes             |
  | -------------- | ---- | ----------------- |
  | kibibyte(KiB)  | 2^10 | 1,024             |
  | mebibyte(MiB)  | 2^20 | 1,048,576         |
  | gebibyte(1GiB) | 2^30 | 1,073,741,824     |
  | tebibyte       | 2^40 | 1,099,511,627,776 |





# IP

> [IP 클래스, 서브넷 마스크, 서브넷팅 완벽 이해하기](https://inpa.tistory.com/entry/WEB-IP-%ED%81%B4%EB%9E%98%EC%8A%A4-%EC%84%9C%EB%B8%8C%EB%84%B7-%EB%A7%88%EC%8A%A4%ED%81%AC-%EC%84%9C%EB%B8%8C%EB%84%B7%ED%8C%85-%EC%B4%9D%EC%A0%95%EB%A6%AC#top)
>
> [CIDR이 무얼 말하는거야?](https://inpa.tistory.com/entry/WEB-%F0%9F%8C%90-CIDR-%EC%9D%B4-%EB%AC%B4%EC%96%BC-%EB%A7%90%ED%95%98%EB%8A%94%EA%B1%B0%EC%95%BC-%E2%87%9B-%EA%B0%9C%EB%85%90-%EC%A0%95%EB%A6%AC-%EA%B3%84%EC%82%B0%EB%B2%95)

- IP(Internet Protocol)

  - 인터넷에 연결되어 있는 모든 장치들(컴퓨터, 노트북, 스마트폰 등)을 식별할 수 있도록 각 장비들에게 부여되는 고유한 주소이다.
  - 일반적으로 마침표로 구분된 3자리 10진수 숫자 4개로 구성된다.
    - 2진법으로 표현하면 32bit가 된다.
    - 그러므로 IP는 총 2의 32승개(약 43억개)의 주소를 갖는다.

  - IP 주소의 구성
    - Network ID + Host ID로 구성된다.
    - Network ID는 한 네트워크 내에서 공유하는 ID이다.
    - Host ID는 동일한 네트워크 내에서 각 호스트들을 구분하기 위해 사용하는 ID이다.
    - 예를 들어 192.168.11.211라는 IP 주소가 있을 때, 앞의 세 숫자(192.168.11)은 network id, 마지막 숫자(211)는 host id이다.
    - 꼭 예시처럼 network id가 세 개의 숫자로 구성되고, 마지막 숫자가 항상 host id인 것은 아니다.
  - 사용할 수 없는 주소
    - 네트워크 주소와 브로드캐스트 주소는 사용할 수 없다.
    - 네트워크 주소는 Host ID의 대역폭의 첫 번째 주소로, 네트워크 자체를 가리키는 주소이다.
    - 브로드캐스트용 주소는 Host ID의 대역폭의 마지막 주소로, 인터넷에 데이터를 전달하기 위해 사용하는 주소이다.
    - 예를 들어 192.168.11까지가 network id라면, host id는 마지막 8bit가 된다. 따라서 이 IP의 네트워크 주소는 192.168.11.0, 브로드 캐스트용 주소는 192.168.11.255가 된다.



- IP 주소 클래스
  - IP를 할당하는 방식 중 하나이다.
    - 지금은 더 이상 사용하지 않는다.
    - 그러나 서브넷을 이해하기 위해 IP 주소 클래스에 대한 이해가 필요하다.
  - 옥텟
    - IP를 8비트로 4등분한 각 부분을 옥텟(Octet)이라 부른다.
    - 예를 들어 192.168.11.211를 2진수로 변경하면 11000000.10101000.00001011.11010011이 되는데, 여기서 콤마로 구분되는 각 부분을 옥텟이라 부른다.
    - 옥텟 별로 IP의 클래스를 나눌 수 있다.
  - IP 클래스
    - 각 클래스마다 network id와 host id의 비율이 다르다.
    - A 클래스의 경우 network id가 8bit, host id가 24bit로 나뉜다.
    - B 클래스의 경우 network id가 16bit, host id가 16bit로 나뉜다.
    - C 클래스의 경우 network id가 24bit, host id가 8bit로 나뉜다.
    - 이 외에도 D, E 클래스가 있으나, D는 멀티 캐스트용 IP, E는 미래에 사용하기 위해 예약된 IP들로, A, B, C가 일반적으로 사용된다.
  - A 클래스
    - 첫 번째 옥텟의 첫 번째 비트는 0으로 고정된다.
    - 따라서 A 클래스의 첫 옥텟은 0~127 사이의 값을 가지게 된다.
    - 호스트 대역이 24bit로, 네트워크 당 나올 수 있는 호스트 주소가 약 1670만개이므로 대규모 네트워크에 적합하다.
  - B 클래스
    - 첫 번째 옥텟의 첫 번째 비트가 1, 두 번째 비트가 0으로 고정된다.
    - 따라서, B클래스의 첫 옥텟은 128-191 사이의 값을 가지게 된다.
    - 호스트 대역이 16bit로, 네트워크 당 나올 수 있는 호스트 주소가 약 65000개이므로, 중간 규모의 네트워크에 적합하다.
  - C 클래스
    - 첫 옥텟의 첫번째 비트가 1, 두 번째 비트가 1, 세 번째 비트가 0으로 고정된다.
    - 따라서 C 클래스의 첫 옥텟은 192~223 사이의 값을 가지게 된다.
    - 호스트 대역이 8bir로, 네트워크 당 나올 수 있는 호스트 주소가 256개이므로 소규모 네트워크에 적합하다.



- 서브넷
  - 하나의 네트워크가 분할되어 나뉜 작은 네트워크이다.
    - 예를 들어 호스트를 60개만 사용하는 기업에게는 C 클래스의 대역폭인 256개도 남는 주소가 생기게 된다.
    - 따라서 이를 절반으로 나누고, 또 다시 절반으로 나눈 64개를 이 기업에게 할당하고, 남는 192개를 다른 곳에 할당하는 방식이다.
  - IP 클래스의 단점을 보완하기 위해 등장했다.
    - IP 클래스 방식의 문제는 다음과 같다.
    - 예를 들어 A 업체는 30000개의 IP 주소를 사용하고 있다.
    - 이 업체에게 B 클래스를 할당하자니 약 35000개의 IP 주소는 사용되지 않음에도 B 업체가 점유하게 되고, C 클래스를 할당하기에는 너무 많은 IP 주소를 사용중이다.
    - 이러한 문제를 해결하기 위해 등장한 것이 서브넷이다.
  - 서브넷 마스크
    - IP 주소와 동일한 32비트로 표현된다.
    - IP 주소와 다른 점은 연속된 1과 연속된 0으로 구성된다는 점이다.
    - 네트워크 ID와 호스트 ID를 보다 편한게 구분하기 위해 사용한다.
    - 서브넷 마스크의 옥텟이 1로 이루어져 있으면(11111111, 255) 네트워크 ID, 0으로 이루어져 있으면(00000000, 0) 호스트 ID이다.
    - 즉, A 클래스는 서브넷 마스크로 255.0.0.0, B 클래스는 255.255.0.0, C 클래스는 255.255.255.0으로 표현된다.
  - 서브네팅
    - 서브넷을 만들기 위해 네트워크를 분할하는 과정이다.
    - 분할 과정에서 서브넷 마스크를 사용한다.
    - 네트워크 영역과 호스트 영역을 쪼개는 작업을 수행한다.



- 서브네팅 계산 방법
  - 주소를 절반으로 나눈다.
  - 서브넷 구분 비트
    - 주소를 절반으로 나누기 위해서 서브넷 구분 비트를 지정해줘야한다.
    - 네트워크 주소를 어느 기준으로 쪼개서 서브넷을 만들지를 정하는 비트이다.
    - 항상 호스트 ID의 왼쪽에서부터 결정되어야 한다.
    - 예를 들어 C 클래스에서 호스트 ID는 하나의 옥텟(8bit, 256)를 지닌다.
    - 8비트의 첫 번째 비트를 서브넷 구분 비트로 사용하면 범위가 두 개로 나뉘게 된다.
    - 예를 들어 00000000은 00000000~011111111과 10000000~11111111으로 나뉘게 된다.
    - 즉 256개의 주소를 128개씩 나누어 사용할 수 있는 것이다.
    - 여기에 두 번째 비트 까지를 서브넷 구분 비트로 사용하면 4 부분으로 나뉘게 된다.
    - 00000000~00111111, 01000000~01111111, 10000000~10111111, 11000000~11111111
    - 즉 256개의 주소를 64개씩 나누어 사용할 수 있게 된다.
  - 예시
    - 192.168.8라는 네트워크 ID를 가지는 C 클래스를 A 기업에 할당하려 한다.
    - 그런데 해당 기업에서는 60개의 주소만 사용할 예정이다.
    - 따라서 이를 서브네팅을 통하여 서브넷을 생성하고 이를 할당하기 위해 아래의 과정을 거친다.
    - 호스트 ID의 bit를 아래와 같이 4부분으로 나누어 64개의 주소로 이루어진 서브넷을 생성한다.
    - 00000000~00111111, 01000000~01111111, 10000000~10111111, 11000000~11111111
    - 위 서브넷 중 하나를 기업에 할당한다. 예를 들어 00000000~00111111를 할당하면, A 기업은 192.168.8.0~63까지의 주소를 할당 받게 된다.
    - 이 때, 네트워크 주소와 브로드 캐스트 주소는 사용이 불가능하므로 실질적으로 1~62까지의 주소를 사용할 수 있다.
    - 마찬가지로 B 기업이 50개의 주소를 필요로 한다면 위에서 분할 한 서브넷 중 하나를 할당한다.
    - 만약 10000000~10111111를 할당했다면 B 기업은 192.168.8.128~191까지의 주소를 할당 받게 되고, 네트워크 주소와 브로드 캐스트 주소를 제외한 129~190까지의 주소를 사용할 수 있다.



- 슈퍼네팅
  - 서브네팅과 반대로 네트워크를 합치는 것이다.



- CIDR(Classless Inter-Domain Routing)
  - 도메인간의 라우팅에서 사용되는 인터넷 주소를, IP 주소 클래스를 사용할 때 보다 더욱 능동적으로 할당할 수 있게 해주는 방식이다.
    - 하나의 CIRD 블럭이 하나의 서브넷이라 이해하면 된다.
    - 서브네팅, 슈퍼네팅도 CIDR의 한 방식이다.
  - CIDR 표기법
    - 서브넷 마스크를 보다 간소화해서 표현할 수 있다.
    - 예를 들어 IP주소가 192.168.11.22/24 일 때 "/24" 부분이 서브넷 마스크를 표현한 부분으로 32bit의 앞에서부터 1의 개수를 나타낸 것이다.
    - 즉, 4개의 옥텟 중 3개의 옥텟이 1(8*3=24)이므로, 위 주소는 C 클래스라는 것을 바로 알 수 있다.
  - 예시
    - IP가 192.168.10.70/26일 경우, 서브넷 마스크의 1의 개수가 26개이다.
    - 따라서 위 IP는 IP 클래스 방식의 C 클래스에 해당하며, 호스트 ID 의 두 번째 비트까지를 서브넷 구분 비트로 사용한다는 것을 알 수 있다.
    - 즉 서브넷 마스크로 표현하면 255.255.255.192(11111111.11111111.11111111.11000000)이 된다.
    - 두 번째 비트 까지를 서브넷 구분 비트로 사용하므로 주소는 총 4개의 서브넷을 나뉜다는 것을 알 수 있다.



# 진수

- 이진수의 음수 표현

  - 1의 보수(One's complement)
    - 양수 값의 비트를 반전시킨 값을 음수로 사용한다.
    - 예를 들어 4bit 기준으로 3은 0011로 표현되는데, 이를 1의 보수로 음수로 변경하면 1100이 된다.
    - 0의 경우 +0(4bit 기준 0000)과 -0(4bit 기준 1111)이 존재한다.
    - 현재에는 거의 사용하지 않는 방식이다.
  - 1의 보수의 뺄셈
    - 덧셈의 경우 일반적인 이진수의 덧셈과 동일하지만, 뺄셈의 경우 아래의 과정을 거쳐 계산한다.
    - 빼는 수의 1의 보수를 구한 후 더한다.
    - 덧셈 결과 자리 올림이 생겼다면 최하위 비트에 1을 더하고 자리올림은 제외한다.
    - 자리 올림이 생기지 않았다면 연산 결과에 1의 보수를 구한 후 - 부호를 붙인다.

  ```txt
  1101-1001 계산
  1001의 1의 보수를 구한다. -> 0110
  1101+0110 = 10011
  자리 올림이 생겼으므로 1을 더한다. -> 10100
  올림된 수를 제외한다. -> 0100
  
  0101-1001 계산
  1001의 1의 보수를 구한다.
  0101+0110 = 1011
  자리 올림이 생기지 않았으므로 1의 보수를 구한 후 -를 붙인다. -> -0100
  ```

  - 2의 보수(Two's complement)

    - 1의 보수에 1을 더해서 구한다.
    - 예를 들어 숫자 5는 8bit 기준으로 00000101로 표현하는데, 이를 1의 보수로 변환하면 11111010이 되고, 여기에 1을 더한 11111011이 2의 보수가 된다.
    - 1의 보수와는 달리 +0, -0이 따로 존재하지 않는다.
  - 2의 보수의 뺄셈

    - 빼는 수의 2의 보수를 구한 후 더한다.
    - 자리 올림이 생겼다면 자리 올림을 제외시킨다.
    - 자리 올림이 생기지 않았다면 연선 결과의 2의 보수를 구한 후 - 부호를 붙인다.
  - 맨 왼쪽 bit가 1이면 음수, 0이면 양수라는 것을 의미한다.



- 16진수(Hexadecimal Number System, HEX)
  - 컴퓨터가 발명되기 이전에 사람들은 10진수를 사용했다.
    - 그러나 컴퓨터가 발명되면서, on, off 밖에 이해할 수 없는 컴퓨터를 위해 사람들은 2진수(**b**inary dig**it**, bit)를 사용하기 시작했다.
    - 편의를 위해 bits 값들을 묶기 시작했고, 초기에는 3 bits를 묶어 octal(8진수)를 사용했다.
    - 그러나 하드웨어 성능의 향상으로 한 묶음에 16가지 경우의 수가 존재하는 4 bits씩 묶을 수 있게 됐다.
    - 이를 그리스어로 6을 뜻하는 Hex와 라틴어로 10을 뜻하는 decimal을 합쳐 HEX-a-decimal이라 부르기로 했다.
  - 0~9까지는 decimal과 동일하며 decimal 10부터는 A부터 시작하는 알파벳을 붙이고, decimal 16은 10으로 표기한다.
    - decimal과 헷갈리지 않게 하기 위하여 숫자 뒤에 `h`를 붙이거나, 숫자 앞에 `0x`를 붙여 16진수라는 것을 표시한다.
  - 16진수는 2진수에서 쉽게 변환이 가능하기에, 컴퓨터에서 광범위하게 사용된다.
    - 16진수는 2의 4제곱이다.
    - 따라서 4bit에 16진수 값 하나를 담을 수 있다.
  - 2진수를 16진수로 변환하는 방법.
    - 2진수를 4개씩 묶어(4bit에 16진수 값 하나가 담기므로) 묶인 2진수를 16진수로 변환한다.
    - 예를 들어 1110011001110001과 같은 2진수가 있을 때, 이를 4개씩 묶으면 다음과 같다.
    - 1110 0110 0111 0001
    - 이를 16진수로 변환하면 E671이 된다.
  - 2진수의 자릿값이 4로 나누어 떨어지지 않을 경우의 변환 방법
    - 예를 들어 101100과 같은 2진수는 10 1100으로밖에 나누어지지 않는다.
    - 이럴 경우 남은 자리에 전부 0을 채운 후 위와 동일한 방법으로 16진수로 변환한다.
    - 0000 0000 0010 1100을 16진수로 변환하면 2C가 된다.
  - 16진수를 2진수로 변환하는 방법
    - 단순히 16진수의 각 자리에 해당하는 수를 2진수로 변경해주기면 하면 된다.
    - 16진수 BE가 있을 때, B(10진수의 11)를 2진수로 변환하면 1011, E(10진수 13)를 2진수로 변환하면 1110이된다.
    - 따라서 BE를 2진수로 변환하면 10111110이 된다.
  - 10진수로의 변환, 혹은 10진수에서 16진수로의 변환은 2진수를 거쳐 반환하면 훨씬 간단하다.
    - 10진수의 16진수 변환은 우선 10진수를 2진수로 변환한 후 2진수로 16진수를 구한다.
    - 16진수를 10진수로 변환할 때에는 위의 과정을 반대로 하면 된다.
  - 사용처
    - URL의 character code에 사용된다.
    - low-level language나 unicode 등의 일부 encoding 등에 사용된다.
    - RGB 기반의 색상을 정의할 때도 사용된다.



# Shorts

- statement(문)와 expression(표현식)
  - expression
    - 하나 이상의 값으로 표현될 수 있는 코드를 말한다.
    - 평가가 가능해서 하나의 값으로 환원된다.
    - 예를 들어 `3+3`이라는 표현식은 평가 된 후 6이라는 값으로 환원된다.

  - statement
    - 실행 가능한 최소한의 독립적인 코드 조각을 말한다.
    - 표현식과 달리 평가와 평가에 따른 값의 환원이 존재하지 않는다.

  - expression은 statement에 포함되는 개념이다.
    - 즉 statement 중 평가되어 값을 반환하는 것들을 expression이라 부른다.




- 컴파일과 런타임
  - 컴파일: 소스 코드를 작성하고 해당 소스 코드를 기계어 코드로 변환하여 실행 가능한 프로그램이 되기 위한 편집 과정.
  - 런타임: 컴파일 과정을 마친 프로그램은 사용자에 의해 실행되며, 이러한 응용프로그램이 동작되는 시점을 런타임이라 부른다.




- 컴파일과 인터프리터
  - 컴파일러: 한 언어에서 다른 언어로 번역하는 역할을 수행, 프로그래밍 언어를 기계어 혹은 바이트 코드 등으로 번역하는 역할에 국한된 의미가 아니다 Java를 C로 번역한다면 이 역시 컴파일이라 할 수 있다.
    - 컴파일러가 실행되면 프로그래밍 언어를 번역해서 하나의 바이너리(혹은 어셈블리)파일로 저장하고 실제 프로그래밍 실행(런타임)은 이 파일을 실행함으로써 일어난다.
    - 즉 통으로 번역하고 번역한 파일을 실행한다.
    - 장점: 바이너리를 실행하기에 실행 속도가 인터프리터에 비해 빠르다.
    - 단점: 수정사항이 생길 때 마다 다시 컴파일을 진행해야 한다.
    - C, C++, Java 등은 컴파일 언어에 속한다.
    - Java의 경우 약간은 애매한 측면이 있다 컴파일러로 통으로 번역을 해 .class 파일을 생성한 뒤 .class 파일을 인터프리터를 통해 한 줄씩 실행한다.
  - 인터프리터:  번역해야 할 파일을 받아 한 줄씩 실행시키는 역할을 수행
    - 한 줄씩 번역하고 한 줄씩 실행한다.
    - 장점: 컴퓨터 마다 컴파일을 해주지 않아도 된다(플랫폼 독립적이다)
    - 단점: 컴파일 언어에 비해 속도가 느리다.
    - Python, Javascript, Ruby 등이 인터프리터 언어에 속한다.



- 배치 프로그램(Batch Program)

  - 정의: 사용자와 상호 작용 없이 일련의 작업들을 작업 단위로 묶어 정기적으로 반복 수행하거나 정해진 규칙에 따라 일괄 처리하는 방법

  - 배치 프로그램의 필수 요소
    - 이벤트 배치: 사전에 정의해 둔 조건 충족 시 자동으로 실행
    - 온디맨드 배치: 사용자의 명시적 요구가 있을 때마다 실행
    - 정기 배치: 정해진 시점(주로 야간)에 정기적으로 실행
  - 배치 스케줄러
    - 일괄 처리를 위해 주기적으로 발생하거나 반복적으로 발생하는 작업을 지원하는 도구.



- 데이터 마이닝(Data Mining)
  - 정의: 대용량의 데이터로부터 사용자의 요구사항에 맞는 의미있는 정보와 지식을 분석하여, 추출하는 방법
  - 등장 배경
    - 기술적 측면: 프로세싱 속도 및 자료저장 구조, 기계학습(ML)기술의 발전, 대량의 데이터 축적 및 데이터마이닝 도구의 발전
    - 비즈니스 측면: Mass 마케팅에서 Target 마케팅으로 패러다임 변화, 대량의 데이터로부터 의미있는 정보 추출을 통한 기업의 새로운 비즈니스 창출/개선/활용.



- 웹 마이닝(Web Mining)
  - 정의: 웹 환경에서 얻어지는 웹 문서, 고객의 정보 및 데이터로부터 특정 행위, 패턴 등의 유용한 정보를 이용하여 마케팅 및 의사결정에 활용하기 위한 마이닝 기법
  - 특징
    - 대용량: 대량의 웹로그를 기반으로 정보를 수집, 자료를 정제 및 클리닝 수행
    - 실시간성: Batch 작업(데이터를 실시간으로 처리하는 것이 아닌 일괄적으로 처리) 성격이 강한 일반 데이터 마이닝과 구별
    - One-to-One: 고객 행위 분석을 통한 개인화 지향(데이터 마이닝은 일반화)
    - 마이닝 기법: 신경망, 연관성, 순서화, 군집화, 의사결정 등에 마이닝 기법 사용



- BCP(Business Continuity Planning)
  - 정의: 지진, 홍수, 천재지변 발생 시, 시스템의 복구, 데이터 복원 등과 같은 단순 복구를 포함하여 기업 비즈니스의 연속성을 보장할 수 있는 체계
  - 등장 배경
    - 재해 대비 시스템 복구 기능이 정보시스템의 필수 요소로 인식되기 시작
    - 기업의 IT 시스템 안정성에 대한 법적 규제가 강화되는 추세
    - 시스템 중지로 인한 영업손실 방지
    - 재해 발생 시 조직 및 개인의 업무 정의 필요
    - 고객 정보 보호 및 업무 연속성 확보를 위한 안정장치 마련
    - 시스템 중단 시 발생하는 기업 이미지 실추 예방
  - 주요 계획
    - 재해 예방: 재해 발생 전 재해 발생 요인에 대응, 위기 관리를 통해 사전에 위기를 정성적, 정량적으로 분석
    - 대응 및 복구: 재해에 대한 정성적, 정량적 평가 항목을 도출, BIA(Business Impact Analysis)를 통한 파급 효과 분석 및 대응 방안 수립, Contingency Plan을 통한 복구 수립



- BCM(Business Continuity Management)
  - 정의: 예기치 않은 상황에서도 비즈니스를 지속적으로 운영하여 비즈니스의 연속성을 보장할 수 있는 경영 방식
  - 특징
    - 복원 능력(Resilience): 업무 중단 발생 시 견뎌낼 수 있는 능력
    - 반복 훈련(Simulation): 계획이 예상대로 수행되고 개선됨을 보장
    - 업무 중단 관리: IMP(Incident Management Plan, 사고 확산 통제), BCP를 통해 업무 중단 관리



- DRS(Disaster Recovery System)
  - 정의: 정보 시스템에 대한 비상 대비체계 유지와 각 업무 조직별 비상사태에 대비한 복구계획 수립을 통한 업무 연속성을 유지할 수 있는 체제
  - 필요성
    - 재해 상황에서 재해복구를 통한 서비스 연속성 확보
    - 연속성 확보를 통한 기업 신뢰도 확보
  - 구축 절차
    - 1단계: 업무 영향 분석
    - 2단계: 재해 복구 전략 수립
    - 3단계: 시스템 구축 및 복구 계획 수립
    - 4단계: 운영 및 모의 훈련



- BIA(Business Impact Analysis)
  - 재해 발생시 영향을 미칠 수 있는 위험을 정의하고, 업무 중단 영향에 대한 정량적, 정성적 분석을 통해 복구 우선순위를 도출하는 과정
  - 목적
    - 핵심 우선순위 결정: 모든 핵심점 사업의 프로세스에 우선순위를 부여함
    - 중단 시간 산정: 경쟁력 있는 기업으로 살아남기 위해 견딜 수 있는 최장시간의 산정
    - 자원 요구: 핵심 프로세스에 대한 지원 요구사항 정의, 시간에 민감한 프로세스에 대부분의 자원을 할당



- 클라우드 컴퓨팅

  - 정의: 하드웨어, 소프트웨어, 데이터 등의 IT 자원이 웹을 통해 표준화된 서비스의 형태로 제공되는 모델 

  - 클라우드로 제공되는 서비스에 대해 사용자는 언제, (IP지원이 되는) 어떤 장비를 통해서든, 원하는 만큼의 서비스를 골라서 사용할 수 있으며, 사용량에 기반하여 비용을 지불하는 비즈니스 모델

  - 3가지 서비스 모델

    > 사진 출처: https://www.redhat.com/ko/topics/cloud-computing/what-is-paas

    ![iaas-paas-saas-diagram](IT_Essential_part3.assets/108083318-4641ac80-70b6-11eb-9b4a-3cb3401ab225.png)

    - SaaS(Software as a Service)
    - PaaS(Platform as a Service)
    - IaaS(Infrastructure as a Service)

  - 4가지 전개 모델

    - Private Cloud: 자사의 내부에 직접 클라우드 인프라를 구축하여 내부에 소속된 사람만 이용
    - Community Cloud: 공통의 관심사를 가진 특정 커뮤니티의 여러 조직이 클라우드를 함께 이용
    - Hybrid Cloud: 내부에 사설 클라우드를 구축하여 운영하다가 필요에 따라 외부의 공용 클라우드를 함께 이용 하는 것
    - Public Cloud: 누구나 함께 이용할 수 있게 구축된 대규모 클라우드서비스

  - 5가지 특징

    - 사용자 중심 서비스: 사용자가 사용한 만큼 비용을 지불
    - 네트워크 접근: 네트워크 기반의 서비스
    - 신속한 서비스 제공
    - 계량 가능한 서비스
    - 컴퓨팅 자원 공유

  - 모바일 클라우드

    - 클라우드 컴퓨팅을 모바일 서비스에 접목시킨 것
    - 구성: 단말, 애플리케이션 제공자, 클라우드 컴퓨팅



- NoSQL	
  - 전통적인 관계형 데이터베이스 보다 덜 제한적인 일관성 모델을 이용하는 데이터의 저장 및 검색을 위한 매커니즘을 제공
  - NoSQL 데이터베이스는 단순 검색 및 추가 작업을 위한 매우 최적화된 키 값 저장 공간.
  - Not only SQL이라 불리기도 한다.
  - MongoDB가 대표적이다.



- TPS(Transaction for second)
  - 초당 트랜잭션의 개수
  - 일정 기간동안 실행된 트랜잭션의 개수를 구하고 다시 1초 구간에 대한 값으로 변경하여 구한다.



- boiler plate code
  - 판에서 찍어낸 듯한 코드라는 의미이다.
  - 아래 조건을 만족하는 코드를 일컫는다.
    - 꼭 필요하고 간단한 기능을 위한 코드다.
    - 반복적으로 사용해야 하는 코드다.
    - 중복되어 많은 양의 코드가 생산된다.



-  스키마(Schema)과 스킴(Scheme)
   - 스키마는 대략적인 계획이나 도식을 뜻한다.
   - 스킴은 구체적이고 확정된 것을 말한다.



- 오버헤드
  - 어떤 처리를 하기 위해 들어가는 간접적인 처리 시간, 메모리 등을 말한다.
  - 예를 들어 A라는 처리를 단순하게 실행한다면 10초가 걸리는데, 안전성을 고려하고 부가적인 B라는 처리를 추가한 결과 처리시간이 15초 걸렸다면, 오버헤드는 5초가 된다.



- mongkey patch

  - 런타임에 프로그램의 특정 기능을 수정하여 사용하는 기법.

    - test를 위해서가 아니라면 지양하는 것이 좋다.
    - 아래 예시를 보면 알 수 있듯, 코드를 예상치 못하게 동작하게 하므로 지양해야 한다.
    - 다만, test 할 때에는 예외로, 만일 의존성을 가지는 함수(예를 들어 API에 요청을 보내 그 응답을 받아오는 함수)를 monkey patch를 통해 다른 함수로 대체하도록 변경하는 등의 방식으로 활용이 가능하다.

  - 어원

    - 원래 이름은 게릴라 패치였다.

    - 이를 발음이 비슷한 고릴라 패치라고 부르다가, 고릴라의 어감이 너무 크고 거대한 느낌이 들어 보다 작은 종인 몽키 패치로 변경되었다.

  - 예시

  ```python
  class Monkey:
      def like_banana(self):
          print("I like banana!")
      
      def hate_banana(self):
          print("I hate Banana")
  
  Monkey.like_banana = hate_banana
  monkey = Monkey()
  monkey.like_banana() # I hate Banana
  ```




- DSL(Domain-Specific Language, 도메인 특화 언어)
  - 특정 분야에 최적화된 프로그래밍 언어.
    - 특정한 목적이 있고 해당 목적의 달성만을 위해서 사용되는 언어
    - SQL의 경우 DB를 조작하는 데 사용하지만 그 이외의 목적으로는 사용하지 않는 DSL이다.
  - 해당 분야 또는 도메인의 개념과 규칙을 사용한다.
  - Python, Java 등의 범용 언어보다 덜 복잡하다.



- Polyglot
  - 여러 언어를 사용하는 것.
  - Polyglot Programming이란 패러다임을 달리 하는 여러 개발 언어를 자유롭게 구사하는 것을 말한다.



- Single Sign-On(SSO, 통합 인증)
  - 한 번의 인증 과정으로 여러 컴퓨터 혹은 애플리케이션 상의 자원을 이용 가능하게 하는 인증
  - 애플리케이션의 대형화, 통합화 추세에 따라 필요성이 증가하게 되었다.



- ETL(Extract, Transform, Load)과 ELT(Extract, Load, Transform)

  - Data를 처리하는 process들이다.
  - ETL
    - 필요한 raw data를 추출한다.
    - 용도에 맞는 포맷으로 변환 한다. 
    - 변환된 data를 data warehouse에 적재한다.
  - ELT
    - 필요한 raw data를 추출한다.
    - 추출한 raw data를 data lake에 load한다.
    - Load 된 data를 필요할 때 data lake에서 꺼내서 transform 한다.

  - ELT의 등장 배경
    - Data가 방대해지기 시작해 transform에 소요되는 시간이 증가하게 되었다.
    - 리소스의 가격 인하 및 클라우드 서비스의 성장으로 raw data를 저장하는데 소요되는 비용이 기존에 비해 인하되었다.
  - Data Warehouse와 Data Lake
    - Data Warehouse는 어느 정도 구조화 된 data들이 모여 있는 곳을 의미한다.
    - Data Lake는 구조화된 data 뿐 아니라 비구조화된 data들도 존재하는 곳이다.



- Programming language
  - Programming language는 규칙과 specification들의 집합이다.
  - 우리가 일반적으로 사용하는 programming language는 사실 규칙과 spec을 구현한 것이다.
    - 예를 들어 Python의 경우 우리가 일반적으로 사용하는 Python은 CPython이라는 Python의 구현체이다.



- magic number

  - 프로그래밍을 할 때 숫자를 변수에 저장하지 않고 바로 입력하는 것을 말한다.
  - 예시
    - 아래 예시에서 140을 magic number라 부른다.

  ```python
  def validate(num):
      if num >= 140:
          return True
  ```

  - 위 코드만 보고서는 140이 뭘 뜻하는지 알 방법이 없다.
    - 함수 이름이 구체적이지 않은 것도 한 몫한다.
  - 만일 아래와 같이 magic number를 사용하지 않는다면, 함수명의 변경 없이도 함수가 하는 일이 보다 분명해진다.

  ```python
  def validate(num):
      MIN_HEIGHT = 140
      if num >= MIN_HEIGHT:
          return True
  ```




- fine-grained와 coarse-grained
  - 프로세스를 어떻게 나눌지에 대한 용어이다.
  - fine-grained
    - 하나의 작업을 여러개의 작은 프로세스를 쪼개어 처리하는 것을 의미한다.
  - coarse-grained
    - 하나의 작업을 큰 단위의 프로세스로 나누어 처리하는 것을 의미한다.





- 루프 불변성(Loop invariant)

  > https://www.geeksforgeeks.org/loop-invariant-condition-examples-sorting-algorithms/

  - 각 iteration의 직전, 직후에 반드시 참이어야 하는 조건 혹은 가정이다.
    - 알고리즘의 타당성, 정확성을 입증하기 위해 사용한다.

  - 루프 불변성의 조건
    - 초기 조건: 첫 순회가 시작하기 전에 참이어야한다.
    - 유지 조건: 각 iteration이 시작되기 전에 참이어야 한다.
    - 종료 조건: 마지막 순회가 종료된 후에도 참이어야 한다.

  - 주의사항
    - 반복 조건과 혼동하기 쉬운데, 반복 조건과는 다르다.
    - 예를 들어 `for(i=0;i<10;i++)`와 같은 코드에서 `i<10`는 이 조건이 참인 동안 반복이 수행되는 반복 조건이지 loop invariant는 아니다.
  - 예시
    - List의 요소들 중 최댓값을 찾는 아래와 같은 code가 있다.
    - 아래 code로 list 내의 최댓값을 찾을 수 있다는 것이 증명되는 이유는, loop invariant가 있기 때문이다.
    - 아래 code의 loop invariant는 `max_num`에는 항상 `lst`의 `i`번째 element까지의 값 중에 최댓값이 담기게 된다는 것이다.
    - 예를 들어, 세 번째 iteration이 끝났을 때, `max_num`에는 7이 담기게 된다.
    - 7이 가장 큰 수라는 것은 list의 3번째 element까지만 고려했을 때 참이다.
    - 만일 이 조건이 거짓이라면, 최댓값을 찾을 수 없다.

  ```python
  lst = [7, 2, 5, 11, 8, 9]
  max_num = -1
  for i in range(len(lst)):
      if lst[i] > max_num:
          max_num = lst[i]
      print("num: {}".format(lst[i]))
      print("max: {}".format(max_num))
      print("-"*8)
  
  """
  num: 7
  max: 7		[7] 중에 7이 최댓값인 것은 True이며, 이는 다음 iteration이 시작되기 직전에도 True이다.
  --------
  num: 2
  max: 7		[7, 2] 중에 7이 최댓값인 것은 True이며, 이는 다음 iteration이 시작되기 직전에도 True이다.
  --------
  num: 5
  max: 7		[7, 2, 5] 중에 7이 최댓값인 것은 True이며, 이는 다음 iteration이 시작되기 직전에도 True이다.
  --------
  num: 11
  max: 11		...
  --------
  num: 8
  max: 11
  --------
  num: 9
  max: 11		[7, 2, 5, 11, 8, 9] 중에 11이 최댓값인 것은 True이며, 이는 반복이 끝난 이후에도 True이다.
  --------
  """
  ```



- Stack frame
  - 메모리의 stack 영역은 함수의 호출과 관련된 지역 변수와 매개 변수가 저장되는 영역이다.
    - 함수의 호출과 함께 할당되며, 함수의 종료와 함께 소멸된다.
    - 이를 통해, 함수가 종료되면, 해당 함수가 호출되기 이전의 상태로 돌아갈 수 있게 된다.
  - Stack overflow
    - 함수의 호출이 중첩되거나, 재귀 함수를 사용할 경우, stack frame이 더 이상 데이터를 저장할 수 없을 정도로 차게 되는 경우가 있다.
    - 이를 stack overflow라 한다.



- CLI, 터미널, 셸
  - CLI
    - 문자로만 이루어진 인터페이스를 뜻한다.
    - 실체가 없는 추상적인 단어로, 문자열을 입력하거나 문자열을 출력하는 것만 가능하다.
  - 터미널
    - 터미널이란 입력과 출력이 가능한 하드웨어 장비를 의미한다(현재는 물리 하드웨어를 지칭하는 단어로는 거의 사용하지 않는다).
    - 콘솔이라고도 부른다.
    - CLI가 물리적으로 구현된 것.
  - 셸
    - CLI로 구현된 터미널에서 실행 가능한 대화형 프로그램
    - 셸은 REPL(Read-Eval-Print Loop)이라고도 불린다.
    - bash는 셸의 한 종류이며, 리눅스의 표준이다.



- vector

  - Vector라는 단어의 기원은 무언가를 옮긴다는 것이다.
    - vehicle, wagon, way 등과 기원이 같다.
    - 또한 바이러스나 병원체를 옮기는 사람 혹은 동물을 뜻할 때 사용하기도 한다.
  - 수학에서 vector란 size와 direction을 가지고 있는 양을 뜻한다.
    - 숫자는 그 자체로 size를 가지고 있다고 볼 수 있다.
    - vector의 direction이 vector의 원래 뜻과 수학을 연관짓는 키이다.
  - 일반적으로 array 형태로 표현되지만 아래와 같이 map 형태로 표현될 수도 있다.
    - key를 축으로, value를 축 위의 점으로 생각한다면 아래와 같은 vector의 형태도 있을 수 있다.

  ```json
  {
      "name":"Happiness",
      "age":3,
      "address":"xxx street",
      "floor":2
  }
  ```



- State Machine
  - 컴퓨터 프로그램과 전자 논리 회로를 설계하는 데 사용하는 수학적 모델.
    - 특정한 상태를 가질 수 있는 오토마타(추상 기계)이다.
    - 한 번에 오로지 한 개의 상태만을 가진다.
  - 유한 개의 state로 구성된 Finite State Machine(FSM)과 무한 개의 state로 구성되는 Infinite State Machine(ISM)으로 나뉜다.
    - 실제 세계는 대부분 FSM으로 구성되어 있다.
    - 그러므로 일반적으로 state machine이라 하면 FSM을 가리킨다.
  - 개념
    - State: Object의 상태를 의미한다.
    - Transition: 한 state에서 다른 state로의 전이를 의미한다.
    - Event(Transition Condition): Transition을 발생시키는 사건이다.
    - Entry Action: 새로운 state로 transition 될 때 행해지는 동작이다.
  - 장점
    - Transition 이후 action에 의해서만 state가 변경되기에 안정성이 높다.
    - 직관적이다.


