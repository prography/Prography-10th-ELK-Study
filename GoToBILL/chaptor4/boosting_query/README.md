# 부스팅 쿼리(Boosting Query)

부스팅 쿼리는 Elasticsearch에서 문서의 관련성 점수를 조정하여 검색 결과의 순위를 세밀하게 제어할 수 있는 강력한 기능입니다.

## 부스팅 쿼리란?

부스팅 쿼리는 특정 조건에 따라 문서의 관련성 점수를 증가(promotion)하거나 감소(demotion)시켜 검색 결과의 순위를 조정하는 데 사용됩니다.

특정 문서를 완전히 제외하지 않으면서도 검색 결과에서의 순위를 조절할 수 있는 유연한 방법을 제공합니다.

## 부스팅 쿼리의 작동 방식

부스팅 쿼리는 두 가지 주요 부분으로 구성됩니다:

1. **Positive Query (긍정 쿼리)**: 
   - 점수를 높이거나 유지하고 싶은 문서를 선택하는 쿼리
   - 이 쿼리와 일치하는 문서는 정상적인 관련성 점수를 받습니다.

2. **Negative Query (부정 쿼리)**:
   - 점수를 낮추고 싶은 문서를 선택하는 쿼리
   - 이 쿼리와 일치하는 문서는 관련성 점수가 감소합니다.
   - 단, 결과에서 완전히 제외되지는 않습니다.

## negative_boost 매개변수

`negative_boost` 매개변수는 부정 쿼리와 일치하는 문서의 점수를 얼마나 감소시킬지 결정합니다:

- 값 범위: 0에서 1 사이
- 값이 0이면: 부정 쿼리와 일치하는 문서를 결과에서 완전히 제외
- 값이 1이면: 점수 감소 없음 (부정 쿼리가 효과 없음)
- 일반적인 값: 0.5 (점수를 절반으로 감소)

## 부스팅 쿼리 예제

```
GET /products/_search
{
  "query": {
    "boosting": {
      "positive": {
        "match": {
          "category": "electronics"
        }
      },
      "negative": {
        "match": {
          "status": "discontinued"
        }
      },
      "negative_boost": 0.5
    }
  }
}
```

이 쿼리는:
1. 카테고리가 "electronics"인 모든 제품을 검색합니다.
2. 이 중 "discontinued" 상태인 제품의 점수를 절반(0.5)으로 감소시킵니다.
3. 결과적으로 단종되지 않은 전자제품이 검색 결과 상위에 표시되지만, 단종된 제품도 결과에는 포함됩니다.

## 다른 예제

### 가격이 높은 제품 강조, 재고 없는 제품 순위 하락

```
GET /products/_search
{
  "query": {
    "boosting": {
      "positive": {
        "range": {
          "price": {
            "gt": 100
          }
        }
      },
      "negative": {
        "term": {
          "in_stock": false
        }
      },
      "negative_boost": 0.3
    }
  }
}
```

### 특정 브랜드 제품 강조, 낮은 평점 제품 순위 하락

```
GET /products/_search
{
  "query": {
    "boosting": {
      "positive": {
        "match": {
          "brand": "Samsung"
        }
      },
      "negative": {
        "range": {
          "rating": {
            "lt": 3
          }
        }
      },
      "negative_boost": 0.4
    }
  }
}
```

## 복잡한 부스팅 쿼리

부스팅 쿼리의 positive와 negative 부분에는 더 복잡한 쿼리를 사용할 수도 있습니다:

```
GET /products/_search
{
  "query": {
    "boosting": {
      "positive": {
        "bool": {
          "must": [
            { "match": { "category": "electronics" } },
            { "match": { "features": "wireless" } }
          ]
        }
      },
      "negative": {
        "bool": {
          "should": [
            { "term": { "stock_status": "backorder" } },
            { "range": { "shipping_days": { "gt": 7 } } }
          ]
        }
      },
      "negative_boost": 0.5
    }
  }
}
```

## 부스팅 쿼리의 장점

1. **검색 결과 순위의 세밀한 제어**: 문서를 제외하지 않고도 순위를 조정할 수 있습니다.

2. **부드러운 필터링(Soft Filtering)**: 일반 필터와 달리 조건에 맞지 않는 문서도 결과에 포함시킬 수 있습니다.

3. **사용자 선호도 반영**: 사용자 선호도나 비즈니스 규칙을 검색 결과에 반영할 수 있습니다.

## 사용 사례

1. **e-커머스 제품 검색**:
   - 할인 상품이나 신상품 강조
   - 품절 상품이나 낮은 평점 상품 순위 하락

2. **콘텐츠 검색**:
   - 최신 콘텐츠 강조
   - 오래된 콘텐츠 순위 하락

3. **호텔 검색**:
   - 고객 평점이 높은 호텔 강조
   - 특정 편의시설이 없는 호텔 순위 하락

4. **구인 검색**:
   - 특정 기술이나 경험을 요구하는 포지션 강조
   - 원격 근무가 불가능한 포지션 순위 하락

부스팅 쿼리는 검색 결과의 관련성을 사용자 요구에 맞게 조정하는 데 탁월한 도구입니다. 검색 경험을 최적화하고 비즈니스 목표를 달성하는 데 큰 도움이 됩니다.
