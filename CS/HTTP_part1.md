# HTTP를 왜 학습해야 하는가

- 모든 것은 HTTP 기반으로 동작한다.
  - 클라이언트와 서버의 통신, 서버와 서버의 통신은 모두 HTTP 기반으로 동작한다.
  - 그러므로 모바일 앱 개발자, 웹 개발자라면 모두 HTTP에 대한 정확한 지식을 지녀야 한다.
    - 그렇다고 HTTP 명세의 모든 것을 알 필요는 없다.
    - 그러나 필요한 부분에 대한 정확한 이해는 반드시 필요하다.



# 인터넷 네트워크

- 클라이언트와 서버는 인터넷을 사이에 두고 통신한다.
  - 인터넷을 사이에 두고 서버와 인터넷이 통신을 하는 방식
    - IP 프로토콜(IP Protocol, Internet Protocol)
    - TCP, UDP



## IP(Internet Protocol)

- 역할
  - 지정한 IP 주소(IP Address)에 데이터 전달
  - 패킷(Packet,Package+Bucket의 합성어)이라는 통신 단위로 데이터 전달



- IP 프로토콜의 동작 방식
  - A(클라이언트)에서 B(서버)로 데이터를 전달한다고 가정
  - A에서 출발지 IP 주소(A의 IP 주소), 목적지 IP 주소(B의 IP 주소), 전송하고자 하는 데이터 등의 정보를 가지고 IP 패킷을 만든다.
  - 생성한 패킷을 인터넷으로 전달한다.
  - 인터넷은 A에서 전달 받은 패킷을 인터넷 노드들을 통해 B까지 전달한다.
  - B에서 전달 받은 후 다시 출발지 IP 주소(B의 IP 주소), 목적지 IP 주소(A의 IP 주소), 전송하고자 하는 데이터 등의 정보를 가지고 IP 패킷을 만든다.
  - 생성한 패킷을 인터넷으로 전달한다.
  - 인터넷은 B에서 전달 받은 패킷을 인터넷 노드들을 통해 A까지 전달한다.



- IP 프로토콜의 한계
  - 비연결성
    - 패킷을 받을 대상이 없거나 서비스 불능 상태(e.g. 패킷을 받을 컴퓨터가 꺼져 있는 경우)여도 패킷 전송
    - 즉, 대상 서버가 패킷을 받을 수 있는 상태인지 모른다.
  - 비신뢰성
    - 중간에 패킷이 사라질 수 있다. 
    - 인터넷 내부의 노드들을 통해 패킷이 전달되는데, 중간에 노드에 이상이 생기면 서버까지의 연결이 끊어지게 되어 패킷이 유실될 수 있다.
    - 패킷이 순서대로 오지 않을 수 있다.
    - 대량의 데이터를 보낼 경우 보통 1,500 byte 단위로 끊어서 보낸다. 2,500 byte의 데이터를 1,500, 1,000 bytr로 끊어서 1,500 byte의 데이터를 먼저 보내고, 1,000 byte의 데이터를 나중에 보낸다 하더라도 인터넷 내부에서 어떤 노드를 거치는가에 따라서 1,000 byte 데이터가 먼저 도착할 수 있다.
  - 프로그램 구분
    - 같은 IP를 사용하는 서버에서 통신하는 애플리케이션이 둘 이상일 수 있다.
    - 예를 들어 내가 사용중인 하나의 IP에서 스트리밍을 통해 음악을 들으면서 웹툰을 볼 수 있다. 이 때 내 IP로 스트리밍 사이트의 패킷과 웹툰 사이트의 패킷이 모두 오게 되는데 이들이 어디서 온 패킷인지 구분할 수 없다.
  - 이와 같은 한계를 극복하기 위해 TCP를 사용한다.



- DNS(Domain Name System)
  - IP의 문제점
    - IP 주소는 100.100.100.1과 같은 형태이기에 기억하기 어렵다는 문제가 있다.
    - 또한 IP 주소는 변경이 가능하므로 IP 주소만 가지고 통신을 하는 것은 한계가 있다.
  - DNS는 도메인 명을 IP 주소로 변환해준다.
    - 서버는 DNS 서버에 도메인 명과 IP 주소를 등록한다.
    - 이후 클라이언트에서 도메인 명으로 접근하면 DNS 서버는 도메인 주소에 해당하는 IP 주소를 반환한다.
    - 이후 클라이언트는 DNS 서버에서 반환 받은 IP 주소로 서버에 접속한다.



## TCP, UDP

