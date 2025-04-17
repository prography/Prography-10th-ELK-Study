# Handling Data Type
- 데이터가 들어올 때 Elasticsearch는 그 값의 타입을 기준으로, 어떻게 색인할지, 어떤 쿼리를 적용할 수 있을지, 정렬/집계가 가능한지 판단한다.

## Date Detection
- Elasticsearch의 dynamic mapping 기능 중 하나로, 문서 안에 "2024-01-01" 같은 날짜처럼 보이는 문자열이 있으면, 자동으로 `type: date`로 추론하여 매핑해주는 기능
- 기본값은 `true`로 설정되어 있다.

🔥 그래도 자동으로 매핑되도록 하는 것보단, 처음부터 date 타입이라고 명시하고 사용하는 것이 관리 측면에서 더 좋다. 

## Date Formats
- 날짜 필드를 type: date로 매핑할 때, "어떤 형식으로 입력될지"를 포맷(format)으로 정의할 수 있다.
- "2024-01-01"이든 "2024/01/01 10:00:00"이든, 미리 정의된 형식에 맞게 들어와야 정상적으로 색인된다.
- Elasticsearch에서 사용할 수 있는 **Date Format(날짜 형식)**은 Java DateTimeFormatter 기반으로 정의되어 있다.

🔥 
- Elasticsearch 내부에서는 항상 EpochMilli를 사용한다.
- Retrieve 할 때는 Date가 INSERT한 그대로 들어가 있지만, Search 하거나, 거맥 내에서 필터를 할 때는 항상 Epoch Milli를 기준으로 사용한다.


## Mapping Date Fields
- format 파라미터를 필드 매핑 안에서 직접 정의하면 Elasticsearch가 날짜 값을 유연하게 해석하여 정확한 인덱싱과 쿼리 해석에 영향을 준다.

