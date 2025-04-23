# Elasticsearch Mapping (매핑)

Elasticsearch에서 매핑은 인덱스의 문서와 필드를 저장하고 인덱싱하는 방법을 정의합니다. 이는 관계형 데이터베이스의 스키마와 유사한 개념입니다. 이 문서에서는 Elasticsearch 매핑의 중요한 개념과 사용법을 설명합니다.

## 1. 매핑 개요

### 1.1 매핑의 역할

매핑은 다음과 같은 역할을 합니다:
- 필드의 데이터 타입 정의 (문자열, 숫자, 날짜 등)
- 필드 인덱싱 방법 설정
- 텍스트 분석 방법 구성
- 필드 간의 관계 정의 (중첩 객체, 부모-자식 관계)

### 1.2 동적 매핑 vs 명시적 매핑

Elasticsearch는 두 가지 매핑 접근 방식을 제공합니다:

#### 동적 매핑 (Dynamic Mapping)
- 문서를 인덱싱할 때 자동으로 필드 타입을 감지하고 매핑을 생성합니다.
- 빠르게 시작할 수 있지만, 복잡한 데이터 구조에서는 정확하지 않을 수 있습니다.

#### 명시적 매핑 (Explicit Mapping)
- 인덱스 생성 시 사전에 매핑을 정의합니다.
- 더 많은 제어와 정확성을 제공하지만, 초기 설정이 필요합니다.

## 2. 주요 데이터 타입

### 2.1 핵심 데이터 타입

#### 텍스트 타입
- **text**: 전문 검색을 위해 분석되는 텍스트입니다.
- **keyword**: 정확한 값 검색, 정렬, 집계에 사용되는 분석되지 않은 문자열입니다.

#### 숫자 타입
- **long**: 64비트 정수
- **integer**: 32비트 정수
- **short**: 16비트 정수
- **byte**: 8비트 정수
- **double**: 64비트 부동 소수점
- **float**: 32비트 부동 소수점
- **half_float**: 16비트 부동 소수점
- **scaled_float**: 고정 배율 인수를 가진 부동 소수점

#### 날짜 타입
- **date**: 날짜와 시간 정보

#### 불리언 타입
- **boolean**: true/false 값

#### 바이너리 타입
- **binary**: Base64 인코딩된 바이너리 데이터

### 2.2 복합 데이터 타입

#### 객체 및 중첩 타입
- **object**: JSON 객체
- **nested**: 독립적으로 쿼리할 수 있는 객체 배열

#### 지리 데이터 타입
- **geo_point**: 위도/경도 좌표
- **geo_shape**: 지리적 모양 (다각형 등)

#### 특수 데이터 타입
- **ip**: IPv4 및 IPv6 주소
- **completion**: 자동 완성 제안을 위한 특수 타입
- **token_count**: 토큰 수를 계산하는 타입

## 3. 매핑 생성 및 관리

### 3.1 인덱스 생성 시 매핑 정의

```
PUT /my_index
{
  "mappings": {
    "properties": {
      "title": {
        "type": "text",
        "analyzer": "standard"
      },
      "description": {
        "type": "text",
        "analyzer": "english"
      },
      "created_at": {
        "type": "date",
        "format": "strict_date_optional_time||epoch_millis"
      },
      "tags": {
        "type": "keyword"
      },
      "user_id": {
        "type": "integer"
      },
      "location": {
        "type": "geo_point"
      },
      "settings": {
        "type": "object",
        "properties": {
          "is_public": {
            "type": "boolean"
          },
          "notification_level": {
            "type": "keyword"
          }
        }
      }
    }
  }
}
```

### 3.2 기존 매핑 조회

```
GET /my_index/_mapping
```

### 3.3 매핑 업데이트

Elasticsearch에서는 기존 필드의 매핑을 변경할 수 없습니다. 하지만 새로운 필드를 추가할 수는 있습니다:

```
PUT /my_index/_mapping
{
  "properties": {
    "new_field": {
      "type": "keyword"
    }
  }
}
```

기존 필드의 매핑을 변경해야 하는 경우:
1. 새 인덱스를 올바른 매핑으로 생성
2. 데이터를 새 인덱스로 리인덱싱
3. 인덱스 별칭을 사용하여 애플리케이션에서 원활하게 전환

### 3.4 동적 매핑 설정

