---
layout: post
title: "Elasticsearch: NoSQL 기반의 검색 엔진"
date: 2025-08-06 00:00:00 +0900
categories: [ELK, Woori FISA]
tags: [elasticsearch, nosql, search engine, elk, '#우리FIS아카데미', '#우리FISA', '#AI엔지니어링', '#K-디지털트레이닝', '#우리에프아이에스', '#글로벌소프트웨어캠퍼스']
---

## 🔍 Elasticsearch: NoSQL 기반의 검색 엔진

오늘은 방대한 양의 데이터를 **빠르고 정교하게 검색**하고 **실시간으로 분석**할 수 있게 해주는 강력한 검색 엔진인 **Elasticsearch**의 기본 개념과 활용법을 학습했다. 특히 ELK(Elasticsearch, Logstash, Kibana) 스택의 핵심으로서 데이터 파이프라인 구축에 필수적인 기술이다.

> ### 💡 Elasticsearch, 왜 배워야 할까?
> Elasticsearch는 단순한 데이터베이스를 넘어, 로그 분석, 전문(Full-Text) 검색, 통계 분석 등 다양한 분야에서 활용된다. 특히 **역색인(Inverted Index)** 구조를 통해 기존 RDBMS가 제공하기 어려운 빠른 검색 속도와 정교한 분석 기능을 제공하여 현대적인 데이터 기반 애플리케이션의 핵심 요소로 자리 잡고 있다.

---

### 🚀 Elasticsearch의 주요 특징

| 특징 | 설명 |
| :--- | :--- |
| **오픈소스 검색 엔진** | Apache Lucene 기반으로 개발되었으며, 누구나 무료로 사용할 수 있다. |
| **전문 검색 (Full-Text Search)** | 여러 필드에 걸쳐 특정 단어가 포함된 문서를 빠르게 찾아낸다. |
| **NoSQL 데이터 저장소** | JSON 문서 형태로 데이터를 저장하는 Document 기반 NoSQL 데이터베이스의 특징을 가진다. |
| **역색인 (Inverted Index)** | 키워드를 통해 문서를 즉시 찾아내는 구조로, 검색 속도가 매우 빠르다. |
| **분산 환경 지원** | 데이터를 여러 노드에 분산 저장(샤딩)하고 복제본을 만들어 안정성과 확장성을 높인다. |
| **통계 분석** | Kibana와 연동하여 수집된 데이터를 시각화하고 복잡한 통계 분석을 수행할 수 있다. |

---

### 📊 관계형 데이터베이스(RDBMS)와의 비교

Elasticsearch의 개념은 RDBMS와 유사한 점이 많아 비교를 통해 쉽게 이해할 수 있다.

| **Elasticsearch** | **RDBMS (e.g., MySQL)** |
| :--- | :--- |
| **Index** | Database |
| **Type (7.x 이전)** | Table |
| **Document** | Row |
| **Field** | Column |
| **Mapping** | Schema |
| **Query DSL / ESQL** | SQL |

> Elasticsearch는 **JSON 형식의 RESTful API**를 통해 데이터를 조작하며, `GET`, `PUT`, `POST`, `DELETE` 등의 HTTP 메서드를 사용한다.

---

### 🛠️ 기본 API 사용법: CRUD와 검색

Elasticsearch는 REST API를 통해 문서(데이터)를 생성, 조회, 수정, 삭제한다.

#### 1. 문서 생성 (Create)
ID를 직접 지정하거나, ES가 자동으로 생성하도록 요청할 수 있다.
```json
// ID를 '1'로 직접 지정하여 문서 생성
PUT /my-index/_doc/1
{
  "title": "Hello World",
  "views": 1234
}
```

#### 2. 문서 조회 (Read)
`_id`를 사용하여 특정 문서를 조회한다.
```json
GET /my-index/_doc/1
```

#### 3. 문서 수정 (Update)
`_update` API를 사용하여 문서의 일부 필드를 수정한다.
```json
POST /my-index/_update/1
{
  "doc": {
    "title": "Hello Elasticsearch!"
  }
}
```

#### 4. 검색 (Search)
`_search` API와 Query DSL을 사용하여 정교한 검색을 수행한다.
```json
POST /my-index/_search
{
  "query": {
    "match": {
      "title": "hello"
    }
  }
}
```

---

### 🗺️ 매핑(Mapping): 데이터 스키마 정의

**매핑(Mapping)** 은 문서의 각 필드가 어떤 데이터 타입(`text`, `keyword`, `long`, `date` 등)으로 저장되고 색인될지를 정의하는 과정이다. RDBMS의 **스키마(Schema)** 와 유사한 역할을 한다.

- **동적 매핑 (Dynamic Mapping)**: 데이터가 처음 입력될 때 Elasticsearch가 자동으로 필드 타입을 추측하여 생성한다.
- **명시적 매핑 (Explicit Mapping)**: 사용자가 직접 인덱스를 생성하며 각 필드의 타입을 명확하게 지정한다. 데이터의 일관성과 검색 정확도를 위해 권장되는 방식이다.

```json
// 'my-index' 생성 시 명시적 매핑 정의
PUT /my-index
{
   "mappings": {
      "properties": {
        "title": { "type": "text" },
        "author": { "type": "keyword" },
        "views": { "type": "long" }
      }
   }
}
```

---

### ✨ 오늘의 회고

- Elasticsearch가 **Apache Lucene** 기반의 **오픈소스 검색 엔진**이자 **NoSQL 데이터베이스**임을 이해했다.
- **역색인(Inverted Index)** 구조 덕분에 **전문 검색(Full-Text Search)** 이 매우 빠르다는 것을 학습했다.
- **Index, Document, Field, Mapping** 등 Elasticsearch의 핵심 개념을 RDBMS와 비교하여 파악했다.
- **RESTful API**와 `PUT`, `POST`, `GET`, `DELETE` 메서드를 사용하여 데이터를 **CRUD(생성, 조회, 수정, 삭제)** 하는 방법을 익혔다.
- `_search` API와 **Query DSL**을 이용한 기본 검색 방법을 실습했다.
- 데이터의 타입을 정의하는 **매핑(Mapping)** 의 중요성과 동적/명시적 매핑의 차이를 이해했다.
