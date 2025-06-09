# Bucket Aggregation
- 문서들을 특정 기준에 따라 **그룹핑(버킷 생성)**하고, 그 안에서 추가적인 분석(예: Metric Aggregation)을 수행할 수 있게 해주는 집계 방식
- 각 버킷은 공통된 특성(예: 필드 값, 값의 범위, 또는 더 복잡한 조건)을 공유하는 문서들의 집합을 나타낸다.

## Types of Bucket Aggregations
- Terms Aggregation
  - 지정된 **필드의 고유한 값들을 기준**으로 문서를 그룹화함
  - [EX] SQL상의 GROUP BY와 유사함
- Range Aggregation
  - 미리 정의된 숫자 범위에 따라 문서를 여러 버킷으로 그룹화함
  - [EX] 0 ~ 10, 11 ~ 20, 21 ~ 30, ... 
- Histogram Aggregation
  - Range Aggregation과 비슷하지만, 지정된 간격을 기준으로 균등하게 나뉜 버킷을 자동으로 생성함.
- Date Histogram Aggregation
  - 일(day), 월(month), 연도(year)와 같은 날짜 간격에 따라 문서를 그룹화함
- Filter / Filters Aggregation
  - filter Aggregation은 특정 조건에 맞는 문서를 하나의 버킷으로 만듦.
  - filters Aggregation은 여러 필터를 정의해 각 조건마다 개별 버킷을 생성함. 즉, **여러 개의 filter를 병렬적으로 실행하고 각 필터마다 하나의 버킷을 생성하는 구조**
- Missing Aggregation
  - 지정된 필드가 존재하지 않는 문서들만을 모아 하나의 버킷을 생성함. 즉, **"값이 비어 있는 필드"**를 기준으로 문서를 찾고 분석할 때 사용
- Nested Aggregation
  - `nested object` 내부의 필드에 대해 집계를 수행함. 즉, **`nested object`를 대상으로 집계를 수행**할 때 사용
- Global Aggregation
  - 쿼리의 필터 조건을 무시하고, 인덱스 내 **전체 문서를 포함**하는 하나의 전역 버킷을 생성함
  - 일반적인 Aggregation은 query 조건의 영향을 받지만, global은 **"조건 상관없이 전체"**를 집계하는 데 사용

### 🔥 Range Aggregation VS Histogram Aggregation
- `Histogram Aggregation` → 중복 없이 균등 간격으로 나눔 (하나의 문서는 하나의 버킷에만 포함)
- `Range Aggregation` → 사용자 정의 범위이며, 겹치는 구간도 허용됨

