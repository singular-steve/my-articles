### 7 상속보다는 조립
상속과 재사용
-상위 클래스의 기능 재사용, 확장하는 방법으로 활용

상속을 통한 기능 재사용시 단점
- 상위 클래스 변경 어려움
  - 상위 클래스 변경의 영향이 모든 하위 클래스로 전파됨
- 클래스 수 증가
  - (예)
    - 초기 설계 : Storage, CompressedStorage, EncryptedStorage
    - 설계 변경으로 인한 추가 : Storage, CompressedStorage, EncryptedStorage + CacheableStorage, CompressedEncryptedStorage, EncryptedCompressedStorage, CacheableEnCryptedStorage ...
- 상속 오용 발생 가능성
  - 상속 구현한 내용을 잘못 사용할 가능성 있음

  ```java
    public class Container extends ArrayList<Luggage> {
      private int maxSize;
      private int currentSize;

      public Container(int maxSize) {
        this.maxSize = maxSize;
      }
      public void put(Luggage lug) throw NotEnoughSpaceException {
        if (!canContain(lug)) throw NotEnoughSpaceException();
        super.add(lug);
        currentSize += lug.size();
      }
      public void extract(Luggage lug) {
        super.remove(lug);
        this.currentSize -+ lug.size();
      }
      public boolean canContain(Luggage lug) {
        return maxSize >= currentSize + lug.size();
      }
    }
  ```

  ```java
    Luggage size3Lug = new Luggage(3);
    Luggage size2Lug = new Luggage(2);
    Luggage size1Lug = new Luggage(1);

    Container c = new Container(5);
    if (c.canContain(size3Lug)) {
      c.put(size3Lug);
    }
    if (c.canContain(size2Lug)) {
      c.add(size2Lug);  // 잘못 사용함 : Container 의 여분 줄지 않음
    }
    if (c.canContain(size1Lug)) {  // 로직 오류 발생
      c.add(size1Lug);
    }
  ```

상속의 단점 해결 방법 : 조립(Composition)
- 여러 객체를 묶어서 더 복잡한 기능을 제공
- 보통 필드로 다른 객체를 참조하는 방식으로 조립
- 객체를 필요한 시점에 생성

```java
public class FlowController {
  private Encryptor encryptor = new Encryptor();

  public void process() {
    ...
    byte[] encryptedData = encryptor.encrypt(data);
    ...
  }
}
```

- (예) Compressor, Encryptor, CacheEngine 구현하여 Storage에서 사용
- (예) 불필요한 기능 상속으로 인항 오용 문제 해결

```java
public class Container {
  private int maxSize;
  private int currentSize;
  private List<Luggage> luggages = new ArrayList();
  ...
  public void put(Luggage lug) throw NotEnoughSpaceException {
    if (!canContain(lug)) throw NotEnoughSpaceException();
    luggages.add(lug);

  }
  ...
}
```

상속보다는 조립 (Composition over inheritance)
- 상속하기에 앞서 조립으로 풀 수 없는지 우선 검토
- 진짜 하위 타입인 경우에만 상속 사용
