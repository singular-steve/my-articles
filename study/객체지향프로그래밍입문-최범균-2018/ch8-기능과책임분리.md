### 8 기능과 책임 분리
기능 분해
- 기능은 하위 기능으로 분해 가능
- (예)
  - 암호 변경
    - 변경 대상 확인
      - 변경 대상 구함
      - 대상 없으면 오류 응답
    - 대상 암호 변경
      - 암호 일치 여부 확인, 불일치하면 암호 불일치 응답
      - 암호 데이터 변경

기능을 누가 제공할 것인가?
- 기능은 책임
- 분리한 기능을 알맞게 분배하여 객체 지향 설계
- (예)
  - 암호변경 : ChangePasswordService
  - 변경대상구하기 : MemberRepository
  - 대상암호변경 : Member

  ```java
  public class ChangePasswordService {
    public Result changePassword(String id, String oldPw, String new Pw) {
      Member mem = memberRepository.findOne(id);
      if (mem == null) {
        return Result.NO_MEMBER;
      }
      try {
        mem.changePassword(oldPw, newPw);
        return Result.SUCCESS;
      } catch(BadPasswordException ex) {
        return Result.BAD_PASSWORD;
      }
    }
  }
  ```

큰 클래스, 큰 메서드
- 클래스나 메서드가 커지면 절차 지향의 문제 발생
  - 큰 클래스 : 많은 필드를 많은 메서드가 공유
  - 큰 메서드 : 많은 변수를 많은 코드가 공유
  - 여러 기능이 한 클래스/메서드에 섞여 있을 가능성 높아짐
- 책임에 따라 알맞게 코드 분리 필요

책임 분배/분리 방법
- 패턴 적용
- 계산 기능 분리
- 외부 연동 분리
- 조건 분기 추상화

패턴 적용
- 전형적 역할 분리
  - 간단한 웹 : 컨트롤러, 서비스, DAO
  - 복잡한 도메인 : 모델 → Entity, Repository, Domain Service
  - AOP : Aspect(공통기능)
  - GoF : Factory, Builder, 전략, 템플릿 메서드, 프록시/데코레이션 등

계산 분리
```java
Member mem = memberRepository.findOne(id);
Product prod = productRepository.findOne(prodId);

int payAmout = prod.price() * orderReq.getAmount();
double pointRate = 0.01;
if (mem.getMembership() == GOLD) {
  pointRate = 0.03;
} else if (mem.getMembership() == SILVER) {
  pointRate = 0.02;
}
if (isDoublePointTarget(prod)) {
  pointRate *= 2;
}
int point = (int) (payAmout * pointRate);
```

- 계산부분을 분리하여 구현
```java
Member mem = memberRepository.findOne(id);
Product prod = productRepository.findOne(prodId);

int payAmout = prod.price() * orderReq.getAmount();
PointCalculator cal = new PointCalculator(...);
int point = cal.calculate();
```

```java
public class PointCalculator {
  ...
  public int calculate() {
    double pointRate = 0.01;
    if (mem.getMembership() == GOLD) {
      pointRate = 0.03;
    } else if (mem.getMembership() == SILVER) {
      pointRate = 0.02;
    }
    if (isDoublePointTarget(prod)) {
      pointRate *= 2;
    }
    return (int) (payAmout * pointRate);
  }
}
```

연동 분리
- 네트워크, 메시징, 파일 등 연동 처리 코드 분리

```java
Product prod = findOne(id);
RestTemplate rest = new RestTemplate();
List<RecoItem> recoItems = rest.get("http://.../recommend?id=" + prod.getId() + "&user=" + userId + "&category=" + prod.getCategory(), RecoItem.class);
```

```java
Product prod = findOne(id);
RecommendService recoService = new RecommendService();
List<RecoItem> recoItems = recoService.getRecoItems(prod.getId(), userId, prod.getCategory());
```

조건 분기 추상화
- 연속적인 if-else 추상화

```java
String fileUrl = "";
if (fileId.startsWith("local:")) {
  fileUrl = "/files/" + fileId.substring(6);
} else if (fileId.startsWith("ss:")) {
  fileUrl = "http://fileserver/files/" + filedId.substring(3);
}
```

```java
FileInfo fileInfo = FileInfo.getFileInfo(fileUrl);
String fileUrl = fileInfo.getUrl();
```

```java
public interface FileInfo {
  String getUrl();
  static FileInfo getFile(...) {...}
}

public class SSFileInfo implements FileInfo {
  private String fileId;

  public String getUrl() {
    return "http://fileserver/files/" + filedId.substring(3);
  }
}
```

의도가 잘 드러나는 이름 사용
- (예) Http로 추천 데이터를 읽어오는 기능 분리
  - RecommendService > HttpDataService

역할 분리와 테스트
- 역할 분리가 잘 되면 테스트 용이해짐

기능과 책임 분리 예제

```java
public class CashClient {
  private SecretKeySpec keySpec;
  private IvParameterSpec ivSpec;

  private Res post(Req req) {
    String reqBody = 
  }
}

```
