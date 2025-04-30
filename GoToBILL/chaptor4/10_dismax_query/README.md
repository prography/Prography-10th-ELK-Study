# Disjunction Max Query (dis_max)

Disjunction Max Query(dis_max)는 Elasticsearch에서 여러 쿼리의 최고 점수만을 사용하여 문서의 관련성을 평가하는 복합 쿼리 유형입니다.

## Disjunction Max Query란?

Disjunction Max Query(종종 dis_max로 축약됨)는 여러 쿼리를 실행하고, 각 쿼리가 생성한 점수 중 **가장 높은 점수**를 해당 문서의 최종 관련성 점수로 사용합니다. 이는 여러 필드에서 동일한 검색어를 찾을 때, 점수가 인위적으로 inflate(부풀려지는) 되는 것을 방지합니다.

## 작동 원리

### 1. Best Score Wins

dis_max 쿼리는 하위 쿼리들 중에서 가장 높은 점수를 문서의 최종 관련성 점수로 선택합니다. 예를 들어:

- title 필드에서의 점수: 0.8
- content 필드에서의 점수: 0.5
- description 필드에서의 점수: 0.3

dis_max 쿼리는 이 중 가장 높은 0.8을 최종 점수로 사용합니다.

### 2. Tie Breaking

단순히 최고 점수만 사용하면, 여러 필드에서 검색어와 일치하는 문서가 하나의 필드에서만 강하게 일치하는 문서와 동일한 점수를 받게 됩니다. 이 문제를 해결하기 위해 dis_max는 `tie_breaker` 매개변수를 제공합니다.

`tie_breaker`는 0과 1 사이의 값으로, 나머지 점수의 일부를 최종 점수에 반영합니다:

최종 점수 = 최고 점수 + (다른 모든 점수 × tie_breaker)

예를 들어, `tie_breaker`가 0.3일 때:
- 최종 점수 = 0.8 + (0.5 × 0.3) + (0.3 × 0.3) = 0.8 + 0.15 + 0.09 = 1.04

## 기본 구문

```
GET /my_index/_search
{
  "query": {
    "dis_max": {
      "queries": [
        { "match": { "title": "quick brown fox" }},
        { "match": { "content": "quick brown fox" }},
        { "match": { "description": "quick brown fox" }}
      ],
      "tie_breaker": 0.3
    }
  }
}
```

## 실제 예제

다음은 제품 검색에서 dis_max 쿼리를 사용하는 예입니다:

```
GET /products/_search
{
  "query": {
    "dis_max": {
      "queries": [
        { "match": { "name": "Speaker" }},
        { "match": { "category": "electronics" }}
      ],
      "tie_breaker": 0.3
    }
  }
}
```

이 쿼리는:
1. 이름에 "Speaker"가 포함되었거나 카테고리가 "electronics"인 제품을 검색합니다.
2. 각 필드에서 가장 높은 점수를 가진 제품이 상위에 표시됩니다.
3. `tie_breaker` 값 0.3으로 인해, 여러 필드에서 일치하는 제품이 약간 더 높은 순위를 갖습니다.

## multi_match 쿼리와의 비교

Elasticsearch는 `multi_match` 쿼리를 통해 여러 필드에서 동일한 검색어를 편리하게 검색할 수 있는 방법을 제공합니다. `multi_match`의 `type` 파라미터에 따라 점수 계산 방식이 달라집니다:

### multi_match 쿼리 (type: best_fields)

`best_fields` 타입(기본값)의 `multi_match` 쿼리는 내부적으로 `dis_max` 쿼리를 사용합니다:

```
GET /my_index/_search
{
  "query": {
    "multi_match": {
      "query": "quick brown fox",
      "fields": ["title", "content", "description"],
      "type": "best_fields",
      "tie_breaker": 0.3
    }
  }
}
```

이는 앞서 본 `dis_max` 예제와 동일한 동작을 합니다.

### multi_match 쿼리 (type: most_fields)

반면 `most_fields` 타입은 모든 필드의 점수를 합산합니다:

```
GET /my_index/_search
{
  "query": {
    "multi_match": {
      "query": "quick brown fox",
      "fields": ["title", "content", "description"],
      "type": "most_fields"
    }
  }
}
```

## dis_max 쿼리가 유용한 상황

1. **여러 필드에서 동일한 검색어 검색**: 제목, 내용, 요약 등 여러 필드에서 검색어를 찾을 때
2. **필드 중요도 차별화**: 일부 필드(예: 제목)에서의 일치를 다른 필드보다 중요하게 취급하고 싶을 때
3. **과도한 점수 합산 방지**: 여러 필드에서 부분적으로 일치하는 문서보다 하나의 필드에서 강하게 일치하는 문서를 우선시하고 싶을 때
4. **다양한 쿼리 유형 결합**: 서로 다른 유형의 쿼리(match, term, range 등)의 결과를 결합할 때

## tie_breaker 값의 의미

- **0**: 완전히 승자독식 방식. 최고 점수만 사용하고 다른 모든 점수는 무시합니다.
- **1**: 모든 점수의 합을 사용합니다 (most_fields와 유사).
- **0~1 사이**: 최고 점수에 다른 점수의 일부를 더합니다. 일반적으로 0.1~0.4 범위의 값이 많이 사용됩니다.

적절한 `tie_breaker` 값은 데이터와 검색 요구사항에 따라 다릅니다. 값이 클수록 여러 필드에서 일치하는 문서가 더 높은 순위를 갖지만, 너무 높으면 dis_max 쿼리의 원래 의도(가장 관련성 높은 필드 강조)가 희석될 수 있습니다.

## 고급 사용 예제

다양한 쿼리 유형과 가중치를 결합한 더 복잡한 예제:

```
GET /products/_search
{
  "query": {
    "dis_max": {
      "queries": [
        { "match": { "name": { "query": "wireless headphone", "boost": 2.0 } }},
        { "match": { "description": "wireless headphone" }},
        { "term": { "tags": { "value": "audio", "boost": 1.5 } }},
        {
          "bool": {
            "should": [
              { "term": { "brand": "Sony" }},
              { "term": { "brand": "Bose" }}
            ]
          }
        }
      ],
      "tie_breaker": 0.2
    }
  }
}
```

이 쿼리는:
1. 이름에 "wireless headphone"이 포함된 제품에 가장 높은 가중치(2.0)를 부여합니다.
2. 설명에 "wireless headphone"이 포함된 제품을 검색합니다.
3. "audio" 태그가 있는 제품에 중간 가중치(1.5)를 부여합니다.
4. Sony 또는 Bose 브랜드 제품을 검색합니다.
5. 각 문서에 대해 위 쿼리 중 가장 높은 점수를 사용하고, `tie_breaker`(0.2)를 통해 다른 쿼리의 점수도 일부 반영합니다.

Disjunction Max Query는 여러 필드나 다양한 쿼리 조건에서 검색할 때 관련성 점수를 더 직관적이고 균형 있게 계산하는 데 매우 유용한 쿼리 유형입니다.
