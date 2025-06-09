### Mapping 전체 구조
```
GET /{index}/_mapping
```

### 특정 필드
```
GET /{index}/_mapping/field/{field_name}
```

### 중첩 필드
```
GET /{index}/_mapping/field/{properties}.{field}
```

### 필드 추가
```
PUT /{index}/_mapping
{
    "properties" : {
        "gender" : {
            "type" : "text"
        }
    }
}
```