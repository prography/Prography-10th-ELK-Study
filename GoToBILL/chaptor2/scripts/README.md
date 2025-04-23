# Elasticsearch 스크립트 (Script)

Elasticsearch에서 스크립트는 문서를 인덱싱하거나 업데이트하거나 검색을 할 때 더 복잡한 연산이나 계산을 가능하게 해주는 기능입니다.

스크립트를 사용하면, 예를 들어 문서 업데이트 시 계산을 하거나, 검색 쿼리에서 커스텀 스코어를 적용하는 등의 작업을 할 수 있습니다.

## 기본 스크립트 예제

```
POST /my_index/_update/100
{
  "script": {
    "source": "ctx._source.price += 10"
  }
}
```

위 예제에서는 `my_index` 인덱스의 ID가 100인 문서의 `price` 필드 값을 10 증가시키고 있습니다.

## 스크립트 사용 예시

### 1. 문서 생성

```
POST /my_index/_doc/100
{
  "title": "Elasticsearch Advanced",
  "author": "Anthony K",
  "publish_date": "2024-05-20",
  "tags": ["search", "analytics"]
}
```

### 2. 문서 조회

```
GET /my_index/_doc/100
```

### 3. 문서 업데이트 (일반적인 방법)

```
POST /my_index/_update/100
{
  "doc": {
    "price": 100
  }
}
```

### 4. 스크립트를 이용한 문서 업데이트

```
POST /my_index/_update/100
{
  "script": {
    "source": "ctx._source.price += 10"
  }
}
```

### 5. 파라미터를 이용한 스크립트

```
POST /my_index/_update/100
{
  "script": {
    "source": "ctx._source.price += params.amount",
    "params": {
      "amount": 10
    }
  }
}
```

### 6. 조건부 실행 (No Operation)

```
POST /my_index/_update/100
{
  "script": {
    "source": """
        if (ctx._source.price > 100) {
            ctx.op = 'noop';
        }
        ctx._source.price += 10;
    """
  }
}
```

여기서 `ctx.op = 'noop'`은 가격이 100을 초과하면 업데이트를 실행하지 않도록 지시합니다.

### 7. 조건부 필드 업데이트

```
POST /my_index/_update/100
{
  "script": {
    "source": """
        if (ctx._source.price > 100) {
            ctx._source.price += 10;
        }
    """
  }
}
```

### 8. 검색 스크립트 (Function Score)

```
POST /my_index/_search
{
  "query": {
    "function_score": {
      "query": {
        "match_all": {}
      },
      "script_score": {
        "script": {
          "source": "doc['price'].value * _score"
        }
      }
    }
  }
}
```

### 9. 집계 스크립트

```
POST /my_index/_search
{
  "size": 0,
  "aggs": {
    "custom_avg": {
      "avg": {
        "script": {
          "source": "doc['price'].value * params.vat_rate",
          "params": {
            "vat_rate": 1.2
          }
        }
      }
    }
  }
}
```

### 10. 저장된 스크립트 (Stored Scripts)

```
POST /_scripts/calculate-vat
{
  "script": {
    "lang": "painless",
    "source": "doc['price'].value * params.vat_rate"
  }
}

POST /my_index/_search
{
  "size": 0,
  "aggs": {
    "custom_avg": {
      "avg": {
        "script": {
          "id": "calculate-vat",
          "params": {
            "vat_rate": 1.2
          }
        }
      }
    }
  }
}
```

## 스크립트 특징

1. **컨텍스트 변수**:
   - `ctx._source`: 업데이트할 문서의 소스 데이터
   - `doc['field']`: 인덱싱된 필드에 빠르게 접근
   - `_score`: 현재 문서의 검색 점수

2. **스크립트 언어**:
   - 기본적으로 `painless`라는 Elasticsearch 전용 스크립트 언어를 사용합니다.
   - 안전하고 빠른 실행을 위해 최적화되었습니다.

3. **스크립트 타입**:
   - 인라인 스크립트: API 호출에 직접 포함된 스크립트
   - 저장된 스크립트: 이름을 지정하여 Elasticsearch에 저장

4. **사용 사례**:
   - 문서 업데이트 시 필드 값 계산
   - 조건부 업데이트 로직 적용
   - 커스텀 스코어링 및 정렬
   - 복잡한 집계 계산
