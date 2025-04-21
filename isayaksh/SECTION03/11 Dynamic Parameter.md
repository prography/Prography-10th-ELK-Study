# Dynamic Parameter

## Dynamic Mapping (동적 매핑)
- Elasticsearch에서 dynamic 매핑 파라미터는 매핑에 명시적으로 정의되지 않은 필드들을 어떻게 처리할지 결정한다.
  - true: (기본값) 새로운 필드가 발견될 때마다 자동으로 매핑에 추가
  - false: 매핑에 명시되지 않은 새로운 필드 무시. 이 필드들은 색인되거나 저장되지 않는다.
  - strict: 매핑에 정의되지 않은 새로운 필드를 포함한 문서는 거부되며, 오류를 반환한다.

