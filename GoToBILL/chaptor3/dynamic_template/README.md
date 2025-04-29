# Elasticsearch 다이나믹 템플릿(Dynamic Template)

## 다이나믹 템플릿이란?

다이나믹 템플릿은 Elasticsearch에서 동적으로 추가되는 필드에 대한 사용자 정의 규칙을 설정할 수 있는 기능입니다. 이 기능을 통해 필드의 이름, 타입, 또는 다른 속성에 기반한 더 세밀한 규칙을 정의할 수 있습니다.

## 다이나믹 템플릿을 사용하는 이유

### 1. 유연성(Flexibility)
다이나믹 템플릿을 사용하면 동적으로 추가되는 필드가 어떻게 인덱싱되고 분석될지를 사용자가 직접 정의할 수 있습니다.

### 2. 제어(Control)
필드 이름이나 데이터 타입과 같은 패턴이나 조건에 기반하여 특정 매핑을 적용할 수 있습니다.

### 3. 효율성(Efficiency)
모든 필드를 사전에 정의할 필요 없이 특정 설정(예: 애널라이저, 인덱싱 옵션)을 자동으로 적용할 수 있습니다.

## 다이나믹 템플릿 설정 방법

다이나믹 템플릿은 인덱스 생성 시 매핑 정의의 일부로 설정할 수 있습니다. 다음은 기본 형식입니다:

```
PUT /my_index
{
  "mappings": {
    "dynamic_templates": [
      {
        "template_name": {
          "match": "pattern",
          "match_mapping_type": "type",
          "mapping": {
            "type": "desired_type",
            "other_options": "..."
          }
        }
    ]
  }
}
```

### 주요 설정 항목

- **template_name**: 템플릿의 이름
- **match**: 필드 이름이 일치해야 하는 패턴
- **match_mapping_type**: 적용할 필드의 데이터 타입
- **mapping**: 패턴이 일치할 때 적용할 실제 매핑 설정

## 활용 예시

다이나믹 템플릿은 다음과 같은 상황에서 유용합니다:

1. 모든 문자열 필드를 기본적으로 `keyword` 타입으로 설정하고 싶을 때
2. 특정 이름 패턴(예: "log_"로 시작하는 필드)을 가진 필드에 특정 애널라이저를 적용하고 싶을 때
3. 숫자 필드를 항상 특정 포맷으로 인덱싱하고 싶을 때

다이나믹 템플릿을 통해 이러한 규칙을 사전에 정의함으로써, 데이터 구조가 변경되더라도 Elasticsearch가 일관된 방식으로 새 필드를 처리할 수 있게 됩니다.

```
PUT /my_index
{
  "mappings": {
    "dynamic_templates": [
      {
        "integers_to_longs": {
          "match_mapping_type": "long",
          "mapping": {
            "type": "long",
            "fields": {
              "raw": {
                "type": "keyword"
              }
            }
          }
        }
      },
      {
        "strings_as_keywords": {
          "match_mapping_type": "string",
          "match": "user_*",
          "unmatch": "*_text",
          "mapping": {
            "type": "keyword"
          }
        }
      },
      {
        "ip_addresses": {
          "match_pattern": "regex",
          "match": "^ip_.*",
          "mapping": {
            "type": "ip"
          }
        }
      },
      {
        "nested_objects": {
          "path_match": "user.address.*",
          "mapping": {
            "type": "nested"
          }
        }
      }
    ]
  }
}
````

1. `integers_to_longs`: 모든 정수 필드가 long 타입으로 매핑되고, 추가로 키워드 서브필드를 가짐 
2. `strings_as_keywords`: `"user_"`로 시작하는 모든 문자열 필드가 keyword 타입으로 매핑됨 (단, "_text"로 끝나는 필드는 제외)
3. `ip_addresses`: `"ip_"`로 시작하는 필드는 IP 주소로 취급되어 ip 타입으로 매핑됨 
4. `nested_objects`: user.address 경로 아래의 모든 객체는 nested 타입으로 매핑됨

## 다이나믹 템플릿 vs 인덱스 템플릿

| 특성 | 다이나믹 템플릿 | 인덱스 템플릿 |
|------|----------------|--------------|
| 정의 | 인덱스 내부의 필드 동적 매핑 규칙 | 인덱스 생성 시 적용되는 인덱스 수준 템플릿 |
| 작동 시점 | 문서 인덱싱 시점에 작동 | 인덱스 생성 시점에 작동 |
| 적용 기준 | 필드 이름/패턴에 따라 매핑 정의 | 인덱스 이름 패턴에 따라 전체 설정 정의 |
| 정의 위치 | 인덱스 매핑 내부에 정의 | 클러스터 수준에서 정의 |
