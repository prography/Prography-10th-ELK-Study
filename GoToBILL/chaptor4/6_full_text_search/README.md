# Elasticsearch의 전체 텍스트 검색 (Full Text Search)

전체 텍스트 검색은 Elasticsearch의 가장 강력한 기능 중 하나로, 대량의 텍스트 데이터에서 관련성 높은 정보를 찾아내는 검색 기술입니다.

## 전체 텍스트 검색이란?

- 전체 텍스트 검색은 문서, 이메일, 웹페이지 등 대량의 텍스트 데이터에서 관련 정보를 찾는 강력한 기술입니다.
- term-level 쿼리와 달리 전체 텍스트 검색은 텍스트를 분석하여 더 유연하고 관련성 높은 결과를 제공합니다.
- 사용자가 입력한 검색어와 정확히 일치하는 것뿐만 아니라 의미적으로 관련된 문서도 찾아냅니다.

## 전체 텍스트 검색 작동 원리

### 1. 텍스트 분석 (Text Analysis)

텍스트 분석은 원본 텍스트를 검색 가능한 토큰(단어)으로 변환하는 과정입니다:

- **토큰화(Tokenization)**: 텍스트를 개별 단어나 용어로 분리
  ```
  "The quick brown fox" → ["The", "quick", "brown", "fox"]
  ```
- **소문자화(Lowercasing)**: 대소문자 구분을 없애 검색을 용이하게 함
  ```
  ["The", "quick", "brown", "fox"] → ["the", "quick", "brown", "fox"]
  ```
- **어간 추출(Stemming)**: 단어의 어근을 추출하여 다양한 형태의 단어를 매칭
  ```
  ["running", "runs"] → ["run", "run"]
  ```
- **불용어 제거(Stop word removal)**: "a", "the", "is"와 같은 일반적인 단어 제거
  ```
  ["the", "quick", "brown", "fox"] → ["quick", "brown", "fox"]
  ```

### 2. 역색인 (Inverted Index)

- 역색인은 토큰에서 문서로의 매핑을 저장하는 자료구조입니다.
- 각 토큰마다 해당 토큰이 등장하는 모든 문서의 목록을 저장합니다.
- 이를 통해 특정 단어가 포함된 모든 문서를 빠르게 찾을 수 있습니다.

예시:
```
"quick" → [document1, document3]
"fox" → [document1, document5]
"brown" → [document1, document7]
```

### 3. 검색 쿼리 처리

사용자가 검색 쿼리를 입력하면:
1. 쿼리 텍스트도 동일한 분석 과정을 거칩니다.
2. 분석된 쿼리 용어를 역색인에서 검색합니다.
3. 일치하는 문서를 찾아 관련성 점수를 계산합니다.

### 4. 관련성 점수 계산 (Relevance Scoring)

Elasticsearch는 TF-IDF(Term Frequency-Inverse Document Frequency) 모델, BM25, 또는 이 모델들의 조합을 사용하여 검색 결과의 관련성을 계산합니다:

- **용어 빈도(Term Frequency, TF)**: 문서 내에서 특정 용어가 얼마나 자주 등장하는지 측정합니다. 용어가 문서에 더 자주 나타날수록 관련성이 높아집니다.

- **역문서 빈도(Inverse Document Frequency, IDF)**: 전체 인덱스에서 특정 용어가 얼마나 희소한지 측정합니다. 희귀한 용어일수록 더 높은 가중치를 받습니다. "the"나 "a"와 같은 흔한 단어는 낮은 IDF 값을 가지며, 전문 용어나 고유 명사는 높은 IDF 값을 가집니다.

- **필드 길이 정규화(Field Length Norm)**: 짧은 필드에 나타나는 용어는 긴 필드에 나타나는 동일한 용어보다 더 관련성이 높을 수 있습니다. 예를 들어, 제목에 나타나는 키워드는 본문에 나타나는 동일한 키워드보다 더 중요할 수 있습니다.

## 주요 전체 텍스트 검색 쿼리 유형

### 1. match 쿼리

가장 기본적인 전체 텍스트 검색 쿼리입니다.

```
GET /my_index/_search
{
  "query": {
    "match": {
      "description": "quick brown fox"
    }
  }
}
```

- 쿼리 텍스트는 분석되어 여러 용어로 분리됩니다.
- 기본적으로 OR 연산을 사용하며, "quick", "brown", "fox" 중 하나라도 포함된 문서를 찾습니다.
- `operator` 파라미터를 사용하여 AND 연산으로 변경할 수 있습니다.

### 2. match_phrase 쿼리

단어의 순서까지 고려하는 쿼리입니다.

```
GET /my_index/_search
{
  "query": {
    "match_phrase": {
      "description": "quick brown fox"
    }
  }
}
```

- "quick brown fox"가 정확히 이 순서로 나타나는 문서만 일치합니다.
- `slop` 파라미터를 사용하여 단어 사이에 허용되는 다른 단어의 수를 지정할 수 있습니다.

### 3. multi_match 쿼리

여러 필드에서 동시에 검색하는 쿼리입니다.

```
GET /my_index/_search
{
  "query": {
    "multi_match": {
      "query": "quick brown fox",
      "fields": ["title", "description", "content"]
    }
  }
}
```

- 지정된 모든 필드에서 검색어를 찾습니다.
- 필드별 가중치를 다르게 설정할 수 있습니다: `"fields": ["title^3", "description^2", "content"]`