인덱스 레벨에서 동적 매핑 동작을 제어할 수 있습니다:

```
PUT /my_index
{
  "mappings": {
    "dynamic": "strict",
    "properties": {
      "title": { "type": "text" },
      "content": { "type": "text" }
    }
  }
}
```

동적 매핑 옵션:
- **true**: 새 필드를 자동으로 추가 (기본값)
- **false**: 새 필드를 무시하지만, 문서는 저장
- **strict**: 매핑되지 않은 필드가 있으면 문서 거부

## 4. 필드 매핑 파라미터

### 4.1 공통 매핑 파라미터

#### 인덱스 관련 파라미터
- **index**: 필드를 검색 가능하게 할지 여부 (default: true)
- **analyzer**: 인덱싱 시 사용할 분석기
- **search_analyzer**: 검색 시 사용할 분석기
- **normalizer**: keyword 필드에 적용할 정규화 규칙

#### 문서 값 관련 파라미터
- **doc_values**: 정렬, 집계를 위한 열 기반 저장소 사용 여부 (default: true)
- **store**: 필드 값을 별도로 저장할지 여부 (default: false)
- **fielddata**: 메모리 내 필드 데이터 캐시 활성화 (text 필드용)

#### 기타 파라미터
- **null_value**: null 값을 처리하는 방법
- **copy_to**: 필드 값을 다른 필드로 복사
- **ignore_above**: 지정된 길이보다 긴 문자열 무시

### 4.2 텍스트 필드 관련 파라미터

```
PUT /my_index
{
  "mappings": {
    "properties": {
      "title": {
        "type": "text",
        "analyzer": "english",
        "search_analyzer": "standard",
        "index_options": "positions",
        "similarity": "BM25",
        "fields": {
          "raw": {
            "type": "keyword"
          }
        }
      }
    }
  }
}
```

- **index_options**: 인덱싱할 정보 수준 (docs, freqs, positions, offsets)
- **similarity**: 유사도 알고리즘 (default: BM25)
- **fields**: 멀티 필드 정의 (동일한 값을 다른 방식으로 인덱싱)

### 4.3 숫자 및 날짜 필드 관련 파라미터

```
PUT /my_index
{
  "mappings": {
    "properties": {
      "price": {
        "type": "double",
        "coerce": true,
        "ignore_malformed": false
      },
      "created_at": {
        "type": "date",
        "format": "yyyy-MM-dd HH:mm:ss||yyyy-MM-dd||epoch_millis"
      }
    }
  }
}
```

- **coerce**: 문자열을 적절한 타입으로 변환 (default: true)
- **ignore_malformed**: 형식이 잘못된 값 무시 (default: false)
- **format**: 날짜 형식 지정

## 5. 고급 매핑 기능

### 5.1 멀티 필드 (Multi Fields)

동일한 값을 여러 방식으로 인덱싱하여 다양한 검색 및 집계 기능을 지원합니다:

```
PUT /my_index
{
  "mappings": {
    "properties": {
      "name": {
        "type": "text",
        "analyzer": "standard",
        "fields": {
          "keyword": {
            "type": "keyword"
          },
          "english": {
            "type": "text",
            "analyzer": "english"
          }
        }
      }
    }
  }
}
```

이 설정으로:
- `name`: 표준 분석기로 전문 검색
- `name.keyword`: 정확한 값 검색 및 집계
- `name.english`: 영어 분석기로 전문 검색

### 5.2 필드 별칭 (Field Aliases)

필드 별칭을 사용하여 기존 필드에 대한 대체 이름을 생성할 수 있습니다:

```
PUT /my_index
{
  "mappings": {
    "properties": {
      "user_identifier": {
        "type": "keyword"
      },
      "user_id": {
        "type": "alias",
        "path": "user_identifier"
      }
    }
  }
}
```

### 5.3 동적 템플릿 (Dynamic Templates)

동적으로 생성되는 필드에 대한 사용자 정의 매핑 규칙을 정의할 수 있습니다:

```
PUT /my_index
{
  "mappings": {
    "dynamic_templates": [
      {
        "strings_as_keywords": {
          "match_mapping_type": "string",
          "mapping": {
            "type": "keyword"
          }
        }
      },
      {
        "integers_as_longs": {
          "match_mapping_type": "long",
          "mapping": {
            "type": "integer"
          }
        }
      },
      {
        "location_fields": {
          "match": "*_location",
          "mapping": {
            "type": "geo_point"
          }
        }
      }
    ],
    "properties": {
      "title": {
        "type": "text"
      }
    }
  }
}
```

