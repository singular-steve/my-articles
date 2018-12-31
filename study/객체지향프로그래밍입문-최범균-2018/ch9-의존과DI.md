### 9 의존과 DI

의존
- 기능구현을 위해 다른 구성요소를 사용하는 것
  - 예 : 객체 생성, 메서드 호출, 데이터 사용
- 의존은 변경이 전파될 가능성을 의미
  - 의존하는 대상이 변경되면 변경될 가능성이 높아짐
    - 예 : 호출하는 메서드의 파라미터가 변경, 호출하는 메서드의 익셉션 타입이 추가

순환 의존
- A → B, B → C, C → A
- 변경 연쇄 전파 가능성 : 클래스, 패키지, 모듈 등 모든 수준에서 순환 의존 없애야함

의존하는 대상이 많다면?
- A → X, B → X, C → X, .... F → X
- X 는 의존하는 대상 A, ... F 가 바뀌는 경우 변경되야할 가능성 있음
- 의존하는 대상은 적을 수록 좋음

의존 대상이 많은 경우 1 : 기능이 많은 클래스

```java
public class UserService {
  public void regist(RegReq regReq) {
    ...
  }
  public void changePw(ChangeReq chgReq) {
    ...
  }
  public void blockUser(String id, String reason) {
    ...
  }
}
```
- 각 기능마다 의존하는 대상이 다를 수 있음
- 한 기능 변경이 다른 기능에 영향을 줄 수 있음


- 기능 별로 분리 고려

  ```java
  public class UserRegistService {
    public void regist(...) {
      ...
    }
  }
  public class ChangePwService {
    public void changePw(...) {
      ...
    }
  }
  public class UserBlockService {
    public void blockUser(...) {
      ...
    }
  }
  ```

의존 대상 많은 경우 2 : 묶어보기
- 여러 의존 대상을 단일 기능으로 묶어서 의존 대상을 줄일 수 있음
- 예
  - AutoDebitService → MinwonFactory, MinwonRepository
  - AutoDebitService → AutoDebitMinwonRegister 로 기능 추상화

의존 대상 객체를 직접 생성하면?
- 생성 클래스가 바뀌면 의존하는 코드도 바뀜
- 의존 대상 객체를 직접 생성하지 않는 방법
  - 팩토리, 빌더
  - **의존 주입(Dependency Injection)**
  - 서비스 로케이터(Service Locator)

의존 주입
- 외부에서 의존 객체를 주입
- 생성자나 메서드를 이용해서 주입

```java
public class ScheduleService {
  private UserRepository repository;
  private Calculator cal;

  public ScheduleService(UserRepository) {
    this.repository = repository;
  }
  public void setCalculator(Calculator cal) {
    this.cal = cal;
  }
}

// 초기화 코드
UserRepository userRepo = new DbUserRepository();
Calculator Calculator = new Calculator();

ScheduleService schSvc = new ScheduleService(userRepo);
schSvc.setCalculator(cal);
```

조립기(Assembler)

- 조립기가 객체 생성, 의존 주입을 처리
  - 예 : 스프링 프레임워크

```java
@Configuration
public class Config {
  @Bean
  public ScheduleService scheduleSvc() {
    ScheduleService svc = new ScheduleService(repo());
    svc.setCalculator(expCal());
    return svc;
  }
  @Bean
  public UserRepository repo() {...}
  @Bean
  public Calculator expCal() {...}
}

// 초기화
ctx = new AnnotationConfigApplicationContext(Config.class);
// 사용할 객체 구함
ScheduleService svc = ctx.getBean(ScheduleService.class);
// 사용
svc.getSchedule(...);
```


DI 장점 1
- 상위 타입을 사용할 경우 의존 대상이 바뀌면 조립기(설정)만 변경하면 됨

```java
public class OrderService {
  private Notifier notifier;

  public OrderService(Notifier notifier) {
    this.notifier = notifier;
  }

  public void order(OrderRequest req) {
    ...
    notifier.notify(...);
  }
}

@Configuration
public class Config {
  @Bean
  public Notifier notifier() {
    return new EmailNotifier();
  }
  @Bean
  public OrderService orderService() {
    return new OrderService(notifier());
  }
}
```

- EmailNotifier 를 상위 Notifier 로 변경하는 예 : 조립기(설정)만 변경

  ```java
  @Configuration
  public class Config {
    @Bean
    public Notifier notifier() {
      return new CompositeNotifier(new EmailNotifier(), new KakaoNotifier());
    }
    ...
  }
  ```

DI 장점 2
- 의존하는 객체 없이 대역 객체를 사용해서 테스트 가능
- 예 : ScheduleService 테스트할 때 의존하고 있는 UserRepository 객체 대신 대역 객체 MemoryUserRepository를 사용하여 ScheduleService의 기능을 테스트함

```java
private MemoryUserRepository userRepo = new MemoryUserRepository();
private ScheduleService svc = new ScheduleService();

@BeforeEach
public void init() {
  svc.setUserRepository(userRepo);
}

@Test
public void givenUser_noCheckPoint_thenGetExpectedSchedule() {
  userRepo.addUser("1", new User(...));
  Schedule schedule = svc.getSchedule("1");
  assertEqulas(EXPECTED, schedule.getType());
}
```

**DI를 습관처럼 사용하기**
- 의존 객체는 주입받도록 코드 작성하는 습관 필요!
