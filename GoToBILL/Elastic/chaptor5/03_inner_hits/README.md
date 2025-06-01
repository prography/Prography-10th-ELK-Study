# Chapter 5 - 03_inner_hits

## Elasticsearch에서 부모-자식 관계의 Inner Hits

Elasticsearch에서 Inner Hits는 부모-자식 관계나 중첩 문서 검색에서 매우 유용한 기능입니다. 특히 `has_parent` 또는 `has_child` 쿼리를 사용할 때, 매칭된 문서와 관련된 문서들을 함께 볼 수 있게 해줍니다.

### Inner Hits의 필요성

부모-자식 관계에서:
- `has_parent` 쿼리는 특정 부모 문서와 연결된 자식 문서만 반환합니다
- `has_child` 쿼리는 특정 자식 문서와 연결된 부모 문서만 반환합니다

하지만 실제 사용 시에는 매칭된 문서와 함께 관련 문서도 같이 보고 싶을 때가 많습니다. 예를 들어:
- "Fantasy 장르의 책이 있는 저자"를 검색하면서, 어떤 Fantasy 책들이 매칭되었는지 함께 보고 싶을 때
- "J.K. Rowling의 책"을 검색하면서, 저자 정보도 함께 확인하고 싶을 때

이러한 경우 Inner Hits를 사용하면 검색 쿼리 하나로 관련 정보를 모두 가져올 수 있습니다.

### 부모-자식 관계에서 Inner Hits 사용 예제

앞서 생성한 library 인덱스를 사용한 예제를 살펴보겠습니다.

#### 1. has_child 쿼리에서 Inner Hits 사용하기

특정 조건의 책을 가진 저자를 검색하면서, 어떤 책이 매칭되었는지 함께 확인:

```elasticsearch
GET /library/_search
{
  "query": {
    "has_child": {
      "type": "book",
      "query": {
        "term": {
          "genre": "Fantasy"
        }
      },
      "inner_hits": {}
    }
  }
}
```

이 쿼리는 Fantasy 장르의 책을 가진 모든 저자를 검색하며, 각 저자별로 어떤 Fantasy 책들이 매칭되었는지 inner_hits 항목에 포함됩니다.

**예상 결과:**
```json
{
  "hits": {
    "total": { "value": 2, "relation": "eq" },
    "hits": [
      {
        "_index": "library",
        "_id": "1",
        "_score": 1.0,
        "_source": {
          "my_join_field": "author",
          "name": "J.K. Rowling"
        },
        "inner_hits": {
          "book": {
            "hits": {
              "total": { "value": 2, "relation": "eq" },
              "hits": [
                {
                  "_source": {
                    "title": "Harry Potter and the Philosopher's Stone",
                    "genre": "Fantasy",
                    "year": 1997
                  }
                },
                {
                  "_source": {
                    "title": "Harry Potter and the Chamber of Secrets",
                    "genre": "Fantasy",
                    "year": 1998
                  }
                }
              ]
            }
          }
        }
      },
      {
        "_index": "library",
        "_id": "2",
        "_score": 1.0,
        "_source": {
          "my_join_field": "author",
          "name": "George R.R. Martin"
        },
        "inner_hits": {
          "book": {
            "hits": {
              "total": { "value": 2, "relation": "eq" },
              "hits": [
                {
                  "_source": {
                    "title": "A Game of Thrones",
                    "genre": "Fantasy",
                    "year": 1996
                  }
                },
                {
                  "_source": {
                    "title": "A Clash of Kings",
                    "genre": "Fantasy",
                    "year": 1998
                  }
                }
              ]
            }
          }
        }
      }
    ]
  }
}
```

#### 2. has_parent 쿼리에서 Inner Hits 사용하기

특정 부모 문서와 연결된 자식 문서를 검색하면서, 부모 정보도 함께 확인:

```elasticsearch
GET /library/_search
{
  "query": {
    "has_parent": {
      "parent_type": "author",
      "query": {
        "match": {
          "name": "J.K. Rowling"
        }
      },
      "inner_hits": {}
    }
  }
}
```

이 쿼리는 J.K. Rowling이 쓴 모든 책을 검색하며, 각 책 결과에 저자 정보도 inner_hits 항목으로 포함됩니다.

**예상 결과:**
```json
{
  "hits": {
    "total": { "value": 2, "relation": "eq" },
    "hits": [
      {
        "_index": "library",
        "_id": "3",
        "_score": 1.0,
        "_routing": "1",
        "_source": {
          "my_join_field": {
            "name": "book",
            "parent": "1"
          },
          "title": "Harry Potter and the Philosopher's Stone",
          "genre": "Fantasy",
          "year": 1997
        },
        "inner_hits": {
          "author": {
            "hits": {
              "total": { "value": 1, "relation": "eq" },
              "hits": [
                {
                  "_source": {
                    "my_join_field": "author",
                    "name": "J.K. Rowling"
                  }
                }
              ]
            }
          }
        }
      },
      {
        "_index": "library",
        "_id": "4",
        "_score": 1.0,
        "_routing": "1",
        "_source": {
          "my_join_field": {
            "name": "book",
            "parent": "1"
          },
          "title": "Harry Potter and the Chamber of Secrets",
          "genre": "Fantasy",
          "year": 1998
        },
        "inner_hits": {
          "author": {
            "hits": {
              "total": { "value": 1, "relation": "eq" },
              "hits": [
                {
                  "_source": {
                    "my_join_field": "author",
                    "name": "J.K. Rowling"
                  }
                }
              ]
            }
          }
        }
      }
    ]
  }
}
```

### Inner Hits 옵션 설정

Inner Hits는 다양한 옵션을 설정할 수 있습니다:

```elasticsearch
GET /library/_search
{
  "query": {
    "has_child": {
      "type": "book",
      "query": {
        "match": {
          "genre": "Fantasy"
        }
      },
      "inner_hits": {
        "name": "matching_books",  // Inner Hits 결과에 사용할 이름
        "size": 3,                // 반환할 Inner Hits 문서 수
        "_source": {              // 반환할 필드 지정
          "includes": ["title", "year"]
        },
        "sort": [                // Inner Hits 정렬 방법
          { "year": "desc" }
        ]
      }
    }
  }
}
```

### Inner Hits의 장점

1. **단일 쿼리로 관련 정보 검색**: 여러 쿼리를 수행할 필요 없이 한 번의 요청으로 관련 문서 정보를 얻을 수 있습니다.

2. **성능 최적화**: 관련 문서를 찾기 위한 추가 쿼리가 필요하지 않으므로 성능이 향상됩니다.

3. **결과 컨텍스트 유지**: 매칭된, 모든 문서 사이의 관계가 결과에 명확하게 표시됩니다.

4. **검색 결과 필터링 및 정렬**: Inner Hits 내에서도 필터링, 정렬, 페이징 등을 적용할 수 있어 유연성이 높습니다.

### 사용 시 주의사항

1. **결과 크기 관리**: Inner Hits로 너무 많은 관련 문서를 요청하면 응답 크기가 커질 수 있으므로, `size` 옵션을 적절히 설정해야 합니다.

2. **필드 필터링**: `_source` 필터링을 사용하여 필요한 필드만 반환하도록 설정하는 것이 좋습니다.

3. **라우팅 고려**: 부모-자식 관계에서 라우팅은 여전히 중요합니다. Inner Hits를 사용할 때도 적절한 라우팅 값을 지정해야 합니다.
