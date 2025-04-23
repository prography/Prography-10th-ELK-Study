# Elasticsearch 동시성 제어 (Concurrency Control)

Elasticsearch는 분산 시스템으로, 여러 클라이언트가 동시에 같은 문서에 접근하여 수정할 수 있습니다. 이러한 환경에서 데이터 일관성을 유지하기 위해 동시성 제어 메커니즘이 필요합니다.

## 1. 동시성 문제

분산 시스템에서 동시성 문제는 다음과 같은 상황에서 발생할 수 있습니다:

- **여러 클라이언트가 동시에 같은 문서를 수정할 때**
- **네트워크 지연이나 노드 장애로 인한 일시적인 비일관성**
- **읽기-수정-쓰기 패턴에서의 충돌**

이러한 문제를 해결하기 위해 Elasticsearch는 여러 동시성 제어 메커니즘을 제공합니다.

## 2. 버전 기반 동시성 제어 (Version-based Concurrency Control)

Elasticsearch의 기본 동시성 제어 메커니즘은 **내부 버전 번호**를 사용합니다:

- 모든 문서는 `_version` 필드를 가지고 있습니다.
- 문서가 생성, 수정, 삭제될 때마다 버전 번호가 증가합니다.
- 문서를 수정할 때 현재 버전 번호를 지정하면, 실제 버전과 일치할 경우에만 작업이 수행됩니다.

### 2.1 버전 기반 동시성 제어 예시

```
// 문서 조회
GET /my_index/_doc/1
// 응답에 _version: 5가 포함됨

// 버전 번호를 지정하여 업데이트 요청
PUT /my_index/_doc/1?version=5
{
  "title": "업데이트된 제목",
  "content": "새로운 내용"
}
```

만약 버전이 일치하지 않으면 Elasticsearch는 `VersionConflictEngineException`을 반환합니다.

## 3. 외부 버전 관리 (External Versioning)

외부 시스템에서 자체 버전 관리를 하는 경우, Elasticsearch와 통합할 수 있습니다:

```
PUT /my_index/_doc/1?version=10&version_type=external
{
  "title": "외부 버전으로 관리되는 문서",
  "content": "내용..."
}
```

외부 버전 관리에서는:
- 제공된 버전이 저장된 버전보다 **크면** 업데이트가 성공합니다.
- 제공된 버전이 저장된 버전보다 **작거나 같으면** 충돌이 발생합니다.

## 4. 낙관적 동시성 제어 (Optimistic Concurrency Control)

Elasticsearch의 동시성 제어는 **낙관적 접근 방식**을 채택합니다:
- 충돌이 거의 발생하지 않을 것이라고 가정합니다.
- 변경 사항을 적용하기 전에 문서 버전을 확인합니다.
- 충돌이 발생한 경우에만 재시도합니다.

### 4.1 낙관적 동시성 제어 구현 예시

클라이언트 측 구현의 일반적인 패턴:

1. 문서 검색 및 버전 저장
2. 문서 수정
3. 버전 번호를 포함하여 업데이트 요청
4. 충돌 시 1단계부터 재시도

```
// 예시 코드 (의사 코드)
function updateDocument(id, updateFn, maxRetries = 3) {
  let retries = 0;
  
  while (retries < maxRetries) {
    try {
      // 1. 문서 검색
      const response = await client.get({
        index: 'my_index',
        id: id
      });
      
      const document = response._source;
      const version = response._version;
      
      // 2. 문서 수정
      const updatedDocument = updateFn(document);
      
      // 3. 버전 정보와 함께 업데이트 요청
      await client.index({
        index: 'my_index',
        id: id,
        body: updatedDocument,
        version: version
      });
      
      return true; // 성공
    } catch (error) {
      if (error.name === 'VersionConflictEngineException') {
        retries++;
        continue; // 재시도
      }
      throw error; // 다른 에러는 재시도하지 않음
    }
  }
  throw new Error('Max retries reached');
}
```

## 5. 동시성 제어를 위한 추가 기법

### 5.1 Sequence Numbers & Primary Terms

최신 버전의 Elasticsearch는 버전 외에도 추가 메커니즘을 사용합니다:

- **Sequence Number (시퀀스 번호)**: 작업(operation)마다 증가하는 번호
- **Primary Term (주 텀)**: 주 샤드가 재할당될 때마다 증가하는 번호

이 두 값을 함께 사용하면 더 정확한 동시성 제어가 가능합니다:

```
PUT /my_index/_doc/1?if_seq_no=123&if_primary_term=2
{
  "title": "시퀀스 번호와 주 텀을 사용한 업데이트",
  "content": "내용..."
}
```

### 5.2 Write-then-read vs Read-then-write

두 가지 일반적인 패턴과 그에 따른 동시성 제어 전략:

- **Write-then-read (쓰기 후 읽기)**: 변경 사항을 즉시 적용한 후 결과를 읽습니다.
  - 버전 충돌에 대한 걱정이 적습니다.
  - 데이터 일관성이 더 좋습니다.

- **Read-then-write (읽기 후 쓰기)**: 현재 상태를 읽고, 수정한 후 쓰기를 합니다.
  - 버전 기반 동시성 제어가 필수적입니다.
  - 충돌 가능성이 높지만 유연성이 좋습니다.

## 6. 동시성 제어와 성능 간의 트레이드오프

동시성 제어와 성능 사이에는 항상 트레이드오프가 있습니다:

- **엄격한 일관성**: 충돌 방지에 좋지만 성능이 떨어질 수 있습니다.
- **느슨한 일관성**: 성능은 좋지만 충돌 위험이 높아집니다.

애플리케이션의 요구사항에 따라 적절한 전략을 선택해야 합니다:

- **중요한 금융 데이터**: 엄격한 일관성을 우선시
- **통계나 로그 데이터**: 성능을 우선시할 수 있음

## 7. 권장 사항

### 7.1 일반적인 권장 사항

- 가능하면 **Bulk API**를 사용하여 배치 처리
- 충돌 발생 시 **자동 재시도 메커니즘** 구현
- 특정 도메인에 맞는 **라우팅 전략** 사용
- 높은 동시성이 예상되는 경우 **샤드 수 조정**

### 7.2. 동시성 문제 완화를 위한 설계 패턴

- **이벤트 소싱 (Event Sourcing)**: 변경 사항 자체보다 이벤트를 저장
- **CQRS (Command Query Responsibility Segregation)**: 읽기와 쓰기 모델 분리
- **비관적 락킹 (Pessimistic Locking)**: 외부 락 메커니즘 활용 (Redis 등)

## 결론

Elasticsearch의 동시성 제어 메커니즘은 분산 환경에서 데이터 일관성을 유지하면서도 성능을 최적화하기 위해 설계되었습니다. 버전 기반 동시성 제어, 외부 버전 관리, 그리고 시퀀스 번호와 주 텀의 활용을 통해 다양한 상황에 맞는 전략을 선택할 수 있습니다.

애플리케이션의 특성과 요구사항에 맞는 동시성 제어 전략을 선택하고, 충돌 관리 메커니즘을 구현함으로써 안정적이고 효율적인 Elasticsearch 시스템을 구축할 수 있습니다.