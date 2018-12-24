### 4 캡슐화 예제

##### 예제 1
```java
public AuthResult authenticate(String id, String pw) {
  Member mem = findOne(id);

  if (mem == null) return AuthResult.NO_MATCH;

  if (mem.getVerificationEmailStatus() != 2) {
    return AuthResult.NO_EMAIL_VERIFIED;
  }

  if (passwordEncoder.isPasswordValid(mem.getPassword(), pw, mem.getId())) {
    return AuthResult.SUCCESS;
  }

  return AuthResult.NO_MATCH;
}
```

Tell, Don't Ask 적용

```java
  ...
  if (mem.isEmailVerified()) {
    return AuthResult.NO_EMAIL_VERIFIED;
  }
  ...

public class Member {
  ...
  public boolean isEmailVerified() {
    return verificationEmailStatus == 2;
  }
  ...
}
```


##### 예제2 (마틴 파울러 - 리팩토링)
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
}

public class Movie {
  public static int REGULAR = 0;
  public static int NEW_RELEASE = 1;
  private int priceCode;

  public int getPriceCode() {
    return priceCode;
  }
}

```

getFrequentRenterPoints 로직 구현을 Movie로 이동
(데이터가 있는 부분에 로직 구현)
```java
... Rental
  public int getFrequentRenterPoints() {
    return movie.getFrequentRenterPoints(daysRented);
  }
...

... Movie
  public int getFrequentRenterPoints(int daysRented) {
    if (priceCode == Movie.NEW_RELEASE && daysRented > 1) {
      return 2;
    } else {
      return 1;
    }
  }
...
```

##### 예제3

```java
...
  Timer t = new Timer();
  t.startTime = System.currentTimeMillis();
  ...
  t.stopTime = System.currentTimeMillis();
  long elaspedTime = t.stopTime - t.startTime;
...

public class Timer {
  public long startTime;
  public long stopTime;
}
```

절차지향적 코드를 캡슐화를 통해 객체지향적 코드로 변경

```java
...
  Timer t = new Timer();
  t.start();
  ...
  t.stop();
  long elaspedTime = t.elaspedTime(MILLISECOND);
...

public class Timer {
  public long startTime;
  public long stopTime;

  public void start() {
    this.startTime = System.currentTimeMillis();
  }
  public void stop() {
    this.stopTime = System.currentTimeMillis();
  }
  public long elaspedTime(TimeUnit unit) {
    switch(unit) {
      case MILLISECOND: return stopTime - startTime;
      ...
    }
  }

}
```

nano초 단위로 내용 변경시 Timer 내부 코드만 변경하면 된다.

##### 예제4

```java
public void verifyEmail(String token) {
  Member mem = findByToken(token);
  if (mem == null) throw new BadTokenException();

  if (mem.getVerificationEmailStatus() == 2) {
    throw new AlreadyVerifiedException();
  } else {
    mem.setVerificationEmailStatus(2);
  }
  // ... 수정사항 DB 반영
}
```

데이터를 가져와서 (판단하고) 변경된 결과를 셋팅하는 패턴 전체를 데이터를 가진 부분으로 이동하여 캡슐화

```java
  ...
    mem.verifyEmail();
  ...

public class Member {
  private int verificationEmailStatus;

  if (isEmailVerified()) {
    throw new AlreadyVerifiedException();
  } else {
    this.verificationEmailStatus = 2;
  }

  public boolean isEmailVerified() {
    return verificationEmailStatus;
  }
}
```
