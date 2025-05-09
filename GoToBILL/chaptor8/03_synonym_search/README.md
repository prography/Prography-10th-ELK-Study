# 동의어 검색(Synonym Search)

Elasticsearch에서 동의어 검색은 서로 다른 단어나 구문을 검색할 때 동등하게 취급할 수 있게 해주는 강력한 기능입니다. 이를 통해 사용자가 다양한 표현으로 검색해도 동일한 결과를 얻을 수 있습니다.

## 개요

동의어 검색을 사용하면 다음과 같은 이점이 있습니다:

- **검색 품질 향상**: 사용자가 다양한 용어로 검색해도 연관된 결과를 제공할 수 있습니다.
- **용어 표준화**: 비슷한 의미를 가진 여러 단어를 하나의 표준 용어로 처리할 수 있습니다.
- **업계 용어 및 약어 처리**: 전문 용어와 그 일반적인 표현을 동등하게 처리할 수 있습니다.

## 동의어 검색 구현 방법

### 1. 동의어 파일 또는 인라인 동의어 정의

동의어는 인덱스 설정에서 직접 정의하거나 외부 파일을 통해 관리할 수 있습니다.

### 2. 커스텀 분석기 생성

동의어 필터를 포함한 커스텀 분석기를 정의하여 텍스트 처리 과정에서 동의어를 적용합니다.

### 3. 분석기를 필드에 적용

매핑 정의에서 동의어 분석기를 사용하도록 필드를 설정합니다.

## 예제 설정

다음은 인라인 동의어를 사용하는 인덱스 설정 예제입니다:

```
DELETE /products

PUT /products
{
  "settings": {
    "analysis": {
      "filter": {
        "synonym_filter": {
          "type": "synonym",
          "synonyms": [
            "laptop, notebook",
            "smartphone, cell phone"
          ]
        }
      },
      "analyzer": {
        "synonym_analyzer": {
          "tokenizer": "standard",
          "filter": [
            "lowercase",
            "synonym_filter"
          ]
        }
      }
    }
  },
  "mappings": {
    "properties": {
      "name": {
        "type": "text",
        "analyzer": "synonym_analyzer"
      }
    }
  }
}
```

이 설정에서:
1. `synonym_filter`라는 동의어 필터를 정의했습니다.
2. 'laptop'과 'notebook', 'smartphone'과 'cell phone'을 동의어로 설정했습니다.
3. 동의어 필터를 사용하는 `synonym_analyzer`라는 커스텀 분석기를 생성했습니다.
4. `name` 필드에 이 분석기를 적용했습니다.

## 예제 데이터 추가

```
POST /products/_doc/1
{
  "name": "Apple iPhone"
}

POST /products/_doc/2
{
  "name": "Samsung Galaxy Smartphone"
}

POST /products/_doc/3
{
  "name": "Dell XPS Laptop"
}

POST /products/_doc/4
{
  "name": "Lenovo ThinkPad Notebook"
}
```

## 동의어 검색 테스트

### 'cell phone' 검색

```
GET /products/_search
{
  "query": {
    "match": {
      "name": "cell phone"
    }
  }
}
```

**결과:**
'cell phone'으로 검색하면 'smartphone'이 포함된 문서도 결과에 포함됩니다. 따라서 문서 1(Apple iPhone)과 문서 2(Samsung Galaxy Smartphone)가 반환됩니다.

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
      "value": 2,
      "relation": "eq"
    },
    "max_score": 0.6931471,
    "hits": [
      {
        "_index": "products",
        "_id": "2",
        "_score": 0.6931471,
        "_source": {
          "name": "Samsung Galaxy Smartphone"
        }
      },
      {
        "_index": "products",
        "_id": "1",
        "_score": 0.5753642,
        "_source": {
          "name": "Apple iPhone"
        }
      }
    ]
  }
}
```

### 'notebook' 검색

```
GET /products/_search
{
  "query": {
    "match": {
      "name": "notebook"
    }
  }
}
```

**결과:**
'notebook'으로 검색하면 'laptop'이 포함된 문서도 결과에 포함됩니다. 따라서 문서 3(Dell XPS Laptop)과 문서 4(Lenovo ThinkPad Notebook)가 반환됩니다.

```json
{
  "took": 2,
  "timed_out": false,
  "_shards": {
    "total": 1,
    "successful": 1,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": {
      "value": 2,
      "relation": "eq"
    },
    "max_score": 0.6931471,
    "hits": [
      {
        "_index": "products",
        "_id": "4",
        "_score": 0.6931471,
        "_source": {
          "name": "Lenovo ThinkPad Notebook"
        }
      },
      {
        "_index": "products",
        "_id": "3",
        "_score": 0.5753642,
        "_source": {
          "name": "Dell XPS Laptop"
        }
      }
    ]
  }
}
```

## 동의어 업데이트

기존 인덱스의 동의어를 업데이트하려면 인덱스를 닫고 설정을 변경한 후 다시 열어야 합니다:

```
POST /products/_close

