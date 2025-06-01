# 검색 결과 하이라이팅(Highlighting)

Elasticsearch에서 하이라이팅은 검색 결과에서 검색어와 일치하는 부분을 강조 표시하는 기능입니다. 이를 통해 사용자는 많은 텍스트 중에서 관련 정보를 빠르게 식별할 수 있습니다.

## 개요

하이라이팅 기능은 다음과 같은 장점을 제공합니다:

- **관련성 확인**: 사용자가 검색 결과에서 검색어의 컨텍스트를 바로 확인할 수 있습니다.
- **가독성 향상**: 중요한 부분을 강조하여 큰 텍스트에서 관련 정보를 빠르게 찾을 수 있습니다.
- **사용자 경험 개선**: 검색 결과의 시각적 품질을 높여 사용자 만족도를 향상시킵니다.

## 인덱스 설정 및 테스트 데이터

먼저 간단한 인덱스와 테스트 데이터를 생성합니다:

```
DELETE /articles

PUT /articles
{
  "mappings": {
    "properties": {
      "title": { "type": "text" },
      "content": { "type": "text" }
    }
  }
}

POST /articles/_doc/1
{
  "title": "The quick brown fox",
  "content": "The quick brown fox jumps over the lazy dog."
}

POST /articles/_doc/2
{
  "title": "The fast fox",
  "content": "A fast fox swiftly leaped over a sleepy dog."
}

POST /articles/_doc/3
{
  "title": "A slow green turtle",
  "content": "The slow green turtle walked under the energetic rabbit."
}
```

## 기본 하이라이팅

가장 기본적인 하이라이팅 방법은 다음과 같습니다:

```
GET /articles/_search
{
  "query": {
    "match": {
      "content": "fox"
    }
  },
  "highlight": {
    "fields": {
      "content": {}
    }
  }
}
```

**결과:**
```json
{
  "took": 10,
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
        "_index": "articles",
        "_id": "1",
        "_score": 0.6931471,
        "_source": {
          "title": "The quick brown fox",
          "content": "The quick brown fox jumps over the lazy dog."
        },
        "highlight": {
          "content": [
            "The quick brown <em>fox</em> jumps over the lazy dog."
          ]
        }
      },
      {
        "_index": "articles",
        "_id": "2",
        "_score": 0.6931471,
        "_source": {
          "title": "The fast fox",
          "content": "A fast fox swiftly leaped over a sleepy dog."
        },
        "highlight": {
          "content": [
            "A fast <em>fox</em> swiftly leaped over a sleepy dog."
          ]
        }
      }
    ]
  }
}
```

이 결과에서 'fox'라는 단어는 `<em>` 태그로 강조 표시되었습니다.

## 사용자 정의 태그 사용

기본 `<em>` 태그 대신 사용자 정의 태그를 사용할 수 있습니다:

```
GET /articles/_search
{
  "query": {
    "match": {
      "content": "fox"
    }
  },
  "highlight": {
    "fields": {
      "content": {}
    },
    "pre_tags": ["<strong>"],
    "post_tags": ["</strong>"]
  }
}
```

