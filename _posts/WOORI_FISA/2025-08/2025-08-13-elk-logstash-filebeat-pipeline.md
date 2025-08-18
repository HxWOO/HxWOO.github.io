---
layout: post
title:  "ELK 스택의 핵심! Logstash와 Filebeat로 로그 파이프라인 구축하기 🛠️"
date:   2025-08-13 10:00:00 +0900
categories: [AI Engineering, Woori FISA]
tags: [ELK, Logstash, Filebeat, Elasticsearch, 데이터파이프라인, 로그수집, '#우리FISA아카데미', '#우리FISA', '#AI엔지니어링', '#K-디지털트레이닝', '#우리에프아이에스', '#글로벌소프트웨어캠퍼스']
---

<br>

## 🚚 데이터 파이프라인의 핵심, Logstash와 Filebeat!

오늘은 ELK 스택의 심장과도 같은 **Logstash**와, 데이터 수집의 첨병 역할을 하는 **Filebeat**에 대해 깊이 있게 학습하고, 여러 서버의 로그를 한 곳으로 모으는 실습을 진행했다. 마치 전국 각지에서 온 택배를 중앙 물류센터에서 분류하고 처리하여 최종 목적지로 보내는 것처럼, Filebeat와 Logstash는 **흩어져 있는 로그 데이터를 수집하고 가공하여 Elasticsearch라는 거대한 창고에 저장**하는 멋진 역할을 수행한다. 📦

> ### 💡 "모든 로그는 이야깃거리를 담고 있다. Logstash는 그 이야기의 번역가다."
> 비정형의 로그 데이터를 정제하고 구조화하여 의미 있는 정보로 만드는 Logstash의 역할은 중앙 집중식 로그 시스템 구축의 핵심이다.

---

### 1. Logstash: 데이터의 변환을 지휘하는 마에스트로 🎻

- **정의**: 다양한 소스에서 로그 또는 이벤트 데이터를 수집, 처리 및 변환하여 다른 시스템으로 전송하는 오픈 소스 **데이터 처리 파이프라인** 도구.
- **파이프라인 3단계**: `Input` → `Filter` → `Output`
    - **Input (입력)**: 데이터가 들어오는 관문. (`beats`, `file`, `jdbc` 등)
    - **Filter (필터)**: 데이터를 정제하고 구조화하는 작업장. (`grok`, `mutate`, `json` 등)
    - **Output (출력)**: 처리된 데이터의 최종 목적지. (`elasticsearch`, `stdout` 등)

| 구분 | **Filebeat (파일비트)** | **Logstash (로그스태시)** |
| :--- | :--- | :--- |
| **역할** | 경량 로그 수집 및 전달 (Agent) | 데이터 수집, 변환, 보강 (Pipeline) |
| **리소스** | 매우 적게 사용 (경량) | 상대적으로 많이 사용 (다양한 기능) |
| **주요 기능** | 로그 파일 감시, 전달 | 데이터 파싱, 필터링, 라우팅 |

---

### 2. Filebeat: 현장에서 발로 뛰는 데이터 수집가 🏃‍♂️

- **정의**: 서버에 에이전트로 설치되어 로그 파일을 수집하고, 이를 Logstash나 Elasticsearch로 전송하는 **경량 로그 수집기**.
- **주요 특징**:
    - **경량성**: 시스템 리소스를 적게 소모하여 서버 부담을 최소화.
    - **안정성**: 네트워크 문제 발생 시 데이터 전송을 보장 (At-Least-Once).
    - **모듈**: Apache, Nginx 등 일반적인 로그를 쉽게 수집하는 모듈 제공.

---

### 3. 실습: 두 개의 서버 로그를 Logstash로 통합하기 🔬

이번 실습에서는 FastAPI로 만든 입출금 서버와 Flask로 만든 로그인 서버, 두 곳에서 발생하는 로그를 각각 다른 Logstash 파이프라인으로 수집하고 처리하는 과정을 진행했다.

#### 🏦 Server 1: 입금/출금 서버 (FastAPI)

FastAPI 서버를 실행하고, `Filebeat`가 `server1.log`를 수집하여 `5045`번 포트로 Logstash에 전송한다.

- **Logstash 파이프라인 (`logstash-pipeline-server1.conf`)**
```conf
input { beats { port => 5045 } }

filter {
  grok {
    match => { "message" => "%{LOGLEVEL:log_level}:\s+%{IP:client_ip}:%{NUMBER:client_port} - \"%{WORD:http_method} %{URIPATH:request_path} HTTP/%{NUMBER:http_version}\" %{NUMBER:response_code} %{WORD:status_message}" }
  }
}

output {
  elasticsearch {
    hosts => ["http://localhost:9200"]
    index => "server1-%{+YYYY.MM.dd}"
  }
}
```

#### 💻 Server 2: 로그인/로그아웃 서버 (Flask)

Flask 서버를 실행하고, `Filebeat`가 `server2.log`를 수집하여 `5044`번 포트로 Logstash에 전송한다.

- **Logstash 파이프라인 (`logstash-pipeline-server2.conf`)**
```conf
input { beats { port => 5044 } }

filter {
  grok {
    match => { "message" => "%{LOGLEVEL:log_level}:\s+%{IP:client_ip}:%{NUMBER:client_port} - \"%{WORD:http_method} %{URIPATH:request_path} HTTP/%{NUMBER:http_version}\" %{NUMBER:response_code} %{GREEDYDATA:status_message}" }
  }
}

output {
  elasticsearch {
    hosts => ["http://localhost:9200"]
    index => "server2-%{+YYYY.MM.dd}"
  }
}
```

#### 🛠️ 다중 파이프라인 관리 (`pipelines.yml`)

Logstash가 위 두 파이프라인을 동시에 실행하도록 `pipelines.yml`에 각 설정 파일의 경로를 지정해주었다.

```yaml
- pipeline.id: server1
  path.config: "C:/devs/ITStudy/05_elk/logstash/config/logstash-pipeline-server1.conf"

- pipeline.id: server2
  path.config: "C:/devs/ITStudy/05_elk/logstash/config/logstash-pipeline-server2.conf"
```

> 🎉 **실습 성공!** 이 구성을 통해 각기 다른 종류의 로그를 별도의 파이프라인에서 독립적으로 처리할 수 있게 되었다. 이 과정에서 Grok 패턴의 강력함을 알 수 있었다.

---

### ✨ 오늘의 회고

**Logstash와 Filebeat**의 관계는 마치 '분업과 협업'의 중요성을 보여주는 것 같았다. Filebeat가 현장에서 가볍고 빠르게 데이터를 모아주면, Logstash는 중앙에서 복잡한 데이터를 처리하며 각자의 역할에 집중하는 모습이 인상 깊었다. 특히 **다중 파이프라인** 설정을 통해 서로 다른 형식의 로그를 동시에 처리하는 실습은 정말 재미있었다. 

오늘의 학습을 통해 흩어져 있던 로그들이 어떻게 중앙에서 관리되고 분석될 수 있는지 그 전체적인 흐름을 이해할 수 있었다. 다음에는 Logstash의 더 다양한 필터 플러그인을 사용해서 데이터를 더욱 유용하게 만들어보는 연습을 해야겠다. 😄
