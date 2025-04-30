# Elasticsearch에서의 검색 방법

Elasticsearch에서는 검색을 수행하는 두 가지 주요 방법이 있습니다:

## URI 검색 (URI Searches)
- URI 검색은 HTTP 요청의 URL 내에서 직접 검색을 수행하는 간단한 방법입니다.
- 이 방법은 DSL 쿼리에 비해 기능이 제한적이지만, 빠르고 간단한 검색에 편리합니다.
- 예: `GET /my_index/_search?q=content:quick`

## DSL 쿼리 (Domain Specific Language Queries)
- DSL 쿼리는 URI 검색보다 더 강력하고 유연합니다.
- Elasticsearch의 쿼리 DSL은 복잡한 쿼리를 구성할 수 있는 JSON 기반 언어입니다.
- 전체 텍스트 검색, 불리언 쿼리, 필터링, 집계 등 다양한 기능을 지원합니다.
- 예: 
    ```
    GET /my_index/_search
    {
      "query": {
        "match": {
          "content": "quick"
        }
      }
    }
    ```

---

```
GET /my_index/_search?q=content:quick AND content:walk
GET /my_index/_search?q=content:quick%20AND%20content:walk
```
위의 쿼리가 잘 될 것 같지만 사실 잘 안 됐습니다.

**아래의 결과처럼 모든 결과 나오는 불상사가 일어났습니다.**

원인은 AND 사이의 공백이 인코딩 에러를 유발했습니다.

따라서 위의 쿼리가 아닌 공백을 `%20`으로 처리해준 쿼리를 실행할 때, 정상적으로 총 결과가 `quick`과 `walk`가 모두 들어간 doc가 출력됩니다. 
```
{
  "took": 2,
  "timed_out": false,
  "_shards": {
    "total": 1,
    "successful": 1,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": {
      "value": 3,
      "relation": "eq"
    },
    "max_score": 0.13353139,
    "hits": [
      {
        "_index": "my_index",
        "_id": "A874gJYBboFKWQtQj4EO",
        "_score": 0.13353139,
        "_source": {
          "content": "quick book",
          "publish_date": "2024-02-01"
        }
      },
      {
        "_index": "my_index",
        "_id": "BM74gJYBboFKWQtQlYGH",
        "_score": 0.13353139,
        "_source": {
          "content": "quick walk",
          "publish_date": "2024-03-01"
        }
      },
      {
        "_index": "my_index",
        "_id": "Bc74gJYBboFKWQtQl4HK",
        "_score": 0.13353139,
        "_source": {
          "content": "quick walk",
          "publish_date": "2024-03-01"
        }
      }
    ]
  }
}
```

## Elasticsearch 쿼리 유형별 해설

아래는 각 Elasticsearch 쿼리의 상세 설명과 용도입니다.

### 1. match 쿼리

```
GET /my_index/_search
{
  "query": {
    "match": {
      "content": "quick"
    }
  }
}
```

**해설**:
- `match` 쿼리는 전체 텍스트 검색을 위한 기본 쿼리입니다.
- "quick"이라는 단어가 `content` 필드에 포함된 모든 문서를 검색합니다.
- 이 쿼리는 분석기를 통해 검색어를 토큰화하고, 필드의 값과 매칭시킵니다.
- 대소문자를 구분하지 않으며, "Quick"이나 "QUICK"을 검색해도 동일한 결과가 나옵니다.

### 2. query_string 쿼리 (기본)

```
GET /my_index/_search
{
  "query": {
    "query_string": {
      "query": "quick"
    }
  }
}
```

**해설**:
- `query_string`은 더 복잡한 검색 문법을 지원하는 고급 쿼리 타입입니다.
- 기본적으로 모든 필드를 대상으로 검색합니다.
- 이 예제에서는 간단히 "quick"이라는 단어가 포함된 문서를 검색합니다.
- `match` 쿼리와 비슷하지만, 더 다양한 연산자(AND, OR, NOT 등)를 지원합니다.

### 3. query_string 쿼리 (OR 연산자 및 필드 지정)

```
GET /my_index/_search
{
  "query": {
    "query_string": {
      "query": "quick OR brown",
      "fields": ["content"]
    }
  }
}
```

**해설**:
- `quick` 또는 `brown` 중 하나라도 포함된 문서를 검색합니다(OR 연산).
- `fields` 파라미터를 통해 검색 대상을 `content` 필드로 제한합니다.
- "quick"이 포함된 문서와 "brown"이 포함된 문서 모두 결과에 포함됩니다.

