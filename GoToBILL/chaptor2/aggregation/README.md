# Elasticsearch Aggregation (집계)

Elasticsearch의 집계 기능은 데이터에 대한 통계 분석, 요약 정보, 그리고 복잡한 비즈니스 인텔리전스 쿼리를 수행할 수 있게 해주는 강력한 도구입니다. 이 문서에서는 Elasticsearch의 주요 집계 유형과 사용 방법에 대해 설명합니다.

## 1. 집계 개요

Elasticsearch 집계는 다음과 같은 장점을 제공합니다:
- 대량의 데이터에서 패턴을 빠르게 발견
- 실시간 분석 및 대시보드 구축
- 복잡한 비즈니스 질문에 대한 답변 제공
- 빠른 검색과 분석을 동시에 수행

## 2. 집계 유형

Elasticsearch는 크게 네 가지 유형의 집계를 제공합니다:

### 2.1 Bucket 집계

문서를 버킷(그룹)으로 분류하는 집계입니다. 조건에 맞는 문서들이 각 버킷에 할당됩니다.

#### terms 집계
필드의 값을 기준으로 문서를 그룹화합니다.

```
GET /my_index/_search
{
  "size": 0,
  "aggs": {
    "popular_colors": {
      "terms": {
        "field": "color",
        "size": 5
      }
    }
  }
}
```

#### range 집계
지정된 범위를 기준으로 문서를 그룹화합니다.

```
GET /my_index/_search
{
  "size": 0,
  "aggs": {
    "price_ranges": {
      "range": {
        "field": "price",
        "ranges": [
          { "to": 50 },
          { "from": 50, "to": 100 },
          { "from": 100 }
        ]
      }
    }
  }
}
```

#### date_histogram 집계
날짜/시간 필드를 기준으로 문서를 시간 간격으로 그룹화합니다.

```
GET /my_index/_search
{
  "size": 0,
  "aggs": {
    "sales_over_time": {
      "date_histogram": {
        "field": "timestamp",
        "calendar_interval": "month"
      }
    }
  }
}
```

#### filter 집계
지정된 필터를 통과하는 문서에 대해 집계합니다.

```
GET /my_index/_search
{
  "size": 0,
  "aggs": {
    "high_value_sales": {
      "filter": { 
        "range": { "price": { "gt": 1000 } }
      },
      "aggs": {
        "avg_price": {
          "avg": { "field": "price" }
        }
      }
    }
  }
}
```

### 2.2 Metrics 집계

숫자 필드에 대한 통계 연산을 수행하는 집계입니다.

#### avg/min/max/sum 집계
기본적인 수학 연산을 수행합니다.

```
GET /my_index/_search
{
  "size": 0,
  "aggs": {
    "avg_price": { "avg": { "field": "price" } },
    "min_price": { "min": { "field": "price" } },
    "max_price": { "max": { "field": "price" } },
    "total_price": { "sum": { "field": "price" } }
  }
}
```

#### stats 집계
여러 기본 통계(count, min, max, avg, sum)를 한 번에 계산합니다.

```
GET /my_index/_search
{
  "size": 0,
  "aggs": {
    "price_stats": {
      "stats": { "field": "price" }
    }
  }
}
```

#### cardinality 집계
필드의 고유 값 개수를 계산합니다.

```
GET /my_index/_search
{
  "size": 0,
  "aggs": {
    "unique_customers": {
      "cardinality": { "field": "customer_id" }
    }
  }
}
```

#### percentiles 집계
데이터의 백분위 분포를 계산합니다.

```
GET /my_index/_search
{
  "size": 0,
  "aggs": {
    "load_time_percentiles": {
      "percentiles": {
        "field": "load_time",
        "percents": [50, 75, 95, 99]
      }
    }
  }
}
```

### 2.3 Pipeline 집계

다른 집계의 결과를 입력으로 사용하는 집계입니다.

#### avg_bucket 집계
다른 집계의 값에 대한 평균을 계산합니다.

```
GET /my_index/_search
{
  "size": 0,
  "aggs": {
    "sales_per_month": {
      "date_histogram": {
        "field": "date",
        "calendar_interval": "month"
      },
      "aggs": {
        "sales": { "sum": { "field": "price" } }
      }
    },
    "avg_monthly_sales": {
      "avg_bucket": {
        "buckets_path": "sales_per_month>sales"
      }
    }
  }
}
```

#### derivative 집계
연속적인 버킷 간의 변화율을 계산합니다.

```
GET /my_index/_search
{
  "size": 0,
  "aggs": {
    "sales_per_month": {
      "date_histogram": {
        "field": "date",
        "calendar_interval": "month"
      },
      "aggs": {
        "sales": { "sum": { "field": "price" } },
        "sales_change": {
          "derivative": { "buckets_path": "sales" }
        }
      }
    }
  }
}
```

#### cumulative_sum 집계
버킷에 걸쳐 누적 합계를 계산합니다.

