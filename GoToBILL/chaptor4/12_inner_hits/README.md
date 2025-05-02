# Inner Hits

Inner Hits는 Elasticsearch에서 중첩 문서나 중첩 쿼리의 세부 정보를 반환할 수 있게 해주는 기능입니다.

## Inner Hits란?

Inner Hits는 Elasticsearch에서 중첩된 문서(nested documents) 또는 중첩 쿼리 내의 개별 항목에 대한
상세 정보를 반환하는 기능입니다. 즉, 부모 문서가 중첩 쿼리와 일치할 때 어떤 특정 하위 문서(또는 sub-document)가 
그 일치의 원인이 되었는지 알 수 있게 해줍니다.

## 필요성

검색 결과에서 부모 문서가 반환될 때, 단순히 "이 문서가 검색 조건과 일치한다"는 정보만으로는 충분하지 않을 수 있습니다.
특히 문서 내에 중첩된 객체 배열이 있는 경우, 어떤 중첩 객체가 검색 조건과 일치했는지 파악하는 것이 중요할 수 있습니다.
Inner Hits는 이 문제를 해결합니다.

## 사용 방법

Inner Hits는 주로 다음 두 가지 쿼리 유형과 함께 사용됩니다:

1. **Nested Query**: 중첩 필드 내의 객체를 검색할 때
2. **Parent-Child Query**: 부모-자식 관계가 있는 문서를 검색할 때

### 예제: Nested Query와 함께 사용

```
GET /products/_search
{
  "query": {
    "nested": {
      "path": "reviews",
      "query": {
        "bool": {
          "must": [
            { "match": { "reviews.rating": 5 } },
            { "match": { "reviews.comment": "excellent" } }
          ]
        }
      },
      "inner_hits": {}  // Inner Hits 설정
    }
  }
}
```

이 쿼리는:
1. 평점이 5점이고 댓글에 "excellent"가 포함된 리뷰가 있는 제품을 검색합니다.
2. `inner_hits` 파라미터를 통해, 각 제품마다 어떤 리뷰가 이 조건과 일치했는지 반환합니다.

### 예제: Parent-Child Query와 함께 사용

```
GET /department/_search
{
  "query": {
    "has_child": {
      "type": "employee",
      "query": {
        "range": {
          "age": { "gte": 50 }
        }
      },
      "inner_hits": {}  // Inner Hits 설정
    }
  }
}
```

이 쿼리는:
1. 50세 이상의 직원이 있는 부서를 검색합니다.
2. `inner_hits` 파라미터를 통해, 각 부서마다 어떤 직원이 이 조건과 일치했는지 반환합니다.

## Inner Hits 옵션

Inner Hits는 다양한 옵션을 지원합니다:

- **size**: 반환할 inner hits의 최대 개수 (기본값: 3)
- **from**: 페이지네이션을 위한 시작 오프셋
- **sort**: inner hits의 정렬 방식 지정
- **_source**: 반환할 필드 지정
- **highlight**: 일치하는 텍스트 강조 표시

예제:

```
GET /products/_search
{
  "query": {
    "nested": {
      "path": "reviews",
      "query": {
        "match": { "reviews.comment": "great" }
      },
      "inner_hits": {
        "size": 5,
        "sort": [
          { "reviews.date": { "order": "desc" } }
        ],
        "_source": ["reviews.rating", "reviews.comment", "reviews.date"],
        "highlight": {
          "fields": {
            "reviews.comment": {}
          }
        }
      }
    }
  }
}
```

## 실제 응답 예시

```
{
  "hits": {
    "total": { "value": 1, "relation": "eq" },
    "hits": [
      {
        "_index": "products",
        "_id": "1",
        "_score": 1.0,
        "_source": {
          "name": "Smartphone XYZ",
          "reviews": [
            { "rating": 5, "comment": "Excellent product!", "date": "2023-05-15" },
            { "rating": 4, "comment": "Good but expensive", "date": "2023-04-20" }
          ]
        },
        "inner_hits": {
          "reviews": {
            "hits": {
              "total": { "value": 1, "relation": "eq" },
              "hits": [
                {
                  "_index": "products",
                  "_nested": {
                    "field": "reviews",
                    "offset": 0
                  },
                  "_score": 1.0,
                  "_source": {
                    "rating": 5,
                    "comment": "Excellent product!",
                    "date": "2023-05-15"
                  },
                  "highlight": {
                    "reviews.comment": [
                      "<em>Excellent</em> product!"
                    ]
                  }
                }
              ]
            }
          }
        }
      }
    ]
  }
}
```

이 응답에서 볼 수 있듯이, "Smartphone XYZ" 제품이 검색 결과로 반환되었고, `inner_hits` 섹션에는 이 제품의 어떤 리뷰가 쿼리 조건과 일치했는지 상세 정보가 포함되어 있습니다.

## 사용 사례

1. **E-commerce 플랫폼**: 특정 속성의 제품 검색 시, 어떤 속성이 검색과 일치했는지 표시
2. **리뷰 시스템**: 특정 키워드가 포함된 리뷰가 있는 제품 검색 시, 해당 리뷰 강조 표시
3. **로그 분석**: 특정 에러가 발생한 로그 항목 검색 시, 어떤 세부 항목이 에러와 관련있는지 표시
4. **문서 관리 시스템**: 중첩된 단락 내에서 특정 키워드 검색 시, 관련 단락 강조 표시

Inner Hits는 복잡한 중첩 구조를 가진 데이터에서 검색 결과의 투명성과 이해도를 높이는 데 매우 유용한 기능입니다.
