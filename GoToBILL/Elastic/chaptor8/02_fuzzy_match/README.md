# 퍼지 매치 쿼리(Fuzzy Match Query)

Elasticsearch에서 퍼지 매치 쿼리는 정확하게 일치하지 않아도 근사적으로 일치하는 용어를 검색할 수 있는 강력한 기능입니다. 이는 오타나 철자 변형에 대응하는 데 특히 유용합니다.

## 개요

Elasticsearch의 퍼지 매치 쿼리는 다음과 같은 특징을 가집니다:

- **근사 일치 검색**: 검색어와 정확히 일치하지 않아도 비슷한 용어를 포함하는 문서를 찾을 수 있습니다.
- **오타 및 철자 변형 처리**: 사용자의 오타나 다양한 철자 표기법에 대응할 수 있습니다.
- **편집 거리**: 검색어와 일치하는 용어 사이에 허용되는 변경 횟수를 지정합니다.

## 주요 파라미터

### fuzziness

fuzziness 파라미터는 검색어와 문서 내 용어 간에 허용되는 편집 거리(변경 횟수)를 결정합니다:

- **fuzziness 0**: 퍼지 기능 없음 (정확한 일치만 허용)
- **fuzziness 1**: 한 번의 편집(문자 대체, 삽입 또는 삭제) 허용
- **fuzziness 2**: 두 번의 편집 허용
- **fuzziness "AUTO"**: 검색어 길이에 따라 퍼지 수준을 자동으로 조정
  - 1~2자: 오타 허용 안 함 (fuzziness = 0)
  - 3~5자: 최대 1회 편집 허용 (fuzziness = 1)
  - 5자 이상: 최대 2회 편집 허용 (fuzziness = 2)

### prefix_length

최소한 몇 개의 시작 문자가 정확히 일치해야 하는지 지정합니다. 이는 오탐(false positive)을 줄이는 데 도움이 됩니다. 예를 들어, `prefix_length: 2`로 설정하면 검색어의 처음 2글자는 반드시 일치해야 합니다.

## 예제 데이터 설정

먼저 테스트용 인덱스와 문서를 생성합니다:

```
DELETE /products
PUT /products
{
  "mappings": {
    "properties": {
      "name": {
        "type": "text"
      },
      "price": {
        "type": "float"
      }
    }
  }
}

POST /products/_doc/1
{
  "name": "Samsung Galaxy",
  "price": 799.99
}

POST /products/_doc/2
{
  "name": "Apple iPhone",
  "price": 999.99
}

POST /products/_doc/3
{
  "name": "Sony Xperia",
  "price": 699.99
}

POST /products/_doc/4
{
  "name": "Google Pixel",
  "price": 899.99
}
```

## 퍼지 검색 실습

### 기본 퍼지 검색 (AUTO 퍼지)

```
GET /products/_search
{
  "query": {
    "fuzzy": {
      "name": {
        "value": "Samsong",
        "fuzziness": "AUTO"
      }
    }
  }
}
```

**결과:**
이 쿼리는 "Samsong"과 유사한 제품 이름을 검색합니다. "Samsung"은 "Samsong"과 1글자 차이이므로 검색 결과에 포함됩니다.

```json
{
  "took": 5,
  "timed_out": false,
  "_shards": {
    "total": 1,
    "successful": 1,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": {
      "value": 1,
      "relation": "eq"
    },
    "max_score": 0.6931471,
    "hits": [
      {
        "_index": "products",
        "_id": "1",
        "_score": 0.6931471,
        "_source": {
          "name": "Samsung Galaxy",
          "price": 799.99
        }
      }
    ]
  }
}
```

### 다양한 퍼지 수준 테스트

#### fuzziness 0 (정확한 일치)

```
GET /products/_search
{
  "query": {
    "fuzzy": {
      "name": {
        "value": "Samsong",
        "fuzziness": 0
      }
    }
  }
}
```

**결과:**
정확히 "Samsong"과 일치하는 문서가 없으므로 결과가 반환되지 않습니다.

#### fuzziness 1 (1자 차이 허용)

```
GET /products/_search
{
  "query": {
    "fuzzy": {
      "name": {
        "value": "Samsong",
        "fuzziness": 1
      }
    }
  }
}
```

**결과:**
"Samsung"과 "Samsong"은 1글자 차이이므로 Samsung Galaxy 제품이 결과에 포함됩니다.

#### fuzziness 2 (2자 차이 허용)

```
GET /products/_search
{
  "query": {
    "fuzzy": {
      "name": {
        "value": "Samsoong",
        "fuzziness": 2
      }
    }
  }
}
```

**결과:**
"Samsung"과 "Samsoong"은 2글자 차이이므로 Samsung Galaxy 제품이 결과에 포함됩니다.

### prefix_length 사용

```
GET /products/_search
{
  "query": {
    "fuzzy": {
      "name": {
        "value": "Pixle",
        "fuzziness": "AUTO",
        "prefix_length": 2
      }
    }
  }
}
```

**결과:**
"Pixle"과 "Pixel"은 철자가 바뀌었지만, 처음 2글자 "Pi"는 동일하므로 Google Pixel 제품이 결과에 포함됩니다.

## 실무 활용 사례

1. **검색 경험 개선**: 사용자가 검색어를 잘못 입력하더라도 관련 결과를 제공하여 사용자 경험을 향상시킵니다.
2. **다국어 지원**: 다양한 언어나 번역에서 발생할 수 있는 철자 변형을 처리합니다.
3. **제품 카탈로그 검색**: 제품명의 다양한 표기법(예: iPhone vs. Iphone vs. i-phone)에 대응합니다.
4. **자동 완성 및 제안**: 타이핑 중 오타가 있어도 적절한 제안을 제공합니다.

## 성능 고려사항

- **리소스 사용**: 퍼지 검색은 정확한 일치 검색보다 더 많은 컴퓨팅 리소스를 사용할 수 있습니다.
- **정밀도와 재현율 균형**: `fuzziness` 값이 높을수록 더 많은 결과를 반환하지만(재현율 증가), 관련성이 낮은 결과도 포함될 수 있습니다(정밀도 감소).
- **prefix_length 최적화**: `prefix_length`를 적절히 설정하면 검색 성능을 향상시키고 오탐을 줄일 수 있습니다.
- **대용량 인덱스 주의**: 대용량 인덱스에서 퍼지 검색은 성능에 영향을 미칠 수 있으므로, 경우에 따라 완성 제안(completion suggester)이나 다른 방법을 고려해볼 수 있습니다.

## 추가 팁

- 퍼지 검색은 단일 용어에 가장 효과적입니다. 복잡한 구문 검색에는 `match` 쿼리와 `fuzziness` 매개변수의 조합이 더 적합할 수 있습니다.
- 고급 애플리케이션에서는 `max_expansions` 파라미터를 사용하여 검색 확장의 수를 제한할 수 있습니다.
- 자동 완성이나 "입력하면서 검색" 기능을 구현할 때는 `prefix_length`를 0으로 설정하는 것이 좋습니다.
