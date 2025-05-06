# 하이라이트란?
- 단어를 검색했을 때, 단어와 일치하는 term을 하이라이트한다.
- 구글, 네이버와 같은 검색 엔진에서 `Elastic Search` 라고 검색하면 `Elasitc` 혹은 `Search`와 일치하는 단어는 강조(`Strong`)되어서 출력된다.

## 하이라이트 사용법
(1) 일반 사용법
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

(2) term 앞뒤에 `em` 태그 아닌 다른 태그 붙이는 방법
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
    "pre_tags": ["<strong>"],   // strong 태그 추가~
    "post_tags": ["</strong>"]
  }
}
```