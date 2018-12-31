### 11 DIP
고수준 모듈
- 의미 있는 단일 기능을 제공
- 상위 수준의 정책 구현

저수준 모듈
- 고수준 모듈의 기능을 구현하기 위해 필요한 하위 기능의 실제 구현

고수준 모듈, 저수준 모듈 예
- 수정한 도면 이미지를 NAS에 저장하고 측정 정보를 DB 테이블에 저장하고 수정 의뢰 정보를 DB에 저장하는 기능
  - 고수준 : 도면 이미지를 저장, 측정 정보를 저장, 도면 수정 의뢰
  - 저수준 : NAS에 이미지 저장, MEA_INFO 테이블에 저장, BP_MOD_REQ 테이블에 저장

고수준이 저수준에 직접 의존하면
- 저수준 모듈 변경으로 인해 고수준 모듈에 영향

```java
public class MeasureService {
  public void measure(MeasureReq req) {
    File file = req.getFile();
    nasStorage.save(file);

    jdbcTemplate.update("insert into MEAS_INFO ...");
    jdbcTemplate.update("insert into BP_MOD_REQ ...");
  }
}
```

```java
public class MeasureService {
  public void measure(MeasureReq req) {
    File file = req.getFile();
    s3storage.upload(file);

    jdbcTemplate.update("insert into MEAS_INFO ...");
    rabitmq.converAndSend(...);
  }
}
```


의존 역전 원칙 DIP : Dependency inversion principle
- 고수준 모듈은 저수준 모듈의 구현에 의존하면 안됨
- 저수준 모듈이 고수준 모듈에서 정의한 추상타입에 의존해야 함
- 예
  - MeasuerService
    - FileService interface ← NasFileStorage
    - MeasureInfoRepository interface ← JpaRepository
    - BlueprintModRequestor interface ← BPModReqDb

고수준 관점에서 추상화
- 예
  - 구현입장 추상화 : ExceptionAdvice → SentryService interface ← SentryClientService
  - 고수준 입장 추상화 : ExceptionAdvice → ExceptionCollector interface ← SentryController

DIP 는 유연함을 높임
- 고수준 모듈의 변경을 최소화하면서 저수준 모듈의 유연함을 높임
- 예
  - ExceptionAdvie → ExceptionCollector interface ← SentryController, ElasticSearchController

DIP 연습
- 상품 상세 정보와 추천 상품 목록 제공 기능
  - 상품 번호를 이용해서 상품 DB에서 상세 정보 구함
  - Daara AIP 이용하여 추천 상품 5개 구함
  - 추천 상품 5개 미만이면 같은 분류의 상품 중 최근 한달 판매가 많은 상품을 ERP에서 구해 5개 추천

- 고수준 모듈
  - 상품 번호로 상품 상세 정보 구함
  - 추천 상품 5개 구함
  - 인기 상품 구함

- 저수준 모듈
  - DB 에서 상세 정보 구함
  - Daara API에서 상품 5개 구함
  - 같은 분류에 속한 상품에서 최근 한달 판매가 많은 상품 ERP에서 구함

- ProductServie
  - ProductRepository inteface ← DBProductRepository
  - ProductRecommender interface ← DaaraRecommender
  - PopuplarProductService interface ←ErpPopuplarProductService
