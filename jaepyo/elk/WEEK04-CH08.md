## 1. Proximity Search (근접 검색)

문장 내 단어 사이의 거리를 허용하여 검색하는 기능. 정확히 붙어 있지 않아도 일정 간격(sloppy match)이면 검색됩니다.

### `match_phrase` + `slop`

- `match_phrase`: 단어 순서를 유지하면서 인접 단어를 찾는 쿼리
- `slop`: 중간에 몇 개의 단어까지 끼어 있어도 허용할지 설정
    
    (`slop = 2`면 최대 2단어 차이 허용)
    

### 예시

```json
GET /docs/_search
{
  "query": {
    "match_phrase": {
      "content": {
        "query": "quick fox",
        "slop": 2
      }
    }
  }
}

```

- `"The quick brown red fox"` 도 매칭됨 (`brown`, `red`가 사이에 있어도 slop ≤ 2)

---

## 2. Fuzzy Match Query (오타 허용 검색)

단어의 철자 오류(타이포)가 있을 때도 검색되도록 허용하는 기능

### `fuzziness`, `prefix_length`

- `fuzziness`: 허용되는 편집 거리 (Levenshtein Distance)
    
    예: `"roam"` ≈ `"foam"`
    
- `prefix_length`: 몇 글자까지는 정확히 일치해야 허용할지 설정

### 예시

```json
GET /users/_search
{
  "query": {
    "fuzzy": {
      "name": {
        "value": "jhon",
        "fuzziness": "1",
        "prefix_length": 1
      }
    }
  }
}

```

- `"john"`도 검색 가능 (`j`는 일치, `h`→`o` 한 글자 차이 허용)

---

## 3. Synonym Search (동의어 검색)

단어는 다르지만 의미가 같을 때, 같은 의미로 검색할 수 있게 하는 기능

### 동작 방식

1. 동의어 파일 또는 리스트 정의 (`ai, artificial intelligence`)
2. custom analyzer 생성
3. 해당 analyzer를 인덱스 매핑에 적용

### 동의어 필터 정의 예시

```json
PUT /my_index
{
  "settings": {
    "analysis": {
      "filter": {
        "synonym_filter": {
          "type": "synonym",
          "synonyms": [
            "ai, artificial intelligence",
            "tv, television"
          ]
        }
      },
      "analyzer": {
        "synonym_analyzer": {
          "tokenizer": "standard",
          "filter": [
            "lowercase",
            "synonym_filter"
          ]
        }
      }
    }
  },
  "mappings": {
    "properties": {
      "content": {
        "type": "text",
        "analyzer": "synonym_analyzer"
      }
    }
  }
}

```

- 모든 노드에 동일한 설정이 적용되어야 클러스터 일관성 유지 가능

---

## 4. Highlight (하이라이트)

검색 결과 중 쿼리와 매칭된 부분을 강조 태그로 감싸서 반환합니다.

### 예시

```json
GET /articles/_search
{
  "query": {
    "match": {
      "body": "elasticsearch"
    }
  },
  "highlight": {
    "fields": {
      "body": {}
    }
  }
}

```

### 기본 출력 결과

```json
"highlight": {
  "body": ["...<em>elasticsearch</em> is a search engine..."]
}

```

- `<em>` 태그는 기본 강조 태그이며, `pre_tags`, `post_tags`로 변경 가능
