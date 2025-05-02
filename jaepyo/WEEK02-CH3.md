## Elasticsearch Analysis (분석기)

### Analysis란?

Elasticsearch에서 텍스트 데이터를 저장하거나 검색할 때 사용하는 전처리 과정입니다.
이 과정을 통해 텍스트를 작고 검색 가능한 단위(Term)로 나누고, 의미 없는 요소를 제거하거나 정규화합니다.

---

### Analysis 전체 처리 흐름

| 순서 | 구성요소 | 설명 |
| --- | --- | --- |
| 1 | Character Filter | 텍스트 내 HTML 태그 제거, 특수 문자 변환 등 |
| 2 | Tokenizer | 문장을 단어 단위로 나누는 작업 (e.g., 공백/구두점 기준) |
| 3 | Token Filter | 불용어 제거, 소문자화, 어간 추출(Stemming) 등 |

> 이 과정을 거쳐 생성된 Term들이 색인되어 검색에 사용됩니다.
> 

---

### 예시

**입력 문장**: "Running in the park is fun!"
**분석기**: "english"

```
1. Character Filter  →  "Running in the park is fun!"
2. Tokenizer         →  ["Running", "in", "the", "park", "is", "fun"]
3. Token Filter      →  ["run", "park", "fun"]
```

---

---

### 왜 중요한가?

- 색인 시 "running" → "run"으로 저장되었는데
- 검색 시 "running"이 "run"으로 분석되지 않으면
    
    → 매칭 실패
    

→ 이를 방지하려면 검색 시에도 동일한 analyzer 사용이 필수입니다.

---

### 어떤 쿼리에 Analyzer가 적용될까?

### 분석이 적용되는 쿼리 (Full-text Query)

| 쿼리 종류 | 설명 |
| --- | --- |
| match | 가장 일반적인 검색 쿼리 (analyzed 필드 검색) |
| match_phrase | 단어 순서까지 고려 |
| multi_match | 여러 필드를 동시에 match |
| common_terms | 자주 등장하는 단어는 relevance 낮게 |

```
GET /my_index/_search
{
  "query": {
    "match": {
      "description": "Running in the park"
    }
  }
}
```

### 분석이 적용되지 않는 쿼리 (Term-Level Query)

| 쿼리 종류 | 설명 |
| --- | --- |
| term | 정확히 일치하는 값을 검색 (분석 안 함) |
| terms | 여러 개의 정확한 값 검색 |
| range | 범위 쿼리 (`gt`, `lt` 등) |
| exists | 필드 존재 여부 검사 |

```
GET /my_index/_search
{
  "query": {
    "term": {
      "status": "Published"
    }
  }
}
```

→ 분석되지 않기 때문에 "status": "published"로 색인되어 있으면 매칭되지 않음

---

### Analyzer 종류

| 분석기 이름 | 설명 | 특징 |
| --- | --- | --- |
| `standard` | 기본 분석기. 대부분의 상황에 적절 | 소문자화, 유니코드 문자 제거, 표준 토크나이저 사용 |
| `simple` | 단순 분석기. 알파벳만 추출 | 대소문자 무시, 비문자(숫자, 기호) 제거 |
| `whitespace` | 공백 기준으로만 분리 | 소문자화 없음, 단순 공백 분리 |
| `keyword` | 전체 텍스트를 하나의 토큰으로 처리 | 분석 없이 문자열 전체 저장. 정렬/필터용 |
| `pattern` | 정규표현식 기반 분석 | 원하는 구분자로 텍스트 분할 가능 |
| `language` | 언어 전용 분석기 (예: `english`, `korean`) | 불용어 제거, 어간 추출(Stemming) 등 언어 특화 처리 포함 |

### `_analyze` API 예시

```
POST /_analyze
{
  "analyzer": "english",
  "text": "Running in the park"
}
```

→ 결과: ["run", "park"]

---

## Inverted Index (역색인)

Elasticsearch는 텍스트 검색 성능을 높이기 위해 역색인 구조를 사용합니다.

### 개념