- 인터넷 프로토콜 스택의 4계층
  - 애플리케이션 계층(HTTP, FTP)
    - 웹 브라우저, 온라인 게임 등 인터넷을 사용하는 애플리케이션
    - 카카오톡을 통해 친구에게 "안녕!" 이라는 메세지를 보낸다.
    - SOCKET 라이브러리를 통해  OS 계층에 이 메시지를 전달한다.
  - 전송 계층(TCP,UDP)
    - 전송 계층과 인터넷 계층은 OS 계층에 속한다.
    - OS 계층의 TCP는 애플리케이션 계층에서 받은 메세지에 TCP 세그먼트를 씌운다.
    - TCP 세그먼트는 출발지 PORT, 목적지 PORT, 전송 제어, 순서, 검증 정보 등이 포함되어 있다.
  - 인터넷 계층(IP)
    - OS 계층의 IP는 TCP 정보가 씌어진 메세지를 IP 패킷으로 만든다.
    - 결국 패킷에는 IP 정보, TCP, 정보, 메세지 정보가 담기게 된다.
  - 네트워크 인터페이스 계층
    - IP 패킷에 Ethernet frame(상세하게 알 필요는 없다)을 씌운다.
    - LAN 카드를 통해서 Ethernet frame을 씌운 IP 패킷을 인터넷으로 전송한다.



- TCP(전송 제어 프로토콜, Transmission Control Protocol)의 특징
  - 연결 지향
    - TCP 3 way handshake(가상 연결)
    - 우선 연결을 하고 메세지를 보낸다.
    - IP 프로토콜의 한계인 비연결성을 보완
  - 데이터 전달 보증
    - 데이터가 성공적으로 전송 되면 데이터를 받은 서버는 데이터를 잘 받았다는 메세지를 클라이언트에 전달한다.
    - 따라서 패킷이 중간에 누락이 될 경우 알 수 있다.
    - IP 프로토콜의 한계인 비신뢰성을 보완
  - 순서 보장
    - TCP 세그먼트에는 순서에 대한 정보가 담겨 있다.
    - 패킷을 A, B, C 순서로 전송했을 때 A, C, B 순서로 도착했다면 서버는 B 부터 다시 보내라는 메세지를 클라이언트에 전송한다.
    - IP 프로토콜의 한계인 비신뢰성을 보완



- TCP 3 way handshake
  - 클라이언트에서 서버로 SYN(synchronize)이라는 접속 요청 메세지를 보낸다.
  - 서버는 SYN 메세지를 성공적으로 받으면 ACK(Acknowledge)이라는 요청 수락 메세지와 SYN(서버 쪽의 SYN)을 다시 클라이언트로 보낸다.
  - 클라이언트는  ACK+SYN(서버 쪽의 SYN)를 받으면 다시 서버로 ACK를 보낸다.
    - 이때 클라이언트의 ACK에 데이터를 함께 전송 가능하다.
  - 그 후 클라이언트는 데이터를 전송한다.



- UDP(사용자 데이터그램 프로토콜, User Datagram Protocol) 특징
  - 특징이 없는 것이 특징으로 하얀 도화지에 비유된다.
    - 기능이 거의 존재하지 않는다.
    - TCP 3 way handshake도 존재하지 않고, 데이터 전달을 보증하지도 않으면 순서를 보장하지도 않는다.
  - IP와 거의 유사하지만 PORT 정보와 체크섬 정보가 존재한다는 차이가 있다.
    - PORT 정보는 TCP에도 존재한다.
    - PORT는 하나의 IP를 여러 애플리케이션이 사용할 경우 다양한 곳에서 전송 받은 패킷이 어디서 온 것인지 구분하게 해준다.
    - 체크섬은 메세지가 제대로 왔는지 검증해준다.
  - 기능이 많이 없음에도 쓰이는 이유는 기능이 없기에 단순하고 빠르기 때문이다.
    - TCP는 다양한 기능을 지니고 있고 복잡한 과정을 거치기에 커스터마이징해서 사용하는 것이 불가능하다.
    - UDP는 단순한 기능을 지니고 있기에 필요한 기능을 추가하여 사용하는 것이 가능하다.
    - 최근에 부상하고 있다.



- PORT
  - 한 번에 둘 이상을 연결해야 하는 경우
    - 단일 IP를 가진 A라는 클라이언트가 B, C라는 각기 다른 서버와 연결중인 경우.
    - B 서버에는 게임 서버 연결, 게임 내의 보이스 채팅 관련 패킷을 받고, C 서버에서는 웹툰 관련 패킷을 받고 있는 경우 어떤 패킷이 어디서 온 것인지 어떻게 확인 할 것인가.
    - 특히 B 서버의 경우 같은 IP에서 패킷이 전송되고 있으므로 더 구분이 어렵다.
  - PORT는 각 패킷 별로 지정된다.
    - 즉, 같은 서버에서 전송된 패킷이라도 각각의 PORT 주소는 다르므로 이것을 통해 구분이 가능하다.
    - IP 가 아파트라면 PORT가 아파트의 동, 호수라고 할 수 있다.
  - PORT 번호
    - 0~65535까지 할당이 가능하다.
    - 0~1023까지는 잘 알려진 포트로, 사용하지 않는 것이 좋다.



