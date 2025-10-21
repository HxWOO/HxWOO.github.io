---
layout: post
title: "🌐 네트워크 계층: 인터넷의 택배 시스템은 어떻게 동작할까? 6️⃣"
date: 2025-10-17 21:00:00 +0900
categories: [컴퓨터 네트워크]
tags: [CS, Network, Network Layer, IP, Routing, Forwarding, SDN, NAT, ICMP]
---

# 🌐 네트워크 계층의 세계로!

지난번엔 전송 계층에서 프로세스 간의 논리적 통신을 담당하는 TCP와 UDP에 대해 알아보았다. 이번에는 한 컴퓨터에서 다른 컴퓨터로 데이터 패킷을 전달하는, 즉 **호스트 대 호스트(host-to-host)** 통신의 핵심인 **네트워크 계층(Network Layer)** 에 대해 깊이 파고들어 보자. 인터넷이라는 거대한 네트워크의 택배 시스템이 어떻게 동작하는지 그 원리를 살펴보자. 🚚

## 1. 네트워크 계층의 두 가지 핵심 기능: Forwarding & Routing

네트워크 계층은 두 가지 중요한 기능을 수행한다.

- **포워딩 (Forwarding):** 라우터에 도착한 패킷을 어떤 출력 링크(output link)로 보내야 할지 결정하는 **로컬한** 동작이다. 라우터의 입력 포트로 들어온 패킷을 적절한 출력 포트로 스위칭하는 과정이다.
- **라우팅 (Routing):** 패킷이 출발지에서 목적지까지 가는 전체 **경로를 결정**하는 과정이다. 라우팅 알고리즘을 통해 얻어진 결과를 바탕으로 각 라우터는 포워딩 테이블을 만들게 된다.

쉽게 비유하자면, 라우팅은 여행자가 전체 지도(네트워크 토폴로지)를 보고 서울에서 부산까지 갈 경로(e.g., 경부고속도로)를 정하는 것이고, 포워딩은 각 고속도로 분기점(라우터)에서 이정표(포워딩 테이블)를 보고 다음 방향(e.g., '부산' 방면)으로 핸들을 꺾는 것과 같다.

## 2. 데이터 플레인 vs 컨트롤 플레인

네트워크 계층의 동작은 논리적으로 두 개의 '플레인(plane)'으로 나눌 수 있다.

### 데이터 플레인 (Data Plane)

- 각 라우터에서 **포워딩** 기능을 수행하는 부분이다.
- 패킷이 입력 포트에서 출력 포트로 어떻게 전달될지를 결정하고 실행한다.
- 속도가 매우 중요하기 때문에 주로 하드웨어로 구현된다. (ns 단위)

### 컨트롤 플레인 (Control Plane)

- 네트워크 전체의 **라우팅**을 담당하는 부분이다.
- 라우팅 알고리즘을 실행하여 포워딩 테이블을 계산하고, 이를 각 라우터의 데이터 플레인에 설치한다.
- 전통적으로는 각 라우터가 라우팅 알고리즘을 실행하며 서로 정보를 교환했지만, 최근에는 **SDN(Software-Defined Networking)** 이라는 새로운 접근 방식이 등장했다.
    - **SDN:** 중앙의 원격 컨트롤러가 전체 네트워크를 조망하며 경로를 계산하고, 각 라우터에 포워딩 규칙을 내려주는 방식이다. 덕분에 더 유연하고 중앙 집중적인 네트워크 관리가 가능해졌다.