> 어떤 문서에 어떤 단어가 포함되어 있는지 저장하는 구조
> 
- `term`: 분석된 단어 (예: run)
- `posting list`: term이 포함된 문서 ID 목록
- `document`: 저장된 개별 JSON 문서

### 예시

**문서 1**: "I love climbing" → ["i", "love", "climbing"]

**문서 2**: "Climbing is fun" → ["climbing", "is", "fun"]

```
"climbing" → [1, 2]
"love"     → [1]
"fun"      → [2]
```

### 동작 방식

1. 텍스트 분석
2. term 추출
3. 각 term에 해당하는 문서 ID 목록(posting list) 구성

---

## Mapping (매핑)

### 개념

Elasticsearch에서의 스키마 정의.
데이터를 색인하기 전에 필드의 데이터 타입과 처리 방법을 정의합니다.

### 주요 구성 요소

- `index`: 색인 이름
- `document`: 저장되는 JSON 객체
- `field`: document 내 개별 키
- `data_type`: 필드의 데이터 타입

### 필드 타입 종류

### 문자열

- `text`: 분석됨. Full-text 검색에 사용
- `keyword`: 분석되지 않음. 정렬, 집계 등에 적합

### 숫자 / 날짜 / 논리값

- `integer`, `long`, `float`, `double`
- `date`: 날짜 타입, `format` 지정 가능
- `boolean`: true/false

### 복합 타입

- `object`: 일반 JSON 객체
- `nested`: 배열 객체 내부 필드 간 관계 유지

### 특수 타입

- `geo_point`, `ip`, `binary` 등

---

## 매핑 변경 - Reindexing

Elasticsearch는 기존 필드의 `type`을 변경할 수 없습니다.
→ 이유: Lucene 인덱스는 필드 타입 기반으로 최적화됨 (변경 불가)

### 해결책: reindex

1. 새 인덱스 생성 (변경된 매핑 적용)
2. `_reindex` API로 기존 인덱스 → 새 인덱스로 데이터 복사
3. `alias`로 기존 인덱스 이름을 새 인덱스로 매핑

```
POST /_reindex
{
  "source": { "index": "old_index" },
  "dest":   { "index": "new_index" }
}
```

> 필드 타입을 바꾸려면 이 과정을 반드시 거쳐야 합니다.
> 

---

## Multi-fields Mapping

하나의 필드를 여러 방식으로 색인하는 기능입니다.

### 예시

```
"title": {
  "type": "text",
  "fields": {
    "raw": { "type": "keyword" }
  }
}
```

→ `title`: 텍스트 검색

→ `title.raw`: 정확히 일치하는 정렬/집계용 필드

---

## Index Templates

새로운 인덱스가 생성될 때 적용할 기본 설정과 매핑을 정의하는 구조

### 사용 예시

- 로그 인덱스: `logs-*`
- 지표 인덱스: `metrics-*`

### 포함 요소

- 인덱스 패턴: 어떤 이름에 적용할지 (`logs-*` 등)
- 매핑: 필드 정의
- 세팅: 샤드 수, 레플리카 수 등
- 앨리어스: 논리적 이름 매핑

---

## Dynamic Mapping (동적 매핑)

매핑을 명시하지 않고 문서를 색인하면, Elasticsearch가 자동으로 타입을 추론합니다.

### 기본 동작

| 데이터 예시 | 추론된 타입 |
| --- | --- |
| "abc" | `text` + `keyword` 하위필드 |
| 123 | `long` |
| 3.14 | `float` |
| true | `boolean` |
| "2024-01-01" | `date` |

### 설정 옵션

- `dynamic: true` (기본) → 자동 매핑
- `dynamic: false` → 매핑 안함 (필드 무시)
- `dynamic: strict` → 정의되지 않은 필드가 오면 에러

---

## Stemming

단어의 어간(기본형)으로 변환하는 작업

| 원본 | 결과 |
| --- | --- |
| running, ran, runs | run |

→ "run" 하나로 통합해서 검색 정확도 향상

## Stop Words

의미 없는 단어(the, is, a 등)를 제거해서 검색 성능과 정확도 향상
