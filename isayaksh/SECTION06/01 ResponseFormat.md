# ResponseFormat
- Elasticsearch는 주로 JSON을 API 요청 및 응답 형식으로 사용하지만, 요청 및 응답에 `YAML`도 지원한다.
  - EX: GET /library/_search?format=yaml
- Elasticsearch는 `pretty` 파라미터를 사용하여 JSON 출력 결과를 사람이 읽기 쉬운 형식으로 제공하는 옵션도 제공한다.
- DevTools를 사용할 때는 기본적으로 `pretty`가 지원되지만, CURL을 사용할 때는 데이터가 압축되어서 제공되는 것이 디폴트이다.
- 따라서, `pretty` 형식으로 응답을 받고 싶다면 파라미터를 추가해야한다.
  - EX: GET /library/_search?pretty

