## 개요

Elasticsearch에서 검색 결과 정렬은 `sort` 매개변수를 사용하여 구현합니다. 정렬은 단일 필드 또는 여러 필드를 기준으로 수행할 수 있으며, 오름차순(ascending) 또는 내림차순(descending)으로 지정할 수 있습니다.

## 예제 데이터 설정

아래 예제에서는 블로그 글(articles)을 표현하는 인덱스를 생성하고 샘플 데이터를 추가합니다.

```
DELETE /articles
PUT /articles
{
  "mappings": {
    "properties": {
      "title": {
        "type": "text",
        "fields": {
          "keyword": {
            "type": "keyword"
          }
        }
      },
      "author": {
        "type": "text",
        "fields": {
          "keyword": {
            "type": "keyword"
          }
        }
      },
      "publish_date": {
        "type": "date"
      },
      "views": {
        "type": "integer"
      },
      "comments": {
        "type": "nested",
        "properties": {
          "user": {
            "type": "text"
          },
          "comment": {
            "type": "text"
          },
          "number_of_comments": {
            "type": "integer"
          }
        }
      }
    }
  }
}
```

샘플 문서 추가:

```
POST /articles/_doc/1
{
  "title": "Introduction to Elasticsearch",
  "author": "John Doe",
  "publish_date": "2023-01-01",
  "views": 150,
  "comments": [
    {
      "user": "Alice",
      "comment": "Great article!",
      "number_of_comments": 5
    },
    {
      "user": "Bob",
      "comment": "Very informative.",
      "number_of_comments": 3
    }
  ]
}

POST /articles/_doc/2
{
  "title": "Advanced Elasticsearch Techniques",
  "author": "Jane Smith",
  "publish_date": "2023-02-15",
  "views": 250,
  "comments": [
    {
      "user": "Charlie",
      "comment": "I learned a lot!",
      "number_of_comments": 8
    }
  ]
}

POST /articles/_doc/3
{
  "title": "Elasticsearch Sorting and Pagination",
  "author": "John Doe",
  "publish_date": "2023-03-05",
  "views": 100,
  "comments": [
    {
      "user": "Dave",
      "comment": "Very helpful.",
      "number_of_comments": 4
    },
    {
      "user": "Eve",
      "comment": "Thanks for the insights.",
      "number_of_comments": 2
    }
  ]
}

POST /articles/_doc/4
{
  "title": "Getting Started with Elasticsearch",
  "author": "Alice Brown",
  "publish_date": "2023-01-20",
  "views": 180,
  "comments": [
    {
      "user": "Frank",
      "comment": "Nice introduction.",
      "number_of_comments": 6
    }
  ]
}
```

## 정렬 예제

### 1. 날짜 기준 정렬 (최신순)

`publish_date` 필드를 기준으로 내림차순(최신순) 정렬:

```
GET /articles/_search
{
  "sort": [
    { "publish_date": { "order": "desc" } }
  ],
  "query": {
    "match_all": {}
  }
}
```

### 2. 숫자 필드 기준 정렬 (오름차순)

`views` 필드를 기준으로 오름차순(적은 것부터) 정렬:

```
GET /articles/_search
{
  "sort": [
    { "views": { "order": "asc" } }
  ],
  "query": {
    "match_all": {}
  }
}
```

### 3. 다중 필드 정렬

먼저 `author.keyword` 필드로 오름차순 정렬한 후, 같은 저자의 글은 `publish_date` 기준으로 내림차순(최신순) 정렬:

```
GET /articles/_search
{
  "sort": [
    { "author.keyword": { "order": "asc" } },
    { "publish_date": { "order": "desc" } }
  ],
  "query": {
    "match_all": {}
  }
}
```

### 4. Nested 필드 정렬

`comments.number_of_comments` 필드를 기준으로 내림차순 정렬하되, 각 문서의 중첩 객체 중 최대값을 사용:

```
GET /articles/_search
{
  "sort": [
    {
      "comments.number_of_comments": {
        "order": "desc",
        "mode": "max",
        "nested": {
          "path": "comments"
        }
      }
    }
  ],
  "query": {
    "match_all": {}
  }
}
```

### 5. 키워드 필드를 사용한 알파벳 정렬

`title.keyword` 필드를 사용하여 알파벳 순으로 정렬:

```
GET /articles/_search
{
  "sort": [
    { "title.keyword": { "order": "asc" } }
  ],
  "query": {
    "match_all": {}
  }
}
```

## 정렬 관련 중요 사항

1. **정렬과 필드 타입**: 
   - `text` 필드는 직접 정렬에 사용할 수 없습니다. 대신 `keyword` 필드나 `fields` 매핑의 `keyword` 하위 필드를 사용해야 합니다.
   - 숫자, 날짜, 지리적 위치 필드는 직접 정렬에 사용할 수 있습니다.

2. **정렬 모드**:
   - 배열이나 중첩 필드의 경우 `mode` 옵션을 설정할 수 있습니다: `min`, `max`, `sum`, `avg`, `median`
   - 중첩 필드 정렬 시 `nested` 매개변수를 지정해야 합니다.

3. **성능 고려사항**:
   - 정렬은 메모리를 많이 사용할 수 있으므로 큰 결과 세트 정렬 시 주의해야 합니다.
   - 자주 사용되는 정렬 필드에 대해 `doc_values`를 활성화해 두는 것이 좋습니다(기본값으로 활성화됨).

4. **페이지네이션과 함께 사용**:
   - 페이지네이션과 함께 정렬을 사용할 때는 일관된 결과를 위해 고유한 값을 가진 필드(예: ID 또는 타임스탬프)를 포함하는 것이 좋습니다.
