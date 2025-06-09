# Mapping and Data Type
- Elasticsearch에서 문서(Document)의 필드와 데이터 타입을 정의하는 스키마
- 어떤 필드가 어떤 데이터 타입인지, 어떤 분석기(analyzer)를 쓸지 등등을 정해주는 설정

## Key Components
- **Index** : 유사한 속성을 가진 문서(Document)들의 집합
- **Document** : 인덱스에 저장된 JSON 객체
- **Field** : 문서(Document)에 존재하는 `KEY: VALUE` 쌍
- **Data Type** : 필드에 어떤 종류의 데이터가 들어가는지 정의

## Data Types
| 타입 | 설명| 
|-|-| 
| text | 전체 텍스트, 분석기 적용됨 (검색용)| 
| keyword | 분석 없이 정확히 일치하는 문자열 (필터, 집계용)| 
| integer, long, float, double | 숫자 타입| 
| boolean | true / false| 
| date | 날짜/시간| 
| geo_point | 위도/경도 좌표| 
| object, nested | JSON 구조 표현용| 
| OOO_range | 범위 기반 검색 |

## Special Data Type
타입 | 설명
| 타입         | 설명                                                                 |
|--------------|----------------------------------------------------------------------|
| geo_point    | 위도/경도 좌표를 저장하고, 거리 계산 또는 위치 기반 검색에 사용        |
| ip           | IPv4 또는 IPv6 주소를 저장하고 필터링/범위 검색에 사용                 |
| completion   | 자동완성(suggest) 기능을 위한 특수한 색인 방식                         |
| token_count  | 필드 내 단어(token)의 개수를 저장 (분석기 기반)                       |
| percolator   | 문서가 아닌 "쿼리"를 색인해서, 나중에 문서가 쿼리에 매칭되는지 테스트할 수 있음 |


### 🔥 Object vs Nested 차이
- Object는 Document를 저장할 때 모든 필드 값을 한번에 저장하게 된다. 따라서, 세밀한 조회가 불가능해진다.