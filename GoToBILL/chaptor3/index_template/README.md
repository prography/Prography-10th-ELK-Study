# Elasticsearch 인덱스 템플릿(Index Templates)

## 인덱스 템플릿이란?
인덱스 템플릿은 Elasticsearch에서 새로운 인덱스를 만들 때 자동으로 적용할 설정, 매핑, 별칭을 미리 정의해 놓는 것입니다. 쉽게 말해 '인덱스 생성 자동화 도구'라고 생각하면 됩니다.

인덱스 템플릿은 특히 비슷한 구조를 가진 여러 인덱스를 관리할 때 유용합니다. 예를 들어 로그 데이터나 메트릭 데이터처럼 시간대별로 인덱스가 계속 생성되는 경우에 매우 효과적입니다.

## 인덱스 템플릿의 주요 개념

### 1. 인덱스 패턴 (Index Patterns)
어떤 이름 패턴을 가진 인덱스에 템플릿을 적용할지 결정합니다. 
예를 들어, `log-*`라는 패턴은 "log-"로 시작하는 모든 인덱스에 이 템플릿을 적용한다는 의미입니다.

### 2. 우선순위 (Priority)
여러 템플릿이 적용될 수 있을 때 어떤 템플릿을 먼저 적용할지를 결정합니다. 
우선순위가 높은 템플릿의 설정이 낮은 템플릿의 설정을 덮어씁니다.

### 3. 설정 (Settings)
샤드 수, 복제본 수 등 인덱스 수준의 설정을 정의합니다.

### 4. 매핑 (Mappings)
인덱스의 필드들의 데이터 타입과 설정을 정의합니다.

### 5. 별칭 (Aliases)
인덱스의 대체 이름으로, 쿼리와 인덱스 관리를 단순화합니다.

## 미리 정의된 인덱스 패턴
Elasticsearch는 다음과 같은 패턴에 우선순위 100으로 설정된 기본 템플릿을 제공합니다:
- logs-*-* (로그 데이터용)
- metrics-*-* (메트릭 데이터용)
- synthetics-*-* (합성 데이터용)
- profiling-* (프로파일링 데이터용)

이런 내장 템플릿들은 로그나 메트릭 데이터를 저장할 때 더 효율적인 기본 설정을 제공합니다.

## 예제 코드 및 설명

### 1. 인덱스 템플릿 생성

```json
PUT /_index_template/logs_template
{
  "index_patterns": ["log-*"],
  "priority": 1,
  "template": {
    "settings": {
      "number_of_shards": 1,
      "number_of_replicas": 1
    },
    "mappings": {
      "properties": {
        "timestamp": {
          "type": "date"
        },
        "message": {
          "type": "text"
        },
        "status": {
          "type": "keyword"
        }
      }
    },
    "aliases": {
      "logs": {}
    }
  }
}
```

- `logs_template`이라는 인덱스 템플릿을 생성합니다.
- `"log-*"` 패턴에 맞는 모든 인덱스(log-로 시작하는 인덱스)에 적용됩니다.
- 우선순위는 1로 설정했습니다.
- 템플릿 내용에는 샤드와 복제본 수, 필드 매핑, 별칭이 정의되어 있습니다.

### 2. 템플릿 조회 및 삭제

```json
GET /_index_template/logs_template
DELETE /_index_template/logs_template
```

### 3. 인덱스 생성 및 데이터 추가

```json
PUT /log-2024-06-18
{
  "mappings": {
    "properties": {
      "extra_field": {
        "type": "text"
      }
    }
  }
}

POST /log-2024-06-18/_doc/1
{
  "extra_field": "This is an extra field for additional information.",
  "message": "System started successfully.",
  "status": "info",
  "timestamp": "2024-06-18T10:15:30Z"
}
```

- `log-2024-06-18` 인덱스를 생성합니다.
- `extra_field`라는 추가 필드를 매핑에 정의합니다.
- 이 인덱스 이름은 템플릿 패턴 `log-*`에 맞기 때문에, 템플릿의 설정과 매핑도 함께 적용됩니다.
- 데이터를 추가합니다.

### 4. 인덱스 설정 및 매핑 조회

```json
GET /log-2024-06-18/_settings
GET /log-2024-06-18/_mapping
```

### 5. 별칭을 통한 검색

```json
GET /logs/_search
{
  "query": {
    "match_all": {}
  }
}
```

- 템플릿에서 정의한 `logs` 별칭을 사용해 모든 로그 데이터를 검색합니다.
- 이 별칭은 `log-*` 패턴에 맞는 모든 인덱스를 가리킵니다.

### 6. 우선순위가 높은 새 템플릿 추가

```json
PUT /_index_template/specific_logs_template
{
  "index_patterns": ["log-2024-*"],
  "priority": 10,
  "template": {
    "settings": {
      "number_of_shards": 2,
      "number_of_replicas": 2
    },
    "mappings": {
      "properties": {
        "user": {
          "type": "keyword"
        }
      }
    },
    "aliases": {
      "specific_logs": {}
    }
  }
}
```

- `specific_logs_template`라는 새 템플릿을 생성합니다.
- 더 구체적인 패턴 `log-2024-*`와 더 높은 우선순위 10을 가집니다.
- 이 템플릿은 `log-2024-*` 패턴에 맞는 인덱스에 대해 원래 템플릿보다 우선 적용됩니다.

### 7. 새 인덱스 생성 및 확인

```json
PUT /log-2024-06-20
{
  "mappings": {
    "properties": {
      "extra_field": {
        "type": "text"
      }
    }
  }
}

GET /log-2024-06-20/_mapping
```

- 새 인덱스를 생성하고 매핑을 확인합니다.
- 이 인덱스는 두 템플릿 패턴 모두에 맞지만, 우선순위가 높은 `specific_logs_template`의 설정(샤드 2개, 복제본 2개)이 적용됩니다.

## 요약
인덱스 템플릿은 반복적인 인덱스 설정 작업을 자동화하고, 일관된 인덱스 구조를 유지하는 데 도움을 주는 도구입니다. 특히 로깅 시스템이나 시계열 데이터와 같이 유사한 구조의 인덱스가 자주 생성되는 환경에서 매우 유용합니다.

우선순위 시스템을 사용하면 일반적인 템플릿과 더 구체적인 템플릿을 함께 사용할 수 있어, 더욱 세밀한 인덱스 관리가 가능합니다.
