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
  // 암호변경기능
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

**책임 분배/분리 방법**
- 패턴 적용
- 계산 기능 분리
- 외부 연동 분리
- 조건 분기 추상화

패턴 적용
- 전형적 역할 분리
  - 간단한 웹 : 컨트롤러, 서비스, DAO
  - 복잡한 도메인 : 모델 → Entity, Value, Repository, Domain Service
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

외부 연동 분리
- 네트워크, 메시징, 파일 등 연동 처리 코드 분리

```java
Product prod = findOne(id);
RestTemplate rest = new RestTemplate();
List<RecoItem> recoItems = rest.get("http://.../recommend?id=" + prod.getId() + "&user=" + userId + "&category=" + prod.getCategory(), RecoItem.class);
```

```java
Product prod = findOne(id);
RecommendService recoService = new RecommendService(); // 외부 연동기능 별도 Class 로 분리
List<RecoItem> recoItems = recoService.getRecoItems(prod.getId(), userId, prod.getCategory());
```

조건 분기 추상화
- 연속적인 if-else 추상화
- interface 와 하위 클래스를 통해 추상화

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

주의할 점 : 의도가 잘 드러나는 이름 사용
- (예) Http로 추천 데이터를 읽어오는 기능 분리
  - RecommendService 가 HttpDataService 보다 의도가 잘 드러남

역할 분리와 테스트
- 역할 분리가 잘 되면 테스트 용이해짐

분리 연습 1

```java
public class CashClient {
  private SecretKeySpec keySpec;
  private IvParameterSpec ivSpec;

  private Res post(Req req) {
    String reqBody = toJson(req);

    Cipher cipher = Cipher.getInstacne(DEFAULT_TRANSFORM);
    cipher.init(Cipher.ENCRYPT_MODE, keySpec, ivSpec);
    String encReqBody = new String(Base64.getEncoder().encode(cipher.doFinal(reqBody)));

    ResponseEntity<String> responseEntity = restTemplate.postForEntity(api, encReqBody, String.class);

    String encRespBody = responseEntity.getBody();

    Cipher ipher2 = Cipher.getInstacne(DEFAULT_TRANSFORM);
    cipher2.init(Cipher.DECRYPT_MODE, keySpec, ivSpec);
    String respBody = new String(cipher.doFinal(Base64.getDecoder().decode(encRespBody)));

    return jsonToObj(respBody);
  }
}
```

- 계산기능 분리 (암복호화)

```java
public class CashClient {
  private Cryptor cryptor;

  private Res post(Req req) {
    String reqBody = toJson(req);

    String encReqBody = cryptor.encrypt(reqBody);

    ResponseEntity<String> responseEntity = restTemplate.postForEntity(api, encReqBody, String.class);

    String encRespBody = cryptor.decrypt(encReqBody);

    return jsonToObj(respBody);
  }
}

public class Cryptor {
  private SecretKeySpec keySpec;
  private IvParameterSpec ivSpec;

  public String encrypt(String plain) {
    Cipher cipher = Cipher.getInstacne(DEFAULT_TRANSFORM);
    cipher.init(Cipher.ENCRYPT_MODE, keySpec, ivSpec);
    return new String(Base64.getEncoder().encode(cipher.doFinal(reqBody)));
  }

  public String decrypt(String encrypted) {
    Cipher ipher2 = Cipher.getInstacne(DEFAULT_TRANSFORM);
    cipher2.init(Cipher.DECRYPT_MODE, keySpec, ivSpec);
    return new String(cipher.doFinal(Base64.getDecoder().decode(encRespBody)));
  }
}
```

분리 연습 2
```java
public class Rental {
  private Movie movie;
  private int daysRented;

  public int getFrequentRenterPoints() {
    if (movie.getPriceCode() == Movie.NEW_RELEASE && daysRented > 1) {
      return 2;
    } else {
      return 1;
    }
  }
  ...
}

public class Movie {
  public static int REGULAR = 0;
  public static int NEW_RELEASE = 1;
  private int priceCode;

  public int getPriceCode() {
    return priceCode;
  }
  ...
}
```

- if-else 블록 추상화

```java
public class Rental {
  private Movie movie;
  private int daysRented;

  public int getFrequentRenterPoints() {
    return movie.getFrequentRenterPoints(daysRented);
  }
  ...
}

public abstract class Movie {
  public abstract int getFrequentRenterPoints(int daysRented);
  ...
}

public class NewReleaseMovie extends Movie {
  public int getFrequentRenterPoints(int daysRented) {
    return daysRented > 1 ? 2 : 1;
  }
}

public class RegularMovie extends Movie {
  public int getFrequentRenterPoints(int daysRented) {
    return 1;
  }
}
```

분리 연습 3

- 회원가입 기능 설계
  - 사용자는 이메일, 이름, 암호 입력 (모두 필수)
  - 암호가 정해진 규칙을 통과하지 않으면 다시 입력 : 규칙1, 규칙2, ...
  - 같은 이메일로 가입한 회원이 있으면 다시 입력
  - 이메일 인증을 위한 메일 발송 (유효성 검증을 위해 암호화된 토큰을 사용)
  - 회원 가입 완료


- 하위기능으로 나누어 추상화


- 회원가입기능
  - 웹 요청 : RegistController
    - 필수값 검증 : RegistCommandValidator - 계산분리
    - 회원가입 처리
  - 회원 가입 : RegistService
    - 암호 규칙 검사 : PasswordPolicy - 계산분리
      - 검사에 통과하지 못하면 가입실패
    - 중복 이메일 가입여부 확익
      - 이메일로 회원 조회 : MemberRepository
      - 존재하면 가입실패
    - 인증메일발송 : AuthMailSender, MailAuthRequestor
      - 토큰생성 : AuthTokenGen
      - 토큰저장 : TokenRepository
      - 인증메일전송
    - 회원정보 저장 : MemberRepository
