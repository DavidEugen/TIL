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



## HTTP(HyperText Transfer Protocol)

우리가 전송하는 대부분의 데이터들이 HTTP 메세지로 전송

- HTML, TEXT
- IMAGE, 음성, 영상, 파일
- JSON, XML (API)
- 거의 모든 형태의 데이터 전송 가능
- 서버간에 데이터를 주고 받을 때도 대부분 HTTP 사용

### 특징

- 클라이언트 서버 구조
- 무상태 프로토콜(스테이스리스), 비연결성
- HTTP 메시지
- 단순함, 확장 가능

### 1. 클라이언트 서버 구조

- Request Response 구조
- 클라이언트는 서버에 요청을 보내고, 응답을 대기 
- 서버가 요청에 대한 결과를 만들어서 응답

### 2. 무상태 프로토콜 (Stateless)

- 서버가 클라이언트의 상태를 보존하지 않음 
- 장점: 서버 확장성 높음(스케일 아웃) 
- 단점: 클라이언트가 추가 데이터 전송

> Stateful, Stateless 차이
>
> Stateful - 상태 유지 (대화의 맥락을 알고 있다. 다른 상태들을 알고 있다.)
>
> ​	예) 한 점원이 계속 응대 
>
> Stateless - 무상태 (현재 요청에 대해서만 알고 있다. 다른 상태들을 모르고 있다.)
>
> ​	예) 요청마다 다른 점원이 응대
>
> **상태 유지** : 중간에 다른 점원으로 바뀌면 안된다. 
> 중간에 다른 점원으로 바뀔 때 상태 정보를 다른 점원에게 미리 알려줘야 한다.)
>
> **무상태**: 중간에 다른 점원으로 바뀌어도 된다.
> 갑자기 고객이 증가해도 점원을 대거 투입할 수 있다.
> 갑자기 클라이언트 요청이 증가해도 서버를 대거 투입할 수 있다. 
>
> 무상태는 응답 서버를 쉽게 바꿀 수 있다. -> **무한한 서버 증설 가능**

#### 무상태의 한계

- 모든 것을 무상태로 설계 할 수 있는 경우도 있고 없는 경우도 있다. 

  > 무상태
  >
  > 예) 로그인이 필요 없는 단순한 서비스 소개 화면 
  >
  > 상태 유지
  >
  > 예) 로그인
  >
  > 로그인한 사용자의 경우 로그인 했다는 상태를 서버에 유지 
  >
  > 일반적으로 브라우저 쿠키와 서버 세션등을 사용해서 상태 유지 

- 상태 유지는 최소한만 사용

### 3. 비 연결성(connectionless)

- HTTP는 기본이 연결을 유지하지 않는 모델 

- 일반적으로 초 단위의 이하의 빠른 속도로 응답

- 1시간 동안 수천명이 서비스를 사용해도 실제 서버에서 동시에 처리하는 요청은 수십개 이 하로 매우 작음

  예) 웹 브라우저에서 계속 연속해서 검색 버튼을 누르지는 않는다. 

- 서버 자원을 매우 효율적으로 사용할 수 있음

#### 비 연결성 한계와 극복

- TCP/IP 연결을 새로 맺어야 함 - 3 way handshake 시간 추가
- 웹 브라우저로 사이트를 요청하면 HTML 뿐만 아니라 자바스크립트, css, 추가 이미지 등 수 많은 자원이 함께 다운로드
- 지금은 HTTP 지속 연결(Persistent Connections)로 문제 해결 
- HTTP/2, HTTP/3에서 더 많은 최적화