# URI와 웹 브라우저 요청 흐름

- URI(Uniform Resource Identifier)
  - 정의
    - U(Uniform): 리소스를 식별하는 통일된 방식.
    - R(resource): 자원, URI로 식별할 수 있는 모든 것.
    - I(Identifier): 다른 항목과 구분하는데 필요한 정보.
    - 주민을 식별할 때 주민등록번호를 사용하듯이, 자원을 식별할 때 사용하는 방법이다.
  - URI, URL(Uniform Resource Location), URN((Uniform Resource Name))의 차이
    - URI는 로케이터(**L**ocator), 이름(**N**ame) 또는 둘 다 추가로 분류될 수 있다.
    - 즉, URL, URN은 URI에 포함되는 개념이다.
  - URL
    - 자원의 위치를 나타낸다. 
    - 위치는 변경 가능하다.
  - URN
    - 자원의 이름을 나타낸다. 
    - 이름은 변경 불가능하다. 
    - URN 이름만으로는 실제 리소스를 찾을 수 있는 방법이 보편화되지 않았다.
  - 아래 내용부터는 URL을 URI와 같은 의미로 사용할 것이다.



- URL 문법

  - `schema://[userinfo@]host[:port][/path][?query][#fragment]`([ ]는 생략 가능)
    - https://www.google.com/search?q=hello&hl=ko라는 주소를 기준으로 보면 다음과 같다.
    - schema: https
    - host: www.google.com
    - path: /search
    - query: q=hello&hi=ko
  - schema
    - 주로 프로토콜(어떤 방식으로 자원에 접근할 것인가 하는 약속 규칙)을 사용한다.
    - http, https, ftp 등이 프로토콜이다
    - HTTP는 하이퍼 텍스트 전송 프로토콜의(Hypertext Transfer Protocol)의 약자로, **서로 다른 시스템들 사이에서 통신을 주고받게 해주는 가장 기초적인 프로토콜**이며, 서버에서 브라우저로 데이터를 전송해 주는 용도로 가장 많이 사용된다.
    - http는 80포트, https는 443포트를 주로 사용하며, 포트는 생략 가능하다.
    - https는 http에 보안이 추가된 것이다(HTTP Secure).
    - HTTP 프로토콜의 문제점은 서버에서부터 브라우저로 전송되는 정보가 암호화되지 않는다는 것이었다.
    - HTTPS 프로토콜은 **SSL(보안 소켓 계층)**을 사용함으로써 이 문제를 해결했다. 
    - SSL은 서버와 브라우저 사이에 안전하게 암호화된 연결을 만들 수 있게 도와주고, 서버 브라우저가 민감한 정보를 주고받을 때 이것이 도난당하는 것을 막아준다.
    - SSL 인증서는 사용자가 사이트에 제공하는 정보를 암호화하는데, 쉽게 말해서 데이터를 암호로 바꾼다고 생각하면 쉽다. 
    - 이렇게 전송된 데이터는 중간에서 누군가 훔쳐 낸다고 하더라도 데이터가 암호화되어있기 때문에 해독할 수 없다. 
    - 그 외에도 HTTPS는 **TLS(전송 계층 보안)** 프로토콜을 통해서도 보안을 유지한다. 
    - TSL은 데이터 무결성을 제공하기 때문에 데이터가 전송 중에 수정되거나 손상되는 것을 방지하고, 사용자가 자신이 의도하는 웹사이트와 통신하고 있음을 입증하는 인증 기능도 제공한다.
    - 또한 보안 뿐 아니라 **검색엔진 최적화(SEO)**에 도움을 주며 **가속화된 모바일 페이지(AMP, Accelerated Mobile Pages)**를 만들고 싶다면 HTTPS 프로토콜을 사용해만 한다.

  - userinfo
    - URL에 사용자 정보를 포함해서 인증해야 할 때 사용한다.
    - 거의 사용하지 않는다.
  - host
    - 호스트명
    - 도메인명 또는 IP 주소를 사용 가능하다.
  - port
    - 접속 포트
    - 일반적으로 생략하며, 생략시 http는 80, https는 443 포트로 자동으로 접속된다.
  - path
    - 리소스 경로
    - 계층적 구조로 되어있다.
    - 예를 들어 5번째 회원 정보를 보고자 한다면  `info/members/5`와 같이 URL을 입력하게 된다(info 계층의 하위 계층인 member 계층의 하위에 있는 5번째 멤버).
  - query
    - key=value 형태
    - ?로 시작하며, &로 추가가 가능하다.
    - query parameter, query string 등으로 불린다.
  - fragment
    - html 내부 북마크 등에서 사용.
    - 서버에 전송하는 정보는 아니다.
    - #으로 시작한다.



