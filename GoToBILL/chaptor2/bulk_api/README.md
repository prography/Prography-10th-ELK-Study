# Elasticsearch Bulk API

Bulk API는 Elasticsearch에서 여러 문서에 대한 작업(인덱싱, 업데이트, 삭제)을 단일 요청으로 처리할 수 있게 해주는 중요한 API입니다. 이 API를 사용하면 네트워크 왕복을 줄이고 성능을 크게 향상시킬 수 있습니다.

## 1. Bulk API 기본 개념

Bulk API는 여러 작업을 한 번의 HTTP 요청으로 처리합니다. 각 작업은 두 줄로 구성됩니다:
1. 작업 메타데이터: 어떤 작업을 수행할지 지정 (인덱스, 생성, 업데이트, 삭제)
2. 작업 데이터: 해당 작업에 필요한 문서 데이터 (삭제 작업은 제외)

## 2. Bulk API 문법

Bulk API는 다음과 같은 형식을 사용합니다:

```
POST /_bulk
{ "index" : { "_index" : "test", "_id" : "1" } }
{ "field1" : "value1" }
{ "delete" : { "_index" : "test", "_id" : "2" } }
{ "create" : { "_index" : "test", "_id" : "3" } }
{ "field1" : "value3" }
{ "update" : { "_index" : "test", "_id" : "4" } }
{ "doc" : { "field2" : "value2" } }
```

### 2.1 지원하는 작업 유형

- **index**: 문서를 인덱스에 추가하거나 기존 문서를 대체합니다.
- **create**: 문서를 인덱스에 추가합니다. 같은 ID의 문서가 이미 존재하면 실패합니다.
- **update**: 기존 문서를 부분 업데이트합니다.
- **delete**: 문서를 삭제합니다.

### 2.2 특정 인덱스에 대한 Bulk 작업

특정 인덱스에 대한 Bulk 작업을 수행할 수도 있습니다:

```
POST /my_index/_bulk
{ "index" : { "_id" : "1" } }
{ "field1" : "value1" }
{ "update" : { "_id" : "2" } }
{ "doc" : { "field2" : "value2" } }
```

이 경우 모든 작업이 `my_index`에 적용되므로, 각 작업 메타데이터에서 `_index`를 지정할 필요가 없습니다.

## 3. Bulk API 요청 형식

Bulk API는 일반적인 JSON 배열을 사용하지 않고, 각 작업을 새 줄로 구분된 JSON 객체(NDJSON)로 제공해야 합니다:

```
{ "action": { "metadata" } }\n
{ "data" }\n
```

각 줄은 반드시 줄 바꿈 문자(`\n`)로 끝나야 하며, 마지막 줄도 예외가 아닙니다.

## 4. Bulk API 응답

Bulk API는 각 작업의 결과를 포함한 JSON 객체를 반환합니다:

```
{
  "took": 30,
  "errors": false,
  "items": [
    {
      "index": {
        "_index": "test",
        "_id": "1",
        "_version": 1,
        "result": "created",
        "status": 201
      }
    },
    {
      "delete": {
        "_index": "test",
        "_id": "2",
        "status": 404,
        "error": {
          "type": "document_missing_exception",
          "reason": "[2]: document missing"
        }
      }
    },
    {
      "create": {
        "_index": "test",
        "_id": "3",
        "_version": 1,
        "result": "created",
        "status": 201
      }
    },
    {
      "update": {
        "_index": "test",
        "_id": "4",
        "_version": 2,
        "result": "updated",
        "status": 200
      }
    }
  ]
}
```
응답에는 다음 정보가 포함됩니다:
- `took`: 작업 수행에 소요된 시간(밀리초)
- `errors`: 하나 이상의 작업이 실패했는지 여부
- `items`: 각 작업의 결과를 포함하는 배열

## 5. Bulk API 최적화 및 성능 팁

### 5.1 최적의 배치 크기

Bulk 요청의 최적 크기는 문서 크기, 클러스터 구성, 하드웨어 등에 따라 다르지만, 일반적으로 다음 가이드라인을 따를 수 있습니다:

- 배치당 1,000~5,000개의 문서
- 배치당 5MB~15MB의 데이터

### 5.2 성능 최적화 팁

- **적절한 배치 크기 선택**: 너무 크거나 작은 배치는 성능을 저하시킬 수 있습니다.
- **충분한 메모리 할당**: Bulk 요청은 메모리를 많이 사용할 수 있으므로, JVM 힙 크기를 적절히 설정하세요.
- **병렬 처리**: 여러 클라이언트에서 병렬로 Bulk 요청을 보내면 처리량을 높일 수 있습니다.
- **재시도 메커니즘 구현**: 일시적인 오류로 인한 실패를 처리하기 위해 재시도 로직을 구현하세요.
- **연결 재사용**: HTTP 연결을 재사용하여 연결 설정 오버헤드를 줄이세요.

## 6. 실제 사용 예시

### 6.1 Python에서 Bulk API 사용 예시

```
from elasticsearch import Elasticsearch, helpers

es = Elasticsearch(["http://localhost:9200"])

actions = [
    {"_index": "test", "_id": "1", "_source": {"field1": "value1"}},
    {"_index": "test", "_id": "2", "_source": {"field1": "value2"}},
    {"_index": "test", "_id": "3", "_source": {"field1": "value3"}}
]

helpers.bulk(es, actions)
```

### 6.2 JSON 파일에서 대량의 데이터 로드

```
import json
from elasticsearch import Elasticsearch, helpers

es = Elasticsearch(["http://localhost:9200"])

def load_json(file_path):
    with open(file_path, 'r') as f:
        return json.load(f)

data = load_json("data.json")

actions = [
    {
        "_index": "my_index",
        "_id": doc["id"],
        "_source": doc
    }
    for doc in data
]

helpers.bulk(es, actions)
```

## 7. 주의사항 및 제한사항

- **단일 작업 실패**: 하나의 작업이 실패해도 다른 작업은 계속 처리됩니다.
- **전체 일관성 보장 없음**: Bulk API는 트랜잭션을 지원하지 않으므로, 일부 작업만 성공하는 경우가 있을 수 있습니다.
- **메모리 사용량**: 큰 Bulk 요청은 많은 메모리를 사용할 수 있으므로 주의해야 합니다.
- **타임아웃**: 큰 Bulk 요청은 타임아웃될 수 있으므로, 클라이언트와 서버 측 타임아웃 설정을 적절히 조정해야 합니다.

## 8. Bulk API와 동시성 제어

Bulk API에서 버전 기반 동시성 제어를 사용할 수 있습니다:

```
POST /_bulk
{ "index" : { "_index" : "test", "_id" : "1", "version" : 1, "version_type" : "external" } }
{ "field1" : "value1" }
```

각 작업 메타데이터에 `version`과 `version_type`을 지정하여 동시성 충돌을 방지할 수 있습니다.

## 결론

Elasticsearch Bulk API는 대량의 데이터를 효율적으로 처리하기 위한 강력한 도구입니다. 네트워크 왕복을 줄이고 인덱싱 성능을 크게 향상시킬 수 있으며, 대량 데이터 로딩, ETL 프로세스, 로그 수집 등의 시나리오에서 특히 유용합니다.

적절한 배치 크기와 최적화 전략을 사용하여 Bulk API의 성능을 최대화하고, 응용 프로그램의 요구사항에 맞게 구성하는 것이 중요합니다.