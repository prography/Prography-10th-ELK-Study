## 페이지네이션 개요

Elasticsearch에서 페이지네이션은 큰 검색 결과 세트를 작고 관리하기 쉬운 단위로 나누는 기능입니다. 이는 대량의 검색 결과를 효율적으로 처리하고 사용자에게 표시할 때 특히 유용합니다.

페이지네이션을 구현하는 가장 일반적인 방법은 검색 쿼리에서 `from`과 `size` 매개변수를 사용하는 것입니다:

- **from**: 결과 집합의 시작 인덱스를 지정합니다 (0부터 시작)
- **size**: 반환할 문서의 개수를 지정합니다 (기본값은 10)

![페이지네이션 시각화](https://i.imgur.com/placeholder.png)

## 예제 코드

### 1. 인덱스 준비하기

먼저 테스트용 인덱스를 생성하고 10개의 문서를 추가합니다:

```
DELETE /my_index

PUT /my_index
{
  "settings": {
    "index": {
      "number_of_shards": 1,
      "number_of_replicas": 1
    }
  }
}

POST /my_index/_doc/1
{
  "content": "1"
}
POST /my_index/_doc/2
{
  "content": "2"
}
POST /my_index/_doc/3
{
  "content": "3"
}
POST /my_index/_doc/4
{
  "content": "4"
}
POST /my_index/_doc/5
{
  "content": "5"
}
POST /my_index/_doc/6
{
  "content": "6"
}
POST /my_index/_doc/7
{
  "content": "7"
}
POST /my_index/_doc/8
{
  "content": "8"
}
POST /my_index/_doc/9
{
  "content": "9"
}
POST /my_index/_doc/10
{
  "content": "10"
}
```

### 2. 기본 페이지네이션 - 첫 5개 결과 가져오기

`size` 매개변수만 사용하여 처음 5개의 결과를 가져옵니다:

```
GET /my_index/_search
{
  "size": 5,
  "query": {
    "match_all": {}
  }
}
```

### 3. 다음 페이지 가져오기 - 두 번째 5개 결과

`from` 매개변수를 사용하여 인덱스 5부터 시작하는 결과를 가져옵니다:

```
GET /my_index/_search
{
  "from": 5,
  "query": {
    "match_all": {}
  }
}
```

### 4. 사용자 지정 페이지 크기 - 첫 3개 결과

더 작은 페이지 크기로 첫 페이지를 가져옵니다:

```
GET /my_index/_search
{
  "from": 0,
  "size": 3,
  "query": {
    "match_all": {}
  }
}
```

### 5. 사용자 지정 페이지 크기 - 두 번째 페이지

3개 항목씩 표시할 때 두 번째 페이지의 결과를 가져옵니다:

```
GET /my_index/_search
{
  "from": 3,
  "size": 3,
  "query": {
    "match_all": {}
  }
}
```

## 페이지네이션 작동 방식

Elasticsearch에서 페이지네이션은 다음과 같이 작동합니다:

1. `from` 매개변수는 결과 집합의 시작 인덱스를 지정합니다 (기본값: 0)
2. `size` 매개변수는 반환할 문서의 수를 지정합니다 (기본값: 10)
3. 페이지 번호로 변환하는 공식: `from = (page_number - 1) * size`

예를 들어:
- 페이지 크기가 10일 때, 2페이지는 `from=10, size=10`
- 페이지 크기가 20일 때, 3페이지는 `from=40, size=20`

## 주의사항

1. **성능 고려사항**: `from + size`가 10,000을 초과하면 Elasticsearch는 기본적으로 오류를 반환합니다. 이는 깊은 페이지네이션이 클러스터에 부담을 줄 수 있기 때문입니다.

2. **대안**:
   - 대용량 데이터셋에서 깊은 페이지네이션이 필요한 경우 [Search After](https://www.elastic.co/guide/en/elasticsearch/reference/current/paginate-search-results.html#search-after) 또는 [Scroll API](https://www.elastic.co/guide/en/elasticsearch/reference/current/paginate-search-results.html#scroll-search-results)를 사용하는 것이 좋습니다.
   - Search After는 실시간 사용자 인터페이스에 적합합니다.
   - Scroll API는 대량의 데이터 내보내기와 같은 배치 처리에 더 적합합니다.

3. **정렬**: 일관된 페이지네이션 결과를 위해 항상 명시적인 정렬 기준을 지정하는 것이 좋습니다.
