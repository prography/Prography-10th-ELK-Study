# Elasticsearch 멀티 필드(Multi-Fields)

## 개요
멀티 필드는 Elasticsearch에서 하나의 필드를 여러 방식으로 인덱싱할 수 있게 해주는 기능입니다. 이를 통해 검색, 정렬, 집계 등 다양한 목적에 최적화된 방식으로 데이터를 활용할 수 있습니다.

## 멀티 필드의 이점
1. **다양한 검색 요구 지원**: 
   - `text` 타입으로는 전문 검색(full-text search)이 가능합니다.
   - `keyword` 타입으로는 정확한 값 매칭과 정렬, 집계가 가능합니다.

2. **텍스트 분석의 유연성**:
   - 동일한 필드를 다른 애널라이저로 분석할 수 있습니다.
   - 예를 들어, 영어 분석기와 한국어 분석기를 동시에 적용할 수 있습니다.

3. **성능 최적화**:
   - 각 필드 타입은 특정 연산에 최적화되어 있습니다.
   - 검색용 text 필드와 집계용 keyword 필드를 분리하면 더 효율적인 쿼리가 가능합니다.

4. **데이터 중복 없이 다양한 기능 활용**:
   - 실제 데이터는 한 번만 저장하고, 다양한 인덱싱 방식을 적용할 수 있습니다.

## 예제 코드

```json
// 인덱스 삭제
DELETE /multi_fields_index

// 인덱스 생성 (매핑 정의)
PUT /multi_fields_index
{
  "mappings": {
    "properties": {
      "title": {
        "type": "text",
        "fields": {
          "raw": {
            "type": "keyword"
          }
        }
      }
    }
  }
}

// 문서 추가 1
POST /multi_fields_index/_doc/1
{
  "title": "Elasticsearch: The Definitive Guide"
}

// 문서 추가 2
POST /multi_fields_index/_doc/2
{
  "title": "Elasticsearch: Search Everything"
}

// 검색 쿼리 1: 전문 검색 (text 타입 활용)
GET /multi_fields_index/_search
{
  "query": {
    "match": {
      "title": "Elasticsearch"
    }
  }
}

// 검색 쿼리 2: 정확한 매칭 (keyword 타입 활용)
GET /multi_fields_index/_search
{
  "query": {
    "term": {
      "title.raw": "Elasticsearch: The Definitive Guide"
    }
  }
}

// 집계 쿼리: 고유 제목 목록 (keyword 타입 활용)
GET /multi_fields_index/_search
{
  "aggs": {
    "unique_titles": {
      "terms": {
        "field": "title.raw"
      }
    }
  }
}
```

## 검색 결과 설명

### 첫 번째 검색 쿼리 (match)
- `title` 필드는 "text" 타입으로, 문장을 단어 단위로 쪼개어 처리합니다.
- 검색어 "Elasticsearch"가 포함된 모든 문서가 검색됩니다.
- **결과**: 두 문서 모두 "Elasticsearch"라는 단어를 포함하고 있어서 둘 다 검색됩니다.

### 두 번째 검색 쿼리 (term)
- `title.raw` 필드는 "keyword" 타입으로, 전체 문장을 하나의 값으로 취급합니다.
- 검색어 "Elasticsearch: The Definitive Guide"와 정확히 일치하는 문서만 찾습니다.
- **결과**: 첫 번째 문서만 정확히 일치하므로 하나만 검색됩니다.

### 집계 쿼리 (terms)
- `terms` 집계는 필드의 고유한 값들을 세는 기능입니다.
- keyword 타입만 이 집계를 할 수 있습니다(text 타입은 불가능).
- **결과**: "Elasticsearch: The Definitive Guide"와 "Elasticsearch: Search Everything"이라는 두 개의 고유 제목이 집계됩니다.

## 실제 활용 예시
- 상품명: text 타입으로 검색, keyword 타입으로 정렬 및 집계
- 이메일: text 타입으로 내용 검색, keyword 타입으로 정확한 이메일 매칭
- 주소: text 타입으로 주소 내 키워드 검색, keyword 타입으로 지역별 집계
