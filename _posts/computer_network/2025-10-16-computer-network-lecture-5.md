---
layout: post
title: "🚚 전송 계층: TCP는 어떻게 신뢰성과 효율성을 모두 잡았을까? 5️⃣"
date: 2025-10-16 21:00:00 +0900
categories: [컴퓨터 네트워크]
tags: [CS, Network, Transport Layer, TCP, Congestion Control, Flow Control, QUIC]
---

지난 포스트에서는 신뢰성 있는 데이터 전송(RDT)의 원리를 rdt3.0까지 살펴보며 Stop-and-Wait 방식의 한계를 확인했다. 패킷을 하나 보내고 ACK를 받을 때까지 기다리는 방식은 신뢰성은 보장하지만, 네트워크의 성능을 최대로 활용하지 못했다.

오늘은 이 문제를 해결하고, 인터넷 통신의 핵심으로 자리 잡은 **TCP(Transmission Control Protocol)** 에 대해 깊이 파고들어 본다. TCP는 어떻게 신뢰성과 효율성, 두 마리 토끼를 모두 잡았을까? 🐰

## 🚀 TCP의 특징: 파이프라이닝과 흐름 제어

TCP는 rdt3.0의 문제를 **파이프라이닝(Pipelining)** 기법으로 해결한다. ACK를 기다리지 않고 여러 개의 패킷을 연속으로 보내 채널 이용률을 극대화하는 것이다. 이와 함께 다음과 같은 핵심적인 특징들을 가진다.

- **Point-to-point:** 단일 송신자와 단일 수신자 간의 연결.
- **Reliable, in-order byte stream:** 데이터를 신뢰성 있게, 순서대로 전달한다.
- **Full duplex:** 한 연결 안에서 양방향 데이터 전송이 가능하다.
- **Connection-oriented:** 데이터를 보내기 전, **3-way handshake**를 통해 연결을 설정한다.
- **Flow controlled:** 수신자의 버퍼가 넘치지 않도록 송신자의 전송 속도를 조절한다.
- **Congestion controlled:** 네트워크의 혼잡 상황을 감지하고 전송 속도를 조절하여 네트워크 전체의 안정성을 유지한다.

## 📦 TCP 세그먼트 구조

TCP가 이 모든 기능을 수행할 수 있는 비밀은 바로 헤더에 있다.

