# Elasticsearch 검색 정리

## 1. 검색 방법

Elasticsearch에서는 **다큐먼트(Document)**를 검색할 때 여러 방법을 제공한다.

### 1.1 URL Search (URI Search)

- HTTP 요청의 URL에 쿼리 파라미터(`q`)를 직접 작성하여 간단한 검색을 수행한다.
- **형식**:
    
    ```
    GET /<index>/_search?q=<field>:<value>
    
    ```
    
- **예시**:
    
    ```bash
    GET /products/_search?q=name:iphone
    
    ```
    
    → `products` 인덱스에서 `name` 필드가 `iphone`인 문서를 검색한다.
    
- **주의점**:
    - **복잡한 쿼리 작성이 어렵다**: 단순한 조건 검색에만 적합하다.
    - **공백이나 특수문자**가 포함된 경우 반드시 **URL 인코딩**을 해주어야 한다.
        
        (예: `q=name:iphone%20pro`)
        
    - 스코어 점수 계산 등 세밀한 제어는 불가능하다.

### 1.2 DSL (Domain Specific Language) Search

-  형식으로 훨씬 복잡하고 정교한 쿼리를 작성할 수 있다.
- 다양한 조건(AND, OR, NOT), 필터링, 정렬, 페이징 등을 자유롭게 설정할 수 있다.
- **형식**:
    
    ```bash
    GET /<index>/_search
    {
      "query": {
        ...
      }
    }
    
    ```
    
- **예시**:
    
    ```bash
    GET /products/_search
    {
      "query": {
        "match": {
          "name": "iphone"
        }
      }
    }
    
    ```
<br>

```
- **URL Search는 단순 검색용**으로만 사용하는 것이 좋다.
- 복잡한 조건(AND/OR 조합, 필터링, 범위 조건 등)은 반드시 **DSL**을 사용한다.
- 성능 최적화를 위해 **필터링(filter)** 과 **정렬(sort)** 을 분리하는 것이 좋다.
```

---

## 1. Term-Level Query란?

- **정확히 일치하는 값**을 찾는 쿼리이다.
- 검색어를 **아날라이즈(analyze)** 하지 않고, 입력한 **값 그대로** 매칭을 시도한다.
- 주로 **비분석 필드**(analyzed=false, 예: `keyword`, `integer`, `date`, `boolean`)에 사용한다.
- **대소문자 구분(case-sensitive)** 된다.

> ❗ 텍스트 필드(text 타입)처럼 분석(analyze)된 필드에는 제대로 동작하지 않을 수 있다.
> 

---

## 2. Term-Level Query의 특징

| 항목 | 내용 |
| --- | --- |
| 매칭 방식 | 정확한 값 일치 (analyze 없음) |
| 적용 필드 | `keyword`, `date`, `long`, `integer`, `boolean` 타입 등 |
| 대소문자 구분 | O (case-sensitive) |
| 분석 여부 | X (tokenizer, analyzer 적용되지 않음) |
| 용도 | 아이디, 상태값, 태그, 날짜, 숫자 등 정형 데이터 조회 |

---

## 3. 왜 Term-Level Query가 필요한가?

- 텍스트 기반 `match` 쿼리는 내부적으로 **토크나이저**와 **필터**를 적용하여 분석(analyze)한 후 매칭한다.
- 예를 들어, `match` 쿼리는 `"Quick Brown Fox"`를
    
    → `quick`, `brown`, `fox` 로 토큰화해서 검색한다.
    
- 반면 **Term-Level Query**는 **원본 그대로** `Quick Brown Fox`를 찾는다.
    
    **소문자로 변환하거나 어간 추출(stemming)하지 않는다.**
    

---

## 4. 주요 Term-Level Queries 종류

### 4.1 term query

- 하나의 값이 정확히 일치하는 문서를 찾는다.

**예시**:

```
GET /products/_search
{
  "query": {
    "term": {
      "status.keyword": "available"
    }
  }
}

```

> status 필드가 available인 문서만 반환.
> 

---

### 4.2 terms query

