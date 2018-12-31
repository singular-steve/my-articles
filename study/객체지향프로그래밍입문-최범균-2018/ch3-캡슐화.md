### 3 캡슐화
캡슐화
- 기능의 구현을 외부에 감춤
- 정보 은닉
- 왜?? 외부에 영향 없이 객체 내부 구현 변경 가능

캡슐화하지 않으면
- 요구사항의 변화가 데이터 구조 / 사용에 변화를 발생시킴 → 데이터를 사용하는 코드 변경

캡슐화 장점
- 캡슐화된 기능을 사용하는 코드 영향 최소화하고 내부 구현을 변경할 수 있는 유연함 제공

캡슐화를 위한 규칙
- Tell, Don't Ask : 데이터를 요청하지 말고, 해달라고 요청하라!
- (예)

  ```java
  accout.getMembership() == REGULAR // WRONG!

  accout.hasRegularPermission() // GOOD!
  ```
- Demeter 의 법칙
  - 메서드에서 생성한 객체의 메서드만 호출
  - 파라미터로 받은 객체의 메서드만 호출
  - 필드로 참조하는 객체의 메서드만 호출
  - (예)
  
  ```java
  acc.getExpDate().isAfter(now) // WRONG!

  acc.isExpired() // GOOD!
  acc.isValid(now) // GOOD!
  ```