- 웹 브라우저 요청 흐름
  - https://www.google.com/search?q=hello&hl=ko와 같은 주소를 입력하면 아래와 같은 과정을 거친다.
  - 요청 보내기
    - DNS 서버를 조회하여 IP 주소를 찾아낸다.
    - 웹 브라우저는 HTTP 요청 메시지를 생성한다.
    - 3 way handshake를 통해 서버가 요청을 받을 수 있는 상태인지 확인한다.
    - 서버가 요청을 받을 수 있는 상태라면, 소켓 라이브러리를 통해 TCP/IP 계층(os 계층)으로 전달한다.
    - TCP/IP 계층에서는 HTTP 메시지에 TCP/IP 패킷을 씌운 요청 패킷을 생성한다.
    - IP와 포트번호(포트번호는 URL에 나와있다. 위 주소에는 생략되어 있으나 https 프로토콜이므로 443 포트를 사용한다)를 가지고 있으므로 이에 해당하는 곳(서버, 예시의 경우 구글 서버)으로 요청 패킷을 전송한다.
    - 인터넷 내부의 노드들을 탐색하며 서버를 찾는다.
  - 응답 받기
    - 서버에서 요청 패킷을 전달 받은 후에는 HTTP 요청을 해석한다.
    - 해석 후 클라이언트에서 요청한 데이터가 담긴 응답 메시지를  생성한다.
    - 이후 응답 메시지에 TCP/IP 패킷을 씌운 응답 패킷을 생성하고 클라이언트로 전송한다.
    - 클라이언트에서는 응답 패킷을 전달 받은 후 거기에 담긴 응답 메세지를 해석한다.
    - 이후 렌더링하여 화면에 나타나게 된다.





# HTTP 기본

- 모든 것을 HTTP 프로토콜에 담아서 전송한다.
  - HTML, TEXT, IMAGE, 음성, 영상, 파일, JSON, XML 등
  - 거의 모든 형태의 데이터를 전송 가능하다.
  - 서버 간에 데이터를 주고 받을 때도 대부분 HTTP를 사용한다.



- HTTP의 역사
  - HTTP/0.9(1991년): GET 메서드만 지원, HTTP 헤더가 존재하지 않았다.
  - HTTP/1.0(1996년): 메서드, 헤더 추가
  - HTTP/1.1(1997년): 가장 많이 사용하는 버전(1999, 2014년에 개정이 이루어짐).
  - HTTP/2(2015년): 성능 개선
  - HTTP/3(진행중): TCP 대신 UDP 사용, 성능 개선



- 기반 프로토콜
  - TCP: HTTP/1.1, HTTP/2
  - UDP: HTTP/3



- 주요 용어 정리
  - Resources
    - HTTP request의 타겟을 리소스라 부른다.
    - 즉 HTTP request를 통해 추가, 수정, 삭제 등을 수행하고자 하는 데이터를 리소스라 부른다.
  - Representations
    - 주어진 resource의 과거, 현재, 혹은 원하는 상태를 반영하기 위한 정보이다.
    - Resource는 문서, 사진 hypertext, json 등 다양한 형태로 존재할 수 있는데, 이들을 어떤 방식으로 보여줄지와 관련이 있다.



## HTTP의 특징

- 클라이언트 서버 구조
  - Request, Response 구조
  - 클라이언트는 서버에 요청을 보내고, 응답을 대기한다.
  - 서버가 요청에 대한 결과를 만들어서 응답한다.