이 설정으로:
- 문자열 필드는 모두 `keyword` 타입으로 매핑
- 정수 필드는 `integer` 타입으로 매핑
- `*_location` 패턴의 필드는 `geo_point` 타입으로 매핑

## 6. 매핑 관련 권장 사항

### 6.1 성능 최적화 팁

- **필요한 필드만 인덱싱**: 검색하지 않는 필드는 `index: false` 설정
- **doc_values 최적화**: 집계나 정렬이 필요 없는 필드는 `doc_values: false` 설정
- **적절한 데이터 타입 선택**: 정수 범위에 맞는 가장 작은 타입 사용 (long 대신 integer 등)
- **text 필드의 fielddata 제한**: 메모리 사용량 증가를 방지하기 위해 필요한 경우에만 활성화

### 6.2 매핑 설계 모범 사례

- **초기 매핑 계획**: 데이터 구조를 분석하고 명시적 매핑 설계
- **필드 이름 규칙 수립**: 일관된 이름 규칙으로 동적 템플릿 적용 용이
- **nested vs object 선택**: 배열 내 객체의 독립적 쿼리가 필요하면 nested 타입 사용
- **멀티 필드 활용**: 동일한 데이터에 대해 다양한 검색 및 집계 요구사항 충족
- **인덱스 템플릿 사용**: 여러 인덱스에 동일한 매핑 적용

### 6.3 매핑 마이그레이션 전략

기존 매핑을 변경해야 할 경우:

1. **인덱스 별칭 활용**: 애플리케이션은 항상 별칭을 통해 인덱스 접근
2. **새 인덱스 생성**: 변경된 매핑으로 새 인덱스 생성
3. **리인덱싱**: 데이터를 새 인덱스로 복사
4. **별칭 전환**: 별칭을 새 인덱스로 변경하여 무중단 전환

```
# 별칭 생성
POST /_aliases
{
  "actions": [
    { "add": { "index": "my_index_v1", "alias": "my_index" } }
  ]
}

# 새 인덱스로 리인덱싱
POST /_reindex
{
  "source": {
    "index": "my_index_v1"
  },
  "dest": {
    "index": "my_index_v2"
  }
}

# 별칭 전환
POST /_aliases
{
  "actions": [
    { "remove": { "index": "my_index_v1", "alias": "my_index" } },
    { "add": { "index": "my_index_v2", "alias": "my_index" } }
  ]
}
```

## 7. 매핑과 스키마 진화 (Schema Evolution)

### 7.1 점진적 매핑 업데이트

새로운 필드를 추가하며 매핑을 점진적으로 발전시키는 방법:

```
PUT /my_index/_mapping
{
  "properties": {
    "new_field": {
      "type": "keyword"
    }
  }
}
```

### 7.2 스키마 변경 사례 연구

복잡한 매핑 변경 예:
1. `name` 필드 타입을 keyword에서 text로 변경
2. `user_preferences` 객체를 nested 타입으로 변경
3. 새로운 위치 관련 필드 추가

이 경우 리인덱싱 시 변환 스크립트를 사용할 수 있습니다:

```
POST /_reindex
{
  "source": {
    "index": "my_index_v1"
  },
  "dest": {
    "index": "my_index_v2"
  },
  "script": {
    "source": """
      ctx._source.location = [
        ctx._source.longitude,
        ctx._source.latitude
      ];
      ctx._source.remove('longitude');
      ctx._source.remove('latitude');
    """
  }
}
```

## 결론

Elasticsearch 매핑은 데이터의 구조와 인덱싱 방법을 정의하는 핵심 요소입니다. 적절한 매핑 설계는 검색 성능과 기능성에 직접적인 영향을 미치므로, 데이터 특성과 애플리케이션 요구사항을 고려하여 신중하게 설계해야 합니다.

동적 매핑의 편의성과 명시적 매핑의 제어력 사이의 균형을 맞추고, 데이터가 진화함에 따라 매핑도 함께 발전시키는 전략을 수립하는 것이 중요합니다. 인덱스 별칭과 리인덱싱을 통해 기존 데이터를 유지하면서 매핑을 효과적으로 업데이트할 수 있습니다.