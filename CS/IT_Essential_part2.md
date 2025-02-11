# RPC

> https://velog.io/@jakeseo_me/RPC%EB%9E%80
>
> https://www.getoutsidedoor.com/2019/07/11/what-is-grpc/

- RPC(Remote Procedure Call, 원격 프로시저 호출)
  - 별도의 원격 제어를 위한 코딩 없이 다른 주소 공간에서 함수나 프로시저를 실행할 수 있게 하는 프로세스 간 통신 기술
    - 프로세스간 통신을 위해 사용하는 IPC(Inter Process Communication) 방법의 한 종류.
    - 분산 컴퓨팅 환경에서 프로세스 간 상호 통신 및 컴퓨팅 자원의 효율적인 사용을 위해서 발전된 기술이다.
    - 원격지의 프로세스에 접근하여 프로시저 또는 함수를 호출하여 사용한다.
    - RPC를 이용하면 프로그래머는 함수가 실행 프로그램에 로컬 위치에 있든 원격 위치에 있든 동일한 코드를 이용할 수 있다.
  - 함수와 프로시저의 차이
    - 함수(function): input에 따른 output의 발생을 목적으로 하는 것으로, Return값을 필수로 가져야한다.
    - 프로시저(Procedure): output 값 자체에 집중하기보단, 명령 단위가 수행하는 절차에 집중한 개념으로, Return 값이 없을 수도 있다.
  - RPC의 기능
    - 언어나 환경에 구애받지 않고 Client-Server 간의 커뮤니케이션을 가능하게 한다.
    - 일반적으로 프로세스는 자신의 주소공간 안에 존재하는 함수만 호출하여 실행 가능하다.
    - 그러나, PRC는 네트워크를 통한 메시징을 통해 다른 주소 공간의 함수나 프로시저를 호출할 수 있게 해 준다.
    - 이를 통해 MSA(Micro Service Architecture) 서비스를 만들 때, 언어나 환경에 구애받지 않고, 비즈니스 로직을 개발하는 데 집중할 수 있다.



- IDL(Interface Definition Language)

  - 소프트웨어 컴포넌트의 인터페이스를 묘사하기 위한 명세 언어
    - 어느 한 언어에 국한되지 않는 언어 중립적인 방법으로 인터페이스를 표현함으로써, 같은 언어를 사용하지 않는 소프트웨어 컴포넌트 사이의 통신을 가능하게 한다.
    - 즉, 두 개의 시스템을 연결하는 다리 역할을 한다.

  - RPC는 IDL을 사용하여 인터페이스를 명시한다.
    - 서버와 클라이언트가 서로 다른 언어를 사용하는 경우도 있다.
    - 서로 다른 언어가 요청과 응답을 주고 받기 위해 인터페이스를 통해 규칙을 명세하고, 각자의 시스템이 이해할 수 있는 형태로 변형하는 작업이 필요한데, IDL이 이러한 역할을 담당한다.
  - 정의된 IDL을 기반으로 rpcgen이라는 rpc 프로토콜 컴파일러를 통해 코드(stub)가 생성된다.
    - Stub은 클라이언트에게는 서버의 프로시저 호출을 위한 참조자가 된다.
    - 서버에게는 프로시저 이해를 위한 참조가 된다(서버 측의 stub을 skeleton이라 부른다.).
    - stub은 원시소스코드 형태로 만들어지므로 클라이언트, 서버 프로그램에 포함하여 빌드한다.





- RPC의 구조 및 동작 과정

  ![img](IT_Essential_part2.assets/operating-system-remote-procedure-call-1-16532903490084-16533502803281.png)

  ![img](IT_Essential_part2.assets/operating-system-remote-call-procedure-working-16532903624116-16533502803292.png)

  - 클라이언트가 클라이언트 스텁 프로시저를 작동시키고, 파라미터를 전달한다.
  - 클라이언트 스텁이 메시지에 파리미터를 Marshall(Pack, 직렬화)한다.
    - Marshall: 파라미터 값을 표준화된 형식으로 변환시키고, 각 파라미터를 메시지에 복사하는 과정
    - 데이터 형태를 XDR(eXternal Data Representation) 형식으로 변환한다.
    - XDR로 변환하는 이유는 integer, float과 같은 기본 데이터 타입에 대해서 머신마다 메모리 저장 방식(i.e. little endian, big endian)이 CPU 아키텍처 별로 다르며, 네트워크 전송 과정에서 바이트 전송 순서를 보장하기 위해서이다.
  - 클라언트 스텁이 메시지를 전송 계층(transport layer)에 넘겨준다.
    - 전송 계층은 해당 메시지를 원격 서버로 보낸다.
  - 서버 측에서 전송 계층이 받아온 메시지를 서버 스텁에 넘겨준다.
    - 서버 스텁은 메시지를 Demarshall(Unmarshall, Unpack)한다.
  - 서버 스텁은 프로시저를 호출한다.
  - 서버 프로시저가 종료되면, 서버 스텁을 반환하고, 서버 스텁은 반환 값을 Marshall하여 메시지에 담는다.
    - 이 메시지를 전송 계층에 전달한다.
  - 전송 계층은 반환 값을 다시 클라이언트의 전송 계층에 전달한다.
  - 클라이언트의 전송 계층은 받은 메시지를 클라이언트 스텁에 전달한다.
  - 클라이언트 스텁은 받아온 반환 파라미터값을 Demarshall하고, 실행한다.



## gRPC

> https://chacha95.github.io/2020-06-15-gRPC1/
>
> https://grpc.io/docs/what-is-grpc/introduction/

- gRPC
  - Google에서 개발한 RPC 시스템
  - TCP/IP 프로토콜과 HTTP 2.0 프로토콜을 사용한다.
    - HTTP 2.0은 HTTP 1.1의 프로토콜을 계승해 성능을 더욱 발전시켰다.
  - HTTP 2.0의 특징
    - Multiplexed Streams: 한 connection으로 동시에 여러 개의 메시지를 주고 받을 수 있으며, Response는 순서와 상관 없이 stream으로 주고 받는다.
    - Stream Prioritization: 리소스간 우선 순위를 설정해 클라이언트가 먼저 필요로하는 리소스부터 보내준다.
    - Server Push: 서버는 클라이언트의 요청에 대해 요청하지 않은 리소스를 마음대로 보내줄 수 있다.
    - Header Compression: Header table과 Huffman Encoding 기법을 사용해 header를 압축한다.
  - IDL로 protocol buffers를 사용한다.
    - google에서 자체 개발한 IDL이다.
    - 구조화된 데이터를 직렬화 할 수 있게 해준다.
    - 다른 방식(XML, Json 등)에 비해 간단하고, 파일 크기도 3~10배 작으며 속도도 20~100배 빠르다는 장점이 있다.
    - Json 같은 IDL도 사용이 불가능 한 것은 아니다.
  - Python, Java, C++, C#, JavaScript, Ruby 등에서 사용 가능하다.



- gRPC의 구조

  ![Concept Diagram](IT_Essential_part2.assets/landing-2-16534518558762.svg)

  - 다른 RPC들과 마찬가지로 service를 정의하고, 원격으로 호출할 수 있는 메서드들을 정의하고, type을 반환한다.
  - 서버에서는 위와 같은 명세를 구현하고 gRPC 서버를 실행시킨다.
  - 클라이언트는 stub을 통해 gRPC 서버와 통신한다.



### Protocol Buffers

- Protocol Buffers
  - Google에서 개발한 IDL이다.
    - Open source로 개발되었다.
  - 구조화된 데이터를 직렬화하는 데 사용한다.
  - 일반적인 text 파일과 동일하지만, `.proto`라는 확장자를 지닌다.



