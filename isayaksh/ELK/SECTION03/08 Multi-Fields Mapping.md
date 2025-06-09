# Multi-Fields Mapping
- 하나의 필드 값을 여러 방식으로 색인(indexing)할 수 있게 해주는 기능
- Mapping 정보를 TEXT 혹은 KEYWORD로 중복 설정하여 조회와 정확한 매치의 용도로도 사용할 수 있다.
- Elasticsearch에서 기본적으로 text 타입 필드는 분석(analyze) 과정을 거치기 때문에 대소문자 구분 없이 처리되지만, keyword 타입을 함께 매핑해두면 대소문자까지 일치하는 정확 매치가 가능하다.

## Multi-Fields Mapping의 장점
- Versatility : 같은 데이터를 다양한 쿼리에 사용할 수 있다.
- Effinciency : 서로 다른 목적의 쿼리를 위해 데이터를 중복 색인할 필요가 없다. -> 하나의 필드만 정의하고 그 안에 서브 필드를 정의하면 된다.
- Flexibility : 같은 필드에 다양한 분석기를 적용해서 검색 목적에 따라 최적화할 수 있다.
