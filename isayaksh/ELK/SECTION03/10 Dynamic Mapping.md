# Dynamic Mapping
- 스키마가 없는 상태에세 Document를 추가하였을 때 자동으로 매핑을 생성해주는 기능
- 새로운 필드를 자동으로 감지하고 적절한 데이터 타입을 추측해서 매핑(mapping)을 생성해주는 기능
- 들어오는 데이터의 구조를 사전에 알 수 없는 상황에서 미리 매핑을 명시하지 않아도, 새로운 필드를 실시간으로 처리할 수 있게 해준다.
- 🔥 Date 타입을 자동으로 감지하는 기능이 있기는 하지만, 필드의 목적이 처음부터 Date라고 생각한다면 맨 처음 필드를 매핑할 때 정의하는 것을 지향하는 것이 관리 측면에서 좋다.

## Default Dynamic Mapping Behavior
- String : value가 text 같다면, keyword를 서브필드로 갖는 text로 매핑된다.
- Numbers : 숫자 value는 데이터 형식에 따라 long, double, ...와 매핑된다.
- Floating point(소수점) : Float가 매핑된다.
- Dates : 날자 형식으로 보이는 string을 date로 매핑한다.
- booleans : true, false로 보이는 value는 boolean으로 매핑된다.
- Object: 이건 그냥 알아서 당연히;;

