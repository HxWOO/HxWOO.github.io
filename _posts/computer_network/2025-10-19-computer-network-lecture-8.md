---
layout: post
title: "🌐 네트워크의 길잡이, 라우팅: 최적의 경로는 어떻게 찾을까? 8️⃣"
date: 2025-10-19 20:00:00 +0900
categories: [컴퓨터 네트워크]
tags: [CS, Network, Routing, Link State, Distance Vector, OSPF, BGP, SDN, ICMP]
---

# 🗺️ 라우팅: 최적의 경로를 찾아서

네트워크의 핵심은 바로 **라우팅(Routing)** 이다. 출발지에서 목적지까지 데이터를 어떤 경로로 보낼지 결정하는 과정으로, 컨트롤 플레인(Control Plane)에서 일어난다. 라우팅 알고리즘은 '좋은' 경로, 즉 비용이 적거나, 가장 빠르거나, 덜 혼잡한 경로를 찾는 것을 목표로 한다.

라우팅 경로를 결정하는 컨트롤 플레인을 구성하는 방식은 크게 두 가지로 나뉜다.

- **라우터별 컨트롤 (Per-router control):** 각 라우터가 서로 정보를 교환하며 **독자적으로** 라우팅 테이블을 계산한다. 분산 방식이라 유연하지만, 라우터 간 정보 동기화가 완벽하지 않을 수 있다. 가장 널리 쓰이는 방식!
- **중앙 집중형 컨트롤 (Logically centralized control):** **SDN(Software-Defined Networking)** 컨트롤러라는 중앙 지휘소에서 모든 라우터의 테이블을 계산하고 배포한다. 일관성이 보장된다는 큰 장점이 있다.

