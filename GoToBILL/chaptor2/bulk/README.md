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