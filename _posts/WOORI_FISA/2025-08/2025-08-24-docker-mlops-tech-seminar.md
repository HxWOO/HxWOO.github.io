---
layout: post
title:  "🛠️ 나의 첫 MLOps 파이프라인 구축기 (feat. Docker)"
date:   2025-08-24 10:00:00 +0900
categories: [WOORI FISA, Project]
tags: [Docker, MLOps, MLflow, FastAPI, Streamlit, MSA, '#우리FIS아카데미', '#우리FISA', '#AI엔지니어링', '#K-디지털트레이닝', '#우리에프아이에스', '#글로벌소프트웨어캠퍼스']
---

## MLOps, Docker로 첫발을 내딛다

기술 세미나 주제는 **"Docker로 배우는 MLOps"**. 이론 공부를 넘어, 실제 운영 환경을 내 손으로 구축해보는 시간이었다. 모델만 냅다 만드는 게 아니라, 데이터 수집부터 배포, 모니터링까지 전체 파이프라인을 설계하는 아주 특별한 경험이었다. 과연 성공했을까? 🤔

> ### 🔥 우리 팀의 목표
> - **코드 기반 인프라 (IaC)**: `docker-compose.yml` 하나로 모든 환경을 통일한다. "제 컴퓨터에선 됐는데요?"는 이제 그만!
> - **MLOps 파이프라인 정복**: 데이터 수집-학습-배포-모니터링의 흐름을 직접 만들고 이해한다.
> - **MSA 아키텍처 체험**: 각 기능을 독립 서비스로 쪼개 유연한 시스템을 설계해본다.

---

### 🧑‍💻 시스템 아키텍처: 우리는 이렇게 만들었다

프로젝트의 심장, MSA 기반 아키텍처다. 각자 맡은 서비스가 `docker-compose` 안에서 유기적으로 돌아가도록 설계했다.

![MLOps Diagram](https://raw.githubusercontent.com/Fisa05-Docker-MLOps/tech-seminar/main/mlops_diagram.png)

| 서비스 | 역할 | 핵심 기술
| :--- | :--- | :--- 
| **MLflow Server** | 모델 실험/버전 관리, 아티팩트 저장 | `MLflow`, `minio`
| **PostgreSQL DB** | 데이터 저장, MLflow 백엔드 | `mySQL`
| **DB Ingestion** | 외부 데이터 수집 및 DB 적재 | `Python`
| **Inference Server** | 예측 API 제공 | `FastAPI`
| **Visualizer Server** | 예측 결과 시각화 대시보드 | `Streamlit`

나는 이 중에서 **Visualizer Server**를 맡았다. 사용자가 모델의 예측 결과를 눈으로 보고, 직접 피드백까지 남길 수 있는 창구를 만드는 일. 데이터와 사람이 만나는 접점을 만든다는 게 참 매력적이었다.

---

### 🛠️ 프로젝트 연장 모음

- **컨테이너**: `Docker`, `Docker-Compose`
- **ML 파이프라인**: `MLflow`, `minio`
- **백엔드**: `FastAPI`
- **데이터베이스**: `mySQL`
- **프론트엔드**: `Streamlit`

이 도구들을 엮어 일관성 있고 재현 가능한 MLOps 환경을 만드는 것이 핵심 과제였다.

---

### ✨ 파이프라인은 이렇게 흐른다: 각 서버의 역할

우리 파이프라인은 각자 명확한 역할을 가진 서비스들의 협력으로 이루어진다.

1.  **데이터 수집 (`db-ingestion-server`)**: 모든 것의 시작. Kaggle API로 원본 데이터를 내려받아 간단한 전처리를 거친 후, `PostgreSQL` 데이터베이스에 안정적으로 저장하는 임무를 수행한다. 파이프라인에 신선한 피를 공급하는 역할!

2.  **모델 학습 및 실험 관리 (`mlflow-client` & `mlflow-server`)**: `mlflow-client`가 `PostgreSQL`에서 데이터를 꺼내 모델을 학습시킨다. 이 과정에서 사용된 모든 파라미터, 성능 메트릭, 그리고 학습된 모델 파일(아티팩트)까지 `mlflow-server`에 꼼꼼하게 기록한다. 여기서 가장 성능 좋은 모델을 선별해 MLflow Model Registry에 'Production' 버전으로 딱지를 붙여 등록한다. 👑

3.  **실시간 추론 API (`inference-server`)**: `FastAPI`로 만들어진 이 서버는 Model Registry를 계속 주시하다가, 'Production' 딱지가 붙은 최신 모델을 자동으로 가져온다. 그리고 `/predict`라는 API 엔드포인트를 통해 모델의 예측 기능을 외부 세계에 제공한다.

4.  **사용자 대시보드 (`visualizer-server`)**: `Streamlit`으로 구현된 사용자 인터페이스. 사용자가 대시보드에서 "예측" 버튼을 누르면, `visualizer-server`는 `inference-server`의 `/predict` API로 요청을 보낸다. 그리고 응답으로 받은 예측 결과를 사용자에게 친절한 그래프와 함께 보여주는 임무를 맡았다. 모델과 사용자가 만나는 최전선!

> #### 🎉 성공의 순간!
> `docker-compose up`... 수많은 서비스가 터미널 위로 올라오며 각자의 역할을 수행하기 시작했다. 그리고 마침내, 내가 만든 Streamlit 대시보드에 FastAPI 서버가 예측한 결과가 딱! 하고 떴을 때. 그 짜릿함은 정말 잊을 수가 없다. 이게 바로 MLOps구나! 온몸으로 느낀 순간이었다.

---

### 회고: 무엇을 얻었나

이번 프로젝트는 단순한 코딩 그 이상이었다. **시스템 전체를 보는 엔지니어의 시야**를 얻었다고 할까.

- **큰 그림**: 각 서비스가 API로 통신하며 하나의 파이프라인을 이루는 전체 구조를 이해하게 됐다.
- **자동화의 힘**: MLOps가 왜 필요한지, 반복 작업을 줄이는 것이 얼마나 중요한지 뼈저리게 느꼈다.
- **진짜 협업**: MSA 환경에서 각자 서비스에 책임을 다하고, API 명세를 맞추며 함께 결과물을 만드는 즐거움을 알게 됐다.

물론 아직 갈 길은 멀다. CI/CD도 도입해야 하고, 모델 재학습 파이프라인도 자동화해야 한다. 하지만 이번 경험은 앞으로 더 복잡한 시스템에 도전할 수 있다는 강력한 자신감을 심어주었다. 🔥

프로젝트의 더 자세한 내용은 [우리 팀 GitHub](https://github.com/Fisa05-Docker-MLOps/tech-seminar)에서 확인할 수 있다. 끝!