![Untitled](https://raw.githubusercontent.com/HxWOO/HxWOO.github.io/master/assets/images/computer_network_6/Untitled.png)

## 3. 라우터는 어떻게 생겼을까?

라우터는 패킷을 포워딩하기 위해 특별히 설계된 고성능 컴퓨터다. 그 내부 구조를 간단히 살펴보자.

![Untitled](https://raw.githubusercontent.com/HxWOO/HxWOO.github.io/master/assets/images/computer_network_6/Untitled%201.png)

- **입력 포트 (Input Ports):** 물리적 링크로부터 패킷을 수신하고, 링크 계층 프로토콜을 처리한 뒤, 포워딩 테이블을 **조회(lookup)** 하여 패킷을 어디로 보낼지 결정한다. 여기서 지연이 발생하면 큐잉이 일어날 수 있다.
- **스위칭 패브릭 (Switching Fabric):** 입력 포트와 출력 포트를 연결하는 라우터의 핵심부다. 패킷을 입력 포트에서 올바른 출력 포트로 신속하게 전달하는 역할을 한다. 메모리, 버스, 인터커넥션 네트워크 등 다양한 방식으로 구현된다.
- **출력 포트 (Output Ports):** 스위칭 패브릭으로부터 전달받은 패킷을 저장(큐잉)했다가, 링크를 통해 다음 목적지로 전송한다. 출력 링크의 전송 속도보다 패킷이 더 빨리 도착하면 버퍼에 쌓이게 되고, 버퍼가 가득 차면 **패킷 손실(drop)** 이 발생한다.
- **라우팅 프로세서 (Routing Processor):** 컨트롤 플레인 기능을 수행한다. 라우팅 프로토콜을 실행하고, SDN 컨트롤러와 통신하며 포워딩 테이블을 계산하고 갱신한다.

### Longest Prefix Matching

포워딩 테이블을 조회할 때, 라우터는 목적지 IP 주소와 가장 길게 일치하는 항목(prefix)을 선택한다. 이를 **Longest Prefix Matching** 규칙이라고 한다. 덕분에 더 구체적인 경로 정보가 일반적인 경로 정보보다 우선순위를 갖게 되어 효율적인 라우팅이 가능하다.

![Untitled](https://raw.githubusercontent.com/HxWOO/HxWOO.github.io/master/assets/images/computer_network_6/Untitled%202.png)

## 4. 인터넷의 주소 체계: IP 프로토콜

인터넷의 모든 장치는 **IP(Internet Protocol)** 주소라는 고유한 식별자를 가진다. 현재 널리 사용되는 IPv4를 기준으로 알아보자.

### IPv4 데이터그램 포맷

![Untitled](https://raw.githubusercontent.com/HxWOO/HxWOO.github.io/master/assets/images/computer_network_6/Untitled%207.png)

- **Version:** IP 프로토콜의 버전 (e.g., 4).
- **Header length:** 헤더의 길이. 옵션이 없으면 20바이트다.
- **Length:** 데이터그램 전체(헤더+데이터)의 길이.
- **TTL (Time to Live):** 패킷이 네트워크 안에서 영원히 떠도는 것을 방지하기 위한 값. 라우터를 하나 거칠 때마다 1씩 감소하며, 0이 되면 패킷은 폐기된다.
- **Source/Destination IP address:** 32비트 길이의 출발지 및 목적지 IP 주소.

### IP 주소와 서브넷

- **IP 주소:** 호스트나 라우터의 **인터페이스(interface)** 에 할당되는 32비트 숫자다.
- **서브넷 (Subnet):** IP 주소는 네트워크 부분을 나타내는 **서브넷 파트(subnet part)** 와 해당 네트워크 내에서 장치를 식별하는 **호스트 파트(host part)** 로 나뉜다. 라우터 없이 서로 통신할 수 있는 장치들의 집합을 하나의 서브넷이라고 볼 수 있다.
- **서브넷 마스크 & CIDR:** `/24`와 같이 표현하며, IP 주소의 앞 24비트가 서브넷 부분임을 나타낸다. 이를 **CIDR(Classless InterDomain Routing)** 표기법이라고 하며, 기존의 A, B, C 클래스 방식보다 유연하게 주소를 할당하고 관리할 수 있게 해준다.

![Untitled](https://raw.githubusercontent.com/HxWOO/HxWOO.github.io/master/assets/images/computer_network_6/Untitled%208.png)

### IP 주소는 어떻게 받을까? (DHCP)

우리가 카페에서 와이파이에 접속하면 자동으로 IP 주소를 할당받는데, 이는 **DHCP(Dynamic Host Configuration Protocol)** 덕분이다.

1.  **Discover:** "나 여기 새로 왔는데, IP 주소 좀 줄 사람?" (브로드캐스트)
2.  **Offer:** DHCP 서버가 "이 주소 어때?" 라고 제안.
3.  **Request:** "좋아, 그 주소로 할게!"
4.  **ACK:** DHCP 서버가 "알겠어, 이제 그 주소는 네 거야." 라고 확정.

DHCP는 IP 주소뿐만 아니라 서브넷 마스크, 기본 게이트웨이(첫 번째 홉 라우터) 주소, DNS 서버 주소 등 네트워크 접속에 필요한 모든 정보를 제공해주는 고마운 프로토콜이다.

## 5. 주소 고갈 문제와 해결사 NAT

IPv4 주소는 약 40억 개로, 이미 오래전에 모두 소진되었다. 😱 이 문제를 해결하고 IPv4의 생명을 연장해준 일등공신이 바로 **NAT(Network Address Translation)**, 우리에게는 **공유기**로 더 익숙한 기술이다.

NAT는 사설 네트워크(e.g., 우리 집)에서 사용하는 **사설 IP 주소**를 인터넷(공용 네트워크)에서 사용하는 **공인 IP 주소** 하나로 변환해주는 기술이다. 덕분에 하나의 공인 IP 주소만으로 여러 장치가 동시에 인터넷을 사용할 수 있게 된다.

## 6. ICMP: 네트워크의 상태를 알려주는 프로토콜

**ICMP(Internet Control Message Protocol)** 는 호스트와 라우터가 네트워크 상태에 대한 정보를 주고받기 위해 사용하는 프로토콜이다. 오류 리포팅(e.g., "목적지에 도달할 수 없음")이나 간단한 쿼리(e.g., "살아있니?")에 사용된다. 우리가 흔히 사용하는 `ping`이나 `traceroute` 같은 네트워크 진단 도구들이 바로 이 ICMP를 이용해 만들어졌다.

---

네트워크 계층은 우리가 매일 사용하는 인터넷의 근간을 이루는 매우 중요한 부분이다. 라우터 내부의 복잡한 동작부터 IP 주소 할당의 비밀까지, 보이지 않는 곳에서 수많은 기술들이 협력하여 안정적인 통신을 만들어내고 있다.
