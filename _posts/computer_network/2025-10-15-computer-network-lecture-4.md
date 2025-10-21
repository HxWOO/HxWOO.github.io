---
layout: post
title: "🚚 전송 계층: 데이터는 어떻게 신뢰성 있게 전달될까? 4️⃣"
date: 2025-10-15 20:00:00 +0900
categories: [컴퓨터 네트워크]
tags: [CS, Network, Transport Layer, TCP, UDP, RDT]
---

# 📨 전송 계층의 역할과 원리

전송 계층(Transport Layer)은 서로 다른 호스트에서 실행되는 애플리케이션 프로세스 간의 **논리적 통신(logical communication)** 을 제공한다. 실제 데이터는 라우터와 스위치를 거쳐 물리적으로 전달되지만, 전송 계층 덕분에 우리는 마치 두 프로세스가 직접 연결된 것처럼 데이터를 주고받을 수 있다.

- **송신 측(Sender):** 애플리케이션의 메시지를 **세그먼트(segment)** 단위로 분할하고, 네트워크 계층으로 전달한다.
- **수신 측(Receiver):** 네트워크 계층에서 받은 세그먼트를 원래의 메시지로 재조립하여 애플리케이션으로 전달한다.

인터넷의 전송 계층에는 대표적으로 두 가지 프로토콜이 있다: **TCP**와 **UDP**.

## 1. 전송 계층 vs 네트워크 계층

두 계층의 가장 큰 차이점은 통신의 대상을 어디까지 보느냐에 있다.

- **전송 계층 (Transport Layer):** **프로세스 대 프로세스(process-to-process)** 의 논리적 통신을 담당한다.
- **네트워크 계층 (Network Layer):** **호스트 대 호스트(host-to-host)** 의 통신을 담당한다.

즉, 네트워크 계층은 IP 주소를 보고 데이터를 올바른 컴퓨터까지 배달하는 택배 기사라면, 전송 계층은 그 컴퓨터 안에서 포트 번호를 보고 올바른 애플리케이션(프로세스)에게 데이터를 전달하는 집배원과 같다.