PUT /products/_settings
{
  "analysis": {
    "filter": {
      "synonym_filter": {
        "type": "synonym",
        "synonyms": [
          "laptop, notebook",
          "smartphone, cell phone",
          "tablet, pad"
        ]
      }
    }
  }
}

POST /products/_open
```

이제 'tablet'과 'pad'가 동의어로 추가되었습니다.

## 분석기 테스트

동의어 분석기가 올바르게 작동하는지 확인할 수 있습니다:

```
POST /products/_analyze
{
  "analyzer": "synonym_analyzer",
  "text": "tablet"
}
```

**결과:**
'tablet'은 'pad'의 동의어로 처리됩니다.

```json
{
  "tokens": [
    {
      "token": "tablet",
      "start_offset": 0,
      "end_offset": 6,
      "type": "word",
      "position": 0
    },
    {
      "token": "pad",
      "start_offset": 0,
      "end_offset": 6,
      "type": "SYNONYM",
      "position": 0
    }
  ]
}
```

## 외부 동의어 파일 사용

대규모 동의어 목록의 경우 외부 파일을 사용하는 것이 더 효율적입니다:

```
POST /products/_close

PUT /products/_settings
{
  "analysis": {
    "filter": {
      "synonym_filter": {
        "type": "synonym",
        "synonyms_path": "analysis/synonyms.txt"
      }
    }
  }
}

POST /products/_open
```

`synonyms.txt` 파일은 각 줄에 동의어 집합을 포함해야 합니다:

```
laptop, notebook
smartphone, cell phone, mobile phone
tablet, pad, slate
great, good, excellent
```

## 실무 활용 사례

1. **전자상거래 검색 개선**: 제품의 다양한 이름과 설명을 통일하여 검색 경험을 향상시킵니다.
2. **의학/법률 용어 처리**: 전문 용어와 일반 용어를 매핑하여 전문가와 일반인 모두가 효과적으로 검색할 수 있습니다.
3. **다국어 지원**: 여러 언어로 된 동일한 개념을 동의어로 처리할 수 있습니다.
4. **스펠링 변형 처리**: 미국식 영어와 영국식 영어의 철자 차이(예: color/colour)를 처리할 수 있습니다.

## 성능 고려사항

- **인덱싱 오버헤드**: 동의어 처리는 인덱싱 시간과 인덱스 크기를 증가시킬 수 있습니다.
- **검색 품질과 성능 균형**: 너무 많은 동의어는 검색 정확도를 떨어뜨릴 수 있습니다.
- **동의어 유지 관리**: 외부 파일을 사용하면 인덱스를 재구축하지 않고도 동의어를 업데이트할 수 있습니다.
- **인덱스 분석기 vs 검색 분석기**: 때로는 인덱싱과 검색에 다른 분석기를 사용하는 것이 효율적일 수 있습니다.

## 추가 팁

- 동의어 매핑은 단방향(one-way) 또는 양방향(two-way)으로 설정할 수 있습니다:
  - `laptop => notebook`(단방향): 'laptop'을 검색하면 'notebook'이 포함된 문서도 검색되지만, 그 반대는 아닙니다.
  - `laptop, notebook`(양방향): 두 용어 중 하나를 검색하면 다른 용어가 포함된 문서도 검색됩니다.
- 분석 체인의 순서가 중요합니다. 일반적으로 동의어 필터는 stemming 필터 전에 적용하는 것이 좋습니다.
- 검색 시 동의어를 동적으로 적용하려면 검색 시점 분석기를 사용할 수 있습니다.
