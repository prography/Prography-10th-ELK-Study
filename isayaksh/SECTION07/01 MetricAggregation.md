# Metric Aggregation
- 수치 데이터를 기반으로 계산을 수행하는 집계 방식이다.
- 흔히 사용되는 예는 합계, 평균, 최대, 최소, 개수 등의 통계를 구하는 것이다.

### MAX값 조회
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

### MIN값 조회
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

## AVG값 조회
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

### `cardinality` 조회
- `products` 인덱스에서 `ategory` 필드의 **고유한 값의 수(=중복 제거된 개수)**를 계산하는 데 사용
- 🔥 category 필드가 텍스트(text) 타입이면 안 되고, **정렬 가능한 필드 (keyword)**여야 동작한다.
- 🔥 고유 값의 개수"를 근사치로 추정하기 때문에 **속도와 메모리 효율이 뛰어나지만, 정확도는 상황에 따라 다소 떨어질 수 있다.**
- 🔥 사용 예시
  - 대시보드/모니터링에서 지표 추세 확인(실시간 추적)
  - 로그에서 에러 코드의 유니크 개수를 빠르게 파악(대략 얼마나 다양한 에러가 나는지 확인)
  - 쿼리 필터 조건 변경에 따른 영향 분석(정확한 값보다 변화량(trend)이 중요함)
  - 
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

### `stats` 조회
- count, min, max, avg, sum의 모든 결과를 조회하는데 사용
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