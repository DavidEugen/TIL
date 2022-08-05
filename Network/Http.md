# Http

## 인터넷 프로토콜 스택 4계층

- application 계층 - HTTP, FTP
- 전송 계층 - TCP, UDP
- 인터넷 계층 - IP
- 네트워크 인터페이스 계층



## 프로토콜 계층에서 본 데이터 전송

- 1. application 에서 Socket 라이브러리를 통해서 메세지 데이터 전달.
- 2. TCP 정보 생성, 메세지 데이터 포함
- 3. IP 패킷 생성, TCP 데이터 포함
- 4. LAN 카드 통해 ethernet frame로 감싸 인터넷에 전달



## IP(Internet Protocol) 

### 역할

- 지정한 IP 주소(IP Address)에 데이터 전달 

- 패킷(Packet)이라는 통신 단위로 데이터 전달

### 한계

#### - 비연결성

- 패킷을 받을 대상이 없거나 서비스 불능 상태여도 패킷 전송

#### - 비신뢰성

- 중간에 패킷이 사라지면?

- 패킷이 순서대로 안오면? 

#### - 프로그램 구분

- 같은 IP를 사용하는 서버에서 통신하는 애플리케이션이 둘 이상이면?



## TCP 특징

TCP : 전송 제어 프로토콜(Transmission Control Protocol)

- 연결지향 - TCP 3 way handshake (가상 연결) 
- 데이터 전달 보증
- 순서 보장

신뢰할 수 있는 프로토콜 

현재는 대부분 TCP 사용 ( http 3 이후 UDP 주로 사용)



## UDP 특징

UDP : 사용자 데이터그램 프로토콜(User Datagram Protocol)

기능이 거의 없음 

- 연결지향 - TCP 3 way handshake X 
- 데이터 전달 보증 X
- 순서 보장 X

데이터 전달 및 순서가 보장되지 않지만, 단순하고 빠름 정리

IP와 거의 같다. +PORT +체크섬 정도만 추가

애플리케이션에서 추가 작업 필요



## DNS
도메인 네임 시스템(Domain Name System)

Domain Name 과 Ip 주소 매핑 

Domain Name으로 IP 주소 변환



## URI

URI(Uniform Resource Identifier)

URI는 로케이터(locator - URL), 이름(name - URN) 또는 둘 다 추가로 분류될 수 있다



**U**niform: 리소스 식별하는 통일된 방식

**R**esource: 자원, URI로 식별할 수 있는 모든 것(제한 없음) 

**I**dentifier: 다른 항목과 구분하는데 필요한 정보

URL: Uniform Resource Locator 

URN: Uniform Resource Name

URL - Locator: 리소스가 있는 **위치**를 지정
URN - Name: 리소스에 **이름**을 부여
위치는 변할 수 있지만, 이름은 변하지 않는다. ex 책의 isbn URN
URN 이름만으로 실제 리소스를 찾을 수 있는 방법이 보편화 되지 않음 

보통 URL을 URI 개념으로 쓴다... 



## URL 문법

```scheme://[userinfo@]host[:port][/path][?query][#fragment]```

ex)

```https://www.google.com:443/search?q=hello&hl=ko```

**scheme** - 프로토콜(https) 어떤 방식으로 자원에 접근할 것인가 하는 약속 규칙

[userinfo@] - 없음

**host** - 호스트명(www.google.com) 

[:port] - 포트 번호(443)

[/path] - 패스(/search)

[?query] - 쿼리 파라미터(q=hello&hl=ko)





## 웹 브라우저 요청 흐름

1. URL을 통해 HTTP Request Message 생성

```https://www.google.com:443/search?q=hello&hl=ko``` 라는 요청을 보낸다면

```GET /search?q=hello&hl=ko HTTP/1.1 Host: www.google.com```

프로토콜 계층에서 본 데이터 전송 1.에서 매세지가 HTTP 메세지

이는 곧 IP 패킷에 쌓여 전달.

2. 서버에서 처리후 응답 메세지 전달.

``` text
HTTP/1.1 200 OK
Content-Type: text/html;charset=UTF-8 Content-Length: 3423

<html> 
  <body>...</body>
</html>
```



3. 응답 메세지로 웹브라우저에서 렌더링
