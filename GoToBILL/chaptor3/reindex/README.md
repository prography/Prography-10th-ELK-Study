# Elasticsearch 리인덱싱과 별칭(Alias) 사용하기

## 리인덱싱 예제 코드

다음은 Elasticsearch에서 필드 타입을 변경하기 위해 리인덱싱을 수행하는 예제입니다:

```
# 원본 인덱스 삭제 후 생성 (product_id를 integer 타입으로 정의)
DELETE /original_index
PUT /original_index
{
  "mappings": {
    "properties": {
      "product_id": {
        "type": "integer"
      }
    }
  }
}

# 원본 인덱스에 데이터 입력
POST /original_index/_doc/1
{
  "product_id": 100
}
POST /original_index/_doc/2
{
  "product_id": 200
}

# 새 인덱스 삭제 후 생성 (product_id를 keyword 타입으로 변경)
DELETE /new_index
PUT /new_index
{
  "mappings": {
    "properties": {
      "product_id": {
        "type": "keyword"
      }
    }
  }
}

# 리인덱싱 실행 (단순 복사)
POST /_reindex
{
  "source": {
    "index": "original_index"
  },
  "dest": {
    "index": "new_index"
  }
}

# 새 인덱스 검색
GET /new_index/_search
{
  "query": {
    "match_all": {}
  }
}

# 데이터 변환을 포함한 리인덱싱 실행 (정수를 문자열로 변환)
POST /_reindex
{
  "source": {
    "index": "original_index"
  },
  "dest": {
    "index": "new_index"
  },
  "script": {
    "source": """
        ctx._source.product_id = ctx._source.product_id.toString()
    """
  }
}

# 별칭 생성 - 원본 인덱스에 base_index 별칭 추가
POST /_aliases
{
  "actions": [
    {
      "add": {
        "index": "original_index",
        "alias": "base_index"
      }
    }
  ]
}

# 별칭을 통한 검색
GET /base_index/_search
{
  "query": {
    "match_all": {}
  }
}

# 별칭 전환 - 원본 인덱스에서 새 인덱스로 별칭 이동
POST /_aliases
{
  "actions": [
    { "remove": { "index": "original_index", "alias": "base_index" } },
    { "add": { "index": "new_index", "alias": "base_index" } }
  ]
}
```

## 별칭(Alias)을 사용하는 이유

Elasticsearch에서 별칭은 다음과 같은 중요한 이유로 사용됩니다:

### 1. 무중단 인덱스 전환 (Zero Downtime)
- 애플리케이션은 항상 동일한 별칭(`base_index`)을 통해 데이터에 접근
- 인덱스 구조 변경 시 새 인덱스를 준비하고 데이터를 마이그레이션한 후, 별칭만 전환
- 애플리케이션 코드 수정 없이 백엔드 인덱스 교체 가능

### 2. 인덱스 버저닝
- 인덱스를 버전별로 관리 가능 (예: `products_v1`, `products_v2`)
- `products`라는 별칭이 항상 현재 사용 중인 버전을 가리킴
- 매핑이나 설정 변경 시 유연한 대응 가능

### 3. 다중 인덱스 접근 간소화
- 하나의 별칭이 여러 인덱스를 동시에 가리킬 수 있음
- 시계열 데이터(예: `logs_2023`, `logs_2024`)에 `logs` 별칭을 부여하여 단일 쿼리로 검색 가능
- 정기적으로 생성되는 인덱스들을 논리적 그룹으로 관리

### 4. 읽기/쓰기 작업 분리
- `products_write`와 `products_read` 같은 별칭으로 작업 분리 가능
- 인덱스 재구성 중에도 애플리케이션의 정상 작동 보장
- 성능 최적화에 활용 가능

위 예제에서는 `base_index` 별칭을 사용하여 기존 `original_index`에서 새로운 구조의 `new_index`로 전환하는 과정을 보여줍니다. 이를 통해 필드 타입 변경과 같은 스키마 수정을 애플리케이션에 영향 없이 수행할 수 있습니다.