- 무상태 프로토콜(stateless)
  - 서버가 클라이언트의 상태를 보존하지 않는다.
  - stateful할 경우의 예시
    - 고객(클라이언트): 사과 하나에 얼만가요?
    - 점원(서버): 500원 입니다.
    - 고객:10개 구매하겠습니다.
    - 점원: 사과 10개는 5000원 입니다. 카드, 현금 중 어떤 걸로 구매하시겠어요?
    - 고객: 신용카드로 구매하겠습니다.
    - 점원: 5000원 결제 완료되었습니다(사과 10개, 신용카드 결제라는 상태 유지)
  - stateful인데 서버가 변경될 경우의 예시(서버가 변경되면 요청이 불가능하다)
    - 고객: 사과 하나에 얼만가요?
    - 점원A: 500원 입니다.
    - 고객:사과 10개 구매하겠습니다.
    - 점원B: 무엇을 2개 구매하시겠어요?
    - 고객: 신용카드로 구매하겠습니다.
    - 점원C: 무슨 제품 몇 개를 신용카드로 구매하시겠어요?
  - stateless일 경우 예시
    - 고객(클라이언트): 사과 하나에 얼만가요?
    - 점원(서버): 500원 입니다.
    - 고객:사과 10개 구매하겠습니다.
    - 점원: 사과 10개는 5000원 입니다. 카드, 현금 중 어떤 걸로 구매하시겠어요?
    - 고객: 사과 10개를 신용카드로 구매하겠습니다.
    - 점원: 5000원 결제 완료되었습니다(사과 10개, 신용카드 결제라는 상태 유지)
  - stateless인데 서버가 변경될 경우의 예시(서버가 변경되도 정상적으로 요청이 가능하다)
    - 고객: 사과 하나에 얼만가요?
    - 점원A: 500원 입니다.
    - 고객:사과 10개 구매하겠습니다.
    - 점원B: 사과 10개는 5000원 입니다. 카드, 현금 중 어떤 걸로 구매하시겠어요?
    - 고객: 사과 10개를 신용카드로 구매하겠습니다.
    - 점원C: 5000원 결제 완료되었습니다(사과 10개, 신용카드 결제라는 상태 유지)
  - Stateful, Stateless 차이
    - stateful: 중간에 서버가 변경되면 안되며, 만일 변경된다면 기존 상태 정보를 서버에 미리 알려줘야 한다.
    - stateless: 중간에 서버가 변경되도 되므로 무한한 서버 증설이 가능하다.
    - 예를 들어 클라이언트가 있고, 서버 A, B가 있을 때, stateful일 경우 클라이언트는 A 서버에 사과라는 제품명과 10개라는 수량 정보를 보내는 단계까지 진행했다.
    - 이 때, 서버A에 장애가 발생한다면, 이전에 클라이언트가 보냈던 제품명, 수량 정보는 서버A에만 저장되어 있으므로, 클라이언트는 서버 B에서 제품명, 수량 정보를 보내는 단계를 다시 거쳐야 한다.
    - 그러나 stateless일 경우 결재를 처음부터 진행할 필요가 없다.
  - stateless는 다음과 같은 장단점이 존재한다.
    - 서버 확장성이 높다는 장점이 있다(스케일 아웃, 서버의 수평 확장).
    - 클라이언트가 서버에 추가 데이터를 전송해야 한다는 단점이 있다.
  - stateless의 한계
    - 모든 것을 무상태로 설계할 수 없는 경우도 존재한다.
    - 로그인 등의 정보는 상태를 유지해야 한다.
    - 이를 위해 브라우저 쿠키나 서버의 세션 등을 사용해서 상태를 유지한다.
    - 상태 유지는 최소한만 사용해야 한다.



- 비연결성
  - 연결을 유지하는 모델
    - 클라이언트 A, B, C가 있고 서버가 있다.
    - 클라이언트 A, B, C가 각각 서버에 요청을 보내고 응답을 받는다.
    - 이렇게 하면 클라이언트 A, B, C와 서버는 연결이 유지된 상태다.
    - 서버는 요청과 응답이 이루어지지 않는 클라이언트와도 계속 연결이 되어있으므로 서버 자원이 소모되게 된다.
  - 연결을 유지하지 않는 모델
    - 클라이언트 A, B, C가 각각 서버에 요청을 보내고 응답을 받는다.
    - 서버는 응답을 보낸 후 클라이언트와의 연결을 종료한다.
    - 즉, 요청할 때 연결하고, 응답을 보낸 후 연결을 끊는다.
    - 서버는 클라이언트와 연결을 유지하지 않으므로 최소한의 자원만 사용 가능하다.
  - 비연결성을 지닌 이유
    - 일반적으로 초 단위 이하의 빠른 속도로 응답이 이루어진다.
    - 1시간 동안 수천명이 서비스를 이용해도 실제 서버에서 동시에 처리하는 요청은 수십개 이하로 매우 적다.
  - 한계
    - TCP/IP 연결을 요청이 올 때 마다 새로 맺어야 한다(3 way handshake에 소모되는 시간이 추가 된다).
    - 웹 브라우저로 사이트를 요청하면 HTML 뿐만 아니라 JS, CSS, 이미지 등 수 많은 자원이 함께 응답이 온다. 따라서 매번 자원을 요청할 때마다 새로 연결해야 하는 것은 비효율적이다.
  - HTTP 지속 연결(Persistence Connections)
    - 위와 같은 문제점을 해결하기 위해 HTTP 지속 연결을 사용한다.
    - 일정 기간 동안 추가 요청이 없으면 연결을 끊는 방식이다.
    - HTTP/2, HTTP/3 에서 최적화가 이루어졌다. 



