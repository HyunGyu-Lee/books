# Comparable을 구현할지 고민하라
- Comparable 인터페이스는 compareTo 메소드를 보유하고 있다.
- compareTo 메소드는 단순 동치성 비교, 순서 비교, 제네릭한 특징을 갖고 있다.
- Comparable을 구현했다는 뜻은 해당 클래스의 인스턴스들에 순서가 있다는 뜻이다.
- 검색, 극단값 계싼, 자동 정렬되는 컬렉션 관리도 쉽게 가능하다.
- Comparable을 구현하는 것만으로도 자바 라이브러리의 제네릭으로 구현된 알고리즘, 컬렉션 프레임워크의 힘을 누릴 수 있다.

#### \#1. Comparable 인터페이스의 일반 규약
```java
public interface Comparable<T> {
    int compareTo(T t);
}
```
- 이 객체와 주어진 객체의 순서를 비교한다. 이 객체가 작으면 음의 정수, 같으면 0, 크면 양의 정수를 반환한다.
  - 비교할 수 없는 객체가 주어진 경우 ClassCastException을 던진다.
- Comparable을 구현한 클래스는 모든 x, y에 대해 sgn(x.compareTo(y)) = -sgn(y.compareTo(x))여야한다.
  - 여기서 sgn은 부호함수로 표현식의 값이 음수, 0, 양수 일 때 -1, 0, 1을 반환하도록 정의한다.
- Comparable을 구현한 클래스는 `추이성`을 보장해야 한다. 즉 x.compareTo(y) > 0 && y.compareTo(z) > 0이면 x.compareTo(z) > 0이어야 한다.
- Comparable을 구현한 클래스는 모든 z에 대해 x.compareTo(y) == 0이면 sgn(x.compareTo(z)) == sgn(y.compareTo(z))여야한다.
- (x.compareTo(y) == 0) == (x.equals(y))여야한다.
  - 이 사항은 필수는 아니지만 지키면 좋다.
  - 지키지 않을경우 반드시 명시해주는 것이 좋다.
  - 이 사항을 지켜주면 compareTo의 줄지어진 순서와 equals의 동치성 비교가 일관성을 갖게된다.
  - 정렬된 컬렉션 (TreeMap, TreeSet)등은 동치성 비교 시 equals대신 compareTo를 사용한다.

#### \#2. compareTo 메소드 작성요령
- 비교 인자가 null이면 NPE를 던진다. (실제로 null값에 접근 시 자동으로 발생)
- 객체 참조 필드 비교 시 해당 필드의 compareTo 메소드를 재귀적으로 호출
- Comparable을 구현하지 않은 필드나 표준이 아닌 순서로 비교하려면 Comparator를 대신 이용한다.
  - Comporator는 직접 구현 혹은 자바에서 제공하는 것을 사용
- 정수, 실수 등 기본타입의 경우 박싱된 클래스의 compare 메소드를 호출 (e.g. Float.compare)
- 클래스의 핵심필드가 여러 개라면 중요도에 따라 우선적으로 비교한다.
  - 핵심필드 비교결과가 0이 아니라면(같지 않으면) 이미 순서가 결정된 것으로 이후 필드는 비교할 필요가 없다. 
```java
public int compareTo(PhoneNumber pn) {
    int result = Short.compare(areaCode, pn.areaCode);
    if (result == 0) {
        result = Short.compare(prefix, pn.prefix);
        if (result == 0) {
            result = Short.compare(lineNum, pn.lineNum);
        }
    }
    return result;
}
```
#### \#3. Comparator 활용
- 자바8부터는 Comparator 인터페이스가 일련의 비교자 생성 메소드를 제공 및 메소드 체이닝 방식으로 비교자를 생성할 수 있다.
- 이렇게 생성한 비교자는 compareTo에서 편하게 사용할 수 있다.
  - 단, 약간의 성능저하가 있을수있다.
```java
private static final Comparator<PhoneNumber> PN_COMPORATOR = 
    Comparator.comparingInt((PhoneNumber pn) -> pn.areaCode)  // 람다식 사용 시 타입 추론을 도와주도록 타입을 명시
              .thenComparingInt(pn -> pn.prefix)
              .thenComparingINt(pn -> pn.lineNum);

public int compareTo(PhoneNumber pn) {
    return PN_COMPORATOR.compare(this, pn);
}
```
- 객체 비교를 위해 Comparator.comparing 메소드도 활용 가능하다.

#### \#4. 주의사항
- 값의 차로 비교하는 방식일 때 해시코드를 종종 사용하는 경우가 있다.
```java
public int compareTo(Object o) {
    return this.hashCode() - o.hashCode();
}
```
- 위 방식은 사용하면 안된다.
  - 정수 오버플로가 발생하거나 부동소수점 계산 방식에 따른 오류가 발생할 수 있다.
- 대신 아래왁 같이 구현할 수 있다.
```java
public int compareTo(Object o) {
    return Integer.compare(this.hashCode(), o.hashCode());
}
```