```
GET /my_index/_search
{
  "size": 0,
  "aggs": {
    "sales_per_month": {
      "date_histogram": {
        "field": "date",
        "calendar_interval": "month"
      },
      "aggs": {
        "sales": { "sum": { "field": "price" } },
        "cumulative_sales": {
          "cumulative_sum": { "buckets_path": "sales" }
        }
      }
    }
  }
}
```

### 2.4 Matrix 집계

여러 필드에 걸쳐 작동하고 행렬 결과를 생성하는 집계입니다.

#### matrix_stats 집계
여러 숫자 필드 간의 통계와 상관 관계를 계산합니다.

```
GET /my_index/_search
{
  "size": 0,
  "aggs": {
    "correlation_matrix": {
      "matrix_stats": {
        "fields": ["height", "weight", "age"]
      }
    }
  }
}
```

## 3. 복합 집계 예제

### 3.1 중첩 집계

버킷 집계 내에 다른 집계(버킷 또는 메트릭)를 중첩할 수 있습니다.

```
GET /my_index/_search
{
  "size": 0,
  "aggs": {
    "products_by_category": {
      "terms": {
        "field": "category",
        "size": 10
      },
      "aggs": {
        "avg_price": { "avg": { "field": "price" } },
        "price_range": {
          "range": {
            "field": "price",
            "ranges": [
              { "to": 50 },
              { "from": 50, "to": 100 },
              { "from": 100 }
            ]
          }
        }
      }
    }
  }
}
```

### 3.2 필터와 집계 조합

필터된 문서 집합에 대해 집계를 수행할 수 있습니다.

```
GET /my_index/_search
{
  "query": {
    "range": {
      "date": {
        "gte": "now-1y"
      }
    }
  },
  "size": 0,
  "aggs": {
    "categories": {
      "terms": {
        "field": "category",
        "size": 10
      },
      "aggs": {
        "avg_price": { "avg": { "field": "price" } },
        "top_products": {
          "top_hits": {
            "size": 3,
            "sort": [
              { "rating": { "order": "desc" } }
            ]
          }
        }
      }
    }
  }
}
```

### 3.3 시계열 분석

날짜 히스토그램과 다양한 집계를 결합하여 시계열 데이터를 분석할 수 있습니다.

```
GET /my_index/_search
{
  "size": 0,
  "aggs": {
    "sales_over_time": {
      "date_histogram": {
        "field": "date",
        "calendar_interval": "month"
      },
      "aggs": {
        "sales": { "sum": { "field": "price" } },
        "sales_deriv": {
          "derivative": { "buckets_path": "sales" }
        },
        "moving_avg": {
          "moving_avg": {
            "buckets_path": "sales",
            "window": 3,
            "model": "simple"
          }
        }
      }
    }
  }
}
```

## 4. 집계 최적화 및 성능 향상

### 4.1 데이터 구조 최적화

#### 루신 doc values 활용
대부분의 집계는 doc values를 사용합니다. 집계에 사용하지 않는 필드는 doc values를 비활성화하여 디스크 공간을 절약할 수 있습니다.

```
PUT /my_index
{
  "mappings": {
    "properties": {
      "text_content": {
        "type": "text",
        "doc_values": false
      },
      "category": {
        "type": "keyword"
      }
    }
  }
}
```

#### 필드 데이터 캐시 제한
text 필드에 대한 집계는 필드 데이터 캐시를 사용합니다. 이 캐시의 크기를 제한하여 메모리 사용량을 관리할 수 있습니다.

```
PUT _cluster/settings
{
  "persistent": {
    "indices.fielddata.cache.size": "10%"
  }
}
```

### 4.2 샤딩 전략

집계 성능은 샤드 수와 분포에 크게 영향을 받습니다. 각 샤드에서 로컬 결과를 계산한 후 조합하기 때문입니다.

- 시계열 데이터의 경우 날짜별로 인덱스를 생성하고 적절한 수의 샤드를 사용합니다.
- 작은 데이터 세트에서는 샤드 수를 최소화하여 병합 오버헤드를 줄입니다.
- 큰 데이터 세트에서는 샤드를 적절히 분배하여 병렬 처리를 최대화합니다.

### 4.3 쿼리 최적화

#### 필터 컨텍스트 사용
가능한 경우 query 대신 filter 컨텍스트를 사용합니다. 필터는 캐시될 수 있으며 관련성 점수를 계산하지 않습니다.

```
GET /my_index/_search
{
  "query": {
    "bool": {
      "filter": [
        { "term": { "status": "active" } },
        { "range": { "price": { "gte": 100 } } }
      ]
    }
  },
  "aggs": {
    "categories": {
      "terms": { "field": "category" }
    }
  }
}
```

#### 부분 집계 결과
대략적인 결과가 허용되는 경우, 더 빠른 집계를 위해 정확도를 희생할 수 있습니다.

```
GET /my_index/_search
{
  "size": 0,
  "aggs": {
    "approx_unique_users": {
      "cardinality": {
        "field": "user_id",
        "precision_threshold": 100
      }
    }
  }
}
```

