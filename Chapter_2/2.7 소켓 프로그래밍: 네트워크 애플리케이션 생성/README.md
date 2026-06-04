# 2.7 소켓 프로그래밍: 네트워크 애플리케이션 생성

지금까지는 HTTP, SMTP, DNS 같은 이미 존재하는 애플리케이션 계층 프로토콜을 살펴보았다.

이번 절에서는 애플리케이션 개발자가 직접 네트워크 프로그램을 만들 때 사용하는 **소켓(socket)**을 다룬다.

소켓은 애플리케이션 프로세스와 트랜스포트 계층 사이의 인터페이스이다.

애플리케이션은 소켓을 통해 네트워크로 데이터를 보내고, 네트워크에서 도착한 데이터를 받는다.

## 2.7.1 소켓이란?

네트워크 애플리케이션은 보통 서로 다른 종단 시스템에서 실행되는 두 프로세스가 통신하는 구조이다.

예를 들어 웹 브라우저 프로세스와 웹 서버 프로세스가 HTTP 메시지를 주고받는다.

이때 프로세스가 네트워크로 직접 패킷을 조작하는 것은 아니다.

애플리케이션은 운영체제가 제공하는 소켓 API를 사용한다.

소켓을 비유하면 애플리케이션과 네트워크 사이의 문과 같다.

프로세스는 데이터를 소켓 밖으로 밀어 넣고, 반대편 프로세스는 자신의 소켓을 통해 데이터를 읽는다.

소켓 프로그래밍을 할 때 개발자는 어떤 트랜스포트 계층 서비스를 사용할지 선택해야 한다.

대표적으로 다음 두 가지가 있다.

1. **UDP**
2. **TCP**

## 2.7.2 UDP 소켓 프로그래밍

UDP는 연결을 설정하지 않고 데이터를 보내는 트랜스포트 계층 프로토콜이다.

UDP를 사용하는 애플리케이션은 데이터를 보낼 때마다 목적지 IP 주소와 포트 번호를 함께 지정한다.

UDP의 특징은 다음과 같다.

- 연결 설정 과정이 없다.
- 데이터는 독립적인 데이터그램으로 전송된다.
- 신뢰적인 전송을 보장하지 않는다.
- 순서 보장을 제공하지 않는다.
- TCP보다 단순하고 오버헤드가 작다.

UDP는 손실이 조금 발생해도 빠른 전송이 중요한 애플리케이션에서 사용될 수 있다.

예를 들어 실시간 음성, 실시간 영상, 간단한 질의/응답 프로토콜에서 UDP가 사용될 수 있다.

### UDP 클라이언트 동작

UDP 클라이언트의 기본 흐름은 다음과 같다.

1. UDP 소켓을 생성한다.
2. 보낼 메시지를 준비한다.
3. 서버의 IP 주소와 포트 번호를 지정해 메시지를 보낸다.
4. 서버의 응답을 기다린다.
5. 응답을 받은 뒤 소켓을 닫는다.

Python 형태로 보면 다음과 같다.

```python
from socket import *

serverName = "hostname"
serverPort = 12000

clientSocket = socket(AF_INET, SOCK_DGRAM)

message = input("Input lowercase sentence: ")
clientSocket.sendto(message.encode(), (serverName, serverPort))

modifiedMessage, serverAddress = clientSocket.recvfrom(2048)
print(modifiedMessage.decode())

clientSocket.close()
```

`SOCK_DGRAM`은 UDP 소켓을 의미한다.

`sendto`는 메시지와 목적지 주소를 함께 전달한다.

UDP는 연결이 없기 때문에 클라이언트 소켓을 만들 때 서버와 미리 연결하지 않는다.

### UDP 서버 동작

UDP 서버는 특정 포트에 소켓을 바인딩하고 클라이언트의 메시지를 기다린다.

기본 흐름은 다음과 같다.

1. UDP 소켓을 생성한다.
2. 서버 포트 번호에 소켓을 바인딩한다.
3. 클라이언트 메시지를 기다린다.
4. 메시지를 처리한다.
5. 클라이언트 주소로 응답을 보낸다.

```python
from socket import *

serverPort = 12000
serverSocket = socket(AF_INET, SOCK_DGRAM)
serverSocket.bind(("", serverPort))

print("The server is ready to receive")

while True:
    message, clientAddress = serverSocket.recvfrom(2048)
    modifiedMessage = message.decode().upper()
    serverSocket.sendto(modifiedMessage.encode(), clientAddress)
```

서버는 `recvfrom`을 통해 메시지와 클라이언트 주소를 함께 얻는다.

그리고 `sendto`를 사용해 해당 클라이언트에게 응답을 보낸다.

## 2.7.3 TCP 소켓 프로그래밍

TCP는 연결 지향형 프로토콜이다.

UDP와 달리 데이터를 보내기 전에 클라이언트와 서버 사이에 TCP 연결을 먼저 설정한다.

TCP의 특징은 다음과 같다.

- 연결 설정 과정이 있다.
- 신뢰적인 데이터 전송을 제공한다.
- 바이트 스트림 방식으로 데이터를 전달한다.
- 순서가 보장된다.
- 흐름 제어와 혼잡 제어를 제공한다.

TCP는 웹, 메일, 파일 전송처럼 데이터가 정확하게 도착해야 하는 애플리케이션에서 주로 사용된다.

### TCP 클라이언트 동작

TCP 클라이언트의 기본 흐름은 다음과 같다.

1. TCP 소켓을 생성한다.
2. 서버의 IP 주소와 포트 번호로 연결을 요청한다.
3. 연결이 만들어지면 데이터를 보낸다.
4. 서버의 응답을 받는다.
5. 소켓을 닫는다.

