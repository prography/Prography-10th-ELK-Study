## RDBMS와 Elasticsearch의 조인 비교

### 전통적인 RDBMS에서의 조인
- 부모-자식 관계를 기반으로 합니다
- 데이터 정규화를 통해 저장 공간을 최적화합니다
- 예시: 저자(부모)와 책(자식) 간의 관계

### Elasticsearch에서의 조인 쿼리
- Elasticsearch도 부모-자식 관계를 사용합니다
- 'join' 필드 타입을 사용하여 관계를 관리합니다
- 부모 문서는 주요 엔티티(예: 저자)입니다
- 자식 문서는 부모와 연관된 엔티티(예: 저자의 책)입니다
- 한 부모 문서는 여러 자식 문서와 연결될 수 있습니다

## 핵심 차이점
- RDBMS는 정규화된 테이블 간의 관계에 중점을 둡니다
- Elasticsearch는 문서 중심 접근 방식으로 관계를 관리합니다
- Elasticsearch의 join은 특별한 필드 타입을 통해 구현됩니다

## Elasticsearch Join 필드 코드 분석

### 인덱스 생성 및 매핑 설정
```elasticsearch
DELETE /library
PUT /library
{
  "mappings": {
    "properties": {
      "my_join_field": {
        "type": "join",
        "relations": {
          "author": "book"  // 'author'를 부모로, 'book'을 자식으로 정의
        }
      },
      "name": { "type": "text" },       // 이름을 위한 공통 필드
      "title": { "type": "text" },      // 책 제목을 위한 필드
      "genre": { "type": "keyword" },   // 책 장르를 위한 필드
      "year": { "type": "integer" }     // 출판 연도를 위한 필드
    }
  }
}
```

### 부모 문서(저자) 생성
```elasticsearch
POST /library/_doc/1
{
  "my_join_field": "author",  // 이것이 부모 문서임을 표시
  "name": "J.K. Rowling"
}

POST /library/_doc/2
{
  "my_join_field": "author",  // 이것이 부모 문서임을 표시
  "name": "George R.R. Martin"
}
```

### 자식 문서(책) 생성
```elasticsearch
POST /library/_doc/3?routing=1
{
  "my_join_field": {
    "name": "book",
    "parent": "1"  // 부모 저자(J.K. Rowling)와 연결
  },
  "title": "Harry Potter and the Philosopher's Stone",
  "genre": "Fantasy",
  "year": 1997
}

POST /library/_doc/4?routing=1
{
  "my_join_field": {
    "name": "book",
    "parent": "1"  // 부모 저자(J.K. Rowling)와 연결
  },
  "title": "Harry Potter and the Chamber of Secrets",
  "genre": "Fantasy",
  "year": 1998
}

POST /library/_doc/5?routing=2
{
  "my_join_field": {
    "name": "book",
    "parent": "2"  // 부모 저자(George R.R. Martin)와 연결
  },
  "title": "A Game of Thrones",
  "genre": "Fantasy",
  "year": 1996
}

POST /library/_doc/6?routing=2
{
  "my_join_field": {
    "name": "book",
    "parent": "2"  // 부모 저자(George R.R. Martin)와 연결
  },
  "title": "A Clash of Kings",
  "genre": "Fantasy",
  "year": 1998
}
```

### 주요 특징
- `routing` 파라미터: 부모와 자식 문서가 같은 샤드에 저장되도록 보장합니다
  - 같은 샤드 안에 저장해야 오류가 안 납니다.
- `parent` 필드: 자식 문서와 부모 문서를 연결하는 ID 참조입니다

### 조인 쿼리 예제

#### Elasticsearch 조인 쿼리 타입 설명
1. `has_parent` 쿼리
   - 자식 문서를 검색하면서 부모 문서의 조건을 적용합니다
   - "특정 부모(저자)를 가진 자식(책)들을 찾는다"는 의미입니다
   - 결과는 항상 자식 문서(책)입니다

2. `has_child` 쿼리
   - 부모 문서를 검색하면서 자식 문서의 조건을 적용합니다
   - "특정 조건의 자식(책)을 가진 부모(저자)를 찾는다"는 의미입니다
   - 결과는 항상 부모 문서(저자)입니다

#### 특정 부모(저자)에 속한 자식(책) 검색
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
      }
    }
  }
}
```
이 쿼리는 이름이 "J.K. Rowling"인 저자(부모)를 가진 모든 책(자식)을 반환합니다.

#### 특정 조건을 만족하는 자식(책)을 가진 부모(저자) 검색
```elasticsearch
GET /library/_search
{
  "query": {
    "has_child": {
      "type": "book",
      "query": {
        "match": {
          "title": "Game"
        }
      }
    }
  }
}
```
이 쿼리는 제목에 "Game"이 포함된 책을 가진 저자(George R.R. Martin)를 반환합니다.

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
      }
    }
  }
}
```
이 쿼리는 Fantasy 장르의 책을 가진 모든 저자(J.K. Rowling과 George R.R. Martin)를 반환합니다.