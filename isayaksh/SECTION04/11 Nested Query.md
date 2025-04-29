# Nested Query
- 객체 내 필드를 쿼리하는 데 사용되는 쿼리이다. nested 객체는 Elasticsearch에서 객체 배열을 처리하기 위해 설계된 특수한 타입이며, 각 객체는 별도의 개체로 처리되어야 한다.
- nested 쿼리는 이러한 nested 객체 내부를 검색하면서도, nested 문서와 상위 문서 간의 관계를 유지할 수 있게 하는 기능이다.
- 🔥 일반 object는 배열 내 필드의 위치 관계를 무시하고 필드 전체에서 조건만 맞으면 문서가 매칭됨. 그래서 서로 다른 객체의 필드가 조건을 충족해도, 문서가 잘못 매칭되는 현상이 생긴다. 이걸 방지하려면 nested object로 선언하고 nested query를 사용해야 한다.

### 예시

**문서**
```json
{
  "_id": "1",
  "products": [
    { "name": "kim", "price": 100 },
    { "name": "lee", "price": 200 }
  ]
}
```

**조회**
```json
{
  "query": {
    "bool": {
      "must": [
        { "match": { "products.name": "kim" }},
        { "match": { "products.price": 200 }}
      ]
    }
  }
}
```
- 일반 object일 경우 결과 : "kim"과 "200"이 각각 존재하므로 → 조건 충족
- Elasticsearch는 배열 안 객체가 서로 다른 거라도 신경 안 씀.
- 해당 문서 전체가 매칭된 걸로 간주

# Why Use Nested Queries?
- 일반 object 필드 → Elasticsearch는 객체를 평탄화하며 필드 간의 관계를 잃게 된다.
  - 예를 들어, name과 price 필드를 가진 product 배열이 있는 경우, 일반 쿼리는 하나의 product에서 name을, 다른 product에서 price를 잘못 매칭할 수 있다.
- 해결 방법
  - Nested Object: 필드를 nested로 정의하면, 배열의 각 객체는 별도의 숨겨진 문서로 인덱싱된다. 이 문서들은 상위 문서와 연결된다.
  - Nested Query: nested 쿼리는 이러한 nested 문서 내부를 상위 문서와의 관계를 유지한 채로 검색할 수 있게 해준다.

### 🔥 정리
```json
{
  "_id": "1",
  "products": [
    { "name": "kim", "price": 100 },
    { "name": "lee", "price": 200 }
  ]
}
```
- 위와 같은 데이터에서 object는 데이터를 아래와 같이 관리한다.
```json
{
    "name" : ["kim", "lee"]
    "price" : [100, 200]
}
```

- nested object는 아래와 같이 관리한다.
```json
{
    [{"name": "kim", "price": 100}, {"name": "lee", "price": 200}]
}
```