- 개요

  - 구조 정의하기
    - 직렬화 하려는 데이터의 구조를 정의한다.
    - `message`라는 키워드를 사용하며, field라 불리는 name-value 쌍으로 정의한다.

  ```protobuf
  message HelloRequest {
    string name = 1;
  }
  
  message HelloReply {
    string message = 1;
  }
  ```

  - 서비스 정의하기
    - `message` 키워드를 통해 정의해둔 데이터들로 RPC method parameter들과 return type을 정의한다.

  ```protobuf
  service Greeter {
    rpc SayHello (HelloRequest) returns (HelloReply) {}
  }
  ```

  - 컴파일하기
    - protocol buffer compiler(`protoc`)을 통해 proto 파일을 컴파일한다.
    - 자신이 원하는 언어로 컴파일이 가능하며 컴파일 과정을 통해 message를 통해 정의한 데이터에 접근할 수 있는 방법과, raw bytes로 직렬화하거나 parsing하는 메서드가 각 언어에 맞게 생성된다.







# Session과 JWT

> https://jwt.io/introduction

- 사용자 인증에 사용되는 두 가지 방식이다.
  - [쿠키, 세션, 웹 스토리지, IndexedDB]에서 본 것과 같이 HTTP는 기본적으로 stateless하다.
  - 그러나 사용자 정보와 같이 지속적으로 유지되야 하는 정보가 존재한다.
    - 클라이언트가 서버에 요청을 보낼 때마다 로그인을 할 수는 없다.



- 기존에는 session 방식을 사용했으나 최근에는 JWT 방식을 많이 사용한다.
  - 각각의 장단점이 있으므로 서비스에 더 적합한 방식을 택하면 된다.



## Session 방식

- Session 방식의 인증 과정은 다음과 같다.
  - Client(browser)가 server로 login 요청을 보낸다.
  - Server는 username과 password 등을 확인하고 일치하면 Session(DB 혹은 Redis 등 데이터를 저장하고 key값 기반으로 데이터를 조회할 수 있는 저장소)에 key, value, expire_date 등을 저장한다.
  - Server는 session_id(key)를 client에 반환하고, client는 해당 정보를 cookie에 저장한다.
  - 이후, 인증이 필요한 요청을 서버로 보낼 때마다,  session_id를 함께 보낸다.
  - 서버는 session_id가 유효한지 확인하고, session_id로 session에 저장된 data를 조회하여 처리 후 client에 응답을 보낸다.



- 장점
  - Client가 가지고 있는 정보는 session_id뿐이므로 중간에 탈취 당해도 정보 유출의 위험이 없다.
  - 서버단에서 session에 대한 통제가 가능하다(예를 들어 모바일과 데스크탑의 동시 접속을 막으려고 할 경우 이미 한 기기에서 로그인한 사용자라면 해당 기기로 발급된 session을 삭제하고 새로운 기기에 대한 session을 생성하면 된다).



- 단점
  - 사용자 수가 증가하면 session도 늘려줘야한다.
  - 매 요청마다 session을 조회해야 한다.



## JWT 방식

> https://jwt.io/introduction
>
> https://velopert.com/2389

