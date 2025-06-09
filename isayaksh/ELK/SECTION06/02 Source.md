# `_source`란?
- Elasticsearch에서 `_source`는 문서를 인덱스에 저장할 때의 원본 JSON 데이터를 의미한다.
- 사용자가 색인할 때 보낸 실제 문서 내용 그대로를 저장하고, 검색 시 이 _source 필드로 다시 보여주는 것이다.
- 정리하면, **`_source`는 색인 시 입력한 JSON 전체 자체를 반환하는 것이다.**

예를 들어 다음과 같이 문서를 색인하면:

```json
PUT /library/_doc/1
{
  "title": "1984",
  "author": "George Orwell"
}
```
이 문서를 조회할 때 나오는 _source는 아래와 같다:

```json
"_source": {
  "title": "1984",
  "author": "George Orwell"
}
```

## _source를 보고싶지 않다면?
- 검색 결과에서 실제 내용을 보여주거나, 필요 시 **`_source`를 비활성화해 저장 공간을 줄일 수도 있다** (`_source: false`)
- 즉, 데이터를 주고 받는 비용을 줄일 수 있다!
- 요청을 보낼 때, `_source: false`로 파라미터를 전달하면 된다.
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

## _source의 특정 값만 보고 싶다면?
- `includes` 파라미터를 통해 특정 필드 값만 `_source`를 통해 조회할 수 있다.
- 반대로, `excludes` 파라미터를 통해 특정 필드 값만 제외할 수도 있다.

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