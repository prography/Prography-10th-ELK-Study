## Elasticsearch 응답 형식(Response Format)

Elasticsearch는 API 요청과 응답에 다양한 형식을 지원합니다. 기본적으로 JSON 형식을 사용하지만, 가독성 향상과 다양한 통합을 위한 옵션들을 제공합니다.

### 기본 응답 형식

- Elasticsearch는 기본적으로 **JSON**(JavaScript Object Notation)을 API 요청과 응답의 기본 형식으로 사용합니다.
- JSON은 구조화된 데이터를 표현하는 표준 형식으로, 대부분의 프로그래밍 언어와 도구에서 쉽게 파싱할 수 있습니다.

### 지원되는 응답 형식 옵션

#### 1. YAML 형식

- Elasticsearch는 JSON 외에도 **YAML**(YAML Ain't Markup Language) 형식을 지원합니다.
- YAML은 JSON보다 더 읽기 쉬운 형식으로, 특히 복잡한 구조의 데이터를 표현할 때 유용합니다.
- `format=yaml` 파라미터를 사용하여 YAML 형식으로 응답을 받을 수 있습니다.

```
GET /library/_search?format=yaml
{
  "query": {
    "has_child": {
      "inner_hits": {},
      "type": "book",
      "query": {
        "match": {
          "title": "Game"
        }
      }
    }
  }
}
```

#### 2. 가독성 향상을 위한 Pretty 출력

- `pretty` 파라미터를 사용하여 JSON 출력을 보기 좋게 포맷팅할 수 있습니다.
- 이는 JSON 응답을 사람이 읽기 쉬운 형태로 들여쓰기와 줄바꿈을 적용합니다.
- 디버깅이나 개발 과정에서 특히 유용합니다.

```
GET /library/_search?pretty
{
  "query": {
    "has_parent": {
      "parent_type": "author",
      "query": {
        "match": {
          "name": "J.K. Rowling"
        }
      }
    }
  }
}
```

### HTTP 클라이언트를 통한 응답 형식 지정

Elasticsearch API를 HTTP 클라이언트(예: curl)로 호출할 때도 동일한 형식 옵션을 사용할 수 있습니다.

**추가로 --cacert /tmp/ca.crt에 대해서 제가 잘 몰랐어서 부가 설명 드리겠습니다.**

- `--cacert`: curl에서 HTTPS 연결 시 서버의 SSL/TLS 인증서를 검증하기 위한 CA 인증서 파일을 지정하는 옵션입니다.
- `/tmp/ca.crt`: CA 인증서 파일의 경로와 이름입니다. 이 파일은 서버의 인증서가 신뢰할 수 있는지 확인하는 데 사용됩니다.

이 옵션이 필요한 이유는:

`Elasticsearch`가 `SSL/TLS(HTTPS)`로 보안이 설정된 경우, 클라이언트는 서버의 인증서가 신뢰할 수 있는지 확인해야 합니다.

따라서 로컬에서 curl을 사용하기 위해서는 Elasticsearch Docker 컨테이너 안에 있는 인증서를 로컬로 옮겨줘야 합니다.


#### 기본 JSON 응답 예제:

```bash
curl -X GET --cacert /tmp/ca.crt -u elastic:changeme https://localhost:9200/library/_search -H "Content-Type: application/json" -d '{
  "query": {
    "has_parent": {
      "parent_type": "author",
      "query": {
        "match": {
          "name": "J.K. Rowling"
        }
      }
    }
  }
}'
```

#### Pretty 파라미터를 적용한 응답 예제:

```bash
curl -X GET --cacert /tmp/ca.crt -u elastic:changeme https://localhost:9200/library/_search?pretty -H "Content-Type: application/json" -d '{
  "query": {
    "has_parent": {
      "parent_type": "author",
      "query": {
        "match": {
          "name": "J.K. Rowling"
        }
      }
    }
  }
}'
```

### 응답 형식 선택 시 고려사항

1. **JSON vs YAML**:
   - JSON: 대부분의 프로그래밍 환경과 호환성이 높고 표준화되어 있습니다.
   - YAML: 사람이 읽고 쓰기에 더 친화적인 구문을 가지고 있어 복잡한 설정이나 구성 파일에 적합합니다.

2. **Pretty 파라미터**:
   - 개발 및 디버깅 단계에서는 가독성을 위해 `pretty` 파라미터를 사용하는 것이 좋습니다.
   - 프로덕션 환경에서는 불필요한 공백과 줄바꿈이 응답 크기를 증가시키므로 사용을 피하는 것이 좋습니다.

3. **응답 크기**:
   - 대용량 데이터를 처리할 때는 포맷팅이 응답 크기와 처리 시간에 영향을 미칠 수 있으므로 주의해야 합니다.

### 응용 예제

#### has_parent 쿼리 결과를 YAML 형식으로 요청:

```
GET /library/_search?format=yaml
{
  "query": {
    "has_parent": {
      "parent_type": "author",
      "query": {
        "match": {
          "name": "J.K. Rowling"
        }
      }
    }
  }
}
```

#### has_child 쿼리 결과를 예쁘게 포맷팅하여 요청:

```
GET /library/_search?pretty
{
  "query": {
    "has_child": {
      "inner_hits": {},
      "type": "book",
      "query": {
        "match": {
          "title": "Game"
        }
      }
    }
  }
}
```

### 결론

Elasticsearch는 기본적으로 JSON 형식을 사용하지만, YAML 형식 지원과 Pretty 출력 옵션을 통해 다양한 상황에 맞게 응답 형식을 조정할 수 있습니다. 

개발 과정에서는 가독성을 위해 포맷팅 옵션을 활용하고, 프로덕션 환경에서는 성능을 고려하여 적절한 형식을 선택하는 것이 중요합니다.
