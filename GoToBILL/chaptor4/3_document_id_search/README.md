```
GET /my_index/_search
{
  "query": {
    "ids": {
      "values": ["document_id_1", "document_id_2", "document_id_3"]
    }
  }
}
```
