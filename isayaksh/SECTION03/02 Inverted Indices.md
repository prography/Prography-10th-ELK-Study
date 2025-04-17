# Inverted Indices
- 단어(토큰)를 키로 삼고, 그 단어가 등장한 문서의 위치나 ID를 값으로 가지는 구조
- 빠르고 효율적인 FULL-TEXT 조회를 용이하게 하는 근본적인 데이터 구조

## Key Concept
- **Terms** : 각 단어와 토큰들이 모두 인덱싱된다.
- **Posting List** : 문서에서 각 단어(term)가 등장한 ID, 위치, 빈도 등의 정보를 모아놓은 리스트

## Inverted Indices가 동작하는 방법

- **Document Parsing** : Analysis(Character Filters → Tokenizer → Token Filters) 처리 후 진행한다.
- **Term Extraction** : Document에서 Terms(단어 or 토큰)을 추출한다. 각 Terms는 Documents 식별자와 연관을 갖는다.
- **Index Creation** : 추출된 term을 기준으로 Inverted Index와 Posting List 생성한다.

### 예시 문서

|문서 ID|	내용|
|-|-|
|1	|"Elasticsearch is fast"|
|2	|"Elasticsearch is scalable"|
|3	|"Search is important"|

### Inverted Index

|Term	|Posting List|
|-|-|
|elasticsearch|	[1, 2]|
|search|	[3]|
|is|	[1, 2, 3]|

## Inverted Indices의 장점
- **Efficiency** : Terms 그리고 Terms와 관련있는 Documents를 빠르게 찾을 수 있게 한다.
- **Scalability** : 큰 볼륨의 텍스트 데이터를 효율적으로 다룰 수 있다.
- **Relevance** : Boolean 쿼리, 구(문장) 쿼리, 근접 검색과 같은 고급 쿼리 기능을 지원한다.

