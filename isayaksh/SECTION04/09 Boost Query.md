# Boost Query
- 특정 쿼리에 부스트를 적용하거나, 특정 기준에 일치하는 문서의 관련성을 낮춤으로써 문서의 관련성 점수에 영향을 줄 수 있게 하는 기능이다.

###  🔥 Boost Query는 Positive Query와 Negative Query를 동시에 사용한다!

- Boost Query 의 동작 방식
  - Positive Query: 홍보하거나 부스트하고 싶은 문서에 일치하는 쿼리이다.
  - Negative Query: 감점하거나 관련성을 낮추고 싶은 문서에 일치하는 쿼리이다.

### 🔥 negative_boost
- 0의 값을 준다면, 매칭되는 결과는 아예 조회가 안된다.
- 1의 값을 준다면, 감점이 아예 없다.
- 만약 0.3을 준다면 negative query와 일치하는 결과의 score는 30%로 감소한다.