- 여러 값을 리스트로 넣어, **여러 값 중 하나라도** 일치하면 검색한다.

**예시**:

```
GET /products/_search
{
  "query": {
    "terms": {
      "category.keyword": ["electronics", "appliances"]
    }
  }
}

```

> category가 electronics 또는 appliances인 문서를 모두 찾는다.
> 

---

### 4.3 range query (참고)

- Term-Level과 비슷하게 숫자, 날짜의 **범위**를 검색할 때 사용된다.

**예시**:

```
GET /products/_search
{
  "query": {
    "range": {
      "price": {
        "gte": 100,
        "lte": 500
      }
    }
  }
}

```

> 가격이 100 이상, 500 이하인 상품 검색.
> 

(※ 하지만 range는 "term" 범주에는 포함되지 않고 별도 range query로 구분함)

---

## 5. Term-Level Query를 사용할 때 주의사항

- `text` 타입 필드에는 직접 사용하면 원하는 결과가 안 나올 수 있다.
    - **해결 방법**: `.keyword` 서브필드를 사용해야 한다. (ex: `status.keyword`)
- **대소문자 주의**: "Apple"과 "apple"은 다르게 인식된다.
- **불필요한 분석기 영향 없음**: 숫자나 boolean에 잘 동작한다.
- **terms 쿼리는 1만개 이하 권장**: 너무 많은 값을 리스트로 던지면 성능 저하가 발생할 수 있다.

---

## 6. Term Query vs Match Query 비교

| 항목 | Term Query | Match Query |
| --- | --- | --- |
| 분석 여부 | X (Analyze 안 함) | O (Analyze 적용) |
| 매칭 방식 | 완전 일치 (Exact Match) | 토큰화 후 매칭 (단어 기반) |
| 주요 대상 필드 | `keyword`, `integer`, `date`, `boolean` | `text` |
| 대소문자 민감성 | 민감(case-sensitive) | 대개 무시(case-insensitive) |
| 예시 | 사용자 ID, 상품 상태, 날짜 | 글 제목, 설명, 본문 검색 |

## 1. Prefix Search

**설명**

- 특정 **필드의 값이 주어진 prefix로 시작**하는 다큐먼트를 검색하는 방법.
- 주로 **자동완성(AutoComplete)** 기능 구현에 사용된다.
    
    (예: `iph`를 입력하면 `iPhone`, `iPhone 12` 결과 출력)
    

**특징**

- **Case-Sensitive** (대소문자 구분)
- **Prefix 길이가 짧을수록** 검색 대상이 많아져 성능 저하가 발생할 수 있다.
- **모든 다큐먼트를 순회**해야 하므로, **대규모 데이터셋에서는 비추천**.

**쿼리 예시**

```
{
  "query": {
    "prefix": {
      "product_name": {
        "value": "iph"
      }
    }
  }
}

```

**주의사항**

- 분석되지 않은 필드(`keyword` 타입)에서 사용하는 것이 일반적이다.
- 대규모 데이터에서는 `edge_ngram` tokenizer를 통한 자동완성 대체가 더 효율적일 수 있다.

---

## 2. Wildcard Search

**설명**

- **특정 패턴**을 만족하는 문자열을 찾을 때 사용.

**문자**

- : 0개 이상의 문자와 매칭
- `?` : 정확히 하나의 문자와 매칭

**특징**

- **맨 앞에 를 사용하는 검색 (`phone`)은 매우 비효율적**이다. (인덱스 전체 스캔)
- **가능하면 앞 wildcard는 피하고**, 접두어 고정 후 검색하는 것을 권장.

**쿼리 예시**

```
{
  "query": {
    "wildcard": {
      "product_name.keyword": {
        "value": "iph*"
      }
    }
  }
}

```

---

## 3. Regular Expression Search

**설명**

- 정규 표현식을 이용하여 복잡한 패턴 매칭을 수행.

**지원 문법 예시**

- `.` : 임의의 문자 1개
- `.*` : 모든 문자열
- `[abc]` : a, b, c 중 하나
- `[a-z]` : 소문자 알파벳 1개
- `[0-9]` : 숫자 하나

