# UPSERT와 REPLACE 연산

Elasticsearch에서는 데이터를 업데이트하는 다양한 방식을 제공합니다. 여기서는 UPSERT와 REPLACE 연산에 대해 알아보겠습니다.

## Upsert (업서트)

**Upsert**는 **Update**와 **Insert**를 결합한 연산입니다. 즉, 주어진 문서 ID에 해당하는 문서가 존재하면 **업데이트**를 하고, 없으면 **새로운 문서를 삽입**합니다.

### 기본 동작

- **문서가 이미 존재하면** 해당 문서를 **업데이트**합니다. (앞에서 설명했듯이 해당 문서를 업데이트하는 것이 아닌 버전을 바꾸는 것이다.)
- **문서가 존재하지 않으면** **새로운 문서**를 생성합니다.

### 예시 1: 문서 필드 업데이트

```
POST /my_index/_update/5
{
  "doc": { "author": "John Doe" },
  "upsert": {
    "title": "Elasticsearch Basics",
    "author": "Jane Doe",
    "publish_date": "2024-05-09",
    "tags": ["search", "analytics"]
  }
}
```

#### 동작 설명

- 문서 ID가 `5`인 문서가 **존재하면**, `author`를 `"John Doe"`로 업데이트합니다.
- 문서 ID가 `5`인 문서가 **존재하지 않으면**, 새로운 문서를 삽입하며, 
  그 내용은 `"title": "Elasticsearch Basics"`, `"author": "Jane Doe"`, `"publish_date": "2024-05-09"`, `"tags": ["search", "analytics"]`입니다.

### 예시 2: 스크립트와 함께 사용

```
POST /my_index/_update/7
{
  "script": {
    "source": "ctx._source.price++"
  },
  "upsert": {
    "title": "Elasticsearch Basics",
    "author": "Jane Doe",
    "publish_date": "2024-05-09",
    "price": 100,
    "tags": ["search", "analytics"]
  }
}
```

#### 동작 설명

- 문서 ID가 `7`인 문서가 **존재하면**, `price` 필드를 **1 증가**시킵니다.
- 문서 ID가 `7`인 문서가 **존재하지 않으면**, 새 문서를 삽입하며 `"price": 100` 값을 추가합니다.

## Replace (교체)

**Replace** 연산은 **기존 문서를 완전히 교체**하는 방식입니다. 즉, 문서가 존재하면 해당 문서를 새 데이터로 **덮어씁니다**. 삽입할 데이터가 제공되면 그 데이터로 기존의 문서 내용이 모두 교체됩니다.

### 예시 1: 문서 생성/교체

```
PUT /my_index/_doc/20
{
  "title": "Elasticsearch Basics",
  "author": "John Doe",
  "publish_date": "2024-05-09",
  "tags": ["search", "analytics"]
}
```

#### 동작 설명

문서 ID가 `20`인 문서를 **새로운 데이터로 교체**합니다. 기존 문서가 있으면 덮어쓰며, 없으면 새로 삽입됩니다.

### 예시 2: 문서 업데이트

```
PUT /my_index/_doc/20
{
  "title": "Updated Elasticsearch Basics",
  "author": "Jane Doe",
  "publish_date": "2024-06-01",
  "tags": ["search", "analytics", "updated"]
}
```

#### 동작 설명

문서 ID가 `20`인 문서를 **완전히 덮어쓰고** `"title"`을 `"Updated Elasticsearch Basics"`로, `"author"`를 `"Jane Doe"`로, `"publish_date"`를 `"2024-06-01"`로 변경하며, `"tags"`도 `"search", "analytics", "updated"`로 업데이트합니다.

## 핵심 차이점

- **Upsert**는 **문서가 존재하면 업데이트**, **문서가 없으면 삽입**하는 방식입니다. 기존 문서를 **부분적으로 수정**할 수 있는 기능이 제공됩니다.

- **Replace**는 문서가 존재하면 **완전히 덮어쓰고**, 새로 삽입하는 방식입니다. 기존 문서가 **완전히 교체**됩니다.

## 정리

- **Upsert**는 문서가 존재하지 않으면 새로 삽입하고, 존재하면 업데이트하는 **혼합 연산**입니다.
- **Replace**는 문서를 **완전히 덮어쓰는** 방식으로, 기존 문서를 새로 교체하는 연산입니다.
