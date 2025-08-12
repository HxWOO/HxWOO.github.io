---
layout: post
title:  "Elasticsearch 집계(Aggregation) 완전 정복! 📊"
date:   2025-08-11 00:00:00 +0900
categories: [AI Engineering, Woori FISA]
tags: [Elasticsearch, Aggregation, ELK, 데이터분석, '#우리FIS아카데미', '#우리FISA', '#AI엔지니어링', '#K-디지털트레이닝', '#우리에프아이에스', '#글로벌소프트웨어캠퍼스']
---

<br>

## 🚀 Elasticsearch 집계(Aggregation) 완전 정복의 여정!

 🗺️ 오늘은 방대한 데이터를 요약하고 분석하는 데 필수적인 Elasticsearch의 다양한 **집계 기능**들을 깊이 있게 탐구하고 실습한 내용을 정리해 보았다. 복잡한 퍼즐 조각들을 맞춰 하나의 아름다운 그림을 완성하는 것처럼, 집계는 **데이터 속에 숨겨진 패턴과 인사이트**를 발견하게 해준다✨

> ### 💡 "데이터는 그 자체로 의미를 가지지 않는다. 가공을 통해 비로소 가치 있는 정보가 된다!"
> 단순히 데이터를 저장하는 것을 넘어, 집계는 데이터에 생명을 불어넣어 비즈니스 의사결정에 필요한 핵심 정보를 제공한다

---

### 1. 메트릭 집계 (Metric Aggregations) 📈

메트릭 집계는 문서 집합에서 단일 값을 계산하는 데 사용된다. 데이터의 핵심적인 통계치를 빠르게 파악할 때 유용하다.

#### 1.1. 합계 (Sum) ➕

특정 필드의 값들의 총합을 계산

```json
GET kibana_sample_data_ecommerce/_search
{
  "size" : 0, // 조회 결과는 보지 않고
  "query": { // 검색하고자 하는 쿼리를 먼저 작성
    "match": {
      "currency" : "EUR"
    }
  },  // 1. 조건에 맞는 DOC를 검색
  "aggs": { // 집계 결과만 출력
    "my-sum-aggregation": {
      "sum": {
        "field": "taxful_total_price"
      }  // 2. 조건에 맞는 DOC 대상으로 한 집계만
    }
  }
}
```

#### 1.2. 최소/최대 (Min/Max) ↔️

특정 필드의 최소값 또는 최대값을 계산

```json
GET kibana_sample_data_ecommerce/_search
{
  "size" : 0,
  "query": {
    "match": {
      "currency" : "EUR"
    }
  },
  "aggs": {
    "my-min-aggregation": {
      "min": {
        "field": "taxful_total_price"
      }
    },
    "my-max-aggregation": {
      "max": {
        "field": "taxful_total_price"
      }
    }
  }
}
```

#### 1.3. 통계 (Stats / Extended Stats) 📊

`stats`는 `count`, `min`, `max`, `avg`, `sum`을 포함한 기본 통계치를 제공함. `extended_stats`는 여기에 분산, 표준편차 등 더 자세한 통계치를 추가로 제공한다.

**예시: 아프리카 지역 매출 통계**
`geoip` 필드 안의 `continent_name` 필드가 'Africa'인 문서를 대상으로 `taxful_total_price`의 통계를 계산한다.

```json
POST kibana_sample_data_ecommerce/_search
{
  "size": 0,
  "query": {
    "match": {
      "geoip.continent_name": "Africa"
    }
  },
  "aggs": {
    "아프리카매출": {
      "stats": { // extended_stats를 사용하면 기본 통계치 모두 조회 가능 (분산, 표준편차 등)
        "field": "taxful_total_price"
      }
    }
  }
}
```

#### 1.4. 고유값 개수 (Cardinality) 🔢

특정 필드의 고유한 값의 개수를 어림잡아 계산한다. 대용량 데이터 처리 시 메모리를 절약하기 위해 근사치를 사용한다.