**특징**

- **계산량이 매우 크다.**
- 인덱스를 직접 탐색해야 해서 대량 데이터에서는 심각한 성능 저하가 생길 수 있다.

**쿼리 예시**

```
{
  "query": {
    "regexp": {
      "username.keyword": "user[0-9]{3}"
    }
  }
}

```

**주의사항**

- prefix, wildcard와 마찬가지로 분석되지 않은 필드(`keyword`)에 사용하는 것이 안전하다.

---

## 4. EXISTS QUERY

**설명**

- **필드가 존재하는 다큐먼트**만 검색할 때 사용.

**특징**

- 필드가 없거나, 값이 `null`이거나, 비어있는 배열인 경우 false로 간주된다.

**쿼리 예시**

```
{
  "query": {
    "exists": {
      "field": "tags"
    }
  }
}

```

**조합 사용 (예시: tags가 존재하지 않는 경우 검색)**

```
{
  "query": {
    "bool": {
      "must_not": {
        "exists": {
          "field": "tags"
        }
      }
    }
  }
}

```

---

## 5. Full Text Search

**설명**

- **텍스트를 분석(analysis)** 한 뒤 **역색인된 데이터**를 검색하는 방식.
- 단순 키워드 매칭이 아니라, **어절 분석**, **소문자화**, **불용어 제거** 등을 적용.

**검색 흐름**

1. 입력 텍스트를 **analyzer**가 전처리한다.
2. 전처리된 텍스트를 기반으로 검색한다.
3. **Relevance Score**를 계산해서 결과를 정렬한다.

**Relevance 계산 요소**

- **TF (Term Frequency)**: 문서 내 등장 빈도
- **IDF (Inverse Document Frequency)**: 전체 문서 중 드물게 나오는 단어일수록 가중치 높음
- **Field Length Norm**: 필드 길이가 짧을수록 가중치 높음

**OR Operator**

- 기본적으로 단어 사이에 **OR 조건**으로 검색된다.
- `"operator": "and"` 옵션을 주면 두 단어 모두 포함하는 다큐먼트만 조회된다.

**쿼리 예시 (기본 OR)**

```
{
  "query": {
    "match": {
      "content": {
        "query": "data science",
        "operator": "or"
      }
    }
  }
}

```

**쿼리 예시 (AND 강제)**

```
{
  "query": {
    "match": {
      "content": {
        "query": "data science",
        "operator": "and"
      }
    }
  }
}

```

---

## 6. Multi Match Search

**설명**

- 하나의 검색어를 여러 필드에 동시에 매칭시키는 방법.

**특징**

- 여러 필드에 대해 동시에 검색하고, **필드별로 가중치(Boost)를 줄 수 있다**.
- tie_breaker를 통해 비슷한 스코어 필드가 여러개 매칭될 경우 점수를 추가 부여할 수 있다.

**쿼리 예시**

```
{
  "query": {
    "multi_match": {
      "query": "iphone",
      "fields": ["title^3", "description", "tags"],
      "tie_breaker": 0.3
    }
  }
}

```

- `title` 필드 매칭 결과에 3배 가중치 부여
- `description`, `tags`는 기본 가중치
- tie_breaker를 통해 여러 필드 매칭시 추가 점수

### 참고사항

- 대량 데이터에서는 **prefix**, **wildcard**, **regexp** 검색을 최소화하고, **analyzed search** (full text search)로 우회하는 것이 성능상 유리하다.
- 자동완성 기능을 구현할 때는 **edge ngram tokenizer**를 이용한 커스텀 analyzer를 사용하면 훨씬 빠르고 효율적이다.
- 검색 정확도를 높이기 위해 **Synonym Token Filter**, **Shingle Token Filter** 등을 추가로 활용할 수 있다.
- 점수 조정이 필요할 경우 **function_score query**를 사용하여 직접 커스텀 scoring이 가능하다.

## Leaf Query

**정의**: 가장 단순한 쿼리. 하나의 필드에 하나의 조건만을 검사하며, 쿼리 안에 다른 쿼리가 없는 형태.