- HTTP 메시지
  - HTTP 메시지에는 거의 모든 형태의 데이터를 전송 가능하다.
  - 시작 라인
    - request-line(요청 메시지의 경우): `method SP(공백) request-target SP HTTP-version CRLF(엔터)`
    - status-line(응답 메시지의 경우): `HTTP-version SP status-code SP reason-phrase CRLF`
  - header-field
    - HTTP 전송에 필요한 모든 부가 정보가 담겨 있다.
    - 많은 종류의 표준 헤더가 있으며, 필요시 임의의 헤더를 추가하는 것이 가능하다.
    - `field-name:OWS(띄어쓰기 허용) filed-value OWS`
    - OWS 위치는 띄어써도 되고 안 띄어써도 된다.
    - filed-name은 대소문자를 구분하지 않으나 field-value는 대소분자를 구분한다.
    - e.g `Host: www.google.com`, `Content-Type:text/html;charset=UTF-8` 등
  - 공백 라인(CRLF)
    - 엔터를 한 번 친다고 생각하면 된다.
    - 이름 그대로 아무 내용도 들어가지 않지만 절대 생략해선 안된다.
  - message body
    - 실제 전송할 데이터가 담긴다.
    - HTML 문서, 이미지, 영상, JSON 등 byte로 표현할 수 있는 모든 데이터를 전송 가능하다.



- 단순하며 확장 가능하다.
  - 크게 성공하는 표준 기술은 대개 단순하지만 확장 가능한 기술이다.
  - HTTP는 단순하다.
    - 스펙도 읽어볼만하다.
  - HTTP 메시지도 매우 단순하다.





## HTTP Request Method

- HTTP 요청 메서드
  - 주어진 리소스에 수행하길 원하는 행동을 나타낸다.
  - HTTP 동사라고 부르기도 한다.



