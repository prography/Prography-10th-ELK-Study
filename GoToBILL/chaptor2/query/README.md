# Elasticsearch Query (쿼리)

Elasticsearch는 다양한 유형의 쿼리를 지원하여 데이터를 검색하고 필터링하는 강력한 기능을 제공합니다. 이 문서에서는 Elasticsearch의 주요 쿼리 유형과 사용 방법에 대해 설명합니다.

## 1. Query와 Filter의 차이점

Elasticsearch에서 데이터를 검색할 때 크게 두 가지 컨텍스트가 있습니다:

### 1.1 Query Context (쿼리 컨텍스트)
- 문서가 쿼리와 **얼마나 잘 일치하는지** 계산합니다.
- 결과에 **관련성 점수(_score)**를 부여합니다.
- 전문 검색(full-text search)에 적합합니다.
- 예: `match`, `match_phrase`, `multi_match` 등

### 1.2 Filter Context (필터 컨텍스트)
- 문서가 조건과 **일치하는지 여부(예/아니오)**만 확인합니다.
- 관련성 점수를 계산하지 않습니다.
- 캐싱이 가능하여 성능이 더 좋습니다.
- 예: `term`, `terms`, `range`, `exists` 등

## 2. 주요 쿼리 유형

### 2.1 Term Level Queries (용어 수준 쿼리)

인덱스에 정확히 저장된 용어를 검색하는 쿼리입니다. 분석되지 않은 정확한 값을 찾을 때 사용합니다.

#### term 쿼리
정확한 용어와 일치하는 문서를 찾습니다.

```
GET /my_index/_search
{
  "query": {
    "term": {
      "status": {
        "value": "active"
      }
    }
  }
}
```

#### terms 쿼리
여러 용어 중 하나와 일치하는 문서를 찾습니다.

```
GET /my_index/_search
{
  "query": {
    "terms": {
      "tags": ["search", "elasticsearch", "query"]
    }
  }
}
```

#### range 쿼리
숫자나 날짜의 범위를 지정합니다.

```
GET /my_index/_search
{
  "query": {
    "range": {
      "age": {
        "gte": 20,
        "lte": 40
      }
    }
  }
}
```

#### exists 쿼리
특정 필드가 존재하는 문서를 찾습니다.

```
GET /my_index/_search
{
  "query": {
    "exists": {
      "field": "email"
    }
  }
}
```

#### prefix 쿼리
특정 접두사로 시작하는 용어를 찾습니다.

```
GET /my_index/_search
{
  "query": {
    "prefix": {
      "username": "john"
    }
  }
}
```

### 2.2 Full-Text Queries (전문 검색 쿼리)

텍스트 분석을 거친 후 검색하는 쿼리입니다. 텍스트를 분석하고 관련성을 계산합니다.

#### match 쿼리
기본적인 전문 검색 쿼리로, 지정된 텍스트와 일치하는 문서를 찾습니다.

```
GET /my_index/_search
{
  "query": {
    "match": {
      "description": "elasticsearch query"
    }
  }
}
```

#### match_phrase 쿼리
정확한 구문을 검색합니다. 단어의 순서와 인접성이 중요합니다.

```
GET /my_index/_search
{
  "query": {
    "match_phrase": {
      "description": "powerful search engine"
    }
  }
}
```

#### multi_match 쿼리
여러 필드에서 동일한 텍스트를 검색합니다.

```
GET /my_index/_search
{
  "query": {
    "multi_match": {
      "query": "elasticsearch",
      "fields": ["title", "description", "content"]
    }
  }
}
```

#### query_string 쿼리
Lucene 쿼리 구문을 사용한 고급 검색을 수행합니다.

```
GET /my_index/_search
{
  "query": {
    "query_string": {
      "default_field": "content",
      "query": "elasticsearch AND (query OR search)"
    }
  }
}
```

### 2.3 Compound Queries (복합 쿼리)

여러 쿼리를 조합하는 쿼리입니다.

#### bool 쿼리
Boolean 논리를 사용하여 여러 쿼리를 조합합니다.

```
GET /my_index/_search
{
  "query": {
    "bool": {
      "must": [
        { "match": { "title": "elasticsearch" } }
      ],
      "filter": [
        { "term": { "status": "published" } },
        { "range": { "date": { "gte": "2023-01-01" } } }
      ],
      "should": [
        { "match": { "description": "search" } }
      ],
      "must_not": [
        { "term": { "tags": "outdated" } }
      ]
    }
  }
}
```

- `must`: 반드시 일치해야 하며, 점수에 영향을 줍니다.
- `filter`: 반드시 일치해야 하지만, 점수에 영향을 주지 않습니다.
- `should`: 일치하면 점수가 높아지지만, 필수는 아닙니다.
- `must_not`: 일치하지 않아야 합니다.

#### function_score 쿼리
검색 결과의 점수를 조정하는 쿼리입니다.

