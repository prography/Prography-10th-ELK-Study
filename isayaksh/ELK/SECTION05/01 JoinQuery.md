# RDB의 JOIN
- `부모 -자식` 관계를 가진다.
- 일반적으로 1개의 부모가 여러개의 자식을 갖는 형태로 이루어진다.
  - EX: 주문(1개) - 주문상품(N개)
- RDB는 정규화를 통해 중복 데이터를 여러개의 테이블에 적재하지 않기 위해 노력한다.
- 하지만, ES에서는 중복 데이터를 분산하여 저장하여 조회를 효율적으로 처리한다.
- 즉, **RDB는 스토리지를 효율적으로 관리하고, ES는 조회 성능을 효과적으로 처리하는데 강점**이 있다.

# ES의 JOIN
- `join` 필드 타입을 사용하여 부모-자식 문서 간의 관계를 관리한다. 이 필드를 통해 두 문서 간의 계층적 관계를 정의할 수 있다.
- 부모 문서(Parent Document): 주체가 되는 기본 엔티티이다. 예: 작가(author)
- 자식 문서(Child Document): 부모와 관련된 부속 엔티티이다. 예: 작가의 책들(books)
- 즉, 하나의 작가 문서 아래에 여러 권의 책 문서를 연결할 수 있도록 구조화된 관계를 만드는 방식이다.

### 🔥 부모-자식 문서 같은 샤드에 저장!
- **부모 문서와 자식문서는 같은 샤드에 저장**되어야 하기 때문에 **자식 문서를 저장할 때 반드시 `routing`을 통해 부모 문서를 포인팅**해야 한다.
```
POST /library/_doc/1
{
  "my_join_field": "author",  // Indicating this is a parent document
  "name": "J.K. Rowling"
}

POST /library/_doc/3?routing=1
{
  "my_join_field": {
    "name": "book",
    "parent": "1"  // Link to the parent author (J.K. Rowling)
  },
  "title": "Harry Potter and the Philosopher's Stone",
  "genre": "Fantasy",
  "year": 1997
}
```
- RDB는 JOIN을 위해 여러 테이블을 조합한다.
- 하지만, ES는 기본적으로 분산 시스템이라, 서로 다른 샤드에 있는 문서는 직접적으로 조인할 수 없다.
- 같은 샤드에 있어야만 내부적으로 parent-child 관계를 유지하고 **쿼리 성능을 확보**할 수 있다.

# 부모 ID로 자식 문서 찾는 방법
```
GET /library/_search
{
  "query": {
    "parent_id": {
      "type": "book",   // 자식 문서의 타입 (join 필드에서 정의한 이름)
      "id": "1"         // 부모 문서의 _id
    }
  }
}
```

### 🔥 부모 문서만 조회하는 방법
```
GET /library/_search
{
  "query": {
    "term": {
      "my_join_field": "author"
    }
  }
}
```

### 🔥 자식 문서만 조회하는 방법
```
GET /library/_search
{
  "query": {
    "term": {
      "my_join_field.name": "book"
    }
  }
}
```