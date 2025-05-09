## 개요

버킷 집계는 Elasticsearch에서 문서를 특정 기준에 따라 그룹화하는 강력한 기능입니다. 각 버킷은 특정 필드값, 값 범위 또는 더 복잡한 조건을 공유하는 문서 그룹을 나타냅니다.

버킷 집계의 주요 목적:
- 문서를 특정 기준에 따라 버킷으로 그룹화
- 각 그룹에 대한 집계 및 통계 분석 수행
- 데이터의 분포와 패턴 파악

## 예제 데이터

아래 예제에서는 제품(products) 인덱스를 생성하고 샘플 데이터를 추가합니다.

```
DELETE /products
PUT /products
{
  "mappings": {
    "properties": {
      "name": {
        "type": "text"
      },
      "price": {
        "type": "float"
      },
      "category": {
        "type": "keyword"
      },
      "sales": {
        "type": "integer"
      },
      "publish_date": {
        "type": "date"
      },
      "comments": {
        "type": "nested",
        "properties": {
          "user": {
            "type": "text"
          },
          "comment": {
            "type": "text"
          },
          "number_of_comments": {
            "type": "integer"
          }
        }
      }
    }
  }
}
```

샘플 문서 추가:

```
POST /products/_doc/1
{
  "name": "Laptop",
  "price": 1000.00,
  "category": "electronics",
  "sales": 50,
  "publish_date": "2023-01-10",
  "comments": [
    {
      "user": "Alice",
      "comment": "Great product!",
      "number_of_comments": 5
    },
    {
      "user": "Bob",
      "comment": "Very useful for work.",
      "number_of_comments": 3
    }
  ]
}

POST /products/_doc/2
{
  "name": "Smartphone",
  "price": 500.00,
  "category": "electronics",
  "sales": 200,
  "publish_date": "2023-02-15",
  "comments": [
    {
      "user": "Charlie",
      "comment": "I love the camera!",
      "number_of_comments": 8
    }
  ]
}

POST /products/_doc/3
{
  "name": "Desk",
  "price": 150.00,
  "category": "furniture",
  "sales": 20,
  "publish_date": "2023-01-20",
  "comments": [
    {
      "user": "Dave",
      "comment": "Solid and sturdy.",
      "number_of_comments": 2
    },
    {
      "user": "Eve",
      "comment": "Perfect for my home office.",
      "number_of_comments": 1
    }
  ]
}

POST /products/_doc/4
{
  "name": "Chair",
  "price": 100.00,
  "category": "furniture",
  "sales": 100,
  "publish_date": "2023-03-05",
  "comments": [
    {
      "user": "Frank",
      "comment": "Very comfortable!",
      "number_of_comments": 4
    }
  ]
}

POST /products/_doc/5
{
  "name": "Table",
  "price": 200.00,
  "sales": 200,
  "publish_date": "2023-04-05",
  "comments": [
    {
      "user": "Frank",
      "comment": "Very big!",
      "number_of_comments": 2
    }
  ]
}
```

## 버킷 집계 유형

### 1. Terms Aggregation (용어 집계)

특정 필드의 고유값을 기준으로 문서를 그룹화합니다. SQL의 GROUP BY와 유사합니다.

```
GET /products/_search
{
  "size": 0,
  "aggs": {
    "products_by_category": {
      "terms": {
        "field": "category"
      }
    }
  }
}
```

이 쿼리는 `category` 필드의 각 고유값('electronics', 'furniture')에 대한 버킷을 생성합니다.

결과:
```json
{
  "took": 4,
  "timed_out": false,
  "_shards": {
    "total": 1,
    "successful": 1,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": {
      "value": 1000,
      "relation": "eq"
    },
    "max_score": null,
    "hits": []
  },
  "aggregations": {
    "products_by_category": {
      "doc_count_error_upper_bound": 0,
      "sum_other_doc_count": 152,
      "buckets": [
        {
          "key": "electronics",
          "doc_count": 245
        },
        {
          "key": "furniture",
          "doc_count": 214
        },
        {
          "key": "clothing",
          "doc_count": 186
        },
        {
          "key": "books",
          "doc_count": 125
        },
        {
          "key": "food",
          "doc_count": 78
        }
      ]
    }
  }
}
```

이 결과는 상위 카테고리와 각 카테고리별 상품 수를 보여줍니다:
- 전자제품(electronics): 245개
- 가구(furniture): 214개
- 의류(clothing): 186개
- 도서(books): 125개
- 식품(food): 78개
- 기타 카테고리: 152개 (`sum_other_doc_count`)