```python
from socket import *

serverName = "hostname"
serverPort = 12000

clientSocket = socket(AF_INET, SOCK_STREAM)
clientSocket.connect((serverName, serverPort))

sentence = input("Input lowercase sentence: ")
clientSocket.send(sentence.encode())

modifiedSentence = clientSocket.recv(1024)
print(modifiedSentence.decode())

clientSocket.close()
```

`SOCK_STREAM`은 TCP 소켓을 의미한다.

TCP에서는 `connect`를 호출해 서버와 연결을 만든 뒤 `send`와 `recv`를 사용한다.

UDP의 `sendto`와 달리 TCP의 `send`에는 목적지 주소를 매번 넣지 않는다.

이미 연결이 특정 서버 프로세스와 연결되어 있기 때문이다.

### TCP 서버 동작

TCP 서버는 클라이언트의 연결 요청을 기다리는 소켓을 만든다.

이 소켓을 **환영 소켓(welcome socket)**이라고 생각할 수 있다.

클라이언트가 연결을 요청하면 서버는 해당 클라이언트와 통신하기 위한 새로운 **연결 소켓(connection socket)**을 만든다.

기본 흐름은 다음과 같다.

1. TCP 소켓을 생성한다.
2. 서버 포트에 소켓을 바인딩한다.
3. 클라이언트 연결 요청을 기다린다.
4. 연결 요청이 오면 새로운 연결 소켓을 만든다.
5. 연결 소켓으로 데이터를 주고받는다.

```python
from socket import *

serverPort = 12000
serverSocket = socket(AF_INET, SOCK_STREAM)
serverSocket.bind(("", serverPort))
serverSocket.listen(1)

print("The server is ready to receive")

while True:
    connectionSocket, addr = serverSocket.accept()
    sentence = connectionSocket.recv(1024).decode()
    capitalizedSentence = sentence.upper()
    connectionSocket.send(capitalizedSentence.encode())
    connectionSocket.close()
```

`accept`는 클라이언트의 연결 요청을 수락하고, 해당 클라이언트와 통신할 새로운 소켓을 반환한다.

서버는 환영 소켓으로 새로운 연결을 기다리고, 연결 소켓으로 실제 데이터를 주고받는다.

## 2.7.4 UDP와 TCP 비교

UDP와 TCP는 모두 애플리케이션이 네트워크로 데이터를 보내기 위해 사용하는 트랜스포트 계층 프로토콜이다.

하지만 제공하는 서비스는 다르다.

| 구분 | UDP | TCP |
| --- | --- | --- |
| 연결 설정 | 없음 | 있음 |
| 전송 방식 | 데이터그램 | 바이트 스트림 |
| 신뢰성 | 보장하지 않음 | 보장 |
| 순서 보장 | 보장하지 않음 | 보장 |
| 목적지 지정 | 보낼 때마다 지정 | 연결 시 지정 |
| 주요 장점 | 단순함, 낮은 오버헤드 | 신뢰성, 순서 보장 |

UDP는 단순하고 빠르지만, 손실이나 순서 뒤바뀜을 애플리케이션이 직접 처리해야 할 수 있다.

TCP는 더 많은 기능을 제공하지만, 연결 설정과 제어 메커니즘 때문에 UDP보다 복잡하다.

## 2.7.5 포트 번호의 역할

IP 주소는 종단 시스템을 식별한다.

하지만 한 호스트 안에는 여러 네트워크 애플리케이션이 동시에 실행될 수 있다.

예를 들어 한 컴퓨터에서 웹 브라우저, 메신저, 메일 클라이언트가 동시에 네트워크를 사용할 수 있다.

이때 어떤 프로세스에게 데이터를 전달해야 하는지 구분하기 위해 **포트 번호(port number)**를 사용한다.

소켓은 보통 다음 정보와 연결된다.

- IP 주소
- 포트 번호
- 트랜스포트 프로토콜

서버 애플리케이션은 잘 알려진 포트에서 요청을 기다린다.

예를 들어 HTTP 서버는 보통 80번 포트, SMTP 서버는 25번 포트에서 요청을 기다린다.

클라이언트는 서버의 IP 주소와 포트 번호를 알아야 해당 애플리케이션 프로세스에 연결할 수 있다.

## 2.7.6 애플리케이션 개발자가 보는 소켓

소켓 프로그래밍에서 개발자는 트랜스포트 계층 아래의 라우팅, 링크 계층, 물리 계층을 직접 다루지 않는다.

개발자는 다음 정도를 결정한다.

- TCP를 사용할지 UDP를 사용할지
- 어느 호스트와 통신할지
- 어떤 포트 번호를 사용할지
- 어떤 메시지 형식을 사용할지
- 받은 데이터를 어떻게 처리할지

즉, 소켓은 복잡한 네트워크 내부 구조를 감추고, 애플리케이션이 통신 기능을 사용할 수 있도록 해주는 API이다.

## 정리

소켓은 애플리케이션 프로세스와 트랜스포트 계층 사이의 인터페이스이다.

UDP 소켓은 연결 없이 데이터그램을 보내며, 목적지 주소를 매번 지정한다.

TCP 소켓은 먼저 연결을 설정하고, 이후 신뢰적인 바이트 스트림을 주고받는다.

네트워크 애플리케이션 개발자는 소켓 API를 사용해 직접 프로토콜과 서비스를 만들 수 있다.

이 절은 앞에서 배운 HTTP, SMTP, DNS 같은 애플리케이션 프로토콜이 실제 프로그램 안에서 어떤 방식으로 구현될 수 있는지 이해하는 출발점이다.