- 메서드의 종류

  - GET
    - 목표 resource에 대한 representation의 전송을 요청한다.
    - 오직 데이터를 받기만 하며 응답에 body는 포함되지 않는다(쿼리 스트링, 쿼리 파라미터 등으로 표시하고자 하는 데이터에 대한 정보를 표시하긴 하지만 이는 요청하는 데이터의 경로를 표시한 것이라 봐야한다).
  - HEAD
    - Response가 content를 담고있지 않다는 것을 제외하면 GET과 동일하다.
    - 특정 representation에 대한 metadata를 얻기 위해 사용한다.
    - GET으로 요청했을 때와 동일한 header를 전송해야 한다.
    - 그러나 일부 예외도 존재하는데 `Content-Length`나 ` Vary` 등의 header field는 생략할 수도 있다.
    - HEAD 메서드는 content를 전달받지 않기에 `Content-Length` 등의 header field를 필요로하지 않기 때문이다.
  - POST
    - 목표 resource가 request와 함께 전송된 representation을 처리하도록 요청한다.
    - 새로운 resource를 생성하거나 이미 존재하는 resource의 representation에 data를 추가하기 위해 사용한다.
    - 만약 POST 요청을 통해 서버에 하나 이상의 resource가 생성되었다면, 서버는 `201(Created)`를 응답으로 보내야하며, `Location` header field를 포함시켜야한다.
    - 요청을 보낼 때 body의 type은 `Content-Type` 헤더에 명시해서 보낸다.
    - 만일 요청에 포함된 representation의 처리 결과가 이미 존재하는 resource의 representation과 동일하다면, 서버는 `303(See Other)` 응답 코드와 `Location` header filed에 이미 존재하는 resource의 identifier를 포함시켜 redirect 시킬 수도 있다.
  - PUT
    - 목표 resource의 상태가 request와 함께 전송된 representation에 정의된 상태로 생성되거나 변경되도록 요청한다.
    - 만약 목표 resource가 현재 representation이 없을 경우 PUT 메서드를 통해 성공적으로 representation을 생성하면, 서버는 반드시 `201(Created)`를 응답으로 보내야한다.
    - 만약 목표 resource가 현재 representation을 가지고 있고, 해당 representation이 요청에 포함된 representation의 상태와 일치하도록 변경되면, 서버는 반드시 `200(OK)` 혹은 `204(No Content)`를 응답으로 보내야한다.
    - 서버는 request와 함께 전송된 representation이 목표 resource의 제약조건과 일치하는지 확인해야한다.
    - 예를 들어 목표 resource의 `Content-Type`이 `text/html`이고 요청에 포함 된 representation이 `image/jpeg`라면, 서버는 목표 resource의 제약조건을 수정하거나, 요청으로 온 representation을 resource의 제약 조건에 맞게 변환하거나, `415(Unsupported Media Type)` 상태 코드를 응답으로 보내 요청을 거부해야한다.

  - DELETE
    - 목표 resource와 그것의 현재 기능 사이의 관계를 제거하도록 요청한다.
    - 단순하게 말하면, 목표 resource의 삭제를 요청한다.
    - 목표 resource의 하나 이상의 현재 representation이 삭제될 수도, 삭제되지 않을 수도 있고, 관련된 저장소 역시 수정되거나 수정되지 않을 수도 있따다.
    - 이는 순전히 서버의 구현과 resource의 특성에 달려있다.
    - 만약 DELETE 메서드가 성공적으로 적용되었고, 삭제가 성공적으로 수행될 것 같다면 `202 (Accepted)`, 삭제가 성공적으로 수행되었다면 `204 (No Content)`, 삭제가 성공적으로 수행되었으면서 상태를 묘사하기 위한 응답 메시지를 보내야 한다면 `200 (OK)`를 보낸다.
  - CONNECT
    - 목료 리소스에 대해 양방향 연결을 시작하는 메서드이다.
    - CONNECT 메서드는 host와 port 번호로만 구성된 특수한 형식의 request target을 사용한다.
    - 만약 포트 번호가 잘못되었을 경우 서버는 반드시 CONNECT 요청을 거부해야하며, 일반적으로 `400 (Bad Request)`를 응답으로 보낸다.
  - OPTIONS
    - 목표 resource가 사용할 수 있는 communication option에 대한 정보를 요청한다.
    - 리소스가 어떤 method, header, content-type 등을 지원하는지 알 수 있다.
    - CORS에서도 사용된다.
  - TRACE
    - Request message의 loop-back을 요청하기 위해 사용한다.
    - 이 요청을 받은 최종 목적지(일반적으로 서버)는 받은 message에서 일부 필드를 제외하고 다시 클라이언트로 전송해야한다.
    - 클라이언트는 TRACE request에 response에서 탈취될 수 있는 민감한 정보를 담아선 안된다.
    - 즉, TRACE request 전송 - 서버에서 일부 필드 제외하고 클라이언트로 response 전송의 순서를 거치는데, TRACE request에 민감한 정보가 담겨있다면, 이 정보는 고스란히 response에 담겨 다시 클라이언트로 돌아오게 될 텐데, 이 때 response가 탈취당하면 민감한 정보도 함께 탈취되기 때문이다.
    - 예를 들어 TRACE request에는 사용자 인증과 관련된 정보를 담아서는 안 된다.
    - 또한 TRACE request를 받은 서버는 request에서 민감한 정보가 포함되었을 것 같은 필드들을 제외한 후 response를 생성해야한다.



- POST와 PUT의 차이
  - POST와 PUT의 근본적인 차이는 request와 함께 전송된 representation의 의도이다.
    - POST에서 목표 resource는 request에 포함된 representation을 resource의 semantic에 따라 처리해야한다.
    - 반면에, PUT에서 request에 포함된 representation은 목표 resource의 상태를 대체한다.
  - 이 근본적인 차이에 의해서 멱등성에서도 차이가 발생한다.
    - PUT은 POST와 달리 멱등성을 지닌다.



- 멱등성과 안정성, 캐시 가능성

  | Method  | Idempotence | Safety | Cacheable                              |
  | ------- | ----------- | ------ | -------------------------------------- |
  | GET     | YES         | YES    | YES                                    |
  | HEAD    | YES         | YES    | YES                                    |
  | POST    | NO          | NO     | Freshness 관련 정보가 있을 때에만 가능 |
  | PUT     | YES         | NO     | NO                                     |
  | DELETE  | YES         | NO     | NO                                     |
  | CONNECT | NO          | NO     | NO                                     |
  | OPTIONS | YES         | YES    | NO                                     |
  | TRACE   | YES         | YES    | NO                                     |
  
  - 멱등성
  
    - 멱등성이란 같은 연산을 여러번 실행하더라도 그 결과가 달라지지 않는 성질을 의미한다.
    - HTTP 메서드의 경우 경우 한 메서드로 여러 번 요청하더라도 서버의 상태가 한 번 요청한 것과 다르지 않다면, 해당 HTTP 메서드는 멱등성이 보장된다고 할 수 있다.
    - 외부 요인으로 리소스의 변경이 일어나는 것은 고려하지 않는다.
    - 즉, GET으로 data 조회, 해당 데이터를 PUT으로 수정, 해당 데이터를 다시 GET으로 조회하면 처음과 마지막의 결과가 달라지겠지만, 이런 경우는 고려하지 않는다.
    - 또한 status code가 다르다고 해서 멱득성이 보장되지 않는 것이 아니다.
  
  - 멱등성이 보장되는 메서드들
  
      - GET, HEAD, OPTIONS 등 단순 조회의 경우 여러번 수행되어도 결과값은 변하지 않기에 멱등성이 보장된다.
  
      - PUT은 목표 resource의 상태를 요청으로 보낸 representation으로 변경하는 것이므로, 여러 번 수행되도 결과는 변하지 않는다.
  
      - DELETE는 있는 데이터를 삭제할 경우와 없는 데이터를 삭제하려는 경우의 응답 코드는 다르겠지만 역시 해당 데이터가 없다는 결과 자체는 변하지 않는다.
  
  - 멱등성이 보장되지 않는 메서드들
  
      - POST의 경우 request와 함께 전송된 representation을 처리해야 하므로, 경우에 따라서 같은 request를 보내더라도 여러 개의 resource가 생성될 수 있다.
  
  - 안정성
  
    - 서버의 상태를 변경시키지 않는 메서드를 안전한 메서드라 부른다.
  
  - 캐시 가능성
  
    - 응답 결과 resource를 캐싱해서 사용할 수 있는가에 관한 개념이다.
    - 캐싱이 가능한 HTTP 메서드들은 캐싱해둔 resource를 사용하여 빠르게 응답을 반환할 수 있다.
  