![Untitled](https://raw.githubusercontent.com/HxWOO/HxWOO.github.io/master/assets/images/computer_network_4/Untitled.png)

## 2. Multiplexing & Demultiplexing

하나의 호스트에서는 여러 개의 네트워크 애플리케이션이 동시에 실행될 수 있다. 전송 계층은 어떻게 이 많은 프로세스들에게서 오는 데이터를 받아서 처리하고, 또 들어오는 데이터를 올바른 프로세스에게 전달할 수 있을까? 바로 **다중화(Multiplexing)** 와 **역다중화(Demultiplexing)** 덕분이다.

- **다중화 (Multiplexing) @ 송신 측:** 여러 소켓(프로세스)으로부터 온 데이터들을 모아 전송 계층 헤더를 붙여서 세그먼트를 만든다.
- **역다중화 (Demultiplexing) @ 수신 측:** 수신된 세그먼트의 헤더 정보(특히 포트 번호)를 보고, 해당 데이터를 올바른 소켓(프로세스)으로 전달한다.

![Untitled](https://raw.githubusercontent.com/HxWOO/HxWOO.github.io/master/assets/images/computer_network_4/Untitled%201.png)

### 역다중화는 어떻게 동작할까?

호스트는 IP 데이터그램을 수신하면, 그 안에 담긴 전송 계층 세그먼트를 꺼낸다. 그리고 세그먼트 헤더에 있는 **출발지/목적지 IP 주소**와 **출발지/목적지 포트 번호**를 사용하여 이 세그먼트를 전달할 정확한 소켓을 찾아낸다.

#### 비연결형 역다중화 (UDP)

UDP 소켓은 **목적지 IP 주소**와 **목적지 포트 번호**, 단 두 가지 정보로 식별된다. 수신된 UDP 세그먼트의 목적지 포트 번호를 보고 해당하는 소켓으로 데이터를 전달한다. 이 때문에 출발지 IP나 포트가 다른 여러 호스트로부터 온 데이터그램이라도, 목적지 IP와 포트 번호만 같다면 모두 같은 UDP 소켓으로 전달될 수 있다.

![Untitled](https://raw.githubusercontent.com/HxWOO/HxWOO.github.io/master/assets/images/computer_network_4/Untitled%202.png)

#### 연결 지향형 역다중화 (TCP)

TCP 소켓은 **(출발지 IP, 출발지 포트 번호, 목적지 IP, 목적지 포트 번호)**, 총 네 가지 정보의 조합으로 식별된다. 따라서 모든 세그먼트는 이 네 가지 값이 모두 일치하는 특정 소켓으로만 전달된다. 덕분에 서버는 동시에 여러 클라이언트와 각각 독립적인 TCP 연결을 유지할 수 있다.

![Untitled](https://raw.githubusercontent.com/HxWOO/HxWOO.github.io/master/assets/images/computer_network_4/Untitled%203.png)

## 3. UDP: User Datagram Protocol

UDP는 "아무것도 꾸미지 않은(no-frills)" 인터넷 프로토콜이다. IP가 제공하는 **최선 노력(best-effort)** 서비스 외에 거의 아무런 기능도 추가하지 않는다.

- **비신뢰적(Unreliable):** 데이터가 손실될 수도, 순서가 뒤바뀌어 도착할 수도 있다.
- **비연결형(Connectionless):** 데이터를 보내기 전에 핸드셰이킹 같은 연결 설정 과정이 없다. 각 세그먼트는 독립적으로 처리된다.

### 그럼 UDP는 왜 쓸까? 🤔

- **빠른 연결:** 연결 설정 과정이 없으므로 지연이 적다.
- **단순함:** 연결 상태를 유지할 필요가 없어 구조가 간단하다.
- **작은 헤더:** TCP보다 헤더 크기가 작아 오버헤드가 적다.
- **혼잡 제어 없음:** TCP처럼 네트워크 혼잡에 따라 전송 속도를 조절하지 않는다. 따라서 개발자가 원한다면 데이터를 최대한 빠르게 전송할 수 있다.

이러한 특징 덕분에 실시간 스트리밍, DNS, SNMP, 그리고 최근의 HTTP/3 등에서 UDP를 사용한다. 물론 신뢰성이 필요하다면 애플리케이션 계층에서 직접 구현해야 한다.

### UDP 세그먼트 구조와 체크섬

![Untitled](https://raw.githubusercontent.com/HxWOO/HxWOO.github.io/master/assets/images/computer_network_4/Untitled%204.png)

UDP 헤더는 출발지/목적지 포트 번호, 길이, 그리고 **체크섬(Checksum)**으로 구성된다. 체크섬은 전송 중 발생할 수 있는 오류를 검출하기 위한 값이다.

- **송신 측:** 세그먼트의 내용을 16비트 정수의 나열로 간주하고 모두 더한 후, 1의 보수를 취해 체크섬 필드에 저장한다. (이때 발생하는 올림(carry)은 다시 더해준다.)
- **수신 측:** 받은 세그먼트로 똑같이 체크섬을 계산하여, 수신된 체크섬 값과 비교한다. 값이 다르면 오류가 발생한 것으로 판단한다.

## 4. 신뢰성 있는 데이터 전송 (RDT)

UDP와 달리 TCP는 신뢰성 있는 데이터 전송을 보장한다. 어떻게 신뢰할 수 없는 채널(e.g., IP) 위에서 데이터의 손실이나 오류 없이 완벽하게 데이터를 전달할 수 있을까? **신뢰성 있는 데이터 전송(Reliable Data Transfer, RDT)** 프로토콜의 원리를 점진적으로 발전시켜보자.

![Untitled](https://raw.githubusercontent.com/HxWOO/HxWOO.github.io/master/assets/images/computer_network_4/Untitled%205.png)

### rdt 2.0: 오류 검출과 피드백

가장 기본적인 아이디어는 수신자가 패킷을 잘 받았는지, 아니면 오류가 있었는지 송신자에게 알려주는 것이다.

- **ACK (Acknowledgements):** "패킷 잘 받았어!" 라는 긍정의 신호.
- **NAK (Negative Acknowledgements):** "패킷에 오류가 있어!" 라는 부정의 신호.

송신자는 패킷을 보내고 ACK나 NAK를 기다린다. NAK를 받으면 해당 패킷을 재전송하고, ACK를 받으면 다음 패킷을 보낸다. 이 방식을 **Stop-and-Wait**라고 한다.

![Untitled](https://raw.githubusercontent.com/HxWOO/HxWOO.github.io/master/assets/images/computer_network_4/Untitled%207.png)

하지만 만약 ACK/NAK 메시지 자체에 오류가 생긴다면? 송신자는 중복된 패킷을 보낼 수도 있고, 수신자는 이를 구분할 방법이 없다. 😱

### rdt 2.1 & 2.2: 순서 번호와 NAK-free

이 문제를 해결하기 위해 **순서 번호(Sequence Number)**를 도입한다. 송신자는 각 패킷에 0, 1, 0, 1... 순으로 번호를 붙인다. 수신자는 이 번호를 보고 중복된 패킷인지 아닌지 판단하여 처리(또는 폐기)할 수 있다.

![Untitled](https://raw.githubusercontent.com/HxWOO/HxWOO.github.io/master/assets/images/computer_network_4/Untitled%2010.png)

더 나아가, **rdt 2.2**에서는 NAK를 없앨 수 있다. 수신자는 NAK를 보내는 대신, 마지막으로 **정상적으로 수신한 패킷의 ACK**를 다시 보낸다. 송신자 입장에서 중복된 ACK를 받는 것은 '방금 보낸 패킷에 문제가 생겼구나'라는 신호(NAK)와 같으므로, 해당 패킷을 재전송하면 된다. 이것이 TCP가 사용하는 방식이다.

![Untitled](https://raw.githubusercontent.com/HxWOO/HxWOO.github.io/master/assets/images/computer_network_4/Untitled%2012.png)

### rdt 3.0: 패킷 손실 처리

만약 패킷이나 ACK가 아예 **손실(loss)** 된다면 어떻게 될까? 송신자는 무한정 기다리게 될 것이다. 이를 해결하기 위해 **타이머(Timer)** 를 사용한다.

송신자는 패킷을 보낼 때 타이머를 설정하고, 일정 시간(timeout) 내에 ACK가 도착하지 않으면 패킷이 손실된 것으로 간주하고 재전송한다. 만약 단순히 지연된 것이었다면 중복 전송이 발생하지만, 순서 번호 덕분에 수신자는 중복을 처리할 수 있다.

![Untitled](https://raw.githubusercontent.com/HxWOO/HxWOO.github.io/master/assets/images/computer_network_4/Untitled%2013.png)

이 Stop-and-Wait 방식의 rdt 3.0은 신뢰성을 보장하지만, 한 번에 패킷 하나씩만 처리하므로 채널의 이용률(utilization)이 매우 낮다는 단점이 있다.

![Untitled](https://raw.githubusercontent.com/HxWOO/HxWOO.github.io/master/assets/images/computer_network_4/Untitled%2016.png)

이 성능 문제를 해결하기 위해 등장한 것이 바로 **파이프라이닝(Pipelining)** 이며, 이는 다음 TCP 파트에서 더 자세히 다루게 될 것이다.

전송 계층의 복잡한 원리를 하나씩 알아가는 과정이 흥미롭다. 신뢰성 있는 통신을 위해 정말 많은 장치들이 숨어있다.
