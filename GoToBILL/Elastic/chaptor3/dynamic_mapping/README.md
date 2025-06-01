# Elasticsearch 다이나믹 매핑(Dynamic Mapping)

## 다이나믹 매핑 개요

Elasticsearch의 다이나믹 매핑에 대한 기본 개념:

* 다이나믹 매핑은 **자동으로 인덱스의 필드 매핑을 생성하고 업데이트**하는 기능입니다.
* 새 문서가 인덱싱될 때 이 기능이 작동합니다.
* 이를 통해 Elasticsearch는 사전에 명시적인 매핑 정의 없이도 실시간으로 새 필드를 처리할 수 있습니다.
* 이 기능은 **데이터 구조를 사전에 알 수 없는 시나리오**에서 특히 유용합니다.

## 기본 다이나믹 매핑 동작

다양한 데이터 타입에 대한 Elasticsearch의 기본 다이나믹 매핑 동작:

* **문자열(Strings)**: 텍스트처럼 보이는 값은 `text` 타입으로 매핑되며, `keyword` 서브필드가 함께 생성됩니다.
* **숫자(Numbers)**: 숫자 값은 감지된 형식에 따라 long, double 등으로 매핑됩니다.
* **부동 소수점(Floating point)**: Float 타입으로 매핑됩니다.
* **날짜(Dates)**: 날짜처럼 보이는 문자열은 date 타입으로 매핑됩니다.
* **불리언(Booleans)**: true 또는 false로 보이는 값은 boolean 타입으로 매핑됩니다.
* **객체(Object)**: object 타입으로 매핑됩니다.

이 문서는 문자열의 경우 기본적으로 `text` 타입으로 매핑되며, 추가로 `keyword` 서브필드가 자동 생성된다는 것을 명확히 보여줍니다. 즉, 문자열의 기본 매핑 타입은 `keyword`가 아니라 `text`입니다.

## 다이나믹 매핑 제어 방법

실제 프로덕션 환경에서는 다이나믹 매핑을 더 세밀하게 제어할 수 있습니다:

```
PUT /controlled_mapping
{
  "mappings": {
    "dynamic": "strict",  // 'true', 'false', 'strict' 중 선택 가능
    "properties": {
      "name": { "type": "text" }
    }
  }
}
```

* `dynamic: true` - 기본값, 새 필드 자동 추가
* `dynamic: false` - 새 필드 저장은 하지만 인덱싱하지 않음
* `dynamic: strict` - 매핑에 없는 필드가 있으면 오류 발생

또한 다이나믹 템플릿을 사용하여 특정 패턴의 필드에 대한 매핑 규칙을 정의할 수도 있습니다.

Elasticsearch의 다이나믹 매핑은 유연성과 개발 속도를 높여주지만, 프로덕션 환경에서는 성능과 검색 품질을 위해 명시적 매핑을 함께 고려하는 것이 좋습니다.