## HTTP Status code

- 200
  - 204 No Content
    - 클라이언트의 요청을 정상적으로 처리했으나, 응답으로 돌려줄 data는 없을 때 사용한다.
    - 예를 들어 PUT이나 DELETE 등으로 새로운 데이터를 수정, 삭제 했을 경우에 사용한다.
    - 사용자가 요청한 data가 존재하지 않는다는 의미로 잘못 사용하지 않도록 주의해야한다.





# Shorts

- 어떤 data를 header에 담고, 어떤 data를 body이 담을 것인가.

  > https://datatracker.ietf.org/doc/html/rfc2616#section-5.3
  >
  > https://codeal.medium.com/headers-vs-body-1d3e754020b2
  >
  > https://softwareengineering.stackexchange.com/questions/277343/what-belongs-in-an-http-request-header-vs-the-request-body

  - Header
    - HTTP request, response에 대한 metadata를 담고 있다.
    - Client는 custom header를 추가할 수 있다.
  - Body
    - HTTP 통신을 통해 전달하고자 하는 data를 담고 있다.
    - HTTP spec 상으로 Body에 어떤 data를 담아야 한다는 규정이나 제한은 없다.
  - 어디에 무엇을 담아야하는가?
    - 전송하고자 하는 data가 metadata라면 header에 담는다.
    - HTTP가 규정한 reserved header에 속하는 data라면, 해당 header에 담는다(e.g. Authorization header에 token을 담는 것).
    - API의 대부분의 endpoint에서 사용하는 data라면 header에 담는다.
    - 만약 위 세 경우 중 어디에도 해당하지 않는다면 body에 담는다.



- Path variable과 Query parameter

  > https://medium.com/@fullsour/when-should-you-use-path-variable-and-query-parameter-a346790e8a6d

  - Path variable(path parameter)
    - URL 경로의 일부를 매개변수로 사용하는 방식이다.
    - 일반적으로 특정 resource를 가리키기 위해 사용한다.
    - 아래의 `1`이 path variable이다.

  ```bash
  $ curl "localhost:8000/docs/1"
  ```

  - Query parameter(query component, query string)
    - URL의 끝 부분에서 key, value의 쌍으로 매개변수를 표현하는 방식이다.
    - URL 경로와는 `?`를 통해 구분하며, 여러 개를 넘겨야 할 경우 `&`로 구분한다.

  ```bash
  $ curl "localhost:8000/docs?genre=romance"
  ```

  - 사용하는 이유
    - HTTP GET method를 사용해서 통신할 때는 body를 함께 전달할 수 없다.
    - 그러나 GET method를 사용하더라도 data를 전달해야 하는 경우가 있는데, 이 때 data를 전달하기 위해 사용한다.
  - 각각을 언제 사용해야 하는가?
    - 만약 resource를 식별해야한다면 path variable을 사용하는 것이 좋다.
    - 그러나 만약 정렬이나 filtering을 해야한다면 query parameter를 사용하는 것이 좋다.
  - Query parameter 사용시 URL의 마지막에 `/`를 붙여야 하는가?
    - 이전에는 붙여야 했지만, 현재는 붙이지 않아도 된다.
    - 즉 `localhost:8000/docs?genre=romance`와 같이 써도 되고, `localhost:8000/docs/?genre=romance`와 같이 써도 된다.

  