**결과:**
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
        "_index": "articles",
        "_id": "1",
        "_score": 0.6931471,
        "_source": {
          "title": "The quick brown fox",
          "content": "The quick brown fox jumps over the lazy dog."
        },
        "highlight": {
          "content": [
            "The quick brown <strong>fox</strong> jumps over the lazy dog."
          ]
        }
      },
      {
        "_index": "articles",
        "_id": "2",
        "_score": 0.6931471,
        "_source": {
          "title": "The fast fox",
          "content": "A fast fox swiftly leaped over a sleepy dog."
        },
        "highlight": {
          "content": [
            "A fast <strong>fox</strong> swiftly leaped over a sleepy dog."
          ]
        }
      }
    ]
  }
}
```

이제 'fox'라는 단어는 `<strong>` 태그로 강조 표시되었습니다.

## 고급 하이라이팅 옵션

Elasticsearch는 다양한, 하이라이팅 옵션을 제공합니다:

### 여러 필드 하이라이팅

여러 필드에 하이라이팅을 적용할 수 있습니다:

```
GET /articles/_search
{
  "query": {
    "multi_match": {
      "query": "fox",
      "fields": ["title", "content"]
    }
  },
  "highlight": {
    "fields": {
      "title": {},
      "content": {}
    }
  }
}
```

### 프래그먼트 크기 및 개수 설정

하이라이팅된 텍스트 주변의 컨텍스트 크기와 반환할 프래그먼트 수를 지정할 수 있습니다:

```
GET /articles/_search
{
  "query": {
    "match": {
      "content": "fox"
    }
  },
  "highlight": {
    "fields": {
      "content": {
        "fragment_size": 50,
        "number_of_fragments": 2
      }
    }
  }
}
```

### 하이라이터 유형 지정

Elasticsearch는 여러 하이라이터 유형을 지원합니다:

```
GET /articles/_search
{
  "query": {
    "match": {
      "content": "fox"
    }
  },
  "highlight": {
    "fields": {
      "content": {
        "type": "unified"
      }
    }
  }
}
```

사용 가능한 하이라이터 유형:
- **unified**: 기본값, 정확도와 성능의 균형을 제공합니다.
- **plain**: 간단하고 빠르지만 정확도가 낮을 수 있습니다.
- **fvh**(Fast Vector Highlighter): 더 정확하지만 필드에 term_vector 설정이 필요합니다.

### 하이라이팅 쿼리 지정

검색 쿼리와 다른 쿼리로 하이라이팅을 적용할 수 있습니다:

```
GET /articles/_search
{
  "query": {
    "bool": {
      "must": [
        { "match": { "content": "fox" } }
      ],
      "filter": [
        { "term": { "title.keyword": "The fast fox" } }
      ]
    }
  },
  "highlight": {
    "fields": {
      "content": {
        "highlight_query": {
          "match": { "content": "fox dog" }
        }
      }
    }
  }
}
```

### 요약 하이라이팅

긴 텍스트의 경우 관련 부분만 보여주는 요약을 생성할 수 있습니다:

```
GET /articles/_search
{
  "query": {
    "match": {
      "content": "fox"
    }
  },
  "highlight": {
    "fields": {
      "content": {
        "fragment_size": 30,
        "number_of_fragments": 1,
        "no_match_size": 0
      }
    }
  }
}
```

## 실무 활용 사례

1. **검색 엔진**: 검색 결과에서 키워드 강조 표시로 사용자 경험 향상
2. **콘텐츠 관리 시스템**: 문서 검색 시 관련 섹션 강조
3. **전자상거래**: 제품 설명에서 검색어와 일치하는 부분 강조
4. **로그 분석**: 에러 메시지나 특정 패턴이 발생한 부분 식별

## 성능 고려사항

- 하이라이팅은 추가적인 처리를 필요로 하므로 검색 성능에 영향을 줄 수 있습니다.
- 대용량 문서나 많은 결과에서 하이라이팅을 사용할 때는 프래그먼트 크기와 개수를 제한하는 것이 좋습니다.
- 하이라이터 유형에 따라 성능과 정확도의 균형이 달라집니다.
- 필드가 매우 큰 경우 `stored` 설정을 활성화하면 하이라이팅 성능이 향상될 수 있습니다.

## 추가 팁

- HTML이 아닌 환경에서는 `pre_tags`와 `post_tags`를 사용하여 다른 형식의 마커를 지정할 수 있습니다.
- 일치하는 용어가 없을 때의 동작은 `no_match_size` 파라미터로 제어할 수 있습니다.
- 필드 내용의 일부만 반환하고 싶을 때는 `number_of_fragments: 0`으로 설정하여 전체 필드 콘텐츠를 하이라이팅할 수 있습니다.
- 분석기 설정에 따라 하이라이팅 결과가 달라질 수 있으므로, 필요한 경우 `highlight_query`를 사용하여 특정 분석기를 적용할 수 있습니다.
