# 정렬

- `publish_date` 필드를 내림차순으로 정렬한 결과
- 해당 필드가 `date` 값이라면, epoch 값이 `"sort": [{epoch time}]`으로 같이 출력됨
```
GET /articles/_search
{
  "sort": [
    { "publish_date": { "order": "desc" } }
  ],
  "query": {
    "match_all": {}
  }
}
```

- `views` 필드를 오름차순으로 정렬한 결과
```
GET /articles/_search
{
  "sort": [
    { "views": { "order": "asc" } }
  ],
  "query": {
    "match_all": {}
  }
}
```

- `author.keyword` 필드 기준 오름차순, `publish_date` 필드 기준 내림차순으로 정렬한 결과
- 🔥 기본적으로 text 타입을 만들면서 keyword 서브필드도 같이 만들어준다. 그래서 author는 본문 검색용, author.keyword는 정렬·집계용으로 분리해 쓸 수 있다.
- 🔥 정렬은 정확한 값을 기준으로 비교해야 하므로, 분석되지 않은 keyword 필드를 사용해야 한다.
- 🔥 만약 그냥 author(=text)를 기준으로 정렬하면 "George", "Orwell"이 따로 쪼개져 있어서 제대로 정렬이 안 된다.
```
GET /articles/_search
{
  "sort": [
    { "author.keyword": { "order": "asc" } },
    { "publish_date": { "order": "desc" } }
  ],
  "query": {
    "match_all": {}
  }
}
```

- 중첩(nested) 필드인 comments.number_of_comments를 기준으로 내림차순 정렬
- `"comments.number_of_comments"`: 중첩 필드 comments 배열 안의 number_of_comments 값을 정렬 기준으로 사용
- `"mode": "max"`: 한 문서에 comments가 여러 개 있을 수 있으므로, 그 중 가장 큰 값(max) 을 기준으로 정렬
- `"nested": { "path": "comments" }`: 해당 필드가 nested 타입이기 때문에, 정렬을 위해 반드시 nested.path를 명시해야 함(그렇지 않으면 에러 발생)
```
GET /articles/_search
{
  "sort": [
    {
      "comments.number_of_comments": {
        "order": "desc",
        "mode": "max",
        "nested": {
          "path": "comments"
        }
      }
    }
  ],
  "query": {
    "match_all": {}
  }
}
```