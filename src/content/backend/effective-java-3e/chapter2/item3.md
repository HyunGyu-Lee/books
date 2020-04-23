# private 생성자나 열거 타입으로 싱글턴임을 보증하라
#### \#1. 싱글턴
- 싱글턴이란 인스턴스를 오직 하나만 생성할 수 있는 클래스를 말한다.
- 싱글턴의 전형적인 예로는 함수와 같은 무상태 객체나 설계상 유일해야하는 시스템 컴포넌트를 들 수 있다.
- 클래스를 싱글턴으로 만들면 이를 사용하는 클라이언트를 테스트하기 어려워질 수 있다.
  - 타입을 인터페이스로 정의한 후 해당 인터페이스를 구현하는 클래스를 싱글턴으로 한 게 아니라면 Mocking이 어렵다
- 싱글턴을 만드는 방법은 보통 둘 중 하나이고, 두 방식 모두 생성자를 private로 감추는 것을 기본으로 한다.

#### \#2. public static final 필드 방식의 싱글턴
```java
public class Elvis {
    public static final Elvis INSTANCE = new Elvis();

    private Elvis() { ... }

    ...
}
```
- private 생성자는 `public static final` 필드인 `INSTANCE`를 초기화할 때 딱 한번 호출된다.
- public이나 protected 생성자가 없으므로 Elvis 클래스가 초기화될때 생성되는 인스턴스가 하나임이 보장된다.
- 단, 권한이 있는 클라이언트는 `AccessibleObject.setAccessible`을 사용해 private 생성자를 호출할 수 있다.
- 이 경우 두 번째 인스턴스가 생성되려할때 예외를 던지도록 구현해두면 방어할 수 있다.

#### \#3. 정적 팩터리 방식의 싱글턴
```java
public class Elvis {
    private static final Elvis INSTANCE = new Elvis();

    private Elvis() { ... }

    public static Elvis getInstance() {
        return INSTANCE;
    }

    ...
}
```
- `Elvis.getInstance`는 항상 똑같은 객체를 반환하므로 제 2의 Elvis 객체는 절대 생성되지 않는다. (리플렉션 공격 방어는 동일하게 적용)

#### \#4. 두 방식의 비교
- `public static final` 방식의 싱글턴은 해당 클래스가 싱글턴임이 API에 명백히 드라난다는 점이다.
- `public static final` 방식의 싱글턴은 간결하다.
- 정적 팩토리 방식의 싱글턴은 API를 바꾸지 않고 싱글턴이 아니게 변경할 수 있다.
- 정적 팩토리 방식의 싱글턴은 정적 제네릭 싱글턴 팩터리로 쉽게 변경할 수 있다.
- 정적 팩토리 방식의 싱글턴은 메소드 참조 방식을 이용할 수 있다.
```java
public class Main {

    public static void doWithElvis(Supplier<Elvis> elvisSupplier) {
        ...
    }

    public static void main() {
        // getInstance를 메소드 참조로 활용
        doWithElvis(Elvis::getInstance);
    } 
}
```
- 이러한 방식의 장점이 굳이 필요없다면, `public static final`방식의 싱글턴이 좋다.

#### \#5. 싱글턴 클래스의 직렬화
- 싱글턴 방식 클래스를 직렬화하려면 단순히 `Serializable` 인터페이스를 구현하는 것으로는 부족하다.
- 모든 인스턴스 필드를 `transient`로 선언하고 `readResolve`메소드를 제공해야한다.
- 그렇지 않으면 역직렬화 할 때 마다 새로운 인스턴스가 생성된다.

#### \#6. 열거 타입을 활용한 싱글턴
```java
public enum Elvis {
    INSTANCE;
}
```
- 열거 타입 싱글턴 방식이 현재 널리 사용되진 않지만 위 소개한 2가지 방법의 단점을 해결해주면서 간결한 방법이기도 하다. 