*   **`precision_threshold`**: 이 값을 전체 문서 개수만큼 주면 정확한 고유값 개수를 보장받을 수 있다.

```json
POST kibana_sample_data_ecommerce/_search
{
  "size": 0,
  "aggs": {
    "고객수총합": {
      "cardinality": {
        "field" : "customer_id",
        "precision_threshold": "4675" // 전체 문서 개수만큼 주면 정확한 고유값 개수 보장
      }
    }
  }
}
```

#### 1.5. 백분위수 (Percentiles) 💯

데이터 분포에서 특정 백분위수에 해당하는 값을 계산

```json
POST kibana_sample_data_ecommerce/_search
{
  "size": 0,
  "aggs": {
    "구매금액백분율": {
      "percentiles": {
        "field" : "taxful_total_price"
      }
    }
  }
}
```

#### 1.6. 백분위 순위 (Percentile Ranks) 🏆

주어진 값들이 데이터 분포에서 몇 번째 백분위수에 해당하는지 계산

```json
POST kibana_sample_data_ecommerce/_search
{
  "size": 0,
  "aggs": {
    "구매금액백분율": {
      "percentile_ranks": {
        "field" : "taxful_total_price",
        "values": [65, 100, 24]
      }
    }
  }
}
```

---

### 2. 버킷 집계 (Bucket Aggregations) 🧺

버킷 집계는 문서를 특정 기준에 따라 그룹화(버킷)하고, 각 버킷에 속하는 문서의 개수를 계산하거나 하위 집계를 수행한다. 데이터의 분포를 시각적으로 이해하는 데 큰 도움이 된다.

#### 2.1. 범위 집계 (Range Aggregation) 📏

지정된 범위(`from`, `to`)에 따라 문서를 그룹화. `from`은 포함(`>=`), `to`는 미포함(`<`)

```json
GET kibana_sample_data_flights/_search
{
  "size": 0,
  "aggs": {
    "거리별 비행구간": {
      "range": { // from-to로 특정 구간을 작성
        "field": "DistanceKilometers",
        "ranges": [
          {"to" : 5000 }, // 첫 구간이라 from 생략 (x < 5000)
          {"from": 5000, // 5000 <= x < 10000
          "to": 10000},
          {"from": 10000,
           "to": 15000},
          {"from":15000} // 마지막 구간이라 to 생략 (x >= 15000)
        ]
      },
      "aggs" : {
        "평균 비행기 티켓 가격-거리별비행구간 기준": {
          "avg" : {
            "field": "AvgTicketPrice"
          }
        },
        "총합 비행기 티켓 가격-거리별비행구간 기준": {
          "sum" : {
            "field": "AvgTicketPrice"
          }
        }
      }
    }
  }
}
```

#### 2.2. 히스토그램 집계 (Histogram Aggregation) 📊

고정된 간격(`interval`)으로 숫자 필드를 그룹화. `offset`을 사용하여 구간 시작점을 조정할 수 있다.

```json
GET kibana_sample_data_flights/_search
{
  "size": 0,
  "aggs": {
    "거리별 비행구간": {
      "histogram": { // interval로 간격을 설정
        "field": "DistanceKilometers",
        "interval": 5000,
        "offset" : -50 // 0으로 떨어지지 않는 음수 구간 설정 시 사용
      },
      "aggs" : {
        "평균 비행기 티켓 가격-거리별비행구간 기준": {
          "avg" : {
            "field": "AvgTicketPrice"
          }
        },
        "총합 비행기 티켓 가격-거리별비행구간 기준": {
          "sum" : {
            "field": "AvgTicketPrice"
          }
        }
      }
    }
  }
}
```

#### 2.3. 날짜 히스토그램 집계 (Date Histogram Aggregation) 📅

시간 기반 필드를 고정된 시간 간격(`calendar_interval`)으로 그룹화. `1m` (분), `1h` (시간), `1d` (일), `1M` (월), `1q` (분기), `1y` (년), `1w` (주) 등의 단위를 사용할 수 있다.