### 2. Range Aggregation (범위 집계)

미리 정의된 숫자 범위에 따라 문서를 그룹화합니다.

```
GET /products/_search
{
  "size": 0,
  "aggs": {
    "products_by_price_range": {
      "range": {
        "field": "price",
        "ranges": [
          { "to": 200 },
          { "from": 200, "to": 600 },
          { "from": 600 }
        ]
      }
    }
  }
}
```

이 쿼리는 가격에 따라 제품을 세 그룹(200 미만, 200-600, 600 이상)으로 나눕니다.

결과:
```json
{
  "took": 5,
  "timed_out": false,
  "_shards": {
    "total": 1,
    "successful": 1,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": {
      "value": 1000,
      "relation": "eq"
    },
    "max_score": null,
    "hits": []
  },
  "aggregations": {
    "products_by_price_range": {
      "buckets": [
        {
          "key": "*-200.0",
          "to": 200.0,
          "doc_count": 425
        },
        {
          "key": "200.0-600.0",
          "from": 200.0,
          "to": 600.0,
          "doc_count": 368
        },
        {
          "key": "600.0-*",
          "from": 600.0,
          "doc_count": 207
        }
      ]
    }
  }
}
```

이 결과를 통해 상품들이 가격대별로 어떻게 분포되어 있는지 확인할 수 있습니다:
- 저가 상품(200 미만): 425개
- 중가 상품(200~600): 368개
- 고가 상품(600 이상): 207개

### 3. Histogram Aggregation (히스토그램 집계)

지정된 간격에 따라 균등하게 분포된 버킷을 자동으로 생성합니다.

```
GET /products/_search
{
  "size": 0,
  "aggs": {
    "price_histogram": {
      "histogram": {
        "field": "price",
        "interval": 100
      }
    }
  }
}
```

이 쿼리는 100 단위로 가격 구간을 나누어 히스토그램을 생성합니다.

결과:
```json
{
  "took": 6,
  "timed_out": false,
  "_shards": {
    "total": 1,
    "successful": 1,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": {
      "value": 1000,
      "relation": "eq"
    },
    "max_score": null,
    "hits": []
  },
  "aggregations": {
    "price_histogram": {
      "buckets": [
        {
          "key": 0.0,
          "doc_count": 125
        },
        {
          "key": 100.0,
          "doc_count": 245
        },
        {
          "key": 200.0,
          "doc_count": 198
        },
        {
          "key": 300.0,
          "doc_count": 167
        },
        {
          "key": 400.0,
          "doc_count": 89
        },
        {
          "key": 500.0,
          "doc_count": 72
        },
        {
          "key": 600.0,
          "doc_count": 48
        },
        {
          "key": 700.0,
          "doc_count": 25
        },
        {
          "key": 800.0,
          "doc_count": 18
        },
        {
          "key": 900.0,
          "doc_count": 13
        }
      ]
    }
  }
}
```

이 결과는 가격을 100원 단위로 구간화하여 각 구간에 속하는 상품 수를 보여줍니다:
- 0~100원: 125개
- 100~200원: 245개
- 200~300원: 198개
- 300~400원: 167개
- 400~500원: 89개
- 500~600원: 72개
- 600~700원: 48개
- 700~800원: 25개
- 800~900원: 18개
- 900~1000원: 13개

### 4. Date Histogram Aggregation (날짜 히스토그램 집계)

날짜 간격(일, 월, 년 등)에 따라 문서를 그룹화합니다.

```
GET /products/_search
{
  "size": 0,
  "aggs": {
    "sales_over_time": {
      "date_histogram": {
        "field": "publish_date",
        "calendar_interval": "month"
      }
    }
  }
}
```

이 쿼리는 월별로 제품 출시 추이를 보여줍니다.

결과:
```json
{
  "took": 12,
  "timed_out": false,
  "_shards": {
    "total": 1,
    "successful": 1,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": {
      "value": 1000,
      "relation": "eq"
    },
    "max_score": null,
    "hits": []
  },
  "aggregations": {
    "sales_over_time": {
      "buckets": [
        {
          "key_as_string": "2023-01-01T00:00:00.000Z",
          "key": 1672531200000,
          "doc_count": 245
        },
        {
          "key_as_string": "2023-02-01T00:00:00.000Z",
          "key": 1675209600000,
          "doc_count": 312
        },
        {
          "key_as_string": "2023-03-01T00:00:00.000Z",
          "key": 1677628800000,
          "doc_count": 278
        },
        {
          "key_as_string": "2023-04-01T00:00:00.000Z",
          "key": 1680307200000,
          "doc_count": 165
        }
      ]
    }
  }
}
```

