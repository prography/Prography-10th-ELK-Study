# Inner hits
- nested 문서나 nested 쿼리 내의 **개별 일치 항목에 대한 세부 정보를 반환**할 수 있도록 해주는 기능이다.
- nested 쿼리를 사용할 때, 어떤 부모 문서가 쿼리에 일치했는지뿐만 아니라, 그 부모 **문서를 일치하게 만든 구체적인 nested 문서(또는 하위 문서)**가 무엇인지도 알고 싶을 수 있다.

### 🔥 Inner hits 실제 사용 예시
1. 이커머스(쇼핑몰) 검색 시스템
- 기업 예시: 아마존, 쿠팡, 11번가 등
- 상품 하나(product)가 여러 버전(색상, 사이즈 등)으로 구성되어 있을 때, 각각은 nested 문서로 저장함.
- 사용자가 "검정색 신발, 사이즈 270"을 검색하면, 그 상품(부모 문서)만 결과로 보여주면 안 되고, 그 상품 안에서 정확히 "검정색 270" 버전이 히트됐다는 걸 보여줘야 함.

👉 그래서 inner_hits를 써서 어떤 버전(옵션)이 매칭됐는지를 뽑아낸다.

2. 뉴스/콘텐츠 서비스
- 기업 예시: 블룸버그, BBC, 네이버 뉴스
- 하나의 뉴스 기사에 여러 관련 태그(nested), 카테고리(nested)가 붙어 있을 때,
- 사용자가 "AI"에 관련된 뉴스를 찾으면, 그냥 기사를 주는 게 아니라, "이 기사가 AI 태그 때문임" 이라고 명확히 알려줘야 함.

👉 그래서 inner_hits를 통해 매칭된 태그나 카테고리를 구체적으로 같이 리턴한다.

3. 채용 포털/이력서 검색
- 기업 예시: LinkedIn, 잡코리아, 사람인
- 이력서 하나에 여러 직무 경험(nested)이 기록되어 있을 때, "Java 개발자 경력" 검색을 하면, 이력서 전체를 주는 게 아니라, 그 사람의 어떤 경력 항목(Java 경력)이 매칭됐는지 정확히 보여줘야 함.

👉 inner_hits로 어떤 경력 세부 항목이 검색 조건에 맞았는지 뽑는다.

4. CRM 고객 데이터 분석
- 기업 예시: Salesforce, HubSpot
- 고객 하나에 여러 활동 기록(nested: 구매, 문의, 클릭 등)이 들어있을 때,
- "지난 1개월 안에 3번 이상 구매한 고객"을 찾을 때,
- 그냥 고객 ID만 주는 게 아니라, 그 고객이 어떤 구매 활동 기록으로 조건을 만족했는지 같이 리턴해야 함.

👉 inner_hits를 써서 구매 내역 세부 항목을 같이 보여준다.