**예시: 일일 평균 비행기 티켓 금액**

```json
GET kibana_sample_data_flights/_search
{
  "size": 0,
  "aggs": {
    "일일 평균 비행기 티켓": {
      "date_histogram": {
        "field": "timestamp",
        "calendar_interval": "1w" // 1주 간격으로 그룹화
        // ,"format": "yyyy-MM-dd" // 결과 포맷 지정 가능
      },
      "aggs": {
        "일일 평균 비행기 티켓 금액": {
          "avg": {
            "field": "AvgTicketPrice"
          }
        }
      }
    }
  }
}
```

#### 2.4. 용어 집계 (Terms Aggregation) 🏷️

`keyword` 타입 필드의 고유한 값들을 기준으로 문서를 그룹화하고, 각 그룹의 문서 개수를 계산. `size`는 버킷을 최대 몇 개까지 생성할 것인지를 지정

```json
GET kibana_sample_data_logs/_search
{
  "size": 0,
  "query": {
    "match_all": {}
  },
  "aggs": {
    "my-terms-aggs": {
      "terms": { // keyword 자료형에 사용
        "field": "host.keyword",
        "size": 10 // 상위 10개 호스트만 표시
      }
    }
  }
}
```

---

### 3. 중첩 집계 (Nested Aggregations) 🧩

버킷 집계 내부에 메트릭 집계나 다른 버킷 집계를 포함하여 더 복잡한 분석을 수행할 수 있다. 레고 블록을 쌓듯이, 여러 집계를 조합하여 원하는 인사이트를 얻을 수 있다.

**예시: 지역별 매출 건수 및 총 매출/평균 매출**

`geoip.continent_name`으로 지역별 버킷을 생성하고, 각 지역 버킷 내에서 `taxful_total_price`의 총합과 평균을 계산

```json
GET kibana_sample_data_ecommerce/_search
{
  "size": 0,
  "aggs": {
    "지역별매출건수": {
      "terms": {
        "field": "geoip.continent_name"
      },
      "aggs":{
        "지역별 총계" : {
          "sum": {
            "field": "taxful_total_price"
          }
        },
        "지역별 평균" : {
          "avg": {
            "field": "taxful_total_price"
          }
        }
      }
    }
  }
}
```

#### 3.1. 복합 집계 (Composite Aggregation) 🔗

여러 필드를 조합하여 복합적인 버킷을 생성하고, 페이지네이션을 지원한다. `after` 파라미터를 사용하여 다음 페이지의 결과를 가져올 수 있다. 대용량 데이터의 연속적인 탐색에 유용

```json
GET kibana_sample_data_logs/_search
{
  "size": 0,
  "query": {
    "match_all": {}
  },
  "aggs": {
    "composite-aggs": {
      "composite": {
        "size": 100,
        "sources": [
          {
            "terms-aggs": {
              "terms": {
                "field": "host.keyword"
              }
            }
          },
          {
            "date-histogram-aggs": {
              "date_histogram": {
                "field": "@timestamp",
                "calendar_interval": "day"
                // ,"format" :"yyyy-MM-dd"
              }
            }
          }
        ]
      }
    }
  }
}
```

**다음 페이지 가져오기 (`after` 사용)**

```json
GET kibana_sample_data_logs/_search
{
  "size": 0,
  "query": {
    "match_all": {}
  },
  "aggs": {
    "composite-aggs": {
      "composite": {
        "size": 100,
          "sources": [
          {
            "terms-aggs": {
              "terms": {
                "field": "host.keyword"
              }
            }
          },
          {
            "date-histogram-aggs": {
              "date_histogram": {
                "field": "@timestamp",
                "calendar_interval": "day"
              }
            }
          }
        ],
        "after": { // 이전 응답의 마지막 버킷 키를 사용하여 다음 페이지 요청
          "terms-aggs": "www.elastic.co",
          "date-histogram-aggs": 1758758400000
        }
      }
    }
  }
}
```

---

### 4. 파이프라인 집계 (Pipeline Aggregations) 🚀

