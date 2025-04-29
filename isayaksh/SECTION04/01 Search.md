# Search
- ES에서 조회를 수행하는 2가지 방법이 있다.

## URI Search
- HTTP 요청의 URL 내에서 직접 검색을 수행하는 간단한 방법입니다. 이 방법은 DSL 쿼리에 비해 제한적이지만 빠르고 간단한 검색에 편리합니다.

```
GET /{index}/_search?q=content:quick
```

## DSL(Domain Specific Language) 쿼리
- URI 검색에 비해 더 강력하고 유연합니다. Elasticsearch의 쿼리 DSL은 복잡한 쿼리를 작성할 수 있는 JSON 기반 언어입니다. 이는 전체 텍스트 검색, 불리언 쿼리, 필터링, 집계 등을 지원합니다.
- 특정 도메인을 조회할 때 최적화된 방법입니다.

### Basic Search
- content 필드에 "quick"이라는 구문이 포함된 문서를 검색합니다.
```
GET /{index}/_search
{
    "query": {
        "match": {
            "content": "quick"
        }
    }
}
```

### Full-Text Search with Query String
- 고급 문법을 사용하여 여러 필드에 걸쳐 전체 텍스트 검색을 수행하려면 `query_string` 쿼리를 사용할 수 있습니다.
```
GET /{index}/_search
{
    "query": {
        "query_string": {
            "query": "quick"
        }
    }
}

GET /{index}/_search
{
    "query": {
        "query_string": {
            "query": "quick OR Brown",
            "fields": ["content"]
        }
    }
}
```

### Boolean Queries
- 불리언 쿼리를 사용하면 `must`, `should`, `must_not`과 같은 논리 연산자를 이용해 여러 쿼리를 결합할 수 있습니다.
```
GET /{index}/_search
{
    "query": {
        "bool": {
            "must": [
                { "match": {"content": "quick"} },
                { "match": {"content": "walk"} }
            ]
        }
    }
}
```

### Filtering Results
- 필터는 **관련성 점수(relevance scoring)에 영향을 주지 않고**, 구조화된 조건을 기반으로 문서를 걸러낼 필요가 있을 때 사용됩니다.

```
GET /{index}/_search
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

### Range Queries
- 범위 쿼리를 사용하면 필드 값이 특정 범위(예: 숫자 범위, 날짜 범위) 안에 있는 문서를 찾을 수 있습니다.

```
GET /{index}/_search
{
    "query": {
        "range": {
            "publish_date" : {
                "gte": "2024-01-01",
                "lte": "2024-12-31"
            }
        }
    }
}
```

### Phrase Search
- 정확한 단어 시퀀스를 찾기 위해 구문 검색을 수행할 수 있습니다.

```
GET /{index}/_search
{
    "query": {
        "match_phrase": {
            "content": "quick brown"
        }
    }
}
```

### (Wildcard Search)
- 와일드카드 검색을 사용하면 * (모든 문자 시퀀스와 일치) 또는 ? (임의의 한 문자와 일치) 같은 패턴을 사용하여 검색할 수 있습니다.

```
GET /{index}/_search
{
    "query": {
        "wildcard": {
            "content": "quic*"
        }
    }
}
```

### 🔥 URI Search가 실제로 사용되는 경우
1. Kibana DevTools나 브라우저에서 "빠른 테스트"할 때
- 복잡한 쿼리 짤 필요 없이 바로 URL에 q= 붙여 검색해볼 수 있음.
- ex: 브라우저 주소창에 바로 입력하거나 curl 한 줄로 테스트.

2. 단순한 키워드 검색 API를 만들 때
- 만약 서비스에서 매우 간단한 검색 기능만 필요한 경우(ex: "제목에 특정 단어 포함된 게시물 찾기")
- DSL로 별도 JSON 짤 필요 없이 바로 q=제목:검색어로 해결.

3. 디버깅이나 임시 데이터 조회할 때
- 서버 로그나 인덱스에 "특정 문서만 찾고 싶다" 할 때
- 빠르게 찾아보기 위해 URI Search를 쓴다.

정리하면, **단순한 테스트 작업에서만 uri search를 사용하고 실제 운영에서는 dsl을 사용하면 된다!**