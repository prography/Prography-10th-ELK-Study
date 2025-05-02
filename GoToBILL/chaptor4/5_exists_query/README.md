# Elasticsearch의 Exists 쿼리

Exists 쿼리는 특정 필드가 존재하는 문서만 검색하는 데 사용되는 Elasticsearch의 term-level 쿼리입니다.

## 기본 개념

- `exists` 쿼리는 지정된 필드가 문서에 존재하는지 확인합니다.
- 필드가 존재하고 null이 아닌 값을 가진 문서만 반환합니다.
- 빈 문자열(`""`)은 존재하는 것으로 간주되지만, null 값이나 누락된 필드는 존재하지 않는 것으로 간주됩니다.

## 예시

### 1. 기본 exists 쿼리

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

이 쿼리는 `email` 필드가 존재하는 모든 문서를 반환합니다.

### 2. 필드가 존재하지 않는 문서 검색 (부정 쿼리)

```
GET /my_index/_search
{
  "query": {
    "bool": {
      "must_not": {
        "exists": {
          "field": "email"
        }
      }
    }
  }
}
```

이 쿼리는 `email` 필드가 없는 모든 문서를 반환합니다.

### 3. 여러 필드 조합 검색

```
GET /my_index/_search
{
  "query": {
    "bool": {
      "must": [
        {
          "exists": {
            "field": "first_name"
          }
        },
        {
          "exists": {
            "field": "last_name"
          }
        }
      ],
      "must_not": {
        "exists": {
          "field": "deleted_at"
        }
      }
    }
  }
}
```

이 쿼리는 `first_name`과 `last_name` 필드가 모두 존재하고, `deleted_at` 필드가 없는 문서를 찾습니다.

### 4. 필터 컨텍스트에서 사용 (성능 최적화)

```
GET /my_index/_search
{
  "query": {
    "bool": {
      "filter": [
        {
          "exists": {
            "field": "email"
          }
        },
        {
          "term": {
            "status": "active"
          }
        }
      ]
    }
  }
}
```

이 쿼리는 `email` 필드가 존재하고 `status`가 "active"인 문서를 찾습니다. 필터 컨텍스트에서 사용하여 성능이 향상됩니다.

## 실제 사용 사례

1. **데이터 품질 검사**: 필수 필드가 누락된 문서 찾기
2. **완전성 검증**: 모든 필수 필드가 있는 문서만 검색
3. **조건부 검색**: 특정 정보가 제공된 경우에만 문서 반환
4. **데이터 클렌징**: 불완전한 데이터 식별하기

## 주의사항

1. **null 값과 빈 값**:
   - `null` 값은 필드가 존재하지 않는 것으로 간주됩니다.
   - 빈 문자열(`""`)과 빈 배열(`[]`)은 필드가 존재하는 것으로 간주됩니다.
   - 그러나 배열이 null 값만 포함하는 경우(`[null]`)는 필드가 존재하지 않는 것으로 간주됩니다.

2. **중첩 객체와 배열**:
   - 중첩 필드는 마침표 표기법을 사용합니다: `user.address.city`
   - 배열 내부 객체의 필드도 검색 가능합니다: `comments.author`

3. **매핑 고려사항**:
   - 필드가 매핑에 정의되어 있지만 문서에 해당 필드가 없으면 존재하지 않는 것으로 간주됩니다.
   - `null_value` 매핑 파라미터를 사용하면 null 값을 다른 값으로 대체할 수 있습니다.

4. **성능 최적화**:
   - `exists` 쿼리는 일반적으로 효율적이지만, 대규모 인덱스에서는 성능에 영향을 줄 수 있습니다.
   - 가능하면 필터 컨텍스트에서 사용하여 결과를 캐싱합시다.
