# Elasticsearch 쿼리 구조: 리프 쿼리와 복합 쿼리

Elasticsearch의 쿼리는 그 구조와 복잡성에 따라 크게 리프 쿼리(Leaf Queries)와 복합 쿼리(Compound Queries)로 구분됩니다.

## 리프 쿼리 (Leaf Queries)

리프 쿼리는 Elasticsearch에서 가장 단순한 형태의 쿼리로, 특정 필드에서 특정 값을 검색하는 데 사용됩니다.

### 리프 쿼리의 주요 특징

- **가장 단순한 형태**: 다른 쿼리를 포함하지 않는 가장 기본적인 쿼리 유형
- **특정 필드 검색**: 문서의 특정 필드에서 지정된 값을 검색
- **단일 조건**: 하나의 조건만 평가 (예: 특정 용어 일치, 범위 검사, 존재 여부 확인)
- **중첩 없음**: 다른 쿼리를 포함하지 않고 직접 조건을 데이터와 대조

### 주요 리프 쿼리 유형과 예시

#### 1. term 쿼리 - 정확한 값 매칭

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
이 쿼리는 `status` 필드 값이 정확히 "published"인 문서만 검색합니다.

#### 2. range 쿼리 - 범위 검색

```
GET /my_index/_search
{
  "query": {
    "range": {
      "publish_date": {
        "gte": "2023-01-01",
        "lte": "2023-12-31"
      }
    }
  }
}
```
이 쿼리는 `publish_date` 필드 값이 2023년 1월 1일부터 12월 31일 사이인 문서를 검색합니다.

#### 3. match 쿼리 - 텍스트 분석 후 검색

```
GET /my_index/_search
{
  "query": {
    "match": {
      "content": "Elasticsearch"
    }
  }
}
```
이 쿼리는 `content` 필드가 "Elasticsearch"를 포함하는 문서를 검색합니다. `match` 쿼리는 텍스트를 분석한 후 검색하므로 전체 텍스트 검색에 적합합니다.

## 복합 쿼리 (Compound Queries)

복합 쿼리는 여러 쿼리를 논리적으로 결합하여 더 복잡하고 정교한 검색 로직을 구현하는 데 사용됩니다.

### 복합 쿼리의 주요 특징

- **쿼리 조합**: 여러 쿼리를 논리 연산자로 결합
- **중첩 구조**: 복합 쿼리 내에 다른 복합 쿼리 포함 가능
- **실행 제어**: 서브 쿼리의 실행 방식과 결과 조합 방법 제어
- **복잡한 검색 로직**: SQL의 WHERE 절과 유사한 복잡한 조건 구현 가능

### Bool 쿼리 - 가장 강력한 복합 쿼리

Bool 쿼리는 Elasticsearch에서 가장 많이 사용되는 복합 쿼리로, 여러 조건을 논리적으로 결합할 수 있습니다.

#### Bool 쿼리의 주요 구성 요소

- **must**: AND 연산자와 동일. 모든 조건이 충족되어야 함
- **must_not**: NOT 연산자와 동일. 해당 조건이 충족되지 않아야 함
- **should**: OR 연산자와 동일. 조건이 충족되면 관련성 점수 증가
- **filter**: must와 유사하지만 관련성 점수 계산 없이 문서 필터링 (성능 최적화)

#### Bool 쿼리 예제와 SQL 비교

1. **AND 조건 (must)**

SQL:
```sql
SELECT * FROM products 
WHERE category = 'electronics' 
AND in_stock = true;
```

Elasticsearch:
```
GET /products/_search
{
  "query": {
    "bool": {
      "must": [
        { "term": { "category": "electronics" } },
        { "term": { "in_stock": true } }
      ]
    }
  }
}
```

2. **OR 조건 (should)**

SQL:
```sql
SELECT * FROM products 
WHERE category = 'electronics' 
OR price < 300;
```

Elasticsearch:
```
GET /products/_search
{
  "query": {
    "bool": {
      "should": [
        { "term": { "category": "electronics" } },
        { "range": { "price": { "lt": 300 } } }
      ],
      "minimum_should_match": 1
    }
  }
}
```

3. **복잡한 조건 조합**

SQL:
```sql
SELECT * FROM products 
WHERE category = 'electronics' 
AND (price < 300 OR in_stock = true) 
AND NOT discontinued = true;
```

