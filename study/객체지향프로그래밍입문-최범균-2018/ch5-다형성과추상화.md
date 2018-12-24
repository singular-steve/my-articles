### 5 다형성과 추상화
다형성
- 여러(poly) 모습(morph)을 갖는 것
- 객체 지향에서의 다형성은 한 객체가 여러 타입을 갖는 것
  - 한 객체가 여러 타입의 기능을 제공
  - 타입 상속으로 다형성 구현

예 : IotTimer 객체는 Timer 객체, Rechargeable 객체 타입 기능을 모두 제공
```java
public class Timer {
  public void start() {...}
  public void stop() {...}
}

public interface Rechargeable {
  void charge();
}

public class IotTimer extends Timer implements Rechargeable {
  public void charge() {...}
}

...
  IotTimer it = new IotTimer();
  it.start();
  it.stop();

  Timer t = it;
  t.start();
  t.stop();

  Rechargeable r = it;
  r.charge();
...
```

추상화(Abstraction)
- 데이터나 프로세스 등 의미가 비슷한 개념이나 의미 있는 표현으로 정의하는 과정
- 두 가지 방식의 추상화
  - 특정한 성질
    - DB의 사용자 테이블 : 아이디, 이름, 이메일
    - Money 클래스 : 통화, 금액
  - 공통 성질 (일반화) - 다형성과 관련됨
    - 프린터 : HP Mxxx, 삼성 SL-Mxxxx

##### 예
SCP로 파일 업로드
HTTP로 데이터 전송
DB 테이블에 삽입

=> 추상화 => 푸시 발송 요청

타입 추상화
- 여러 구현 클래스를 대표하는 상위 타입 도출
  - 인터페이스 타입으로 추상화
  - 추상화 타입과 구현은 타입 상속으로 연결
- 예
  - Interface : <<Interface>> Notifier
    - 기능에 대한 의미 제공, 구현을 제공하지 않음
  - Concrete 클래스 : EmailNotifier, SMSNotifier, KakaoNotifier

추상 타입 사용
- 추상타입을 이용한 프로그래밍
```java
  Notifier Notifier = getNotifier(...);
  notifier.notify(...);
```
- 추상 타입은 구현을 감추어 기능의 구현이 아닌 의도를 더 잘 드러냄

추상 타입 사용에 따른 이점
- 유연함!
- 예제 : 요구사항 변경에 따라 주문 취소 코드 변경
```java
private SmsSender smsSender;

public void cancel(String ono){
  ... 주문 취소 처리
  smsSender.sendSms(...);
}
```

```java
private SmsSender smsSender;
private KakaoPush kakaoPush;

public void cancel(String ono){
  ... 주문 취소 처리
  if (pushEnabled) {
    kakaoPush.push(...);
  } else {
    smsSender.sendSms(...);
  }
}
```

```java
private SmsSender smsSender;
private KakaoPush kakaoPush;
private MailService mailSvc;

public void cancel(String ono){
  ... 주문 취소 처리
  if (pushEnabled) {
    kakaoPush.push(...);
  } else {
    smsSender.sendSms(...);
  }
  mailSvc.sendMail(...);
}
```

```java
// 추상타입을 사용
public void cancel(String ono) {
  ... 주문 취소 처리
  Notifier notifier = getNotifier(...);
  notifier.notify(...);
}

private Notifier getNotifier(...) {
  if (pushEnabled) {
    kakaoPush.push(...);
  } else {
    smsSender.sendSms(...);
  }
}
```

```java
// 한번더 추상화
// 주문 취소 처리 로직 변경 필요 없음
public void cancel(String ono) {
  ... 주문 취소 처리
  Notifier notifier = NotifierFactory.instance().getNotifier(...);
  notifier.notify(...);
}

public interface NotifierFactory {
  Notifier notifier = getNotifier(...);

  static NotifierFactory instance() {
    return new DefaultNotifierFactory();
  }
}

public class DefaultNotifierFactory implements NotifierFactory {
  public Notifier getNotifier(...) {
    if (pushEnabled) {
      return new KakaoNotifier();
    } else {
      return new SMSNotifier();
    }
  }
}

```

추상화는 의존 대상이 변경하는 시점에 한다
- 실제 변경 및 확장이 발생할 때 추상화 시도
- 아직 존재하지 않는 기능에 대한 추상화는 주의 : 추상화 → 추상타입 증가 → 복잡도 증가
