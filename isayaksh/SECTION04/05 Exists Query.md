# Exists Query
- 필드가 Document 내부에 있는지 확인하기 위해서 사용한다.
- 필드를 포함한 Document를 필터링할 수 있다.
- Field Absence: Exists 쿼리를 사용하면, Document에 특정 필드가 존재하지 않을 경우, false를 return한다.
- 🔥 Null value: 필드에 해당하는 값을 NULL로 세팅하면, 해당 필드는 존재하지 않는 것으로 취급한다.
- Empty Arrays: 필드가 빈 배열이라면, 해당 필드는 존재하지 않는 것으로 취급한다.

