# Seaerch Document ID

- 인덱스에 저장된 모든 Document를 조회하면 각 Document의 ID를 조회할 수 있다.
```
GET /my_index/_search
{
  "query": {
    "match_all": {}
  }
}
```

- 앞서 조회한 Document의 ID를 통해 직접 Document를 조회할 수 있다.
```
GET /my_index/_search
{
  "query": {
    "ids": {
      "values": ["gpFQWZEBfwu87jqZ_XLW"]
    }
  }
}
```