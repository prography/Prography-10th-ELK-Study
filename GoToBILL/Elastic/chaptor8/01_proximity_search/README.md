# 근접 검색(Proximity Search)

Elasticsearch에서 근접 검색은 특정 단어들이 서로 얼마나 가까이 위치하는지에 따라 문서를 검색하는 기능입니다. 이는 정확한 구문 검색보다 더 유연한 검색 방법을 제공합니다.

## 개요

Elasticsearch의 근접 검색은 다음과 같은 특징을 가집니다:

- **특정 단어들이 서로 가까이 위치한 문서 검색**: 근접 검색을 통해 특정 단어들이 서로 일정 거리 내에 있는 문서를 찾을 수 있습니다.
- **match_phrase 쿼리**: 이 쿼리는 지정된 필드에서 정확한 단어 시퀀스를 포함하는 문서를 찾습니다.
- **slop 파라미터**: slop 값은 구문 내의 용어 사이에 얼마나 많은 단어가 있을 수 있는지를 정의합니다. slop 값이 0이면 정확한 구문 일치가 필요하며, 값이 높을수록 더 많은 단어가 중간에 허용됩니다.

## 예제 데이터 설정

먼저 테스트용 인덱스와 문서를 생성합니다:

```
DELETE /articles

PUT /articles
{
  "mappings": {
    "properties": {
      "title": {
        "type": "text"
      },
      "content": {
        "type": "text"
      }
    }
  }
}

PUT /articles/_doc/1
{
  "title": "The quick brown fox jumps over the lazy dog",
  "content": "A quick brown fox swiftly jumped over the lazy dog."
}

PUT /articles/_doc/2
{
  "title": "The quick brown fox leaps over the sleepy dog",
  "content": "A quick and fast fox jumped quickly over the lazy and sleepy dog."
}

PUT /articles/_doc/3
{
  "title": "A slow green turtle walks under the energetic rabbit",
  "content": "The green turtle slowly walked under the energetic rabbit."
}
```

## 근접 검색 실습

### 1. slop: 0 (정확한 구문 일치)

slop 값이 0인 경우 정확히 일치하는 구문만 검색됩니다:

```
GET /articles/_search
{
  "query": {
    "match_phrase": {
      "content": {
        "query": "quick fox",
        "slop": 0
      }
    }
  }
}
```

**결과:**
이 쿼리는 "quick"과 "fox"가 정확히 연속해서 나타나는 문서만 반환합니다. 예제 데이터에서는 이러한 문서가 없으므로 결과가 반환되지 않습니다.

### 2. slop: 1 (1개 단어 거리 허용)

slop 값이 1인 경우 최대 1개의 단어가 중간에 있을 수 있습니다:

```
GET /articles/_search
{
  "query": {
    "match_phrase": {
      "content": {
        "query": "quick fox",
        "slop": 1
      }
    }
  }
}
```

**결과:**
이 쿼리는 문서 1을 반환합니다. 문서 1의 내용 "A quick brown fox..."에서 "quick"과 "fox" 사이에 "brown"이라는 한 단어가 있습니다.

### 3. slop: 2 (2개 단어 거리 허용)

slop 값이 2인 경우 최대 2개의 단어가 중간에 있을 수 있습니다:

```
GET /articles/_search
{
  "query": {
    "match_phrase": {
      "content": {
        "query": "quick fox",
        "slop": 2
      }
    }
  }
}
```

**결과:**
이 쿼리는 문서 1과 문서 2를 반환합니다. 문서 2의 내용 "A quick and fast fox..."에서 "quick"과 "fox" 사이에 "and fast"라는 두 단어가 있습니다.

## 실무 활용 사례

1. **자연어 검색 개선**: 사용자가 정확한 구문을 기억하지 못하거나 단어 순서가 약간 다른 경우에도 관련 결과를 제공합니다.
2. **컨텐츠 분석**: 특정 키워드가 서로 가까이 언급된 문서를 식별하여 관련성이 높은 컨텐츠를 분석할 수 있습니다.
3. **표절 감지**: 문장 구조가 유사한 텍스트를 찾는 데 도움이 됩니다.
4. **검색 결과 순위 지정**: 검색 용어가 서로 가까이 있을수록 더 높은 관련성 점수를 부여할 수 있습니다.

## 성능 고려사항

- slop 값이 높을수록 더 많은 결과가 반환될 수 있지만, 관련성이 낮은 결과도 포함될 수 있습니다.
- 복잡한 구문 검색은 단순한 term 쿼리보다 더 많은 리소스를 사용할 수 있습니다.
- 대량의 데이터에서 높은 slop 값을 사용할 때는 성능 영향을 고려해야 합니다.
