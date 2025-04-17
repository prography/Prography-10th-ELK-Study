# Analysis란?
- ES에 텍스트를 INSERT할 때, **검색에 유용하도록 DATA FORMAT을 바꾸는 프로세스**를 의미한다.
- Analysis의 목표는 텍스트를 인덱싱되고 효과적으로 처리될 수 있는 용어나 토큰으로 분해한다.

## Tokenization
- 문장을 쪼개서 검색할 수 있는 최소 단위로 만드는 것을 의미한다.

## Filtering
Tokenization으로 나눠진 단어들에 대해 변형, 제거, 추가 등의 후처리 작업을 의미한다.

### 필터 종류
| 필터 이름 | 설명 |
|-|-|
|lowercase|	전부 소문자로 변환|
|stop|	불용어(stop words) 제거 (예: is, the, a 등)|
|stemmer|	어간 추출 (예: "running" → "run")|
|asciifolding|	특수문자 제거 (예: "café" → "cafe")|
|synonym|	동의어 매핑|
|edge_ngram|	자동완성용 접두어 생성|
|truncate|	토큰 길이 제한|

### 필터 사용 예
```
{
  "settings": {
    "analysis": {
      "analyzer": {
        "my_analyzer": {
          "tokenizer": "standard",
          "filter": ["lowercase", "stop", "stemmer"]
        }
      }
    }
  }
}
```

# Key Components of Analyzers

## Character Filters
- 텍스트를 "쪼개기 전에", 특정 문자나 문자열을 치환하거나 제거하거나 변형

### Analysis 흐름
```
Character Filters → Tokenizer → Token Filters
```

### Character Filters 종류
|이름	|설명|
|-|-|
|html_strip	|HTML 태그 제거 (`<p>`, `<div>` 등)|
|mapping	|특정 문자나 문자열을 다른 것으로 치환|
|pattern_replace	|정규표현식 기반으로 문자 치환|

### Character Filters 예
```
"char_filter": {
  "replace_dash": {
    "type": "pattern_replace",
    "pattern": "-",
    "replacement": " "
  }
}
```

## Tokenizer
- 문장을 받아서 검색 가능한 단어들로 쪼개주는 핵심 처리기

### Tokenizer 종류
|이름 | 설명 | 예시 결과
|-|-|-|
|standard | 기본 토크나이저 (공백, 특수문자 기준) | "Elasticsearch is awesome" → ["elasticsearch", "is", "awesome"]|
|whitespace | 공백만 기준 | "hello-world" → ["hello-world"]|
|keyword | 전체 문자열을 하나의 토큰으로 | "Elasticsearch rules" → ["Elasticsearch rules"]|
|pattern | 정규표현식으로 나눔 | "2023-01-01" (- 기준) → ["2023", "01", "01"]|
|ngram | 글자 단위로 분할 (자동완성에 사용) | "abc" → ["a", "ab", "abc", "b", "bc", "c"]|
|edge_ngram | 접두어로 분할 (검색 보조용) | "search" → ["s", "se", "sea", "sear", "searc", "search"]|

### Tokenizer 사용 예
```
"analyzer": {
  "my_analyzer": {
    "type": "custom",
    "tokenizer": "whitespace",  // 여기!
    "filter": ["lowercase"]
  }
}
```

## Toekn Filters
- 쪼갠 단어들을 더 정리, 변형, 필터링해서 검색 품질을 향상시키는 역할

### Token Filter 종류
|필터 이름 | 설명|
|-|-|
|lowercase | 모든 토큰을 소문자로 변환|
|stop | 불용어 제거 (예: the, is, and 등)|
|stemmer | 단어의 어간 추출 (예: "running" → "run")|
|asciifolding | 발음기호 제거 (예: "café" → "cafe")|
|synonym | 동의어 치환|
|edge_ngram | 자동완성용 접두어 생성|
|truncate | 토큰 길이 제한|
|nori_part_of_speech | 한국어 품사 필터링 (Nori 전용)|