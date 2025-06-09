# Synonym Search
- Elasticsearch에서의 동의어 검색은 **검색 중에 동일하게 취급되어야 하는 단어 또는 구문을 정의**할 수 있게 해준다  
  - 예: laptop, notebook은 동일한 단어로 취급되어야 한다

- 동의어 검색을 생성하는 방법
  - **동의어 파일 또는 인라인 동의어 정의**: 동의어로 취급할 단어 목록을 생성한다
  - **커스텀 분석기 생성**: 동의어 필터가 적용된 커스텀 분석기를 정의하여 원하는 필드에 적용한다
  - **커스텀 분석기를 필드에 적용**: 동의어 검색을 사용할 필드의 매핑을 업데이트한다

### 🔥 실제 예시
(1) 동의어 필터정의
- `synonym_filter`라는 이름의 동의어 필터 정의
- `"laptop"`과 `"notebook"`은 동일한 단어로 취급, `"smartphone"`과 `"cell phone"`도 동일하게 취급
```
"filter": {
    "synonym_filter": {
        "type": "synonym",
        "synonyms": [
            "laptop, notebook",
            "smartphone, cell phone"
        ]
    }
},
```

(2) analyzer 정의
- `synonym_analyzer`라는 이름의 분석기를 정의
- 위에서 만든 동의어 필터(`synonym_filter`)를 적용함
```
"analyzer": {
  "synonym_analyzer": {
    "tokenizer": "standard",
    "filter": [
      "lowercase",
      "synonym_filter"
    ]
  }
}
```

### 🔥 동의어 필터를 재정의하고 싶은 경우!
(1) `products` 인덱스를 close 한다.
```
POST /products/_close
```

(2) `products` 인덱스의 `synonym_filter` 필터의 값을 수정해준다.
```
PUT /products/_settings
{
  "analysis": {
    "filter": {
      "synonym_filter": {
        "type": "synonym",
        "synonyms": [
          "laptop, notebook",
          "smartphone, cell phone",
          "tablet, pad"
        ]
      }
    }
  }
}
```

(3) `products` 인덱스를 open한다.
```
POST /products/_open
```

---

### 🔥 파일을 통해 동의어 필터 생성
(1) 동의어 파일을 생성하기 위해서는 현재 실행중인 도커 vm에 접속한다.
```
docker ps // 실행중인 도커 ID를 확인한다.

docer exec -it {dockerid} /bin/bash // 도커 vm에 접속한다.

cd config/anaysis/ // 다음 경로로 이동한다.
```

(2) 해당 디렉토리에서 동의어로 처리할 문자들을 `synonyms.txt` 파일에 작성한 후 생성한다.

(3) ES에서 해당 텍스트 파일을 통해 동의어 필터를 생성한다.
- 동의어 필터를 재생성하기 위해 인덱스를 close, open 하는 절차는 동일하다.
- 앞서 생성한 `synonyms.txt` 파일을 동의어 필터에 데이터로 사용한다.
```
PUT /products/_settings
{
  "analysis": {
    "filter": {
      "synonym_filter": {
        "type": "synonym",
        "synonyms_path": "analysis/synonyms.txt" // 동의어가 정리된 텍스트 파일
      }
    }
  }
}
```