```
GET /my_index/_search
{
  "query": {
    "function_score": {
      "query": { "match": { "title": "elasticsearch" } },
      "functions": [
        {
          "field_value_factor": {
            "field": "popularity",
            "factor": 1.2,
            "modifier": "log1p"
          }
        },
        {
          "gauss": {
            "date": {
              "origin": "now",
              "scale": "30d",
              "decay": 0.5
            }
          }
        }
      ],
      "boost_mode": "multiply"
    }
  }
}
```

## 3. 쿼리 성능 최적화

### 3.1 필터 사용하기
- 가능한 경우 쿼리보다 필터를 사용하세요.
- 필터는 캐시되어 성능이 더 좋습니다.
- 정확한 일치, 범위 등의 조건은 필터로 처리하세요.

### 3.2 검색 비용이 적은 쿼리 선택
- `term`, `terms`, `range` 쿼리는 비용이 낮습니다.
- `wildcard`, `regexp`, `prefix` 쿼리는 비용이 더 높습니다.
- 가능하면 비용이 낮은 쿼리를 사용하세요.

### 3.3 bool 쿼리 최적화
- 필터 조건을 `must` 대신 `filter`에 배치하세요.
- `must_not`을 과도하게 사용하지 않도록 주의하세요.
- 복잡한 `bool` 쿼리를 여러 단계로 나누는 것을 고려하세요.

### 3.4 페이지네이션 최적화
- 깊은 페이지네이션은 피하세요.
- Search After API 또는 Scroll API를 사용하세요.

```
// Search After API 예시
GET /my_index/_search
{
  "size": 10,
  "sort": [
    { "date": "desc" },
    { "_id": "desc" }
  ],
  "search_after": ["2023-05-01", "last_document_id"],
  "query": {
    "match": { "status": "active" }
  }
}
```

## 4. 고급 쿼리 기법

### 4.1 Query와 Filter의 조합
일반적으로 다음과 같은 패턴이 권장됩니다:

```
GET /my_index/_search
{
  "query": {
    "bool": {
      "must": [
        { "match": { "title": "elasticsearch" } }
      ],
      "filter": [
        { "term": { "status": "active" } },
        { "range": { "date": { "gte": "now-1y" } } }
      ]
    }
  }
}
```

### 4.2 Boosting
특정 필드나 쿼리의 점수를 조정할 수 있습니다:

```
GET /my_index/_search
{
  "query": {
    "bool": {
      "should": [
        { "match": { "title": { "query": "elasticsearch", "boost": 3 } } },
        { "match": { "description": "elasticsearch" } }
      ]
    }
  }
}
```

### 4.3 Synonym 사용
유사어 검색을 활용하여 검색 결과를 개선할 수 있습니다:

```
// 인덱스 설정에 synonym filter 추가
PUT /my_index
{
  "settings": {
    "analysis": {
      "filter": {
        "my_synonym_filter": {
          "type": "synonym",
          "synonyms": [
            "elasticsearch, elastic, es",
            "search, find, query"
          ]
        }
      },
      "analyzer": {
        "my_synonym_analyzer": {
          "tokenizer": "standard",
          "filter": ["lowercase", "my_synonym_filter"]
        }
      }
    }
  },
  "mappings": {
    "properties": {
      "title": {
        "type": "text",
        "analyzer": "my_synonym_analyzer"
      }
    }
  }
}
```

## 5. 쿼리 디버깅 및 분석

### 5.1 Explain API
쿼리 실행 계획과 점수 계산 방식을 확인할 수 있습니다:

```
GET /my_index/_explain/1
{
  "query": {
    "match": { "title": "elasticsearch" }
  }
}
```

### 5.2 Profile API
쿼리 성능을 분석할 수 있습니다:

```
GET /my_index/_search
{
  "profile": true,
  "query": {
    "match": { "title": "elasticsearch" }
  }
}
```

## 6. 실무 적용 사례

### 6.1 검색 결과 개선
- 관련성 높은 필드에 가중치 부여
- 최신 문서에 더 높은 점수 부여
- 사용자 행동 데이터 활용

### 6.2 자동 완성 구현
- Completion Suggester 또는 Edge N-gram 사용
- 타이핑할 때마다 검색 쿼리 실행

### 6.3 다국어 검색
- 언어별 분석기 사용
- 번역 서비스와 연동

## 결론

Elasticsearch는 다양한 쿼리 유형과 기능을 제공하여 복잡한 검색 요구사항을 충족시킬 수 있습니다. 적절한 쿼리 유형을 선택하고, 필터를 효과적으로 활용하며, 성능 최적화 기법을 적용하여 강력한 검색 기능을 구현할 수 있습니다.

효율적인 검색을 위해서는 데이터 모델과 검색 요구사항을 고려하여 매핑을 설계하고, 적절한 분석기를 선택하는 것이 중요합니다. 또한, 지속적인 모니터링과 튜닝을 통해 검색 성능을 개선해 나가야 합니다.