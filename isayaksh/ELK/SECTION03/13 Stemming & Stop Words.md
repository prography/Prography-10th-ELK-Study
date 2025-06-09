# Stemming이란?
- 자연어 처리(NLP)와 정보 검색에서 단어를 기본형 또는 어근으로 줄이는 과정이다.
- 하나의 단어에서 파생된 다양한 형태를 하나의 항목으로 분석할 수 있도록 그룹화하는 데 있다.
- 예 ["running", "runner", "ran", "runs"] -> "run"

- Stemming 알고리즘
  - Porter Stemmer: 가장 일반적인 어간 추출 알고리즘 중 하나이며, 일련의 규칙을 적용하여 단어를 어근 형태로 변환한다.
  - Snowball Stemmer: Porter Stemmer보다 더 고급이며 설정 가능한 버전이다.

# Stop Words란?
- 검색 엔진이나 자연어 처리(NLP) 작업에서 텍스트를 처리하기 전에 자주 제거되는 일반적인 단어이다.
- 이 단어들은 보통 매우 흔하고 고유한 의미를 거의 가지지 않기 때문에, 색인 크기를 줄이고 검색 성능을 향상시키기 위해 제외될 수 있다.

## 적용 예시
- BEFORE: "The quick brown fox jumps over the lazy dog."
- AFTER: "quick brown fox jumps over lazy dog"