### 4. bool 쿼리 (must 절)

```
GET /my_index/_search
{
  "query": {
    "bool": {
      "must": [
        { "match": { "content": "quick" } },
        { "match": { "content": "walk" } }
      ]
    }
  }
}
```

**해설**:
- `bool` 쿼리는 여러 쿼리 조건을 결합하는 복합 쿼리입니다.
- `must` 절은 모든 조건이 반드시 충족되어야 합니다(AND 연산).
- 이 쿼리는 "quick"과 "walk" 두 단어가 모두 포함된 문서만 찾습니다.
- 따라서 "quick walk"를 포함한 문서는 검색되지만, "quick book"은 검색되지 않습니다.

### 5. bool 쿼리 (filter 절과 term 쿼리)

```
GET /my_index/_search
{
  "query": {
    "bool": {
      "filter": {
        "term": { "content": "quick" }
      }
    }
  }
}
```

**해설**:
- `filter` 절은 스코어를 계산하지 않고 단순히 조건에 맞는 문서만 필터링합니다(스코어가 0으로 표시됨).
- `term` 쿼리는 검색어를 분석하지 않고 정확히 일치하는 토큰을 찾습니다.
- **중요**: `content` 필드가 `text` 타입이면 문서 내용이 토큰화되어 저장됩니다. 따라서 "quick book"과 "quick walk"는 모두 "quick" 토큰을 포함하고 있어서, `term` 쿼리로 "quick"을 검색하면 모든 문서가 반환됩니다.
- 전체 문구를 정확히 검색하려면 `content.keyword`를 사용해야 합니다: `"term": { "content.keyword": "quick book" }`
- 필터 쿼리는 캐싱되므로 자주 사용되는 필터는 성능이 향상됩니다.

### 6. range 쿼리

```
GET /my_index/_search
{
  "query": {
    "range": {
      "publish_date": {
        "gte": "2024-01-01",
        "lte": "2024-02-28"
      }
    }
  }
}
```

**해설**:
- `range` 쿼리는 특정 범위 내의 값을 검색합니다.
- 이 쿼리는 `publish_date` 필드가 2024년 1월 1일부터 2월 28일 사이인 문서를 검색합니다.
- `gte`는 "greater than or equal to"(이상), `lte`는 "less than or equal to"(이하)를 의미합니다.
- 날짜 외에도 숫자, 문자열 등 다양한 타입에 적용할 수 있습니다.

### 7. match_phrase 쿼리

```
GET /my_index/_search
{
  "query": {
    "match_phrase": {
      "content": "quick book"
    }
  }
}
```

**해설**:
- `match_phrase` 쿼리는 정확한 구문을 검색합니다.
- 단어들의 순서와 인접성이 중요합니다.
- 이 쿼리는 "quick"과 "book"이 정확히 이 순서로 인접해 있는 문서만 찾습니다.
- "quick red book"은 매칭되지 않으며, "book quick" 역시 순서가 다르므로 매칭되지 않습니다.

#### 예시
```
{
  "content": "This is a quick book about programming."
}
```
- 이때는 제대로 검색이 돼서 반환이 됩니당.

### 8. wildcard 쿼리

```
GET /my_index/_search
{
  "query": {
    "wildcard": {
      "content": "qui*"
    }
  }
}
```

**해설**:
- `wildcard` 쿼리는 와일드카드 패턴을 사용한 검색을 지원합니다.
- `*`는 0개 이상의 문자와 일치합니다.
- 이 쿼리는 "qui"로 시작하는 모든 용어가 포함된 문서를 찾습니다(예: "quick", "quill", "quinoa" 등).
- **주의**: 와일드카드 쿼리는 성능에 영향을 줄 수 있으므로, 특히 선행 와일드카드(`*abc`)는 가급적 피해야 합니다.

### 9. match_all 쿼리

```
GET /my_index/_search
{
  "query": {
    "match_all": {}
  }
}
```

**해설**:
- `match_all` 쿼리는 인덱스의 모든 문서를 반환합니다.
- 필터나 조건 없이 모든 문서를 검색하고자 할 때 사용합니다.
- 기본적으로 각 문서의 스코어는 1.0입니다.
- 인덱스 내용을 확인하거나 디버깅할 때 유용합니다.