- JWT

  - 안전한 정보 교환을 위한 개방형 규약([RFC 7519](https://tools.ietf.org/html/rfc7519))이다.
    - 정보를 주고 받을 때, JSON object를 활용한다.
    - 자가수용적(Self-contained)인 특징을 가지고 있어, JWT 토큰 자체에 전달하고자 하는 데이터, 검증할 수 있는 서명 데이터 등이 모두 포함되어 있다.
    - 정보가 전자 서명되어 있어 검증이 가능하고 신뢰할 수 있다.
    - HMAC 알고리즘을 사용하는 비밀키나  RDS 또는 ECDSA와 같은 공개키, 개인키 쌍으로 서명될 수 있다.
  - 형식
    - JWT는 Header, Payload, Signature의 세 부분으로 구성된다.
    - 각 부분은 `.`으로 나뉜다.
  
  ```bash
  eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiaWF0IjoxNTE2MjM5MDIyfQ.SflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV_adQssw5c
  ```



  - JWT의 목적은 데이터 자체를 보호하는 것이 아니다.
    - JWT의 목적은 데이터가 인증된 소스에서 생성되었음을 sign을 통해 증명하는 것이다.
    - Token에 서명을 하면 변조로부터 보호되지만 누구나 읽을 수 있다. 
    - 실제로 header, payload는 어떠한 암호화도 거치지 않고 단지 encoding만 된 상태이기에, decoding만 하면 누구나 정보를 볼 수 있다.
    - 따라서, JWT의 header 또는 payload에 담은 정보가 암호화되지 않은 경우 header 또는 payload에 비밀 정보를 넣어선 안 된다.



- Header

  - JWT를 어떻게 계산할 것인지에 대한 정보가 담겨 있다.
  - 일반적으로 token의 type과 sign 알고리즘(e.g. HMAC SHA256 or RSA.)에 대한 정보가 담겨 있다.

  ```json
  {
    "alg": "HS256",
    "typ": "JWT"
  }
  ```

  - 위 정보를 base64Url로 encoding한다.

  ```bash
  eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9
  ```



- Payload

  - 실제로 전달하고자 하는 정보가 담겨있는 부분이다.
  - Claim이 담겨 있는데, claim이란 entitiy와 그에 대한 추가적인 정보를 담고 있는 조각을 의미한다.
    - 즉 name, value의 한 쌍을 claim이라 부르고, name을 entity라 부른다.
  - Claim에는 3가지 종류가 있다.
    - Registered claims: 꼭 토큰에 포함시켜야 하는 것은 아니지만, 토큰에 포함시키는 것을 추천하는 claim들로, JWT 스펙에 정의되어 있는 claim들이다. iss(issuer), exp(expiration time), sub(subject), aud(audience) 등이 이에 속한다.
    - Public claims: JWT를 사용하는 사람들이 정의하는 claim들이다. 충돌을 피하기 위해서  [IANA JSON Web Token Registry](https://www.iana.org/assignments/jwt/jwt.xhtml)나 URI로 정의한다.
    - Private claims: JWT를 사용하여 통신을 주고 받는 당사자들이 협의한 claim들이다.

  ```json
  {
    "sub": "1234567890",
    "name": "John Doe",
    "iat": 1516239022
  }
  ```

  - 위 정보를 base64Url로 encoding한다.

  ```bash
  eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiaWF0IjoxNTE2MjM5MDIyfQ
  ```



- Signature

  - 생성 과정
    - Header를 base64Url로 encoding한 값과, payload를 base64Url로 encoding한 값을 `.`을 사이에 두고 합한다.
    - 비밀키와 header에서 `alg`에 지정한 암호화 알고리즘을 사용하여 생성한다.

  ```json
  // Signature를 생성하는 슈도코드
  HMACSHA256(
    base64UrlEncode(header) + "." +
    base64UrlEncode(payload),
    your-256-bit-secret
  )
  ```

  - 위 정보를 base64Url로 인코딩한다.

  ```bash
  SflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV_adQssw5c
  ```



- 인증 과정
  - Client에서 Server로 사용자의 인증 정보를 전송한다.
  - Server는 인증 정보가 정확한지 확인 후 JWT를 생성하여 client에 전달한다.
  - Client는 이를 어딘가(client의 플랫폼에 따라 다르다)에 저장한다.
  - 이후부터 발생하는 모든 인증이 필요한 요청에 JWT를 담아서 Server로 보낸다.
  - 서버는 JWT를 비밀키를 통해 JWT의 signature를 검증하여 변조 여부를 확인하고, 이상이 없으면  JWT로부터 사용자 정보를 추출한다.



- 장점
  - 서버에서 관리하지 않으므로, 사용자가 늘어 JWT를 많이 발급해도 서버의 자원 사용이 증가하지 않는다.
  - 쿠키가 아닌 다른 곳에 저장(일반적으로 local storage)이 가능하므로 쿠크릴 사용할 수 없는 모바일 애플리케이션 등에서도 사용이 가능하다.



- 단점
  - Payload, Header등이 암호화되지 않은 상태로 그대로 노출되므로, 개인정보가 담길 경우 위험할 수 있다.
  - 서버에서 JWT를 저장하는 것이 아니므로, 서버에서 발급된 순간 서버가 통제할 수 있는 방법이 없다.
    - 예를 들어, 특정 사용자를 강제로 로그아웃 시킬 경우, 세션 기반 인증에서는 해당 시용자의 세션을 삭제하면 그만이지만, JWT 방식에서는 불가능하다.



- JWT의 보안 전략
  - 만료 기한을 최대한 짦게 설정한다.
    - 토큰이 탈취되더라도 빠르게 만료되기에 피해를 최소화 할 수 있다.
    - 사용자가 자주 로그인해야 한다는 문제가 있다.
  - Sliding Session
    - Client가 JWT를 server로 보낼 때 마다 JWT의 갱신을 요청하는 방법이다.
    - 매 요청마다 갱신이 이루어지도록 할 수도 있고, 글 쓰기, 결제 등 중간에 인증이 만료되선 안 되는 작업에만 한정적으로 적용할 수도 있다.
  - Refresh Token
    - 최초 토큰을 생성할 때, Access Token과 Refresh Token을 발급하는 방법이다.
    - Client가 만료된 Access Token으로 Server와 통신을 진행하다 Access Token이 만료된다.
    - Server는 client에 access token이 만료되었다는 응답을 보내고, client는 access token보다 유효기간이 긴 refresh token을 server로 보내 access token의 발급을 요청한다.
    - Server는 client가 보낸 refresh token이 server에 저장된 refresh token과 비교하여, 유효한 경우 새로운 access token을 발급하고, 유효하지 않은 경우 다시 로그인을 요구한다.
    - 서버가 refresh token을 관리하므로 refresh token을 만료시켜 access token의 재발급을 막을 수 있다는 장점이 있다.
    - 그러나 refresh token을 어딘가에 저장하고 조회해야 하기에 I/O가 발생하고, 이는 JWT의 장점을 충분히 활용하지 못하는 것이다.
    - 또한 client는 refresh token이 탈취되지 않도록 추가적인 처리가 필요하다.



- Python으로 JWT 생성하기

  - 대부분의 언어에 JWT를 다루는 라이브러리가 개발되어 있다.
    - python의 경우 PyJWT가 있다.
  - 그러나 JWT의 생성 과정을 위해서 JWT를 생성하는 스크립트를 간단하게 작성하면 아래와 같다.

  ```python
  import json
  import base64
  import hmac
  import hashlib
  
  
  class JWTCreator:
      def __init__(self, header, payload, secret_key):
          self.header = header
          self.payload = payload
          self.secret_key = secret_key
          self.algorithms = {
              "HS256":hashlib.sha256,
              "HS512":hashlib.sha512
          }
          self.algorithm = self.algorithms[header["alg"]]
          self.segments = []
      
      def encode_header(self):
          str_header = json.dumps(self.header, separators=(",", ":")).encode()
          encoded_header = base64.urlsafe_b64encode(str_header).replace(b"=", b"")
          self.segments.append(encoded_header)
      
      def encode_payload(self):
          str_payload = json.dumps(self.payload, separators=(",", ":")).encode()
          encoded_payload = base64.urlsafe_b64encode(str_payload).replace(b"=", b"")
          self.segments.append(encoded_payload)
      
      def sign(self):
          # header와 payload를 더한 값을 만든다.
          signing_input = b".".join(self.segments)
          # header에 담긴 alogrithm으로 sign을 생성한다.
          signature = hmac.new(self.secret_key.encode("utf-8"), signing_input, self.algorithm).digest()
          encoded_signature = base64.urlsafe_b64encode(signature).replace(b"=", b"")
          self.segments.append(encoded_signature)
          
      def create_jwt(self):
          self.encode_header()
          self.encode_payload()
          self.sign()
          
          encoded_string = b".".join(self.segments)
  
          return encoded_string.decode("utf-8")
  
  
  header = {
      "alg": "HS256",
      "typ": "JWT"
  }
  
  payload = {
    "sub": "1234567890",
    "name": "John Doe",
    "admin": True,
    "iat": 1516239022
  }
  
  secret_key = "skansmfqhtlfwpfhqhslRkdhkswjsekzmthdnfahstmxjcjfjatodruTejfkRkaWkrshffkTdj"
  
  jwt_creator = JWTCreator(header, payload, secret_key)
  print(jwt_creator.create_jwt())
  ```

  



# Text Encoding

- Encoding
  - 컴퓨터는 전기 신호로 정보를 표현한다.
    - 전기 신호가 있으면 1, 없으면 0이 되며 이 신호의 유무가 데이터의 최소 단위가 되고 이를 비트(bit)라 한다.
    - 하나의 비트는 0 또는 1만 표현할 수 있기 때문에 다양한 데이터를 표현하는 것은 불가능하다.
    - 따라서 다양한 데이터를 표현하기 위해서 여러 비트를 하나로 묶은 byte라는 단위를 사용한다.
    - 바이트란 일정한 개수의 비트로 이루어진 연속된 비트열로, 바이트를 구성하는 비트의 개수가 정해진 것은 아니지만 일반적으로 8비트를 1바이트로 사용한다.
    - 한 개의 비트는 0 또는 1이라는 2개의 정보를 표현할 수 있는데 이를 8개로 묶으면 256(2의 8승)가지의 정보를 표현할 수 있다.
  - 문자열 역시 byte로 표현되는데, 문자를 byte로 변환하는 것을 encoding이라 부른다.



- ASCII(American Standard Code for Information Interchange)

  - 이름에서 알 수 있듯이 미국에서 정의한 부호 체계이다.

  - 7비트로 character 하나를 표현한다.
    - 즉 128개의 문자를 표현할 수 있으며, 33개의 출력 불가능한 제어 문자들과 95개의 출력 가능한 문자들로 이루어져 있다.
    - 제어 문자들 중 대부분은 더 이상 사용되지 않는다.
    - 95개의 문자는 52개의 영문 알파벳 대소문자와 10개의 숫자, 32개의 특수문자, 하나의 공백 문자로 구성된다.
  - 8비트가 아닌 7비트를 사용하는 이유는 1비트는 통신 에러 검출을 위해 사용하기 때문이며, 이를 위해 사용되는 비트를 Parity bit라 부른다.



- ANSI(American National Standards Institute) 코드
  - American National Standards Institute에서 만든 문자 코드이다.
  - ASCII 코드만으로는 영어가 아닌 언어를 표현하기에는 부족하다.
    - 따라서 1bit를 추가하여 0~127까지는 기존의 ASCII코드와 동일한 문자를 할당하고 나머지 128개로 각 나라의 문자를 표현하는 방식으로 만들었다.
  - 특정 인코딩 방식을 말하는 것이 아니라 code page를 의미한다.
    - 각 언어별로 code page가 존재하며, 어떤 코드 페이지를 사용하는 가에 따라 정의된 문자가 달라진다.
    - 예를 들어 cp949는 한글을 위한 code page이다.
  - EUC-KR
    - 한글을 위한 code page로 완성형 코드 조합이다.
    - 완성형 코드란 완성된 문자 하나하나 코드 번호를 부여한 것이다. 반대 개념으로 조합형 코드가 있다. 예를 들어 "감"이라는 문자가 있을 때  "ㄱ", "ㅏ", "ㅁ"에 각기 코드를 부여하고 이를 조합하여 만드는 것이 조합형, "감"이라는 문자 자체에 코드를 부여한 것이 완성형 코드이다.
    - ASCII에 속한 문자들은 1byte, 한글은 2bytes로 표현하는 가변 길이 인코딩 방식이다.
  - cp949
    - MS에서 EUC-KR을 확장하여 만든 완성형 코드 조합이다.



- Unicode
  - 등장 배경
    - 1byte로 표현 불가능한 언어들(자음과 모음의 조합으로 한 글자가 완성되는 한글, 1만자가 넘어가는 한자가 존재하는 중국어 등)이 존재.
    - ANSI 방식의 경우도 각 언어마다 각기 다른 code page를 지니고 있어 사용에 여러 언어를 동시에 사용하는 데 불편함이 있었다.
    - 이와 같은 문제를 해결하기 위해 세상의 모든 언어를 하나의 코드 표에 담은 unicode가 등장했다.
  - 총 1,114,112개의 문자를 표현 가능하다.
    - 따라서 기본다중언어판(BMP, Basic Multilingual Plane)이라 불리는 16비트(2bytes) 영역에 모든 문자를 집어넣을 수 있으리라 생각했다.
    - 그러나 고어, 사어, 각종 특수 문자등으로 인해 65,536개로도 부족하게 되었다.
    - 이에 따라 BMP의 2048자를 대행 코드 영역(Surrogates)으로 할당하고, 이 중 1024자를 상위 대행(high surrogates), 1024자를 하위 대행(low surrogates)로 정의하여 이 둘의 조합으로 다시 1,048,576자를 추가로 정의할 수 있도록 했다.
    - 따라서 기존의 65,536+추가된 1,048,576으로 총 1,114,112개의 문자를 표현 가능해졌다.
  - 유니코드는 전체 공간이 17개의 판(plane)으로 나뉘어 있다.
    - unicode가 표현할 수 있는 전체 문자의 수 인 1,114,112를 2bytes로 표현할 수 있는 문자의 수인 65,536로 나누면 17이 된다.
    - 1개의 BMP와 16개의 보충언어판(SMP, Supplementary Multilingual Plane)로 구성된다.



- UTF(Unicode Transformation Format)
  - 이름에서도 알 수 있듯 unicode로 인코딩하는 방식이다.
  - 뒤에 숫자가 붙는데 이 숫자는 몇 bit를 1byte로보고 인코딩 할 것인지를 의미한다.
    - 예를 들어 UTF-8은 8bit를 1byte로 보고 인코딩한다.
  - UTF-8 
    - 일반적으로 가장 많이 쓰이는 인코딩 방식이다.
    - 문자를 1byte~6bytes까지 가변적으로 인코딩하는 방식.
    - 예를 들어 ASCII 코드상의 문자들은 1bytes로, 아시아 문자는 3bytes로 인코딩한다.
    - 표현 가능한 길이는 최대 66bytes지만 다른 인코딩과의 호환을 위해 46bytes까지만 사용한다.



- Base64
  
  - Binary data를 64진법으로 표현된 ASCII코드 문자열로 변환시키는 인코딩이다.
    - 64개의 문자를 사용한다고 하여 Base 64라는 이름이 붙었다.
    - ASCII 문자들 중 제어문자를 제외하고 아래와 같은 64개의 문자들을 사용 한다.
    - `A-Z`, `a-z`, `0-9`, `+`, `/`
    - 여기에 추가로  `=`도 사용하는데, 이는 padding을 위해 사용한다.
    - 64개의 문자를 사용하므로, 64개의 문자를 표현할 수 있는 6bit(2**6==24)를 사용한다.
  
  - 64인 이유는 2의 제곱수 중 ASCII 코드를 가장 많이 표현할 수 있는 수이기 때문이다.
  
    - ASCII 문자는 128개의 문자를 표현할 수 있다.
    - 그러나 제어문자를 제외하면 실제 표현 가능한 문자는 95개뿐이다.
    - 따라서 2의 제곱수 중 64가 ASCII 코드를 가장 많이 표현할 수 있는 수이다.
  
  - 24bit 크기의 버퍼를 사용하여 데이터를 담는다.
  
    - 대부분의 컴퓨터는 8bit를 1byte로 사용한다.
    - 그러나 base64는 6bit로 하나의 charater를 표현한다.
    - 따라서 8과 6의 최소공배수인 24bit를 버퍼의 크기로 사용한다.
    - 만일 24bit를 채우지 못 할 경우 남은 bit를 0으로 채우는 padding을 거친다.
    - 즉 3byte의 data를 4개의 charater로 표현한다.
  
  - 인코딩 예시1
  
    > 색인표는 [링크](https://en.wikipedia.org/wiki/Base64) 참고
  
    - Hello를 base64로 인코딩 하는 과정은 아래와 같다.
    - 먼저 각 charater를 ascii code 값으로 변환하면 `72 101 108 108 111`가 된다.
    - 그 후 각 ascii code를 2진수로 변환하면`01001000 01100101 01101100 01101100 01101111`가 된다.
    - Base64는 3byte(24bits)를 버퍼 사이즈로 사용하는데, 위 값은 3의 배수가 아닌 5이므로 맨 마지막에 `00000000`을 추가해준다.
    - 결국 `01001000 01100101 01101100 01101100 01101111 00000000`이 된다.
    - 이를 6bit씩 끊으면 `010010 000110 010101 101100 011011 000110 111100 000000`이 된다.
    - Base64 index에서 각 2진수에 해당하는 값을 찾아 변환해주면 `SGVsbG8`이 된다.
    - 위에서 6bit 덩어리들의 개수를 4의 배수로 맞추기 위해 `00000000`를 padding을 했다는 것을 표시하기 위해 `=`을 추가하면 최종적으로 `SGVsbG8=`가 된다.
  
  - 인코딩 예시2
  
    - Theo를 base64로 인코딩하는 과정은 아래와 같다.
    - 각 charater를 ascii code 값으로 변환하면 `84, 104, 101, 111`가 된다.
    - 이를 2진수로 변환하면 `01010100 01101000 01100101 01101111`이 되는데, 이 역시 3의 배수가 아닌 4이므로 3의 배수가 되도록 `00000000`을 2개 추가해준다.
    - 그 후 이를 6bits씩 자르면 `010101 000110 100001 100101 011011 110000 000000 000000`이 되고, 이를 base64 index에서 찾아 변환하면 `VGhlbw`가 된다.
    - 위에서 전체 byte의 개수를 3의 배수로 맞추기 위해 `00000000`을 2개 추가했으므로  `=`을 2개 추가하면 `VGhlbw==`가 된다.
    
  - Padding
  
    - 마지막에 추가되는 padding  문자 `=`의 개수는 위 처럼 모자란 3의 배수가 되기에 byte의 개수를 추가해주는 방법이 있다.
    - 다른 방법도 하나 있는데, 6bits씩 자른 덩어리들의 마지막 덩어리의 bit수를 보고도 알 수 있다.
    - 예를 들어 `11`이면 6bit를 채우기 위해서는 `00`을 두 개 추가하여 `110000`이 되어야 한다. 이 때 추가한 `00`의 개수만큼 `=`을 추가하면 전체 byte의 개수를 몰라도 padding을 추가할 수 있다.
    - 또 다른 방법으로는 6bit씩 자른 마지막 덩어리의 bits수를 3으로 나눈 나머지 만큼 `=`을 추가해도 된다.
    
  
  ```python
  import base64
  from faker import Faker
  
  
  fake = Faker()
  text = fake.name()
  binaries = []
  for char in text:
      binary = str(format(ord(char), "b"))
      while len(binary) != 8:
          binary = "0" + binary
      binaries.append(binary)
  
  n = len(binaries)
  print(n)
  num = 3
  cnt = 1
  while True:
      if n <= num*cnt:
          print(num*cnt-n)
          break
      cnt += 1
  
  binary_string = "".join(binaries)
  six_bits = [binary_string[i:i+6] for i in range(0,len(binary_string), 6)]
  
  print(len(six_bits[-1]) % 3)
  
  print(base64.b64encode(text.encode()))
  ```
  
  - Python으로 변환하기
  
    - 사실 Python의 경우 `base64` 패키지를 사용하면 훨씬 간편하게 인코딩이 가능하다.
  
  ```python
  def convert_to_binary(ascii_code):
      binary_string = ""
      while ascii_code:
          if ascii_code % 2 == 0:
              binary_string += "0"
          else:
              binary_string += "1"
          ascii_code //= 2
  
      while len(binary_string) != 8:
          binary_string += "0"
  
      return binary_string[::-1]
  
  
  def get_base64_index(six_bits):
      index = 0
      position = 5
      for bit in six_bits:
          if int(bit):
              index += 2 ** position
          position -= 1
  
      return index
  
  
  def make_base64_index():
      base64_index = {
          62: "+",
          63: "/"
      }
      num_to_plus = 65
      for i in range(62):
          if i == 26:
              num_to_plus += 6
          if i == 52:
              num_to_plus = -4
          base64_index[i] = chr(i + num_to_plus)
  
      return base64_index
  
  
  base64_index = make_base64_index()
  
  ascii_codes = [ord(char) for char in input()]
  binary_string = ""
  for ascii_code in ascii_codes:
      binary_string += convert_to_binary(ascii_code)
  
  base64_encoded = ""
  num_padding = 0
  for i in range(0, len(binary_string), 6):
      six_bits = binary_string[i:i + 6]
  
      length = len(six_bits)
      if length != 6:
          num_padding = length % 3
          for _ in range(6 - length):
              six_bits += "0"
  
      print(base64_index[get_base64_index(six_bits)], end="")
  
  for _ in range(num_padding):
      print("=", end="")
  ```
  
  - ASCII는 text를 bytes로 변환하는 데 반해, base64는 data를 text로 변환한다.
    - 즉 ASCII는 text를 byte로 변환하기 위한 encoding이고, base64는 data를 text로 변환하기 위한 encoding이다.
    - 즉, 위에서도 확인했듯 base64의 결과값은 bytes가 아니라 string이다.
  - Base64를 사용하는 이유
    - Base64를 사용하여 인코딩하면, 인코딩하기 전 보다 사이즈가 더 커지게 된다.
    - 그럼에도 base64를 사용하는 이유는 ASCII코드의 경우 8bits 중 7bits로 data를 표현하고, 나머지 1bit를 처리하는 방식이 시스템별로 통일되어 있지 않고, 제어문자의 경우도 시스템 별로 다른 코드 값기 때문에 문제가 생길 수 있다.
    - 따라서 ASCII 코드들 중 모든 시스템에 동일하게 적용될 수 있는 64개의 문자로만 데이터를 전달하기 위해 base64를 사용한다.
    - 그냥 2진수를 보내면 되는 것 아닌가라고 생각할 수 있지만, text만을 input으로 받는 경우 2진수를 바로 보낼 수 없다.
  
  - Base32, base16 등
    - 이들은 모두 base64와 동일한 과정으로 거치지만 각각 5bit, 4bit로 data를 표현하여 표현할 수 있는 문자의 개수가 base64보다는 적다.



- 한글 유니코드

  - 한글은 유니코드 중 0x0000 ~ 0xFFFF 영역을 표현하는 기본 영역 내부에 정의되어 있다.
  - 한글을 표현하는 유니코드 범위
    - `  Hangul Syllables`는 한글 한 음절을 표현한 것이다.

  | 유니코드 범위  | 한글 문자 코드                  | 사이즈 | 사용  | 문자                                                 |
  | -------------- | ------------------------------- | ------ | ----- | ---------------------------------------------------- |
  | U+1100..U+11FF | Hangul Jamo                     | 256    | 256   | 한글                                                 |
  | U+3000..U+303F | CJK Symbols and Punctuation     | 64     | 64    | 한글(15자), 한글(2자), 일반(43자), 상속(4자)         |
  | U+3130..U+318F | Hangul Compatibility Jamo       | 96     | 94    | 한글                                                 |
  | U+3200..U+32FF | Enclosed CJK Letters and Months | 256    | 254   | 한글(62자), 가타카나(47자), 일반(145자)              |
  | U+A960..U+A97F | Hangul Jamo Extended-A          | 32     | 29    | 한글                                                 |
  | U+AC00..U+D7AF | Hangul Syllables                | 11184  | 11172 | 한글                                                 |
  | U+D7B0..U+D7FF | Hangul Jamo Extended-B          | 80     | 72    | 한글                                                 |
  | U+FF00..U+FFEF | Halfwidth and Fullwidth Forms   | 240    | 225   | 한글(52자), 가타카나(55자), 로마자(52자), 일반(66자) |

  - 이 중에서 한글 자모를 표현하는 유니코드는 아래와 같다.
    - `Hangul Jamo`
    - `Hangul Compatibility Jamo`



- 한글의 초성, 중성, 종성 분리하기

  ```python
  # 한글 유니코드 중에서 초성, 중성, 종성의 개수
  NUM_CHO, NUM_JUNG, NUM_JONG = 19, 21, 28
  # 하나의 초성에 중성과 종성을 조합해 만들 수 있는 음절의 개수
  NUM_COMBINATION_PER_CHOSUNG = NUM_JUNG * NUM_JONG
  
  # 한글 유니코드 범위 (가 ~ 힣)
  BASE_CODE_POINT = ord('가')
  LAST_CODE_POINT = ord('힣')
  
  # 한글 유니코드의 초성 리스트
  CHOSUNG = [
      'ㄱ', 'ㄲ', 'ㄴ', 'ㄷ', 'ㄸ', 'ㄹ', 'ㅁ', 'ㅂ', 'ㅃ', 'ㅅ', 'ㅆ', 'ㅇ', 'ㅈ', 'ㅉ', 'ㅊ', 'ㅋ', 'ㅌ', 'ㅍ', 'ㅎ'
  ]
  
  # 한글 유니코드의 중성 리스트
  JUNGSUNG = [
      'ㅏ', 'ㅐ', 'ㅑ', 'ㅒ', 'ㅓ', 'ㅔ', 'ㅕ', 'ㅖ', 'ㅗ', 'ㅘ', 'ㅙ', 'ㅚ', 'ㅛ', 'ㅜ', 'ㅝ', 'ㅞ', 'ㅟ', 'ㅠ', 'ㅡ', 'ㅢ', 'ㅣ'
  ]
  
  # 한글 유니코드의 종성 리스트(종성 없음을 포함)
  JONGSUNG = [
      ' ', 'ㄱ', 'ㄲ', 'ㄳ', 'ㄴ', 'ㄵ', 'ㄶ', 'ㄷ', 'ㄹ', 'ㄺ', 'ㄻ', 'ㄼ', 'ㄽ', 'ㄾ', 'ㄿ', 'ㅀ', 'ㅁ', 'ㅂ', 'ㅄ', 'ㅅ', 'ㅆ', 'ㅇ', 'ㅈ', 'ㅊ', 'ㅋ', 'ㅌ', 'ㅍ', 'ㅎ'
  ]
  
  def is_hangul(character):
      if not BASE_CODE_POINT <= ord(character) <= LAST_CODE_POINT:
          return False
      return True
  
  def decompose_korean(character):
      diff = ord(character) - BASE_CODE_POINT
      cho_idx = diff // (NUM_COMBINATION_PER_CHOSUNG)
      jung_idx = (diff - (cho_idx * NUM_COMBINATION_PER_CHOSUNG)) // NUM_JONG
      jong_idx = (diff - (cho_idx * NUM_COMBINATION_PER_CHOSUNG)) % NUM_JONG
  
      return CHOSUNG[cho_idx], JUNGSUNG[jung_idx], JONGSUNG[jong_idx]
  
  
  korean_text = "깍두기"
  for char in korean_text:
      if is_hangul(char):
          print(f"Character: {char}, Decomposition: {decompose_korean(char)}")
      else:
          print(f"Character {char} is not Hangul")
  ```

  - 사전 지식
    - 한글은 유니코드 상에서 "가"(AC00)부터 "힣"(D7AF)까지 **연속적**으로 위치한다.
    - 유니코드 상에서 한글 초성의 개수는 19개, 중성의 개수는 21개, 종성의 개수는 28개 이다.
    - 따라서 유니코드 상에서 하나의 초성에 모든 중성과 종성을 조합해서 만들 수 있는 음절의 개수는 588(중성의 개수 * 종성의 개수)개이다.
    - 결국 "가"의 위치를 0으로 봤을 때, "ㄱ"으로 시작하는 음절은 0(가)~587(깋)까지 위치하고, "ㄲ"으로 시작하는 음절은 588(까)~1175(낗)까지 위치하는 식으로 배치가 될 것이다.

  - 초성을 찾는 방법
    - 먼저 음절의 유니코드 값에서 한글 유니코드의 첫 음절인 "가"의 유니코드 값을 빼야 한다.
    - 음절의 유니코드에서 한글 유니코드의 첫 음절인 "가"의 유니코드 값을 뺀 값은 "가"를 0으로 봤을 때의 음절의 위치를 의미한다.
    - 예를 들어 "깍"의 유니코드 값에서 "가"의 유니코드 값을 빼서 나오는 589는 "가"를 0으로 봤을 때 "깍"이 589번째에 위치한다는 의미이다.
    - 따라서 이 값을 하나의 초성에 모든 중성과 종성을 조합해서 만들 수 있는 음절의 개수인 588로 나누면 그 몫으로 초성을 알아낼 수 있다.
    - "깍"의 유니코드 값에서 "가"의 유니코드 값을 빼면 589가 나오고, 이를 588로 나눈 몫은 1이다.
    - 한글 유니코드에 있는 초성 중에서 "ㄲ"은 두 번째에 위치한다.
    - 따라서 몫인 1을 index로 사용하여 "깍"의 초성은 "ㄲ"이라는 것을 알 수 있게 된다.
  - 중성을 찾는 방법
    - 찾으려는 음절의 유니코드에서 한글 유니코드의 첫 음절인 "가"의 유니코드 값을 뺀 값에 다시 초성의 index값과 하나의 초성에 모든 중성과 종성을 조합해서 만들 수 있는 음절의 개수를 곱하여 뺀 뒤, 이를 종성의 개수인 28로 나누면 그 몫을 통해 중성을 얻을 수 있다.
    - 여기에 다시 초성의 index값과 하나의 초성에 모든 중성과 종성을 조합해서 만들 수 있는 음절의 개수를 곱하여 뺀 값은 해당 음절의 초성의 첫 번째 값을 0으로 봤을 때의 음절의 위치를 의미한다.
    - `cho_idx * NUM_COMBINATION_PER_CHOSUNG`는 유니코드 상에서 해당하는 초성의 첫 번째 음절의 위치를 찾는 것이다.
    - 예를 들어 유니코드상에서 "깍"의 초성인 "ㄲ"의 첫 번째 값은 "까"이고, "가"를 0으로 봤을 때 588번째에 위치한다.
    - 이 때, "가"를 0으로 봤을 때 "깍"의 위치인 589에서 "가"를 0으로 봤을 때 "까"의 위치인 588을 빼면 "까" 0으로 봤을 때 "깍"의 위치를 구할 수 있다.
    - 그리고 이 값을 종성으로 나누면 그 몫이 중성의 인덱스가 된다.
  - 종성을 찾는 방법
    - 찾으려는 음절의 유니코드에서 한글 유니코드의 첫 음절인 "가"의 유니코드 값을 뺀 값에 다시 초성의 index값과 하나의 초성에 모든 중성과 종성을 조합해서 만들 수 있는 음절의 개수를 곱하여 뺀 뒤, 이를 종성의 개수인 28로 나누면 그 나머지를 통해 종성을 얻을 수 있다.
    - 과정은 중성을 구하는 과정과 동일하며 몫이 아닌 나머지가 종성의 인덱스가 된다는 점만 다르다.







# REST

> https://wonit.tistory.com/454
>
> https://meetup.toast.com/posts/92
>
> https://shoark7.github.io/programming/knowledge/what-is-rest
>
> https://blog.npcode.com/2017/04/03/rest%ec%9d%98-representation%ec%9d%b4%eb%9e%80-%eb%ac%b4%ec%97%87%ec%9d%b8%ea%b0%80/
>
> https://pages.apigee.com/rs/apigee/images/api-design-ebook-2012-03.pdf

- REST(REpresentational State Transfer)

  - HTTP 저자 중 하나인 로이 필딩이 자신의 박사 학위 논문에서 처음 소개했다.
    - HTTP가 설계의 우수성에 비해 제대로 사용되지 못하는 상황이 안타까워 HTTP의 장점을 최대한 활용할 수 있는 아키텍쳐로 REST를 소개했다.
  - Representation
    - 같은 데이터라도 표현(represent)하는 방식이 다를 수 있다.
    - 예를 들어 같은 데이터라도 이를 JSON, XML, HTML 등 다양한 양식으로 표현할 수 있다.
    - 로이 필딩은 자원(resource)과 자원을 표현하는 양식(representation)을 분리했다.
    - 서버는 자원을 전송하는 것이 아니라 자원의 표현을 전송해서 소통하는 것이다.
    - 표현 방식은 서버와 클라이언트 사이의 협상에 의해 결정된다.
    - 서버에서 전송된 자원의 표현은 원래 자원의 표현 방식과 같을 수도 있고 다를 수도 있지만, 이는 인터페이스 뒤에 가려져 알 수 없다.
    - 결국 representation이라는 것은 어떤 리소스의 특정 시점의 상태를 반영하고 있는 정보이다.
    - Representation은 representation data와 representation metadata로 구성된다.
    - representation data는 해당 representation이 표현하려 한 데이터이고, representation metadata는 Content-Type 등과 같은 값 들이다(HTTP header에 있는 값들이 모두 representation metadata에 속하는 것은 아니다).
  - State Transfer
    - State는 웹 애플리케이션의 상태를 의미하며, transfer는 이 상태의 전송을 의미한다.
    - 사용자가 링크를 클릭하면 현재 페이지에서 다른 페이지로의 이동(즉, 상태의 변화)이 발생한다.
    - 이 상태의 변경은 representation의 전송(transfer)를 통해 이루어진다.
    - Transfer는 상태의 전이를 의미하는 것이 아니다. 사용자가 링크를 클릭함으로서 웹 어플리케이션의 상태가 전이된 것은 사실이지만, transfer가 의미하는 것은 network component 사이의 전송을 말한다(일반적으로 서버에서 클라이언트로의 전송).
    - 리소스의 상태와 애플리케이션의 상태는 둘 다 동일하게 state로 표현되었지만, 당연하게도 그 둘은 같은 것을 가리키는 것이 아니다.
  - 결론
    - 위의 내용을 종합하면 REST는 결국 아래와 같은 뜻이다.
    - 하나의 웹 네트워크는 전이 가능한 여러 상태를 갖는데, 이 때 사용자가 링크를 클릭하면 URI로 매핑된 해당 상태로 전이한다. 이 때 그 상태 정보는 표현에 의해 조작(혹은 전처리)되어 전송된다.
  - 주의
    - REST는 URI를 만드는 규칙이 아니다.
    - 웹 프로토콜이나 어플리케이션의 구조화 스타일이다.
    - 실제로 로이 필딩의 논문에서는 URI의 작성 규칙에 대한 내용은 존재하지 않는다.



- REST의 구성 요소
  - 자원(Resource)
    - 모든 자원에 고유한 ID(웹의 경우 URI)가 존재한다.
    - 클라이언트는 ID를 통해 자원을 지정하고 해당 자원의 상태에 대한 조작을 서버에 요청한다.
    - 즉 ID는 자원 그 자체가 아닌 자원에 대한 참조이다.
  - 행위(Verb)
    - HTTP 프로토콜의 method를 사용한다.
    - GET, POST 등
  - 표현(Representation of Resource)
    - 클라이언트가 자원의 상태에 대한 조작을 요청하면, 서버는 이에 적절한 응답(representation)을 보낸다.
    - JSON, XML 등 다양한 형태의 representation이 있을 수 있다.



- REST의 제약 조건
  - 인터페이스 일관성(Uniform interface)
    - 서버의 리소스에 접근할 때 인터페이스가 일관적이어야한다.
    - 여기에 또 4가지 제약 조건이 존재한다.
  - 무상태(Stateless)
    - 클라이언트의 상태가 서버에 저장되어서는 안 된다.
    - 클라이언트에서 서버로 전송되는 정보에는 서버가 요청을 처리하기 위해 필요한 모든 정보가 포함되어 있어야한다.
  - 캐시처리가능(Cacheable)
    - 클라이언트는 응답을 캐싱할 수 있어야한다.
    - 응답 내의 데이터에 해당 요청이 캐시가 가능한지, 불가능한지를 명시해야한다.
    - 캐시가 가능한 요청이라면, 클라이언트에서 동일한 요청이 왔을 때 응답 데이터를 재사용 할 수 있어야한다.
    - `cache-control` 헤더에 캐시 가능 여부를 명시해준다.
  - 계층화(Layered System)
    - 클라이언트는 단순히 REST 서버에 요청을 보내고 응답만 받으면 그만이다.
    - 즉 해당 응답이 최종 서버가 보낸 것인지 중개자가 보낸 것인지 알 필요가 없다.
    - 이를 통해 서버는 중개 서버(게이트웨이, 프록시 등)나 로드 밸런싱 등의 기능을 활용하여 확장 가능한 시스템을 구성할 수 있다.
  - 클라이언트/서버 구조
    - 클라이언트와 서버의 관심사를 명확히 분리해야한다.
    - 서버는 서버의 관심사(데이터 저장소로부터 필요한 자원을 어떻게 하면 잘 관리할지)에 집중하고, 클라이언트는 클라이언트의 관심사(어떤 방식으로 서버에 정보를 요청하고, 어떻게 보여 줄 지)에만 집중한다.
    - 이를 통해 각각 개발해야 하는 부분이 명확해지고 서로 간의 의존성이 줄어들게 된다.
  - Code on demand(optional)
    - 다른 조건들과 달리 선택사항이다.
    - 클라이언트는 서버로부터 실행시킬 수 있는 로직(python script 등)을 받아 클라이언트의 기능을 일시적으로 확장하는데 사용할 수 있다.



- Uniform interface의 4가지 제약 조건

  - 요청 내에서의 자원 식별
    - 요청에서 개별 자원에 대한 식별이 가능해야한다.
    - 웹을 예로 들면 URI에서 자원에 대한 식별이 가능해야한다.
  - 표현을 통한 리소스 조정
    - 서버는 클라이언트에 리소스를 조정(CRUD)하는 요청을 보낼 때, 해당 자원을 지칭하는 메시지와 메타데이터의 표현(representation)을 함께 보내야 한다.
    - 서버는 클라이언트에서 받아온 정보를 바탕으로 리소스를 조정한다.
    - 또한 서버 역시 클라이언트에 조정에 대한 응답을 보낼 때 데이터를 그대로 보내지 않고 JSON, XML등의 표현(representation)으로 변환하여 보낸다.
  - 자기 서술적 메시지
    - 각 메시지는 메시지 자신을 어떻게 처리해야 하는지 충분한 정보를 포함해야 한다.
    - 예시

  ```json
  // 아래 메시지를 응답으로 받은 클라이언트는 아래 정보를 어떻게 처리해야하는지 알 수 없다.
  {
      "id":1,
      "title":"Elasticsearch Guide"
  }
  
  // 아래와 같이 Content-type을 응답 header에 추가함으로써 해당 메시지가 Json 형식이며, 그에 맞는 처리를 해줘야 한다는 것을 알 수 있도록 한다.
  Content-type: application/json
  {
      "id":1,
      "title":"Elasticsearch Guide"
  }
  
  // 다만 id, title이 무엇을 지칭하는지는 여전히 알 수 없다. 따라서 명세를 담은 Link를 응답 header에 추가하여 id와 title이 무엇인지 알 수 있게 해준다.
  Content-type: application/json
  Link: <https://localhost:8000/docs/books>; rel="profile"
  {
      "id":1,
      "title":"Elasticsearch Guide"
  }
  ```

  - Hypermedia As The Engine Of Application State(HATEOAS)
    - Hypermedia를 통해서 애플리케이션의 상태 전이가 가능해야한다.
    - 또한 Hypermedia에 자기 자신에 대한 정보가 담겨야한다.
    - 상태의 전이란 특정 자원을 요청한 후 그와 연계된 요청으로 다시 요청을 보내는 것을 말한다.
    - 예를 들어 특정 게시글의 조회 요청 이후에는 다음 게시글 조회, 해당 게시글에 대한 댓글 쓰기 등의 연계된 요청이 있을 수 있다.
    - 이러한 것들을 응답 메시지에 함께 넣어줘야한다.

  ```json
  {
    "data": {
      "id": 100,
      "name": "article 100",
      "content": "100번째 게시글입니다.",
      "self": "http://localhost:8080/api/article/100", // 현재 api 주소
      "profile": "http://localhost:8080/docs#query-article", // 해당 api의 문서
      "next": "http://localhost:8080/api/article/101", // 다음 article을 조회하는 URI
      "comment": "http://localhost:8080/api/article/100/comment", // article의 댓글 달기
    }
  }
  ```



- HAL(Hypertest Application Lanaguage)

  - JSON, XML 데이터 내에 외부 리소스에 대한 링크를 추가하기 위한 특별 데이터 타입이다.
    - HATEOAS를 보다 쉽게 충족시킬 수 있게 해준다.
    - 일반적으로 데이터를 담기 위해 사용하는 resource 필드와, hypermedia를 담기 위한 link 필드가 존재한다.
  - 아래와 같이 두 가지 데이터 타입을 갖는다.
    - `application/hal+json`
    - `application/hal+xml`

  - 예시

  ```json
  {
      "data": {
          "id": 100,
          "name": "article 100",
      	"content": "100번째 게시글입니다.",
      },
      "_links": {
          "self": {
              "href": "http://localhost:8080/api/article/100"
          },
          "profile": {
              "href": "http://localhost:8080/docs#query-article"
          },
          "next": {
              "href": "http://localhost:8080/api/article/101"
          },
          "comment":{
              "href": "http://localhost:8080/api/article/100/comment"
          }
      }
  }
  ```





## REST API

- REST API
  - REST의 제약조건을 만족하는 API를 구현한 것을 REST API라 부른다.
  - HTTP API
    - HTTP를 사용해서 서로 정해둔 스펙으로 데이터를 주고 받는 것.
    - REST API는 HTTP API에 보다 엄격한 제약 조건을 추가한 것이다.
    - 대부분의 경우 모든 제약 조건을 만족하는 REST API는 흔치 않다.
    - 따라서 엄밀히 따지면 REST API가 아닌 HTTP API일 때에도 자원, 행위, 표현이 포함되어 있으면 REST API라고 부르는 경우가 많다.



- REST API 설계의 기본 규칙

  - 용어
    - document: 객체의 인스턴스나 DB 레코드와 유사한 개념
    - collection: 서버에서 관리하는 디렉터리.
    - store: 클라이언트에서 관리하는 resource 저장소

  ```bash
  # sports와 players는 collection, soccer는 document이다.
  http:// restapi.example.com/sports/soccer/players/13
  ```

  - URI는 resource를 표현해야한다.
    - 동사를 사용하지 않는다. 
    - 대문자보다는 소문자를 사용한다.
    - Document 이름으로는 단수명사를 사용해야한다.
    - Collection과  store 이름으로는 복수명사를 사용해야 한다.
    - 추상적인 resource name 보다는 구체적인 resource name이 낫다.
  - 자원에 대한 행위는 HTTP method로 표현한다.
    - URI에 HTTP method가 들어가면 안 된다(e.g. `GET /members/get/1` X).
    - URI에 행위에 대한 동사 표현이 들어가면 안 된다(e.g. `GET /members/find/1` X).
    - 경로 부분 중 변하는 부분은 고유한 값이어야한다(e.g. `DELETE /members/1`, 에서 변하는 부분인 1은 해당 자원을 나타내는 고유한 값이어야 한다).
  - 특수 문자 관련 규칙
    - `/`는 계층 관계를 나타내는데 사용한다.
    - URI의 마지막 문자로 `/`를 사용하지 않는다.
    - `-`은 가독성을 높이는데 사용한다.
    - `_`은 사용하지 않는다.
    - 파일의 확장자는 포함하지 않는다.



- 명사가 아닌 동사를 쓰는 것이 허용되는 경우들

  - 도메인에 따라 API를 통해 서버에 resource가 아닌 response를 요청해야하는 경우가 있을 수 있다.
    - 예를 들어 특정 값을 계산해서 해당 결과를 반환 받는 다던가, 문자를 다른 언어로 번역하는 경우 등이 있다.
    - 계산 결과와 번역 결과 모두 그 자체로 resource는 아니지만, 클라이언트가 서버에 반환을 요청한 값이다.
  - 이럴 경우 예외적으로 동사를 사용하는 것이 허용된다.
    - 예를 들어 아래 요청은 유로화를 한화로 변경하는 요청이다.
    - 유로화와 한화 모두 서버가 가지고 있는 resource는 아니며, 서버는 클라이언트로부터 요청을 받으면 정해진 로직에 따라 환율을 계산하여 그 결과를 반환하면 된다.

  ```http
  GET /convert?from=EUR&to=KRW&amount=100
  ```

  - 또 다른 경우는 검색이 필요할 경우이다.
    - 일반적으로 단순한 검색은 resource 뒤에 query parameter를 사용하여 아래와 같이 요청한다.

  ```http
  GET dogs/?q=red
  ```

  - 그러나, 특정 resource에 대한 검색이 아닌, 모든 resource에 대해 검색을 해야 하는 경우, resource명이 아닌 동사 `search`를 사용한다.

  ``` http
  GET /search?q=fluffy+fur
  ```

  - 단, 이 경우 동사를 사용한 이유를 API 문서에 명시해야한다.



- Response의 attribute name

  - 서버 개발자는 response로 반환될 attribute name늘 지정해야한다.
    - 예를 들어 아래 예시에서는 `created_dt`라는 attribute name을 지정했다.

  ```json
  {
    "created_dt": "Thu Nov 03 05:19;38 +0000 2011"
  }
  ```

  - 이 때 attribute name의 형식을 어떤 형식으로 할지가 문제가 될 수 있다.
    - 같은 값을 반환하더라도 아래와 같이 다양한 형식으로 attribute name을 지정할 수 있기 때문이다.

  ```json
  {
    "created_dt": "Thu Nov 03 05:19;38 +0000 2011",
    "CreatedAt": "Thu Nov 03 05:19;38 +0000 2011",
    "createdAt": "Thu Nov 03 05:19;38 +0000 2011"
  }
  ```

  - Attribute name의 형식은 camelCase를 따르는 것이 권장된다.
    - 팀 내에서 정한 규칙이 있다면, 해당 규칙을 따르는 것이 가장 좋다.
    - 만일 정해진 규칙이 없거나 규칙을 정해야 하는 상황이라면, camelCase를 사용하는 것이 권장된다.
    - 이는 response를 받아서 처리하는 언어인 JavaScript가 camelCase를 사용하기 때문이다.

  ```javascript
  // JavaScript에서 response를 받아서 처리할 때는, 아래와 같은 attribute name 형식 보다는
  var timing = myObject.created_at;
  var timing - myObject.CreatedAt;
  
  // 아래와 같은 camelCase가 더 자연스럽다.
  var timing - myObject.createdAt;
  ```



- 예외 처리

  - 클라이언트의 요청을 처리하는 중에 에러가 발생한 경우 서버는 아래와 같은 정보를 클라이언트에 반환해야한다.
    - 에러를 표현할 수 있는 적절한 HTTP status code
    - 에러가 발생한 원인
    - 에러를 해결할 수 있는 방법(관련된 문서의 링크나 간략한 설명 등)

  - 예시
    - 첫 번째는 에러가 발생했음에도 정상 처리 되었다는 200 status code를 반환하고, 두 번째는 해당 error를 어떻게 처리해야하는지에 대한 설명이 누락되어 있다.
    - 세 번째는 올바른 HTTP status code를 반환하고, error의 원인과 해결 방법에 대해 알 수 있도록 링크를 제공한다.

  ```json
  // HTTP Status Code: 200
  {
    "type" : "OauthException", 
    "message":"(#803) Some of the aliases you requested do not exist: foo.bar"
  }
  
  // HTTP Status Code: 401
  {
    "code" : 401, 
    "message": "Authentication Required"
  }
  
  // HTTP Status Code: 401
  {
    "status" : "401", 
    "message":"Authenticate",
    "code": 20003, 
    "moreinfo": "http://www.twilio.com/docs/errors/20003"
  }
  ```



- Version 정보 표기하기

  - Version은 API에서 매우 중요한 요소 중 하나이다.
  - Version은 다양한 방식으로 표현할 수 있다.
    - 첫 번째와 같이 날짜를 사용할 수도 있고
    - 두 번째와 같이 중간에 넣을 수도 있고
    - 세 번째와 같이 query parameter로 받을 수도 있다.

  ```http
  /2010-04-01/Accounts/
  /services/data/v20.0/sobjects/Account
  ?v=1.0
  ```

  - 가장 바람직한 방식은 URL의 가장 앞쪽에 `v` prefix를 사용하여 표현하되, 숫자에 소숫점 이하는 표기하지 않는 것이다.
    - 소수를 버리는 이유는 인터페이스를 단순하게 유지하기 위함이다.

  ```http
  /v1/dogs
  ```

  - HTTP header에 버전 정보를 넣는 것이 더 바람직하지 않은가?
    - 이론적으로는 API의 버전 정보를 header에 넣는 것이 맞다.
    - 그러나 대부분의 기업들 이러한 방식을 채택하지 않는데, 일반적으로 header에는 한 번 정해지면 잘 변경되지 않는 값들을 넣기 때문이다.
    - Version과 같이 지속적으로 변경되는 정보를 header에 넣을 경우 URL에 넣는 것에 비해 변경된 정보를 식별하는 것이 쉽지 않다.

