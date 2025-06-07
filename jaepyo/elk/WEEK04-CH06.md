## Elasticsearch 페이지네이션

### 1. `from` + `size` 방식

- **설명**: 특정 위치(`from`)에서부터 원하는 개수(`size`)만큼 결과를 잘라서 반환
- **예시**:
    
    ```json
    GET /products/_search
    {
      "from": 10,
      "size": 5,
      "query": {
        "match_all": {}
      },
      "sort": [
        { "price": "asc" }
      ]
    }
    
    ```
    
- **단점**:
    - MySQL의 OFFSET 방식과 유사
    - 정렬된 전체 결과를 메모리에 올린 뒤 잘라냄
    - 대량 데이터가 있는 경우 성능 저하가 발생

---

### 2. `search_after` 방식

- **설명**: 이전 페이지의 마지막 정렬값을 기준으로 다음 페이지를 요청
- **조건**:
    - 반드시 `sort` 기준이 있어야 함
    - 고유하지 않은 필드로 정렬할 경우 `_id` 같은 고유한 필드를 보조 정렬 기준으로 추가해야 함

### 첫 페이지 요청

```json
POST /library/_search
{
  "size": 3,
  "sort": [
    { "year": "desc" },
    { "_id": "desc" }
  ],
  "query": {
    "match_all": {}
  }
}

```

### 응답 예시

```json
{
  "hits": {
    "hits": [
      {
        "_id": "10",
        "_source": {
          "title": "Book A",
          "year": 2024
        },
        "sort": [2024, "10"]
      },
      {
        "_id": "9",
        "_source": {
          "title": "Book B",
          "year": 2023
        },
        "sort": [2023, "9"]
      },
      {
        "_id": "8",
        "_source": {
          "title": "Book C",
          "year": 2023
        },
        "sort": [2023, "8"]
      }
    ]
  }
}

```

### 다음 페이지 요청

```json
POST /library/_search
{
  "size": 3,
  "sort": [
    { "year": "desc" },
    { "_id": "desc" }
  ],
  "search_after": [2023, "8"],
  "query": {
    "match_all": {}
  }
}

```

- `search_after`에는 이전 응답의 마지막 문서의 `sort` 값을 넘김
- 이를 반복하면 효율적인 deep paging이 가능

---

## 유의사항

- `search_after`는 `sort` 필수
- 정렬 기준이 고유하지 않다면 `_id` 등 보조 정렬 필드 추가
- `search_after` 값은 `sort` 순서와 정확히 일치해야 함
- `from + size`는 작은 범위에서만 사용하는 것이 바람직함