파이프라인 집계는 다른 집계의 결과에 대해 추가적인 집계를 수행한다. 이미 계산된 집계 결과를 바탕으로 더 심층적인 분석을 가능하게 함

#### 4.1. 누적 합계 (Cumulative Sum) 📈

이전 버킷의 값을 포함하여 현재 버킷까지의 누적 합계를 계산한다. `buckets_path`를 사용하여 어떤 메트릭 집계의 결과를 누적할지 지정, 시계열 데이터 분석에서 트렌드를 파악하는 데 매우 유용

**예시: 일자별 매출 및 총 누적 매출**

```json
GET kibana_sample_data_ecommerce/_search
{
  "size": 0,
  "query": {
    "match_all": {}
  },
  "aggs": {
    "일자별매출": {
      "date_histogram": {
        "field": "order_date",
        "calendar_interval": "day"
      },
      "aggs": {
        "매출": {
          "sum": {
            "field": "taxful_total_price"
          }
        },
        "총매출": {
          "cumulative_sum": { // 누적합
            "buckets_path": "매출" // "매출"이라는 이름의 메트릭 집계 결과를 사용
          }
        }
      }
    }
  }
}
```

#### 4.2. 버킷 최대/최소 (Max/Min Bucket) 🎯

버킷 집계 결과 중 특정 메트릭의 최대값 또는 최소값을 가진 버킷을 찾는다. 특정 기간 동안 가장 성과가 좋았던/나빴던 시점을 찾아낼 때 유용

**예시: 일자별 평균 구매 개수가 가장 많거나 적은 날**

```json
POST kibana_sample_data_ecommerce/_search
{
  "size": 0,
  "query": {
    "match_all": {}
  },
  "aggs": {
    "일자별": {
      "date_histogram": {
        "field": "order_date",
        "calendar_interval": "day"
      },
      "aggs": {
        "평균구매개수": {
          "avg": {
            "field": "total_quantity"
          }
        }
      }
    },
    "일자별평균구매개수가가장많은날": {
      "max_bucket": {
        "buckets_path": "일자별>평균구매개수" // "일자별" 버킷 집계 내의 "평균구매개수" 메트릭 사용
      }
    },
    "일자별평균구매개수가가장적은날": {
      "min_bucket": {
        "buckets_path": "일자별>평균구매개수"
      }
    }
  }
}
```

---

### 5. 실습 예제: `bank` 인덱스 분석 🏦

`bank` 인덱스에 데이터를 벌크(bulk)로 삽입한 후, 다양한 집계 쿼리를 통해 데이터를 분석한다.

**`bank` 인덱스 데이터 삽입 예시:**

```json
PUT _bulk
{"index":{"_index":"bank", "_id": "1"}}{"date": "2018-06-01", "bank": "NH농협은행", "branch": "1호점", "location": "종각", "customers": 2314}
{"index":{"_index":"bank", "_id": "2"}}{"date": "2017-06-01", "bank": "NH농협은행", "branch": "1호점", "location": "강남", "customers": 5412}
...
{"index":{"_index":"bank", "_id": "19"}}{"date": "2018-11-01", "bank": "기업은행", "branch": "3호점", "location": "불광", "customers": 151}
{"index":{"_index":"bank", "_id": "20"}}{"date": "2020-10-01", "bank": "기업은행", "branch": "4호점", "location": "불광", "customers": 1302}
```

#### 5.1. `bank`의 모든 고객 수 합계 💰

```json
GET bank/_search
{
  "query": {
    "match_all" : {}
  },
  "size": 0,
  "aggs": {
    "고객수 총 합": {
      "sum": {
        "field": "customers"
      }
    }
  }
}
```

#### 5.2. 최소 고객을 보유한 은행의 고객 수 (및 은행 정보) 📉

**주의:** Elasticsearch는 집계 결과를 직접 쿼리에 재사용하는 기능을 제공하지 않는다. 따라서 최소값을 찾은 후, 해당 값으로 다시 검색을 수행해야 함