#### 집계 결과 캐싱
자주 사용되는 집계의 경우 요청 캐시를 활성화할 수 있습니다.

```
GET /my_index/_search?request_cache=true
{
  "size": 0,
  "aggs": {
    "popular_categories": {
      "terms": { "field": "category" }
    }
  }
}
```

## 5. 고급 집계 기능

### 5.1 스크립트 기반 집계

필드 값을 직접 사용하는 대신 스크립트를 사용하여 집계를 수행할 수 있습니다.

```
GET /my_index/_search
{
  "size": 0,
  "aggs": {
    "discounted_stats": {
      "stats": {
        "script": {
          "source": "doc['price'].value * 0.8"
        }
      }
    }
  }
}
```

### 5.2 중첩 문서 집계

중첩 문서에 대한 집계를 수행하려면 `nested` 집계를 사용해야 합니다.

```
GET /my_index/_search
{
  "size": 0,
  "aggs": {
    "reviews": {
      "nested": {
        "path": "reviews"
      },
      "aggs": {
        "average_rating": {
          "avg": {
            "field": "reviews.rating"
          }
        }
      }
    }
  }
}
```

### 5.3 복합(Composite) 집계

여러 버킷 소스를 조합하여 모든 가능한 조합을 생성하는 집계입니다. 큰 데이터셋에서 페이지네이션을 지원합니다.

```
GET /my_index/_search
{
  "size": 0,
  "aggs": {
    "sales_by_category_and_month": {
      "composite": {
        "sources": [
          {
            "category": {
              "terms": { "field": "category" }
            }
          },
          {
            "month": {
              "date_histogram": {
                "field": "date",
                "calendar_interval": "month"
              }
            }
          }
        ]
      }
    }
  }
}
```

### 5.4 결측값 처리

집계 시 필드 값이 없는 문서를 처리하는 방법입니다.

```
GET /my_index/_search
{
  "size": 0,
  "aggs": {
    "avg_price": {
      "avg": {
        "field": "price",
        "missing": 0
      }
    }
  }
}
```

## 6. 집계 사용 사례

### 6.1 대시보드 및 시각화

Kibana와 같은 도구를 사용하여 집계 결과를 시각화할 수 있습니다. 일반적인 대시보드 요소로는 다음이 있습니다:

- 시간 경과에 따른 메트릭 추이
- 카테고리별 분포
- 지리 데이터 시각화
- 상위/하위 항목 리스트
- 이상치 감지

### 6.2 비즈니스 인텔리전스

집계를 통해 비즈니스 인사이트를 얻을 수 있습니다:

- 판매 트렌드 및 예측
- 사용자 행동 분석
- 제품 성능 모니터링
- 마케팅 캠페인 효과 측정

### 6.3 로그 및 이벤트 분석

시스템 로그 및 이벤트 데이터를 분석할 수 있습니다:

- 오류 발생 패턴 식별
- 성능 병목 현상 감지
- 사용자 세션 분석
- 보안 이벤트 집계

## 7. 실무 권장 사항

### 7.1 일반적인 권장 사항

- **쿼리 결과 제한**: 집계에 필요한 문서만 필터링하여 성능 향상
- **샘플링 활용**: 대규모 데이터셋에서는 샘플링을 통해 근사치를 빠르게 얻기
- **인덱스 설계**: 집계를 고려한 매핑 및 인덱스 설계
- **캐싱 활용**: 자주 사용되는 집계 결과 캐싱

### 7.2 성능 모니터링

집계 성능을 모니터링하기 위한 방법:

- **프로파일 API 활용**: 집계 실행 계획 및 시간 분석
- **클러스터 상태 모니터링**: 집계 중 노드 부하 확인
- **핫 샤드 식별**: 집계 부하가 높은 샤드 식별 및 최적화

### 7.3 스케일링 전략

대규모 집계를 처리하기 위한 스케일링 전략:

- **수직 확장**: 더 많은 메모리 및 CPU 리소스 할당
- **수평 확장**: 클러스터에 더 많은 노드 추가
- **샤드 재배포**: 핫 샤드를 여러 노드에 분산
- **데이터 모델 최적화**: 집계에 최적화된 데이터 구조 설계

## 결론

Elasticsearch의 집계 기능은 데이터에서 의미 있는 인사이트를 추출하기 위한 강력한 도구입니다. 버킷, 메트릭, 파이프라인, 매트릭스 집계를 조합하여 복잡한 분석 요구사항을 충족할 수 있습니다.

성능 최적화를 위해 적절한 매핑, 샤딩, 필터링 전략을 사용하고, 실시간 분석과 대시보드를 위해 집계 결과를 효과적으로 시각화할 수 있습니다.

집계 기능을 마스터하면 Elasticsearch를 단순한 검색 엔진을 넘어 강력한 분석 플랫폼으로 활용할 수 있습니다.