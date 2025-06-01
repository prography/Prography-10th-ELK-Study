## 개요

메트릭 집계는 숫자 필드에 대한 다양한 통계 지표를 계산하는 기능입니다. Elasticsearch에서 제공하는 주요 메트릭 집계 유형은 다음과 같습니다:

- **sum**: 필드 값의 합계
- **average**: 필드 값의 평균
- **min**: 필드의 최소값
- **max**: 필드의 최대값
- **stats**: 위의 모든 지표(min, max, sum, avg)와 함께 count까지 한 번에 계산
- **cardinality**: 필드의 고유값 개수(유니크 값 카운트)

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
      }
    }
  }
}

POST /products/_doc/1
{
  "name": "Laptop",
  "price": 1000,
  "category": "electronics",
  "sales": 50
}

POST /products/_doc/2
{
  "name": "Smartphone",
  "price": 500,
  "category": "electronics",
  "sales": 200
}

POST /products/_doc/3
{
  "name": "Desk",
  "price": 150,
  "category": "furniture",
  "sales": 20
}

POST /products/_doc/4
{
  "name": "Chair",
  "price": 100,
  "category": "furniture",
  "sales": 100
}
```

## 메트릭 집계 예제

### 1. Max 집계 - 최대값 찾기

price 필드의 최대값을 찾는 쿼리:

```
GET /products/_search
{
  "size": 0,
  "aggs": {
    "max_price": {
      "max": {
        "field": "price"
      }
    }
  }
}
```

결과로 1000(Laptop의 가격)이 반환됩니다.

### 2. Min 집계 - 최소값 찾기

price 필드의 최소값을 찾는 쿼리:

```
GET /products/_search
{
  "size": 0,
  "aggs": {
    "min_price": {
      "min": {
        "field": "price"
      }
    }
  }
}
```

결과로 100(Chair의 가격)이 반환됩니다.

### 3. Average 집계 - 평균값 계산

sales 필드의 평균값을 계산하는 쿼리:

```
GET /products/_search
{
  "size": 0,
  "aggs": {
    "average_sales": {
      "avg": {
        "field": "sales"
      }
    }
  }
}
```

결과로 모든 제품의 평균 판매량이 반환됩니다.

### 4. Cardinality 집계 - 고유값 개수 세기

category 필드의 고유값 개수를 세는 쿼리:

```
GET /products/_search
{
  "size": 0,
  "aggs": {
    "unique_categories": {
      "cardinality": {
        "field": "category"
      }
    }
  }
}
```

결과로 2(electronics와 furniture의 두 카테고리)가 반환됩니다.

### 5. Stats 집계 - 여러 통계치 한 번에 계산

price 필드에 대한 여러 통계치를 한 번에 계산하는 쿼리:

```
GET /products/_search
{
  "size": 0,
  "aggs": {
    "price_stats": {
      "stats": {
        "field": "price"
      }
    }
  }
}
```

이 쿼리는 price 필드의 다음 통계치를 한 번에 반환합니다:
- count: 문서 개수
- min: 최소값
- max: 최대값
- avg: 평균값
- sum: 합계

## 메트릭 집계 활용

메트릭 집계는 다음과 같은 상황에서 유용하게 사용할 수 있습니다:

1. **비즈니스 인사이트 도출**: 판매 데이터의 평균, 최소, 최대값을 통해 비즈니스 성과를 측정
2. **데이터 요약**: 대량의 데이터를 간결한 통계로 요약
3. **이상치 감지**: min/max 값을 통해 데이터의 이상치 식별
4. **퍼포먼스 모니터링**: 시스템 메트릭(CPU, 메모리 등)의 통계 분석

## 메트릭 집계 관련 고려사항

1. **필드 타입**: 메트릭 집계는 주로 숫자 타입 필드(integer, long, float, double 등)에 적용됩니다.
2. **문서 수**: 대량의 문서에 대한 집계 연산은 리소스를 많이 사용할 수 있으므로 성능에 주의해야 합니다.
3. **정확도**: cardinality 집계는 대용량 데이터에서 근사치를 제공하며, precision_threshold 파라미터로 정확도를 조정할 수 있습니다.
4. **결합 사용**: 메트릭 집계는 버킷 집계(bucket aggregations)와 함께 사용하여 더 복잡한 분석이 가능합니다.