**최소 고객 수 찾기:**

```json
GET bank/_search
{
  "query": {
    "match_all" : {}
  },
  "size": 0,
  "aggs": {
    "고객이 가장 적은 은행의 일평균 방문고객수": {
      "min": {
        "field": "customers"
      }
    }
  }
}
```

**최소 고객 수를 가진 문서 검색 (예: 100명):**

```json
GET bank/_search
{
  "query": {
    "match": {
      "customers": 100
    }
  }
}
```

**정렬을 통한 최소값 문서 찾기:**
`sort` 파라미터를 사용하여 `customers` 필드를 오름차순으로 정렬하고 `size: 1`로 가장 작은 값을 가진 문서를 직접 가져올 수 있다.

```json
GET bank/_search
{
  "size": 1,
  "sort":[
    {
      "customers": {
        "order": "asc"
      }
    }
  ]
}
```

#### 5.3. 불광지역에 있는 모든 은행 검색 📍

```json
GET bank/_search
{
  "query": {
    "match": {
      "location": "불광"
    }
  }
}
```

#### 5.4. 불광지역에 있는 모든 은행 고객 수 합계 🏦

```json
GET bank/_search
{
 "size": 0,
  "query": {
    "match": {
      "location": "불광"
    }
  },
  "aggs" : {
    "불광 은행 총 고객수":  {
      "sum" : {
        "field" : "customers"
      }
    }
  }
}
```

#### 5.5. `customers` 수가 100명 이상 152명 미만인 구간의 문서 검색 🔍

**쿼리 DSL의 `range` 사용:**

```json
GET bank/_search
{
    "query": {
        "range": {
          "customers": {
            "gte": 100, // Greater Than or Equal (이상)
            "lt": 152  // Less Than (미만)
          }
        }
    }
}
```

**집계의 `range` 사용 (해당 범위의 문서에 대한 통계):**

```json
GET bank/_search
{
    "size": 0,
    "aggs": {
      "고객수": {
        "range": {
            "field": "customers",
            "ranges" : [
                {"from": 100,
                "to":152}
            ]
        },
      "aggs": {
        "고객수합": {
          "stats": {
            "field":"customers"
          }
        }
      }
      }
    }
}
```

#### 5.6. 500명 단위로 구간을 나누어 `customers` 필드 집계 📈

```json
GET bank/_search
{
  "size": 0,
  "aggs": {
    "500명단위customers": {
      "histogram": {
        "field": "customers",
        "interval": 500
      },
      "aggs": {
        "고객수 합": {
          "sum": {
            "field": "customers"
          }
        }
      }
    }
  }
}
```

#### 5.7. 월별로 구간을 나누어 `date` 필드 집계 🗓️

```json
GET bank/_search
{
  "size": 0,
  "aggs": {
    "월별구간": {
      "date_histogram": {
        "field": "date",
        "calendar_interval": "1M", // 1개월 간격
        "format": "yyyy-MM"
      },
      "aggs": {
        "월별 고객수": {
          "sum": {
            "field": "customers"
          }
        }
      }
    }
  }
}
```

#### 5.8. 년도별로 구간을 나누어 `date` 필드 집계 🗓️

```json
GET bank/_search
{
  "size": 0,
  "aggs": {
    "연도별구간": {
      "date_histogram": {
        "field": "date",
        "calendar_interval": "1y", // 1년 간격
        "format": "yyyy"
      },
      "aggs": {
        "연도별 고객수": {
          "sum": {
            "field": "customers"
          }
        }
      }
    }
  }
}
```

#### 5.9. 각 지역에 은행이 몇 개씩 있는지, 위치 키워드 문자열 별로 버킷을 나누어 집계 🗺️

`location.keyword` 필드를 사용하여 정확한 지역명으로 그룹화

```json
GET bank/_search
{
  "size": 0,
  "aggs": {
    "location별 은행 개수": {
      "terms": {
        "field": "location.keyword"
      },
      "aggs": {
        "고객수합": {
          "sum": {
            "field": "customers"
          }
        }
      }
    }
  }
}
```

