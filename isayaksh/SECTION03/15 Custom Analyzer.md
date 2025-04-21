# Custom Analyzer
- Elasticsearch에서 사용자 정의 분석기(custom analyzer)를 생성하면, 텍스트가 처리되고 색인되는 방식을 특정 요구에 맞게 조정할 수 있다.

- 사용자 정의 분석기는 **tokenizer(토크나이저)**를 지정하고, 선택적으로 **character filters(문자 필터)**와 **token filters(토큰 필터)**를 하나 이상 정의하여 구성한다.

## 사용자 정의 분석기 생성 단계
- Define the Custom Analyzer : 인덱스 설정에서 구성 요소(tokenizer, filter)를 지정하여 사용자 정의 분석기를 정의한다.
- Apply the Custom Analyzer : 매핑에서 특정 필드에 사용자 정의 분석기를 적용한다.

## 기존 인덱스에 새로운 분석기 추가하기
- Dynamic Settings(동적 설정): Elasticsearch를 재시작하지 않고도 런타임 중에 수정할 수 있다. 주로 자주 변경되거나 상황 변화에 따라 조정이 필요한 설정에 적합하다.
- Static Settings(정적 설정): 반드시 Elasticsearch가 시작되기 전에 설정해야 하며, 변경하려면 재시작 또는 인덱스 재생성이 필요하다. 클러스터나 인덱스의 핵심 동작을 정의하는 설정에 사용된다.
- 분석기 설정(analysis setting)은 **정적 설정(static one)**에 해당한다.

### 🔥 Analyzer를 변경했을 때, 이전의 Analyzer로 저장된 데이터는 어떻게 해야할까?
- Reindex를 사용하고 Alias를 사용한다.
- 혹은 `POST /{index}/_update_by_query?conflicts=proceed` 명령어 실행