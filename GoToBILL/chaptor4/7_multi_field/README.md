# 멀티 매치 쿼리(Multi-Match Query)

멀티 매치 쿼리는 Elasticsearch에서 동일한 검색어를 여러 필드에서 검색할 수 있게 해주는 쿼리 유형입니다. 이는 여러 필드에 걸쳐 동일한 검색을 수행해야 할 때 매우 유용합니다.

## 멀티 매치 쿼리의 기본 구조

```
GET /my_index/_search
{
  "query": {
    "multi_match": {
      "query": "검색어",
      "fields": ["필드1", "필드2", "필드3"]
    }
  }
}
```

## 주요 특징

1. **여러 필드 검색**: 한 번의 쿼리로 여러 필드에서 동일한 검색어를 찾을 수 있습니다.

2. **필드별 가중치 조정**: 특정 필드에 더 높은 가중치를 부여할 수 있습니다.
   ```
   "fields": ["title^3", "description^2", "content"]
   ```
   위 예제에서 title 필드는 3배, description 필드는 2배, content 필드는 기본 가중치(1)를 가집니다.

3. **다양한 매치 유형**: 여러 매치 유형(type)을 지정할 수 있습니다:
   - `best_fields`: 가장 잘 매치되는 필드의 점수를 중심으로 계산 (기본값)
   - `most_fields`: 매치되는 모든 필드의 점수를 합산
   - `cross_fields`: 여러 필드를 하나의 큰 필드처럼 취급
   - `phrase`: 구문 검색 수행
   - `phrase_prefix`: 구문 접두사 검색 수행

## 예제

### 기본 사용법
```
GET /blog/_search
{
  "query": {
    "multi_match": {
      "query": "elasticsearch guide",
      "fields": ["title", "content", "summary"]
    }
  }
}
```

이 쿼리는 "elasticsearch guide"라는 검색어를 title, content, summary 세 필드에서 검색합니다.

### 필드 가중치 지정
```
GET /blog/_search
{
  "query": {
    "multi_match": {
      "query": "elasticsearch guide",
      "fields": ["title^3", "content", "summary^2"]
    }
  }
}
```

이 쿼리는 title 필드에서 찾은 결과에 3배, summary 필드에 2배의 가중치를 부여합니다.

### 매치 유형 지정
```
GET /blog/_search
{
  "query": {
    "multi_match": {
      "query": "elasticsearch guide",
      "fields": ["title", "content", "summary"],
      "type": "most_fields"
    }
  }
}
```

이 쿼리는 "most_fields" 유형을 사용하여 매치되는 모든 필드의 점수를 합산합니다.

### 퍼지 검색 적용
```
GET /blog/_search
{
  "query": {
    "multi_match": {
      "query": "elasticsearch guide",
      "fields": ["title", "content", "summary"],
      "fuzziness": "AUTO"
    }
  }
}
```

이 쿼리는 오타에 대한 허용 범위를 자동으로 설정하여 검색합니다.

## 다양한 매치 유형(type) 상세 설명

### 1. best_fields (기본값)
- 가장 높은 점수를 받은 필드의 점수를 기준으로 문서 점수를 계산합니다.
- 단일 필드에서 모든 검색어가 발견될 때 높은 관련성을 갖습니다.
- `tie_breaker` 매개변수를 사용하여 다른 매칭 필드의 점수를 일부 반영할 수 있습니다.
- 최종 점수 = 최고 점수 + (다른 필드의 점수 × tie_breaker 값)

```
GET /blog/_search
{
  "query": {
    "multi_match": {
      "query": "elasticsearch guide",
      "fields": ["title", "content", "summary"],
      "type": "best_fields",
      "tie_breaker": 0.3
    }
  }
}
```

### 2. most_fields
- 모든 매칭 필드의 점수를 합산합니다.
- 여러 필드에 걸쳐 같은 단어가 등장할 때 해당 문서의 관련성이 높아집니다.
- 동의어를 다른 필드에 저장한 경우 유용합니다.

```
GET /blog/_search
{
  "query": {
    "multi_match": {
      "query": "elasticsearch guide",
      "fields": ["title", "content", "summary", "tags"],
      "type": "most_fields"
    }
  }
}
```

### 3. cross_fields
- 여러 필드를 하나의 큰 필드처럼 처리합니다.
- 검색어의 각 용어가 어느 필드에 있든 상관없이 모두 발견되면 높은 점수를 얻습니다.
- 관련 정보가 여러 필드에 분산되어 있을 때 유용합니다(예: 사람 이름이 first_name과 last_name 필드에 나뉘어 있는 경우).

```
GET /people/_search
{
  "query": {
    "multi_match": {
      "query": "John Smith",
      "fields": ["first_name", "last_name"],
      "type": "cross_fields",
      "operator": "and"
    }
  }
}
```

### 4. phrase
- 각 필드에 대해 phrase 쿼리를 실행합니다.
- 검색어 용어의 순서와 근접성이 중요할 때 사용합니다.

```
GET /blog/_search
{
  "query": {
    "multi_match": {
      "query": "elasticsearch guide",
      "fields": ["title", "content"],
      "type": "phrase",
      "slop": 2
    }
  }
}
```

### 5. phrase_prefix
- phrase 쿼리와 유사하지만 마지막 용어는 접두사로 취급합니다.
- 자동 완성 기능을 구현할 때 유용합니다.

```
GET /blog/_search
{
  "query": {
    "multi_match": {
      "query": "elasticsearch gui",
      "fields": ["title", "content"],
      "type": "phrase_prefix",
      "max_expansions": 10
    }
  }
}
```

## 실제 사용 사례

1. **검색 엔진**: 사용자가 검색창에 입력한 내용을 제목, 내용, 태그 등 여러 필드에서 검색합니다.

2. **제품 검색**: 제품명, 설명, 특징, 브랜드 등 여러 필드에서 사용자의 검색어와 일치하는 제품을 찾습니다.

3. **문서 검색**: 제목, 본문, 작성자, 태그 등 문서의 다양한 부분에서 검색을 수행합니다.

4. **연락처 검색**: 이름, 성, 이메일, 회사명 등 여러 필드에서 검색어와 일치하는 연락처를 찾습니다.

멀티 매치 쿼리는 사용자 친화적인 검색 경험을 제공하고 싶을 때 특히 유용하며, 다양한 매개변수를 통해 검색 동작을 세밀하게 제어할 수 있습니다.
