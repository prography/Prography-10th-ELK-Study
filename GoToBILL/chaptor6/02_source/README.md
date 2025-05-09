## 개요

Elasticsearch에서 `_source` 필드는 인덱싱 시 제출된 원본 JSON 문서를 포함합니다. 검색 결과에서 이 필드를 어떻게 반환할지 제어하는 여러 방법이 있습니다.

## 예제 쿼리

### 1. 기본 검색 쿼리

```
GET /library/_search?pretty
{
  "query": {
    "match": {
      "title": "Game"
    }
  }
}
```

이 쿼리는 'title' 필드에 'Game'을 포함하는 모든 문서를 검색합니다. 기본적으로 `_source` 필드 전체가 결과에 반환됩니다.

### 2. _source 필드 비활성화

```
GET /library/_search?pretty
{
  "_source": false,
  "query": {
    "match": {
      "title": "Game"
    }
  }
}
```

`_source: false`로 설정하면 검색 결과에 `_source` 필드가 전혀 포함되지 않습니다. 이는 필요한 정보만 반환하여 응답 크기를 줄이고자 할 때 유용합니다.

### 3. 특정 필드만 포함하기

```
GET /library/_search?pretty
{
  "_source": {
    "includes": "my_join_field.name"
  },
  "query": {
    "match": {
      "title": "Game"
    }
  }
}
```

`includes` 파라미터를 사용하면 `_source`에서 특정 필드만 선택적으로 반환할 수 있습니다. 여기서는 'my_join_field.name' 필드만 반환됩니다.

### 4. 특정 필드 제외하기

```
GET /library/_search?pretty
{
  "_source": {
    "excludes": "my_join_field.name"
  },
  "query": {
    "match": {
      "title": "Game"
    }
  }
}
```

`excludes` 파라미터를 사용하면 `_source`에서 특정 필드를 제외할 수 있습니다. 여기서는 'my_join_field.name'을 제외한 모든 필드가 반환됩니다.

### 5. 여러 필드 제외하기

```
GET /library/_search?pretty
{
  "_source": {
    "excludes": ["my_join_field.name", "title"]
  },
  "query": {
    "match": {
      "title": "Game"
    }
  }
}
```

배열을 사용하여 여러 필드를 동시에 제외할 수 있습니다. 이 쿼리는 'my_join_field.name'과 'title' 필드를 모두 제외합니다.

## 활용 사례

- **응답 크기 최적화**: 큰 문서에서 필요한 필드만 반환하여 네트워크 대역폭 사용을 줄일 수 있습니다.
- **보안**: 민감한 필드를 결과에서 제외할 수 있습니다.
- **성능 향상**: 필요한 데이터만 전송하여 전체 검색 프로세스를 빠르게 할 수 있습니다.

## 참고사항

- `_source` 필드를 비활성화하면 하이라이팅(highlighting)이나 일부 재스코어링(rescoring) 기능과 같은 특정 기능을 사용할 수 없게 됩니다.


- 인덱스 생성 시 `_source` 필드를 완전히 비활성화할 수도 있지만, 이는 업데이트 및 재인덱싱 기능에 영향을 미치므로 신중하게 결정해야 합니다.