![Untitled](https://raw.githubusercontent.com/HxWOO/HxWOO.github.io/master/assets/images/computer_network_8/Untitled.png)

## 1. 라우팅 알고리즘의 양대 산맥: LS vs DV

라우팅 알고리즘은 크게 **링크 상태(Link State, LS)** 와 **거리 벡터(Distance Vector, DV)** 방식으로 나뉜다.

### Link-State (LS) 알고리즘

- **글로벌 정보:** 모든 라우터가 전체 네트워크의 토폴로지(지도)를 다 알고 있다.
- **알고리즘:** 각 라우터는 이 전체 지도를 바탕으로 **다익스트라(Dijkstra) 알고리즘**을 실행해 자기 자신으로부터 다른 모든 노드까지의 최단 경로를 계산한다.
- **대표 주자:** **OSPF**

LS 방식은 모든 라우터가 완전한 정보를 가지고 계산하므로 비교적 안정적이지만, 링크 비용이 트래픽 양에 따라 계속 변하면 경로가 계속 바뀌는 **진동(oscillation) 문제**가 발생할 수 있다.

![Untitled](https://raw.githubusercontent.com/HxWOO/HxWOO.github.io/master/assets/images/computer_network_8/Untitled%202.png)

### Distance-Vector (DV) 알고리즘

- **탈중앙화 정보:** 각 라우터는 자신과 직접 연결된 이웃 라우터하고만 정보를 교환한다.
- **"이웃 말만 믿기":** 라우터는 이웃이 알려주는 "목적지까지 이만큼 걸려"라는 거리 벡터(distance vector) 정보를 전적으로 신뢰하고, 이를 바탕으로 자신의 라우팅 테이블을 갱신한다.
- **알고리즘:** **벨만-포드(Bellman-Ford)** 알고리즘에 기반한다.
- **대표 주자:** **RIP**, EIGRP

DV 방식은 구조가 간단하고 분산적이지만, "나쁜 소식"이 느리게 전파되는 **Count-to-Infinity** 문제에 취약하다. 예를 들어, 특정 경로의 비용이 급증했을 때, 라우터들이 서로 잘못된 정보를 계속 주고받으며 최적 경로를 찾는 데 아주 오랜 시간이 걸릴 수 있다. 😭

## 2. 인터넷의 라우팅: AS의 등장

실제 인터넷은 수십억 개의 장치가 연결된 거대한 네트워크라, 위에서 설명한 방식 그대로를 적용하기 어렵다. 그래서 인터넷은 **자율 시스템(Autonomous System, AS)** 이라는 단위로 묶어서 계층적으로 관리한다.

- **AS (Autonomous System):** 하나의 관리 주체가 운영하는 라우터들의 집합. (하나의 '도메인'처럼 생각할 수 있다)
- **내부 라우팅 (Intra-AS Routing):** AS **내에서** 경로를 결정하는 것. (e.g., OSPF, RIP)
- **외부 라우팅 (Inter-AS Routing):** AS **간의** 경로를 결정하는 것. (e.g., BGP)

![Untitled](https://raw.githubusercontent.com/HxWOO/HxWOO.github.io/master/assets/images/computer_network_8/Untitled%205.png)

### OSPF: 가장 널리 쓰이는 내부 라우팅 프로토콜

**OSPF(Open Shortest Path First)** 는 현재 가장 널리 사용되는 **LS 기반**의 내부 라우팅 프로토콜이다.

- **Open:** 공공재로 개발되어 누구나 사용할 수 있다.
- **동작 방식:** 각 라우터가 자신의 링크 상태(LS) 정보를 AS 내 모든 라우터에게 알리고(flooding), 전체 토폴로지 맵을 만든 뒤 다익스트라 알고리즘으로 최단 경로를 계산한다.
- **계층 구조:** OSPF는 AS를 여러 **area**로 나누어 관리한다. 이를 통해 LS 정보가 넘쳐나는 것을 막고(flooding 범위를 area로 제한), 라우팅 계산의 부담을 줄여 성능을 향상시킨다. 👍

### BGP: AS들을 연결하는 외부 라우팅 프로토콜

**BGP(Border Gateway Protocol)** 는 AS와 AS를 연결하는 인터넷의 **사실상 표준** 외부 라우팅 프로토콜이다.

- **eBGP & iBGP:** 이웃 AS로부터 경로 정보를 받는 **eBGP**와, 그 정보를 AS 내부 라우터들에게 전파하는 **iBGP**로 나뉜다.
- **경로 벡터 (Path Vector):** BGP는 단순히 비용만 따지는 게 아니라, 목적지까지 거쳐가야 할 **AS들의 목록(AS-PATH)** 을 기반으로 경로를 결정한다.
- **정책 기반 라우팅:** BGP의 가장 큰 특징은 **정책(policy)** 이 성능보다 우선될 수 있다는 점이다. 예를 들어, "A라는 회사의 AS는 절대 거치지 않겠다"와 같은 정책을 설정하여 라우팅 경로를 제어할 수 있다.

![Untitled](https://raw.githubusercontent.com/HxWOO/HxWOO.github.io/master/assets/images/computer_network_8/Untitled%207.png)

## 3. SDN: 네트워크를 프로그래밍하다

**SDN(Software-Defined Networking)** 은 네트워크 제어 방식을 혁신하는 새로운 패러다임이다. 기존에는 각 라우터가 알아서 경로를 계산했지만, SDN에서는 **중앙 컨트롤러**가 네트워크 전체를 지휘한다.

- **Control/Data Plane 분리:** 라우터는 데이터 전달(Data Plane)만 담당하는 단순한 스위치가 되고, 라우팅 계산 등 지능적인 역할(Control Plane)은 SDN 컨트롤러가 맡는다.
- **중앙 집중 제어:** 컨트롤러가 전체 네트워크 상황을 보고 각 스위치의 동작 규칙(flow table)을 한 번에 결정하고 내려보낸다.
- **프로그래머블:** 개발자는 컨트롤러의 API를 통해 라우팅, 부하 분산, 접근 제어 등 네트워크 동작을 마음대로 프로그래밍할 수 있게 된다! 💻

![Untitled](https://raw.githubusercontent.com/HxWOO/HxWOO.github.io/master/assets/images/computer_network_8/Untitled%209.png)

### OpenFlow: SDN의 핵심 프로토콜

**OpenFlow**는 SDN 컨트롤러와 스위치(라우터)가 서로 통신하는 표준 방법을 정의한 프로토콜이다. 컨트롤러는 OpenFlow를 통해 스위치의 Flow Table을 수정하여 패킷을 어떻게 처리할지(전달, 폐기, 수정 등) 지시한다.

## 4. ICMP: 네트워크의 조용한 조력자

**ICMP(Internet Control Message Protocol)** 는 네트워크 장치들이 서로 오류 메시지나 제어 메시지를 주고받기 위해 사용하는 프로토콜이다. IP 계층 바로 위에서 동작하며, IP 데이터그램에 담겨 전달된다.

- **오류 보고:** "목적지에 도달할 수 없음", "TTL 만료" 등 라우팅 과정에서 발생하는 문제를 알려준다.
- **네트워크 진단:** `ping` 명령어가 ICMP를 사용하여 특정 호스트가 살아있는지 확인하고, `Traceroute`는 ICMP의 TTL 만료 메시지를 이용해 목적지까지의 경로를 추적한다.

라우팅의 세계는 정말 복잡하지만, 덕분에 우리가 매일같이 인터넷을 안정적으로 사용할 수 있다는 사실이 새삼 놀랍다.
