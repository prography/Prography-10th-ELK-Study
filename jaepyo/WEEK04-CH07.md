## Metric Aggregation

개별 필드 값을 기반으로 **수치적 요약**을 계산합니다.

| 종류 | 설명 | 예시 |
| --- | --- | --- |
| `sum` | 필드 값의 총합 | 총 판매액 구하기 |
| `avg` | 필드 값의 평균 | 평균 사용자 나이 |
| `min` | 필드 값 중 최소값 | 최저 가격 상품 |
| `max` | 필드 값 중 최대값 | 최고 매출 |
| `stats` | min, max, avg, sum, count를 모두 반환 | 종합적인 통계 |

### 예시: 평균 가격 구하기

```
GET /products/_search
{
  "size": 0,
  "aggs": {
    "average_price": {
      "avg": {
        "field": "price"
      }
    }
  }
}

```

---

## Bucket Aggregation

조건에 따라 다큐먼트를 그룹 지어 **버킷**에 넣습니다. 각 버킷은 해당 조건을 만족하는 문서들로 구성됩니다.

### 1. Term Aggregation

특정 필드의 **고유한 값 별로 그룹화**합니다.

```

""
GET /products/_search
{
  "size": 0,
  "aggs": {
    "by_category": {
      "terms": {
        "field": "category.keyword"
      }
    }
  }
}

```

---

### 2. Range Aggregation

숫자나 날짜 필드에 대해 **범위 구간**으로 나누어 집계합니다.

```


GET /products/_search
{
  "size": 0,
  "aggs": {
    "price_ranges": {
      "range": {
        "field": "price",
        "ranges": [
          { "to": 100 },
          { "from": 100, "to": 500 },
          { "from": 500 }
        ]
      }
    }
  }
}

```

---

### 3. Histogram Aggregation

숫자 필드를 **고정된 간격(bin)** 으로 나누어 집계합니다.

```
GET /sales/_search
{
  "size": 0,
  "aggs": {
    "price_histogram": {
      "histogram": {
        "field": "amount",
        "interval": 100
      }
    }
  }
}

```

---

### 4. Date Histogram Aggregation

날짜 필드를 기준으로 **일, 월 등 시간 단위**로 나누어 집계합니다.

```
GET /logs/_search
{
  "size": 0,
  "aggs": {
    "monthly_logs": {
      "date_histogram": {
        "field": "timestamp",
        "calendar_interval": "month"
      }
    }
  }
}

```

---

## Metric + Bucket 조합 예시

카테고리별 평균 가격 구하기

```
GET /products/_search
{
  "size": 0,
  "aggs": {
    "by_category": {
      "terms": {
        "field": "category.keyword"
      },
      "aggs": {
        "average_price": {
          "avg": {
            "field": "price"
          }
        }
      }
    }
  }
}

```

---

### Filter Aggregation

단일 필터 조건을 만족하는 문서만 포함하는 하나의 버킷 생성

```
"aggs": {
  "only_expensive": {
    "filter": {
      "range": { "price": { "gt": 1000 } }
    },
    "aggs": {
      "avg_price": { "avg": { "field": "price" } }
    }
  }
}

```

---

### Filters Aggregation

다중 필터 조건에 따라 여러 버킷 생성

```
"aggs": {
  "price_ranges": {
    "filters": {
      "filters": {
        "cheap": { "range": { "price": { "lt": 100 } } },
        "mid": { "range": { "price": { "gte": 100, "lte": 500 } } },
        "expensive": { "range": { "price": { "gt": 500 } } }
      }
    }
  }
}

```

---

### Composite Aggregation

대량 데이터를 pagination 방식으로 집계 (scroll과 함께 사용 가능)

```
"aggs": {
  "composite_buckets": {
    "composite": {
      "size": 1000,
      "sources": [
        { "category": { "terms": { "field": "category.keyword" } } },
        { "brand": { "terms": { "field": "brand.keyword" } } }
      ]
    }
  }
}

```

---

### Missing Aggregation

특정 필드가 존재하지 않는 문서만 모은 버킷 생성

```
"aggs": {
  "missing_price": {
    "missing": {
      "field": "price"
    }
  }
}

```

---

### Multi Terms Aggregation

여러 필드를 조합하여 복합 키로 그룹핑 (7.12+)

```
"aggs": {
  "category_and_brand": {
    "multi_terms": {
      "terms": [
        { "field": "category.keyword" },
        { "field": "brand.keyword" }
      ]
    }
  }
}

```

---

### Adjacency Matrix Aggregation

여러 조건 조합을 통해 교집합 버킷 생성

```
"aggs": {
  "intersections": {
    "adjacency_matrix": {
      "filters": {
        "cheap": { "range": { "price": { "lt": 100 } } },
        "popular": { "range": { "rating": { "gte": 4.5 } } }
      }
    }
  }
}

```

---

## 고급 Metric Aggregation

### Percentiles / Percentile Ranks

백분위 수 계산

```
"aggs": {
  "load_percentiles": {
    "percentiles": {
      "field": "response_time"
    }
  }
}

```

---

### Cardinality

고유한 값의 개수 추정

```
"aggs": {
  "unique_users": {
    "cardinality": {
      "field": "user_id"
    }
  }
}

```

---

### Top Hits

각 버킷에서 대표 문서 추출

```
"aggs": {
  "by_category": {
    "terms": {
      "field": "category.keyword"
    },
    "aggs": {
      "top_products": {
        "top_hits": {
          "size": 1,
          "_source": ["name", "price"]
        }
      }
    }
  }
}

```

---

### Scripted Metric

사용자 정의 로직으로 집계 수행

```
"aggs": {
  "custom_metric": {
    "scripted_metric": {
      "init_script": "state.transactions = []",
      "map_script": "state.transactions.add(doc.amount.value)",
      "combine_script": "return state.transactions",
      "reduce_script": "double sum = 0; for (s in states) { for (t in s) { sum += t } } return sum"
    }
  }
}

```

---

## Pipeline Aggregation

다른 집계 결과를 기반으로 연산

### Derivative

변화량 계산 (기울기)

### Cumulative Sum

누적합 계산

### Moving Avg

이동 평균 계산

### Bucket Script

여러 metric을 조합한 계산

```
"aggs": {
  "revenue": { "sum": { "field": "price" } },
  "tax": {
    "bucket_script": {
      "buckets_path": {
        "revenue": "revenue"
      },
      "script": "params.revenue * 0.1"
    }
  }
}

```
