# Nested Query (중첩 쿼리)

Nested Query는 Elasticsearch에서 중첩된 객체 배열을 효과적으로 검색하기 위한 특수 쿼리 유형입니다.

## 중첩 객체란?

Elasticsearch에서 중첩 객체는 문서 내에 배열 형태로 포함된 객체를 의미하며, 각 객체는 독립적인 엔티티로 취급됩니다. 일반 객체 필드와 달리, 중첩 객체는 인덱싱 시 별도의 숨겨진 문서로 저장되며 부모 문서와의 관계가 유지됩니다.

## 중첩 쿼리의 필요성

### 일반 객체 필드의 한계

일반 객체 필드를 사용할 경우, Elasticsearch는 객체 구조를 평면화(flattening)하여 저장합니다. 이로 인해 필드 간의 관계가 손실될 수 있습니다.

예를 들어, 다음과 같은 제품 문서가 있다고 가정해 보겠습니다:

```
{
  "name": "Smartphone",
  "variants": [
    { "color": "black", "price": 699 },
    { "color": "white", "price": 799 }
  ]
}
```

일반 객체 필드로 인덱싱하면, 이는 다음과 같이 평면화됩니다:

```
{
  "name": "Smartphone",
  "variants.color": ["black", "white"],
  "variants.price": [699, 799]
}
```

이 경우, "color가 black이고 price가 699인 variant"를 검색하려 할 때 문제가 발생합니다. Elasticsearch는 "black"이 첫 번째 variant의 color이고 "699"가 첫 번째 variant의 price라는 관계를 잃어버렸기 때문에, **color가 black인 variant와 price가 799인 variant**도 이 조건과 일치하는 것으로 간주할 수 있습니다.

### 중첩 객체 및 중첩 쿼리의 해결책

중첩 객체 타입과 중첩 쿼리를 사용하면 이 문제를 해결할 수 있습니다:

1. **중첩 객체 타입**: 객체 배열의 각 항목을 별도의 문서로 인덱싱하되, 부모 문서와의 관계를 유지합니다.
2. **중첩 쿼리**: 이러한 중첩 문서 내에서 쿼리를 실행하고, 일치하는 부모 문서를 반환합니다.

## 중첩 객체 매핑 설정

중첩 쿼리를 사용하기 전에, 해당 필드가 `nested` 타입으로 매핑되어 있어야 합니다:

```
PUT /products
{
  "mappings": {
    "properties": {
      "name": { "type": "text" },
      "variants": {
        "type": "nested",
        "properties": {
          "color": { "type": "keyword" },
          "price": { "type": "integer" }
        }
      }
    }
  }
}
```

## 중첩 쿼리 기본 구문

중첩 쿼리의 기본 구문은 다음과 같습니다:

```
GET /products/_search
{
  "query": {
    "nested": {
      "path": "variants",
      "query": {
        "bool": {
          "must": [
            { "term": { "variants.color": "black" } },
            { "range": { "variants.price": { "lte": 700 } } }
          ]
        }
      }
    }
  }
}
```

이 쿼리는:
1. `variants` 경로에 있는 중첩 객체 내에서 검색합니다.
2. `color`가 "black"이고 `price`가 700 이하인 variant가 있는 제품을 찾습니다.
3. **중요한 점**: 두 조건이 동일한 variant 객체 내에서 충족되어야 합니다.

## 중첩 쿼리의 주요 파라미터

중첩 쿼리는 다음과 같은 주요 파라미터를 지원합니다:

- **path**: 중첩 객체 필드의 경로
- **query**: 중첩 객체 내에서 실행할 쿼리
- **score_mode**: 여러 중첩 문서가 일치할 때 점수를 계산하는 방법 (avg, sum, min, max, none)
- **ignore_unmapped**: 지정된 경로가 매핑되지 않은 경우 오류 대신 빈 결과를 반환할지 여부

## 중첩 쿼리와 Inner Hits 결합

중첩 쿼리만으로는 어떤 중첩 객체가 조건과 일치했는지 알 수 없습니다. 이 정보를 얻으려면 `inner_hits` 파라미터를 함께 사용해야 합니다:

```
GET /products/_search
{
  "query": {
    "nested": {
      "path": "variants",
      "query": {
        "bool": {
          "must": [
            { "term": { "variants.color": "black" } },
            { "range": { "variants.price": { "lte": 700 } } }
          ]
        }
      },
      "inner_hits": {}
    }
  }
}
```

이렇게 하면 검색 결과에 일치하는 중첩 객체의 세부 정보가 포함됩니다.

## 중첩된 중첩 객체 검색

중첩 구조가 여러 단계로 이루어진 경우(예: 제품 > variants > reviews), 여러 중첩 쿼리를 중첩하여 사용할 수 있습니다:

```
GET /products/_search
{
  "query": {
    "nested": {
      "path": "variants",
      "query": {
        "bool": {
          "must": [
            { "term": { "variants.color": "black" } },
            {
              "nested": {
                "path": "variants.reviews",
                "query": {
                  "term": { "variants.reviews.rating": 5 }
                }
              }
            }
          ]
        }
      }
    }
  }
}
```

## 성능 고려사항

중첩 쿼리는 강력하지만, 다음과 같은 성능 영향을 고려해야 합니다:

1. 중첩 객체는 별도의 루씬 문서로 저장되므로, 인덱스 크기가 증가합니다.
2. 중첩 쿼리는 일반 쿼리보다 더 많은 처리를 필요로 합니다.
3. 중첩 객체의 수가 많을수록 쿼리 성능이 저하될 수 있습니다.

이러한 이유로, 중첩 객체를 과도하게 사용하거나 매우 큰 중첩 배열을 가진 문서는 피하는 것이 좋습니다.

## 실제 응용 사례

1. **E-commerce 카탈로그**: 색상, 크기 등 다양한 속성을 가진 제품 변형 검색
2. **리뷰 시스템**: 특정 평점이나 키워드가 포함된 리뷰가 있는 제품 검색
3. **HR 시스템**: 특정 기술이나 경험을 가진 직원이 있는 부서 검색
4. **여행 플랫폼**: 특정 시설이나 서비스를 제공하는 호텔 검색

중첩 쿼리는 복잡한 데이터 구조에서 필드 간의 관계를 유지하면서 정확한 검색을 수행해야 할 때 필수적인 도구입니다.