#### 5.10. 각 은행별로 개점년도로 집계 후 고객 수를 하위 집계 수행 🏦📅

은행별(`bank.keyword`)로 그룹화하고, 각 은행 내에서 개점년도(`date`)별로 다시 그룹화한 후, 해당 년도의 총 고객 수를 계산

```json
GET bank/_search
{
  "size": 0,
  "aggs": {
    "은행별 지점 개수": {
      "terms": {
        "field": "bank.keyword"
      },
      "aggs": {
        "개점년도별": {
          "date_histogram": {
            "field": "date",
            "calendar_interval": "1y",
            "format": "yyyy년"
          },
          "aggs": {
            "총 고객수": {
              "sum": {
                "field": "customers"
              }
            }
          }
        }
      }
    }
  }
}
```

#### 5.11. 각 은행별로 개점년도로 집계 후 고객 수의 평균을 하위 집계 수행 🏦📊

위와 동일한 구조에서 총 고객수 대신 평균 고객수를 계산

```json
GET bank/_search
{
  "size": 0,
  "aggs": {
    "은행별 지점 개수": {
    "terms": {
        "field": "bank.keyword"
    },
    "aggs": {
        "개점년도별": {
        "date_histogram": {
            "field": "date",
            "calendar_interval": "1y",
            "format": "yyyy년"
        },
        "aggs": {
            "평균 고객수": {
            "avg": {
                "field": "customers"
            }
            }
        }
        }
    }
    }
  }
}
```

#### 5.12. 각 은행별로 총 고객 수와 고객 수 평균을 모두 집계 🏦📈

은행별(`bank.keyword`)로 그룹화하고, 각 은행의 총 고객 수(`sum`)와 평균 고객 수(`avg`)를 동시에 계산

```json
GET bank/_search
{
  "size": 0,
  "aggs": {
    "은행별지점개수": {
      "terms": {
        "field": "bank.keyword"
      },
      "aggs": {
        "평균고객수": {
          "avg": {
            "field": "customers"
          }
        },
        "총 고객수": {
          "sum": {
            "field": "customers"
          }
        }
      }
    }
  }
}
```

#### 5.13. 연간 은행 방문 고객수 총합이 가장 큰 년도 찾기 (Pipeline Aggregation) 🏆

`date_histogram`으로 년도별 총 고객수를 계산한 후, `max_bucket` 파이프라인 집계를 사용하여 이 중 가장 큰 값을 가진 년도를 찾는다.

```json
GET bank/_search
{
  "size": 0,
  "aggs": {
    "개점년도별 집계": {
      "date_histogram": {
        "field": "date",
        "calendar_interval": "1y",
        "format": "yyyy년도"
      },
      "aggs": {
        "연도별 은행 고객수": {
          "sum": {
            "field": "customers"
          }
        }
      }
    },
    "연간 은행 방문 고객수 총합이 가장 큰 년도": {
      "max_bucket": {
        "buckets_path": "개점년도별 집계>연도별 은행 고객수" // "개점년도별 집계" 버킷 내의 "연도별 은행 고객수" 메트릭 사용
      }
    }
  }
}
```

---

### ✨ 오늘의 회고

**Elasticsearch 집계**는 데이터의 숨겨진 이야기를 찾아내는 탐정 도구 같았다. 🕵️‍♀️ 다양한 집계 기능을 통해 데이터 속에서 의미 있는 패턴과 인사이트를 발견하는 과정은 정말 흥미로웠다. 특히, **메트릭, 버킷, 중첩, 파이프라인 집계를 조합**하여 복잡한 분석을 수행할 수 있다는 점이 인상 깊었다.

Elasticsearch 집계가 실제 비즈니스 문제 해결에 얼마나 강력한 도구가 될 수 있는지 체감했다. 앞으로는 더 다양한 데이터셋에 집계를 적용해보고, Kibana와 연동하여 시각화하는 연습을 꾸준히 해야겠다. 🌊😄
