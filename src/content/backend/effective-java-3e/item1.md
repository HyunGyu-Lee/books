# 생성자 대신 정적 팩터리 메서드를 고려하라
#### \#1. 클래스의 인스턴스를 생성하는 여러 방법
- 클라이언트가 클래스의 인스턴스를 얻는 전통적인 수단은 public 생성자이다.
- 생성자와 별도로 정적 팩터리 메서드(static factory method)를 제공할 수 있다.
- 아래는 boolean타입을 받아 Boolean(박싱 클래스)로 반환해주는 정적 팩터리 메소드이다.
```java
public static Boolean valueOf(boolean b) {
    return b ? Boolean.TRUE : Boolean.FALSE;
}
```

#### \#2. public 생성자에 비해 정적 팩토리 메소드가 갖는 장점
- 이름을 가질 수 있다.
  - 생성자에 넘기는 매개변수와 생성자만으로는 반환될 객체의 특성을 제대로 설명할 수 없다.
  - 정적 팩토리 메소드는 메소드명만 잘 지으면 반환될 객체의 특성을 쉽게 묘사할 수 있다.
  - `BigInteger(int, int, Random)`과 `BigInteger.probablePrime` 중 어느쪽이 `소수인 BigInteger를 반환한다`를 잘 설명하는지 비교
  - 하나의 시그니처로는 생성자 하나만 만들 수 있다. 하지만 정적 팩토리 메소드는 이름이 다른 동일한 시그니처의 메소드를 만들수 있다.
- 호출될 때 마다 인스턴스를 새로 생성하지는 않아도 된다.
  - **불변 클래스**는 인스턴스를 미리 만들어놓거나, 캐싱하여 재사용하는 식으로 불필요한 객체생성을 피할 수 있다.
  - 특히 생성 비용이 큰 경우 성능을 상당 부분 개선해줄 수 있다. ([플라이웨이트 패턴](https://palpit.tistory.com/198)과 유사)
  - 정적 팩토리 메소드는 클래스를 **인스턴스 통제 클래스(instance-controlled class)**로 만들수 있다.
    - 인스턴스를 통제하면 클래스를 싱글턴으로 만들수도, 인스턴스화 불가화, 동치인 인스턴스가 단 하나만임을 보장하는 등 여러 제약이 가능하다.
- 반환 타입의 하위 타입 객체를 반환할 수 있는 능력이 있다.
  - API를 만들 때 구현 클래스를 공개하지 않고 해당 객체를 반환할 수 있어 API를 작게 유지할 수 있다.
  - 자바8 전에는 인터페이스에 정적 메서드를 선언할 수 없었기 때문에, `Type`류 인터페이스를 반환하는 반환하는 정적 메소드가 필요하다면 해당 정적 메소드를 갖는 `Types` 클래스를 만드는 것이 일반적이었다.
  - 예를들어 컬렉션 프레임워크의 경우 **수정불가(Unmodifiable)**, **동기화(Synchroized)** 등의 기능을 제공하는 객체를 `java.util.Collections` 클래스 하나로 얻어올 수 있도록 설계되어있다. (API 외견이 작고, 해당 API를 잘 사용하기 위해 익혀야 하는 개념과 난이도도 낮아짐)
  - 또한 일반적으로 정적 팩터리 메소드를 사용하는 클라이언트는 해당 구현체가 아닌 인터페이스만을 주로 다루게 된다. (좋은 습관)
- 입력 매개변수에 따라 매번 다른 클래스의 객체를 반환할 수 있다.
  - 반환 타입의 하위 타입이기만 하면 어떤 클래스의 객체를 반환하던 상관이 없다.
    - 예를들어 `EnumSet`의 경우 Enum이 64개 이하이면 long 변수로 관리하는 `RegularEnumSet`, 이상이면 long[]으로 관리하는 `JumboEnumSet`을 반환한다.
    - 클라이언트는 이 두 클래스의 존재를 모르며 알 필요도 없다. API측에서는 세부 구현 클래스를 추가, 삭제해도 클라이언트 측에서 문제가 없다.
- 정적 팩터리 메소드를 작성하는 시점에는 반환할 객체의 클래스가 존재하지 않아도 된다.
  - 이 유연함은 서비스 제공자 프레임워크 (Service Provider Framework)의 근간이 된다. (대표적으로 JDBC)
  - 서비스 제공자 프레임워크는 3가지 컴포넌트로 이루어진다.
    - 구현체의 동작을 정의하는 **서비스 인터페이스** (JDBC에서 `Connection`)
    - 제공자(서비스 구현체)가 구현체를 등록할 때 사용하는 **제공자 등록 API** (JDBC에서 `DriverManager.registerDriver`)
    - 클라이언트가 서비스의 인스턴스를 얻을 때 사용하는 **서비스 접근 API** (JDBC에서 `DriverManager.getConnection`)
    - 서비스 인터페이스의 인스턴스를 생성하는 팩터리 객체를 설명해주는 **서비스 제공자 인터페이스** (JDBC에서 `Driver`)
  - 클라이언트는 **서비스 접근 API**를 사용할 때 원하는 구현체의 조건을 명시할 수 있다.
  - 자바5 부터는 `java.util.ServiceLoader`라는 [범용 서비스 제공자 프레임워크](https://riptutorial.com/ko/java/example/19523/%EA%B0%84%EB%8B%A8%ED%95%9C-serviceloader-%EC%98%88%EC%A0%9C)가 제공된다.

#### \#3. 정적 팩터리 메소드의 단점
- 상속을 하려면 public, protected 생성자가 필요하므로, 정적 팩터리 메소드만 사용하면 상속을 할 수 없다.
- 정적 팩터리 메소드는 프로그래머가 찾기 어렵다.
  - 생성자처럼 API 설명에 명확하게 드러나지 않는다.
  - 자바독에 잘 표현되도록 널리 알려진 이름을 사용해야한다.
    - from : 매개변수를 하나 받아서 해당 타입의 인스턴스를 반환하는 형변환 메서드
      - Date d = Date.from(instant);
    - of : 여러 매개변수를 받아 적합한 타입의 인스턴스를 반환하는 집계 메서드
      - Set<Rank> faceCards = EnumSet.of(JACK, QUEEN, KING);
    - valueOf : from과 of의 더 자세한 버전
      - BigInteger prime = BigInteger.valueOf(Integer.MAX_VALUE);
    - instance / getInstance : (매개변수를 받는다면) 매개변수로 명시한 인스턴스를 반환하지만, 같은 인스턴스임을 보장하지 않는다.
      - StackWaler luke = StackWaler.getInstance(options);
    - create / newInstance : instance 혹은 getInstance와 같지만, 매번 새로운 인스턴스를 생성해 반환함을 보장한다.
      - Object newArray = Array.newInstance(classObject, arrayLen);
    - getType : getInstance와 같지만 생성할 클래스가 아닌 다른 클래스에 팩터리 메서드를 정의할 때 쓴다. **Type**은 팩터리 메서드가 반환할 객체의 타입
      - FileStore fs = Files.getFileStore(path);
      - Drawer drawer = Drawers.getDrawer(opts);
    - newType : newInstance와 같지만, 생성할 클래스가 아닌 다른 클래스에 팩터리 메서드를 정의할 때 쓴다. **Type**은 팩터리 메서드가 반환할 객체의 타입
      - BufferedReader br = Files.newBufferedReader(path);
    - type : getType, newType의 간결한 버전
      - List<String> names = Collections.list(nameArrays);