### 4. query_string 쿼리

고급 쿼리 구문을 지원하는 쿼리입니다.

```
GET /my_index/_search
{
  "query": {
    "query_string": {
      "query": "quick AND brown OR fox",
      "default_field": "description"
    }
  }
}
```

- AND, OR, NOT 연산자 지원
- 와일드카드, 퍼지 검색 등의 고급 기능 지원
- 복잡한 검색에 유용하지만 사용자 입력을 직접 받을 때는 주의가 필요합니다.

### 5. simple_query_string 쿼리

query_string의 더 안전한 버전입니다.

```
GET /my_index/_search
{
  "query": {
    "simple_query_string": {
      "query": "quick +brown -fox",
      "fields": ["title", "description"]
    }
  }
}
```

- 구문 오류가 발생하지 않으며, 잘못된 구문은 무시됩니다.
- `+`는 AND, `-`는 NOT, `|`는 OR 연산을 의미합니다.

## 실제 예제 해설

다음은 기사 검색 인덱스를 사용한 전체 텍스트 검색 예제입니다:

### 인덱스 및 문서 설정

```
DELETE /articles
PUT /articles
{
  "mappings": {
    "properties": {
      "title": {
        "type": "text"
      },
      "content": {
        "type": "text"
      },
      "summary": {
        "type": "text"
      }
    }
  }
}
```

이 코드는 `title`, `content`, `summary` 필드를 가진 `articles` 인덱스를 생성합니다. 모든 필드는 전체 텍스트 검색을 위한 `text` 타입으로 설정됩니다.

### 샘플 문서 색인

4개의 기사가 색인됩니다:

```
POST /articles/_doc/1
{
  "title": "Introduction to Machine Learning",
  "content": "Machine learning is a field of artificial intelligence that uses statistical techniques to give computers the ability to learn from data. Neural networks are a key component of many machine learning algorithms.",
  "summary": "An introduction to machine learning and neural networks."
}
```
(다른 문서들 생략)

### 기본 match 쿼리

```
GET /articles/_search
{
  "query": {
    "match": {
      "content": "learning"
    }
  }
}
```

이 쿼리는 `content` 필드에서 "learning"이라는 단어를 포함하는 모든 문서를 찾습니다. 문서 1, 2, 3이 모두 "learning"을 포함하고 있어 결과에 포함됩니다.

### 대소문자 구분 없는 검색

```
GET /articles/_search
{
  "query": {
    "match": {
      "content": "LEARNING"
    }
  }
}
```

이 쿼리는 `content` 필드에서 "LEARNING"(대문자)를 검색하지만, 텍스트 분석 과정에서 소문자화가 적용되므로 "learning"(소문자)을 포함하는 모든 문서를 찾습니다. 따라서 이전 쿼리와 동일한 결과가 반환됩니다.

### OR 연산자를 사용한 match 쿼리

```
GET /articles/_search
{
  "query": {
    "match": {
      "content": {
        "query": "Data Science",
        "operator": "or"
      }
    }
  }
}
```

이 쿼리는 `content` 필드에서 "Data" 또는 "Science"를 포함하는 문서를 찾습니다. `operator`를 명시적으로 "or"로 지정했습니다(이는 기본값이므로 생략 가능). 문서 4는 두 단어를 모두 포함하므로 가장 높은 점수를 받고, 다른 문서들은 "science"를 포함하지 않지만 "data"를 포함하므로 결과에 포함됩니다.

### AND 연산자를 사용한 match 쿼리

```
GET /articles/_search
{
  "query": {
    "match": {
      "content": {
        "query": "Data Science",
        "operator": "and"
      }
    }
  }
}
```

이 쿼리는 `content` 필드에서 "Data"와 "Science" 모두를 포함하는 문서를 찾습니다. `operator`를 "and"로 지정했으므로 두 단어가 모두 필요합니다. 문서 4만이 두 단어를 모두 포함하므로 결과에 단독으로 포함됩니다.

## 전체 텍스트 검색 최적화 팁

1. **적절한 분석기 선택**: 언어에 맞는 분석기를 사용하여 검색 품질을 향상시킵니다.
   
2. **필드 가중치 조정**: 더 중요한 필드(예: 제목)에 더 높은 가중치를 부여합니다.
   
3. **동의어 사용**: 동의어 매핑을 설정하여 관련 검색어도 결과에 포함시킵니다.
   
4. **하이라이팅**: 검색 결과에서 일치하는 부분을 강조 표시합니다.
   
5. **필드 중첩**: n-gram 또는 edge n-gram 토큰화를 사용하여 부분 일치나 자동 완성 기능을 구현합니다.

## 전체 텍스트 검색 vs Term 레벨 쿼리

| 특성 | 전체 텍스트 검색 | Term 레벨 쿼리 |
|------|-----------------|-------------|
| 텍스트 분석 | 분석 수행 | 분석하지 않음 |
| 대소문자 구분 | 구분하지 않음 (기본값) | 구분함 |
| 부분 단어 일치 | 가능 (분석기에 따라) | 불가능 (정확한 일치만) |
| 관련성 점수 | 계산함 | 일치/불일치만 판단 |
| 사용 사례 | 블로그 포스트, 제품 설명, 이메일 등의 텍스트 검색 | 정확한 값 필터링 (ID, 코드, 상태 등) |
