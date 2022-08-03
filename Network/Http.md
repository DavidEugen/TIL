# Http

## 인터넷 프로토콜 스택 4계층

- application 계층 - HTTP, FTP
- 전송 계층 - TCP, UDP
- 인터넷 계층 - IP
- 네트워크 인터페이스 계층



## 프로토콜 계층에서 본 데이터 전송

- application 에서 Socket 라이브러리를 통해서 메세지 데이터 전달.
- TCP 정보 생성, 메세지 데이터 포함
- IP 패킷 생성, TCP 데이터 포함
- LAN 카드 통해 ethernet frame로 감싸 인터넷에 전달



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