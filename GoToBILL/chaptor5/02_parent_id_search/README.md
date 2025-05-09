## Elasticsearch에서 parent_id를 사용한 검색

Elasticsearch에서 parent_id를 사용하여 자식 문서를 검색하는 방법에 대해 설명합니다. 이 기능은 특정 부모 문서 ID를 알고 있을 때 관련된 모든 자식 문서를 효율적으로 검색할 수 있게 해줍니다.

### parent_id 쿼리 개요

`parent_id` 쿼리는 특정 부모 ID를 가진 자식 문서를 검색하는 특수한 쿼리 타입입니다. `has_parent` 쿼리와 달리 부모 문서의 내용을 검색하는 것이 아니라, 부모 문서의 ID만으로 검색이 가능합니다.

### 사용 사례

- 특정 저자(ID)가 작성한 모든 책을 빠르게 검색할 때
- 특정 부모 문서와 연결된 모든 자식 문서를 조회할 때
- 부모 ID만 알고 있을 때 관련 자식 문서를 찾을 때

### parent_id 쿼리 구문

```elasticsearch
GET /library/_search
{
  "query": {
    "parent_id": {
      "type": "book",
      "id": "1"
    }
  }
}
```

위 쿼리는 부모 ID가 "1"인 모든 "book" 타입의 자식 문서를 검색합니다.

### parent_id 쿼리 예제

아래는 앞서 01_join 예제의 라이브러리 인덱스에서 특정 저자의 책을 검색하는 예제입니다:

```elasticsearch
# J.K. Rowling(ID: 1)이 쓴 모든 책 검색
GET /library/_search
{
  "query": {
    "parent_id": {
      "type": "book",
      "id": "1"
    }
  }
}
```

결과로 "Harry Potter and the Philosopher's Stone"과 "Harry Potter and the Chamber of Secrets" 두 책이 반환됩니다.

```elasticsearch
# George R.R. Martin(ID: 2)이 쓴 모든 책 검색
GET /library/_search
{
  "query": {
    "parent_id": {
      "type": "book",
      "id": "2"
    }
  }
}
```

결과로 "A Game of Thrones"과 "A Clash of Kings" 두 책이 반환됩니다.

### parent_id 쿼리 vs. has_parent 쿼리

1. **parent_id 쿼리**:
   - 부모 문서의 ID만 알고 있을 때 사용
   - 빠른 검색 성능 (ID 기반 검색이므로)
   - 부모 문서의 내용에 대한 검색 조건을 적용할 수 없음

2. **has_parent 쿼리**:
   - 부모 문서의 내용(필드 값)을 기반으로 검색할 때 사용
   - 예: "J.K. Rowling"이라는 이름을 가진 저자의 모든 책
   - 보다 복잡한 검색 조건 적용 가능
   - parent_id 쿼리보다 상대적으로 성능이 느릴 수 있음

### 성능 고려사항

- `parent_id` 쿼리는 ID 기반 검색이므로 매우 효율적입니다
- 대량의 데이터에서 특히 `has_parent` 쿼리보다 더 빠른 성능을 제공합니다
- 부모 ID가 명확히 알려져 있는 경우 항상 `parent_id` 쿼리를 사용하는 것이 좋습니다