Elasticsearch:
```
GET /products/_search
{
  "query": {
    "bool": {
      "must": [
        { "term": { "category": "electronics" } },
        {
          "bool": {
            "should": [
              { "range": { "price": { "lt": 300 } } },
              { "term": { "in_stock": true } }
            ]
          }
        }
      ],
      "must_not": [
        { "term": { "discontinued": true } }
      ]
    }
  }
}
```

4. **필터링 (filter)**

SQL:
```sql
SELECT * FROM products 
WHERE category = 'electronics' 
AND price < 300 
AND in_stock = true;
```

Elasticsearch:
```
GET /products/_search
{
  "query": {
    "bool": {
      "must": [
        { "term": { "category": "electronics" } }
      ],
      "filter": [
        { "range": { "price": { "lt": 300 } } },
        { "term": { "in_stock": true } }
      ]
    }
  }
}
```
`필터링`이 그래서 뭔데? 라는 의문이 들 수도 있습니다.
- `"filter 절"`이라는 말은 Elasticsearch에서 filter 절이 검색 결과의 관련성 점수계산에 영향을 주지 않는다는 의미입니다.


`관련성 점수`라는게 그래서 뭔데?
- 관련성 점수는 검색 쿼리와 문서가 얼마나 잘 일치하는지를 수치화한 값입니다. 
- `Elasticsearch`에서는 이 점수를 기반으로 검색 결과를 정렬합니다(기본적으로 점수가 높은 문서가 먼저 표시됨).
- `용어 빈도(Term Frequency)`: 문서에서 검색어가 얼마나 자주 등장하는지 
- `역문서 빈도(Inverse Document Frequency)`: 검색어가 전체 인덱스에서 얼마나 희소한지 
- `필드 길이(Field Length)`: 짧은 필드에서의 일치가 긴 필드보다 더 관련성이 높게 평가됨

Filter의 성능 이점:
- 점수 계산을 생략하므로 쿼리 실행이 더 빠릅니다.
- 필터 결과는 캐싱되어 재사용될 수 있어 반복 쿼리 시 성능이 향상됩니다.

### 언제 filter를 사용해야 할까요?
다음과 같은 경우에 filter를 사용하는 것이 좋습니다:

`이진(예/아니오) 결정:`

- 범위 검사: 가격, 날짜 범위 
- 정확한 값 일치: 카테고리, 상태, 태그 
- 존재 여부 확인: 필드가 있는지 없는지


`점수가 중요하지 않은 경우:`

- 결과를 다른 기준(예: 날짜, 가격)으로 정렬할 때 
- 다른 쿼리 부분(must/should)이 점수를 결정할 때 
- 단순히 조건에 맞는 문서만 필요할 때

## 실제 데이터 예제

다음은 제품 데이터를 사용한 실제 예제입니다:

### 인덱스 매핑 설정
```
PUT /products
{
  "mappings": {
    "properties": {
      "name": {
        "type": "text"
      },
      "category": {
        "type": "keyword"
      },
      "price": {
        "type": "float"
      },
      "in_stock": {
        "type": "boolean"
      },
      "discontinued": {
        "type": "boolean"
      }
    }
  }
}
```

### 샘플 데이터 색인
```
POST /products/_doc/1
{
  "name": "Smartphone XYZ",
  "category": "electronics",
  "price": 299.99,
  "in_stock": true,
  "discontinued": false
}
```
(총 8개의 제품 데이터가 색인됨)

## 쿼리 선택 가이드라인

1. **단순 조건에는 리프 쿼리 사용**:
   - 단일 필드에 대한 정확한 값 검색: `term` 쿼리
   - 범위 검색: `range` 쿼리
   - 전체 텍스트 검색: `match` 쿼리

2. **복잡한 조건에는 bool 쿼리 사용**:
   - 여러 조건의 AND 조합: `must` 절
   - 여러 조건의 OR 조합: `should` 절과 `minimum_should_match`
   - 특정 조건 제외: `must_not` 절
   - 관련성 점수에 영향 없는 필터링: `filter` 절

3. **성능 최적화**:
   - 관련성 순위가 중요하지 않은 조건은 `filter` 절 사용 (캐싱 이점)
   - 명확한 분류/필터링 목적의 필드는 `keyword` 타입 사용
   - 복잡한 bool 쿼리는 논리적 단위로 중첩하여 가독성과 유지보수성 향상