![Untitled](https://raw.githubusercontent.com/HxWOO/HxWOO.github.io/master/assets/images/computer_network_5/Untitled.png)

- **Sequence Number (순서 번호):** 세그먼트에 포함된 데이터의 첫 번째 바이트에 부여되는 고유 번호. 이를 통해 데이터의 순서를 재조립하고 중복을 제거한다.
- **Acknowledgement Number (확인 응답 번호):** 수신자가 다음에 받기를 기대하는 바이트의 순서 번호. 예를 들어 199번까지 잘 받았다면, ACK 번호는 200이 된다. TCP는 **누적 ACK(cumulative ACK)**를 사용하므로, 200번 ACK는 199번까지의 모든 데이터를 문제없이 받았다는 의미를 내포한다.
- **Header Length:** TCP 헤더의 길이를 나타낸다. 옵션(Options) 필드의 길이가 가변적이라 이 필드가 필요하다.
- **Flags (U,A,P,R,S,F):** 연결 설정(SYN), 데이터 전송(PSH, URG), 연결 종료(FIN), 강제 종료(RST) 등 다양한 제어 정보를 담는다.
- **Receive Window (수신 윈도우):** 흐름 제어(Flow Control)를 위해 사용된다. 수신자가 현재 추가로 받을 수 있는 데이터의 양을 송신자에게 알려준다.
- **Checksum:** 데이터 오류 검출을 위한 값.
- **Options:** 타임스탬프, 윈도우 스케일 등 부가적인 정보를 담는다.

## ⏱️ Timeout 설정: 너무 길지도, 짧지도 않게

패킷 손실을 감지하기 위한 타임아웃 설정은 TCP 성능에 매우 중요하다.

- **너무 짧으면?** 패킷이 잘 전달되고 있는데도 손실로 오해하여 불필요한 재전송이 발생한다.
- **너무 길면?** 패킷이 실제로 손실되었을 때 너무 늦게 반응하여 성능이 저하된다.

문제는 네트워크 상태가 계속 변하기 때문에 RTT(Round Trip Time)가 계속 변한다는 점이다. TCP는 **SampleRTT**를 계속 측정하고, **지수 가중 이동 평균(EWMA)** 을 사용하여 변동성을 완화한 **EstimatedRTT**를 계산한다.

- `EstimatedRTT = (1-α) * EstimatedRTT + α * SampleRTT`

그리고 RTT의 변동폭(DevRTT)까지 고려하여 최종 타임아웃 간격(`TimeoutInterval`)을 동적으로 조절한다.

- `TimeoutInterval = EstimatedRTT + 4 * DevRTT`

## 🔄 TCP의 재전송 시나리오

TCP는 다음과 같은 상황에서 데이터를 재전송하여 신뢰성을 보장한다.

1.  **Timeout:** 가장 기본적인 재전송 매커니즘. ACK를 받기 전에 타이머가 만료되면 해당 세그먼트를 재전송한다.
2.  **Fast Retransmit (빠른 재전송):** 중간에 패킷 하나가 유실되면, 수신자는 유실된 패킷 다음의 패킷들을 받을 때마다 계속해서 동일한 ACK(유실된 패킷의 순서 번호를 가진)를 보낸다. 송신자가 이 **중복 ACK(duplicate ACK)를 3번** 받으면, 타임아웃이 발생하기 전이라도 '아, 패킷이 유실되었구나'라고 판단하고 해당 패킷을 즉시 재전송한다.

![Untitled](https://raw.githubusercontent.com/HxWOO/HxWOO.github.io/master/assets/images/computer_network_5/Untitled%205.png)

## 🌊 흐름 제어 (Flow Control)

송신자가 너무 빨리 보내서 수신자의 버퍼가 넘쳐버리면(overflow), 수신자는 데이터를 버릴 수밖에 없고 이는 엄청난 낭비다. **흐름 제어**는 이를 방지하기 위한 기능이다.

수신자는 TCP 헤더의 **Receive Window (rwnd)** 필드를 통해 자신의 버퍼에 남은 공간의 크기를 송신자에게 계속 알려준다. 송신자는 이 `rwnd` 크기를 초과하지 않는 범위 내에서만 데이터를 전송한다.

![Untitled](https://raw.githubusercontent.com/HxWOO/HxWOO.github.io/master/assets/images/computer_network_5/Untitled%207.png)

## 🤝 연결 관리 (Connection Management)

TCP는 데이터를 보내기 전에 반드시 연결을 설정하고, 데이터 전송이 끝나면 연결을 해제한다.

### 3-Way Handshake (연결 설정)

1.  **[SYN]** 클라이언트 → 서버: "연결 요청할게!" (SYN 비트 = 1, 임의의 순서 번호 `client_isn` 설정)
2.  **[SYN+ACK]** 서버 → 클라이언트: "알겠어! 너의 요청 잘 받았고, 나도 준비됐어!" (SYN 비트 = 1, ACK 비트 = 1, `ack = client_isn + 1`, 서버도 임의의 순서 번호 `server_isn` 설정)
3.  **[ACK]** 클라이언트 → 서버: "응답 고마워! 이제 데이터 보낼게!" (ACK 비트 = 1, `ack = server_isn + 1`)

이 과정을 통해 양쪽 모두 서로의 상태를 확인하고 안전하게 통신을 시작할 수 있다.

### 연결 종료 (Closing Connection)

연결 종료는 **4-Way Handshake**를 통해 이루어지며, 각 측이 `FIN` 플래그를 보내 자신의 데이터 전송이 끝났음을 알리고 상대방의 `ACK`를 받는다.

## 🚦 혼잡 제어 (Congestion Control)

흐름 제어가 송신자와 수신자 '둘'의 관계에 대한 것이라면, **혼잡 제어**는 네트워크 '전체'의 안정을 위한 것이다. 너무 많은 데이터가 한꺼번에 네트워크에 쏟아져 들어와 라우터의 큐가 넘치고 패킷이 손실되는 **혼잡 붕괴(congestion collapse)**를 막기 위한 중요한 기능이다.

TCP는 네트워크의 혼잡을 '패킷 손실'이나 '지연 증가'로 간접적으로 추론하고, **Congestion Window (cwnd)** 라는 값을 조절하여 전송 속도를 제어한다. 송신자가 한 번에 보낼 수 있는 데이터의 양은 `min(rwnd, cwnd)`에 의해 결정된다.

### AIMD (Additive Increase, Multiplicative Decrease)

TCP 혼잡 제어의 기본 전략이다.

- **Additive Increase (합 증가):** 혼잡이 감지되지 않으면, 1 RTT마다 `cwnd`를 1 MSS(Maximum Segment Size)씩 점진적으로 늘린다. (조심스럽게 속도를 올린다)
- **Multiplicative Decrease (곱 감소):** 패킷 손실(혼잡)이 감지되면, `cwnd`를 절반으로 확 줄인다. (위험 신호에 빠르게 반응한다)

### Slow Start & Congestion Avoidance

TCP는 연결 초기에 `cwnd`를 1 MSS에서 시작하여, 매 RTT마다 2배씩 지수적으로 빠르게 늘린다. 이를 **Slow Start**라고 한다. (이름은 느리지만 실제로는 가장 빠르다!)

그러다 `cwnd`가 미리 설정된 임계값(**ssthresh**)을 넘어서면, **Congestion Avoidance** 단계로 전환하여 `cwnd`를 1 MSS씩 선형적으로 증가시킨다(AIMD의 Additive Increase).

- **손실 감지 시:**
    - **Timeout으로 감지:** `ssthresh`를 `cwnd/2`로 설정하고, `cwnd`를 1 MSS로 리셋한 후 다시 Slow Start를 시작한다. (심각한 혼잡으로 간주)
    - **3 중복 ACK로 감지:** `ssthresh`와 `cwnd`를 모두 `cwnd/2`로 설정하고 Congestion Avoidance 단계부터 시작한다. (비교적 덜 심각한 혼잡으로 간주, TCP Reno 방식)

![Untitled](https://raw.githubusercontent.com/HxWOO/HxWOO.github.io/master/assets/images/computer_network_5/Untitled%2012.png)

### TCP CUBIC & BBR

AIMD 방식은 톱니 모양으로 `cwnd`가 변동하여 네트워크 대역폭을 완전히 활용하기 어렵다는 단점이 있다. 이를 개선하기 위해 현대적인 혼잡 제어 알고리즘들이 등장했다.

- **TCP CUBIC:** loss가 발생했던 지점(`W_max`)까지는 3차 함수 곡선 모양으로 빠르게 `cwnd`를 증가시키고, `W_max`에 가까워지면 증가 속도를 늦춰 안정성을 높인다. (Linux, Android의 기본 알고리즘)
- **BBR (Bottleneck Bandwidth and RTT):** loss 기반이 아닌, RTT와 대역폭 자체를 측정하여 파이프를 "거의 꽉 차지만 넘치지 않게" 유지하려는 알고리즘. (Google의 내부 네트워크 및 Youtube 등에서 사용)

## ✨ QUIC: TCP의 진화

지난 40년간 인터넷을 지배해온 TCP도 한계는 있었다. 대표적으로 **Head-of-Line Blocking** 문제(한 패킷이 유실되면 해당 스트림 전체가 막히는 현상)와 3-Way Handshake로 인한 연결 지연 등이 있다.

**QUIC(Quick UDP Internet Connections)** 는 이러한 문제를 해결하기 위해 등장한 새로운 전송 프로토콜이다.

- **UDP 기반:** TCP가 아닌 UDP 위에서 동작하여 커널 수정 없이 빠르게 배포 및 개선이 가능하다.
- **빠른 연결 설정:** 1-RTT만으로 암호화 및 전송 설정이 완료된다.
- **스트림 멀티플렉싱:** 여러 데이터 스트림을 하나의 연결로 처리하면서도, 한 스트림의 패킷 손실이 다른 스트림에 영향을 주지 않는다. (No Head-of-Line Blocking)
- **강화된 보안:** TLS 1.3을 기본으로 내장하여 강력한 암호화를 제공한다.

QUIC은 HTTP/3의 표준 전송 프로토콜로 채택되었으며, TCP의 시대에서 QUIC의 시대로 점차 넘어가고 있다. 전송 계층의 혁신은 현재진행형이다!
