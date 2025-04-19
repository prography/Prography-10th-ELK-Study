# Updating mapping is impossible
- ES에서 인덱스에 한번이라도 Mapping이 생성되면, 수정될 수 없다.
- ES의 불변성(Immutability)은 ES가 데이터를 저장하고 인덱싱하는 방식이다.

## Mapping 정보의 수정을 방지하는 이유?
- Lucene Index Structure : Lucene은 필드 구조를 기반으로 segment에 데이터를 쓰므로, 필드 타입이 정해지면 변경이 불가능하다.
- Data Integrity and Consistency: 이미 색인된 문서와 새로운 문서 간의 필드 정의가 다르면 데이터 일관성이 깨질 수 있다.
- Performance Considerations: 필드 구조 변경은 재색인(reindex)을 요구하며 이는 리소스를 많이 소모하고 성능에 영향을 준다. 분석(Analysis) 과정을 다시 거쳐야 한다.
- Field Type Incompatibility: 같은 필드 이름에 서로 다른 타입을 허용하면 검색 및 집계 연산이 충돌하거나 오류를 일으킬 수 있다.
Index Structure Optimization: Elasticsearch는 인덱스 생성 시 성능 최적화를 위해 내부 구조를 구성하므로, 필드 변경은 이 구조를 무너뜨릴 수 있다.

## INDEX 재생성
- 매핑을 바꾸고 싶으면, 기존 인덱스를 수정하는 게 아니라, 새로운 인덱스를 만들어야 한다.
- 원하는 매핑 구조로 새 인덱스를 정의하고, 기존 인덱스 데이터를 Reindex API를 이용해 복사해야 한다.

### 🔥 Reindex시 주의해야할 점
- integer → string(예: text, keyword)으로 리인덱스할 때, 단순히 Reindex API만 쓰면 숫자 그대로 복사돼서 매핑은 바뀌었어도 데이터는 여전히 숫자이다.
- 따라서, script를 사용해서 형변환을 직접 해야지만 실제 데이터 값이 string으로 변환된다.

### 🔥 애플리케이션에서 인덱스가 변경된지 모른다면?
- 애플리케이션에서 특정 인덱스에 직접 접근하고 있다고 가정해보자.
- 만약, 해당 인덱스를 ES에서 변경했어도, 애플리케이션에서는 해당 정보를 알지 못하고 수정되기 전 인덱스에 계속 접근하게 된다.
- 이러한 경우에 `alias`를 사용하면 인덱스가 변경되어도 애플리케이션 단계에서는 변경된 인덱스를 감지하기 위한 프로세스 없이 그대로 사용하면 된다.
- ES에서는 기존 인덱스의 별칭을 삭제하고, 새로운 인덱스에 기존 인덱스가 사용하던 별칭을 붙이면 된다.