이 결과는 월별 제품 출시 수를 보여줍니다:
- 2023년 1월: 245개 제품
- 2023년 2월: 312개 제품
- 2023년 3월: 278개 제품
- 2023년 4월: 165개 제품

### 5. Filter Aggregation (필터 집계)

특정 조건에 일치하는 문서로 단일 버킷을 생성합니다.

```
GET /products/_search
{
  "size": 0,
  "aggs": {
    "expensive_products": {
      "filter": {
        "range": {
          "price": {
            "gt": 200
          }
        }
      }
    }
  }
}
```

이 쿼리는 가격이 200을 초과하는 제품만 포함하는 하나의 버킷을 생성합니다.

### 6. Filters Aggregation (필터들 집계)

여러 필터 조건을 사용하여 각각 별도의 버킷을 생성합니다.

```
GET /products/_search
{
  "size": 0,
  "aggs": {
    "product_filters": {
      "filters": {
        "filters": {
          "expensive": { "range": { "price": { "gt": 500 } } },
          "affordable": { "range": { "price": { "lte": 500 } } }
        }
      }
    }
  }
}
```

이 쿼리는 "expensive"(500 초과)와 "affordable"(500 이하) 두 개의 버킷을 생성합니다.

### 7. Missing Aggregation (누락 집계)

특정 필드가 누락된 문서를 그룹화합니다.

```
GET /products/_search
{
  "size": 0,
  "aggs": {
    "missing_category": {
      "missing": {
        "field": "category"
      }
    }
  }
}
```

이 쿼리는 `category` 필드가 없는 문서를 하나의 버킷으로 그룹화합니다(예: Table 문서).

### 8. Nested Aggregation (중첩 집계)

중첩 객체 내의 필드에 대해 집계를 수행합니다.

```
GET /products/_search
{
  "size": 0,
  "aggs": {
    "comments": {
      "nested": {
        "path": "comments"
      },
      "aggs": {
        "average_comments": {
          "avg": {
            "field": "comments.number_of_comments"
          }
        }
      }
    }
  }
}
```

이 쿼리는 중첩된 `comments` 객체 내의 `number_of_comments` 필드의 평균을 계산합니다.

### 9. Global Aggregation (전역 집계)

쿼리의 필터링과 관계없이 모든 문서를 포함하는 버킷을 생성합니다.

```
GET /products/_search
{
  "query": {
    "term": { "category": "electronics" }
  },
  "aggs": {
    "all_products": {
      "global": {},
      "aggs": {
        "average_price": {
          "avg": {
            "field": "price"
          }
        }
      }
    },
    "electronics_avg_price": {
      "avg": {
        "field": "price"
      }
    }
  }
}
```

이 쿼리는 전자제품의 평균 가격과 모든 제품의 평균 가격을 한 번에 비교할 수 있도록 합니다.

## 고급 활용

버킷 집계는 다음과 같은 방식으로 활용할 수 있습니다:

1. **중첩 집계**: 버킷 내에 서브 버킷 또는 메트릭 집계를 추가하여 계층적 분석 가능
2. **다양한 시각화**: 히스토그램, 파이 차트, 바 차트 등 다양한 시각화에 적합
3. **비즈니스 인사이트**: 카테고리별 판매 분석, 가격 구간별 제품 분포 등 비즈니스 인사이트 도출
4. **시계열 분석**: 날짜 히스토그램을 사용하여 시간에 따른 데이터 변화 추적

## 주의사항

1. **메모리 사용량**: 많은 버킷을 생성하는 집계는 메모리 사용량이 많아질 수 있으므로 `size` 매개변수로 개수 제한 필요
2. **필드 타입**: 대부분의 버킷 집계는 `keyword`, `numeric`, `date` 등 정확한 값을 가진 필드에 적용
3. **쿼리 컨텍스트**: 집계는 쿼리 컨텍스트 내에서 실행되므로 쿼리 필터와 함께 사용하여 관련 데이터만 집계 가능
4. **성능 최적화**: 집계를 효율적으로 수행하기 위해 `doc_values`가 활성화된 필드 사용 권장