- **Field Level** 검색
- **Single Condition**
- **No Nesting**

### 예시

```


GET /my_index/_search
{
  "query": {
    "term": {
      "status": "published"
    }
  }
}

```

> status 필드가 정확히 published인 문서만 검색 (analyzed되지 않은 keyword 필드여야 함).
> 

---

## Compound Query

**정의**: 여러 쿼리를 조합하여 복잡한 검색을 수행하는 쿼리. 내부에 하나 이상의 쿼리를 포함함.

---

### 1. **Bool Query**

논리 조건을 조합하여 검색하는 가장 대표적인 compound query.

| 옵션 | 의미 | relevance score 영향 |
| --- | --- | --- |
| `must` | AND 조건 (모두 일치해야 함) | O |
| `must_not` | NOT 조건 (일치하면 제외) | O |
| `should` | OR 조건 (하나 이상 일치하면 됨) | O |
| `filter` | 조건은 필수지만 score엔 영향 없음 | ✨빠름 (cache됨) |

### 예시

```


GET /products/_search
{
  "query": {
    "bool": {
      "must": [
        { "term": { "category": "electronics" }},
        { "term": { "in_stock": true }}
      ]
    }
  }
}

```

---

### 2. **Boosting Query**

**목적**: 특정 조건의 문서는 점수를 높이고(positive), 특정 조건은 점수를 낮추고(negative) 싶을 때 사용.

### 예시

```
GET /articles/_search
{
  "query": {
    "boosting": {
      "positive": {
        "match": { "content": "ai" }
      },
      "negative": {
        "match": { "content": "spam" }
      },
      "negative_boost": 0.5
    }
  }
}

```

> "ai"라는 단어가 포함된 문서를 우선적으로, "spam" 포함 문서는 점수를 낮춰서 정렬.
> 

---

### 3. **Disjunction Max Query (dis_max)**

**정의**: 여러 서브 쿼리를 실행하고, 그 중 가장 높은 점수를 결과로 사용. OR 조건이지만 가장 적합한 문서에 높은 점수를 주고 싶을 때 사용.

- `tie_breaker`: 여러 쿼리가 동시에 매칭되었을 때 가산 점을 줄 수 있음.

### 예시

```
GET /books/_search
{
  "query": {
    "dis_max": {
      "queries": [
        { "match": { "title": "climbing" }},
        { "match": { "description": "climbing" }}
      ],
      "tie_breaker": 0.3
    }
  }
}

```

> 제목과 설명 둘 다 일치하면 가산점이 붙고, 하나만 일치해도 검색됨.
> 

---

## Nested Query

**정의**: Elasticsearch에서는 일반 배열 구조는 내부 요소 간의 관계를 고려하지 않음.

`nested` 타입으로 정의된 문서에서 **객체 단위의 관계**를 유지하면서 검색해야 할 때 사용.

### 예시 상황

```
"comments": [
  { "user": "alice", "message": "good" },
  { "user": "bob", "message": "bad" }
]

```

> 일반 쿼리로는 user: bob + message: good 이 서로 다른 객체에 있어도 매칭됨 → 의도한 결과 아님.
> 

### 예시

```
GET /posts/_search
{
  "query": {
    "nested": {
      "path": "comments",
      "query": {
        "bool": {
          "must": [
            { "match": { "comments.user": "alice" }},
            { "match": { "comments.message": "good" }}
          ]
        }
      }
    }
  }
}

```

---

## Inner Hits

**정의**: `nested` 쿼리로 검색된 문서 내부에서 **매칭된 nested 객체를 함께 반환**하도록 설정하는 기능.

→ 어떤 부분이 일치했는지 확인 가능

### 예시

```
GET /posts/_search
{
  "query": {
    "nested": {
      "path": "comments",
      "query": {
        "match": { "comments.message": "great" }
      },
      "inner_hits": {}
    }
  }
}

```

> 매칭된 comments 객체를 inner_hits로 반환 → 디버깅/하이라이팅 등에 유용
>
