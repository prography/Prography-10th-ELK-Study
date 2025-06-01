# Elasticsearch 매핑(Mapping)

**매핑**은 **Elasticsearch**에서 데이터를 **정확하게 저장**하고 **효율적으로 검색**할 수 있도록 도와주는 핵심 설정입니다.

매핑을 통해 필드의 **데이터 타입**, **검색 분석기**, **중첩 객체** 등을 정의하여, **성능**을 최적화하고, **검색**의 정확도를 높이며, **데이터 구조**를 일관성 있게 유지할 수 있습니다.

## 매핑 없이 데이터 저장 시의 문제점

만약 매핑을 **정의하지 않고 데이터**를 저장하면, Elasticsearch가 자동으로 `dynamic mapping`을 사용하여 데이터를 추정하고 저장합니다. 이 경우 **예상치 못한 결과**가 나올 수 있습니다.

## 매핑 예제

### 기본 매핑 예제

```
DELETE /member
PUT /member
{
  "mappings": {
    "properties": {
      "name": { "type": "text" },
      "email": { "type": "keyword" },
      "age": { "type": "integer" },
      "joined": { "type": "date" },
      "active": { "type": "boolean" },
      "location": { "type": "geo_point" },
      "ip_address": { "type": "ip" },
      "profile_picture": { "type": "binary" },
      "preferences": {
        "type": "object",
        "properties": {
          "newsletter": { "type": "boolean" },
          "notifications": { "type": "keyword" }
        }
      }
    }
  }
}
```

### 문서 추가 예제

```
POST /member/_doc/1
{
  "name": "John Doe",
  "email": "john.doe@example.com",
  "age": 30,
  "joined": "2023-05-27T00:00:00Z",
  "active": true,
  "location": {
    "lat": 40.7128,
    "lon": -74.0060
  },
  "ip_address": "192.168.1.1",
  "profile_picture": "aGVsbG8gd29ybGQ=",  // "hello world" in Base64
  "preferences": {
    "newsletter": true,
    "notifications": "daily"
  }
}
```

### 문서 조회

```
GET /member/_doc/1
```

## Object vs Nested 타입 비교

### Object 타입 예제

```
DELETE /my_user
PUT /my_user
{
  "mappings": {
    "properties": {
      "user": {
        "type": "object",
        "properties": {
          "name": { "type": "text" },
          "email": { "type": "keyword" }
        }
      },
      "comments": {
        "type": "object",
        "properties": {
          "user": { "type": "text" },
          "message": { "type": "text" },
          "date": { "type": "date" }
        }
      }
    }
  }
}
```

### 문서 추가 (Object 타입)

```
POST /my_user/_doc/1
{
  "user": {
    "name": "John Doe",
    "email": "john.doe@example.com"
  },
  "comments": [
    {
      "user": "Alice",
      "message": "Nice article!",
      "date": "2023-05-27"
    },
    {
      "user": "Bob",
      "message": "Thanks for the information.",
      "date": "2023-05-28"
    }
  ]
}
```

### Object 타입 검색 (주의: 예상과 다른 결과가 나올 수 있음)

```
GET /my_user/_search
{
  "query": {
    "bool": {
      "must": [
        { "match": { "comments.user": "Alice" } },
        { "match": { "comments.message": "Thanks for the information" } }
      ]
    }
  }
}
```

### Nested 타입 예제

```
DELETE /my_nested_user
PUT /my_nested_user
{
  "mappings": {
    "properties": {
      "user": {
        "type": "object",
        "properties": {
          "name": { "type": "text" },
          "email": { "type": "keyword" }
        }
      },
      "comments": {
        "type": "nested",
        "properties": {
          "user": { "type": "text" },
          "message": { "type": "text" },
          "date": { "type": "date" }
        }
      }
    }
  }
}
```

### 문서 추가 (Nested 타입)

```
POST /my_nested_user/_doc/1
{
  "user": {
    "name": "John Doe",
    "email": "john.doe@example.com"
  },
  "comments": [
    {
      "user": "Alice",
      "message": "Nice article!",
      "date": "2023-05-27"
    },
    {
      "user": "Bob",
      "message": "Thanks for the information.",
      "date": "2023-05-28"
    }
  ]
}
```

### Nested 타입 검색 예제

```
GET /my_nested_user/_search
{
  "query": {
    "nested": {
      "path": "comments",
      "query": {
        "bool": {
          "must": [
            { "match": { "comments.user": "Alice" } },
            { "match": { "comments.message": "Nice article!" } }
          ]
        }
      }
    }
  }
}
```

## 특수 데이터 타입 예제

### Percolator 타입 (저장된 쿼리)

```
DELETE /my-index
PUT /my-index
{
  "mappings": {
    "properties": {
      "query": {
        "type": "percolator"
      },
      "message": {
        "type": "text"
      }
    }
  }
}

POST /my-index/_doc/1?refresh
{
  "query": {
    "match": {
      "message": "Elasticsearch"
    }
  }
}

POST /my-index/_search
{
  "query": {
    "percolate": {
      "field": "query",
      "document": {
        "message": "Elasticsearch is a powerful search engine"
      }
    }
  }
}
```

### Binary 타입

```
DELETE /binary-index
PUT /binary-index
{
  "mappings": {
    "properties": {
      "file": {
        "type": "binary"
      },
      "description": {
        "type": "text"
      }
    }
  }
}

POST /binary-index/_doc/1
{
  "file": "U29tZSBiaW5hcnkgY29udGVudA==",
  "description": "A file containing some binary data"
}

GET /binary-index/_doc/1
```

## 매핑 정보 조회

```
GET /member/_mapping

GET /member/_mapping/field/name

GET /member/_mapping/field/preferences.newsletter
```

## 플랫 매핑 (Flat Mapping)

```
PUT /member_new
{
  "mappings": {
    "properties": {
      "name": { "type": "text" },
      "email": { "type": "keyword" },
      "age": { "type": "integer" },
      "joined": { "type": "date" },
      "active": { "type": "boolean" },
      "location": { "type": "geo_point" },
      "ip_address": { "type": "ip" },
      "profile_picture": { "type": "binary" },
      "preferences.newsletter": { "type": "boolean" },
      "preferences.notifications": { "type": "keyword" }
    }
  }
}

GET /member_new/_mapping
```

## 기존 매핑 업데이트

```
PUT /member_new/_mapping
{
  "properties": {
    "gender": {
      "type": "text"
    }
  }
}
```

## Object 타입과 Nested 타입 비교

- **`object`** 타입: **하위 필드**들은 문서 전체와 연관되어 있고, 쿼리 시 여러 조건을 **단일 객체로 취급**합니다.
- **`nested`** 타입: 각 **하위 필드**가 **독립적인 객체처럼 처리**되어, 복잡한 **다대다 관계**를 효율적으로 처리할 수 있습니다.
