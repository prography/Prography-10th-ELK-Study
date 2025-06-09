# Analyzer
- 색인과 쿼리 과정 중에 텍스트를 처리하는 구성 요소이다.
- 텍스트를 토큰(또는 용어)의 스트림으로 분해하며, 이 과정에서 소문자 변환, 불용어 제거 등 다양한 변환 작업을 적용할 수 있다.

## 구성 요소
- Tokenizer: 텍스트를 토큰(예: 단어)으로 분할한다.
- Character Filters (선택): 토큰화 전에 텍스트를 수정한다 (예: HTML 태그 제거 등).
- Token Filters (선택): 토큰화 이후에 토큰을 수정한다 (예: 소문자 변환, 어간 추출 등).

# Common Built-in Analyzers
## standard
- 텍스트를 유니코드 텍스트 분할 규칙에 따라 단어로 분리한다.
- 구성 요소
  - Tokenizer: standard
  - Token Filters: lowercase, stop(지원함)

## simple
- 텍스트를 문자가 아닌 문자(non-letter) 기준으로 분리하고, 토큰을 모두 소문자로 변환한다.
- 구성 요소
  - Tokenizer: lowercase

## whitespace
- 공백(스페이스, 탭 등)을 기준으로 텍스트를 토큰으로 분리한다.
- 구성 요소
  - Tokenizer: whitespace

## stop
- simple 분석기와 유사하지만, 추가로 불용어(stopword)를 필터링한다.
- 구성 요소
  - Tokenizer: whitespace

## keyword
- 텍스트를 토큰화하지 않는다. 입력 전체를 하나의 토큰으로 처리한다.
- 이는 **정확히 일치하는 검색(exact match)**에 자주 사용된다.
- 구성 요소
  - Tokenizer: keyword

## pattern
- 정규 표현식 패턴을 기반으로 텍스트를 분리한다.
- 구성 요소
  - Tokenizer: pattern