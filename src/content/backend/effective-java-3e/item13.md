# clone 재정의는 주의해서 진행하라
- Cloneable은 복제해도 되는 클래스임을 명시하는 인터페이스지만, 몇 가지 맹점이 있다.
  - clone 메소드가 선언된 곳이 Clonable이 아닌 Object 클래스이다.
  - clone 메소드가 protected로 선언되어 있어 외부 호출이 어렵다.
  - Cloneable 인터페이스의 구현(implement)여부에 따라 clone() 메소드 동작이 달라진다.
    - 이는 일반적인 인터페이스 사용방식이 아니므로 따라하지 않을 것을 권장한다.
- 실무에서는 Cloneable을 구현한 클래스는 public clone메소드를 제공하며 제대로 객체복사가 될 것으로 예상하지만 이를 지키려면 그 클래스와 모든 상위 클래스는 복잡한 기술 명세를 지켜야한다.
- 생성자 호출 없이 객체를 생성할 수 있는 모순적인 상황도 생긴다.

#### \# Cloneable의 규약
- 객체의 복사본을 생성해 반환한다.
  - x.clone() != x
  - x.clone().getClass() == x.clone()
  - x.clone().equals(x)

#### \# clone의 구현 코드
1. 가변상태를 참조하지 않는 클래스용 clone 메소드
```java
@Override
public PhoneNumber clone() {
    try {
        return (PhoneNumber) super.clone(); // Object -> PhoneNumber 공변 반환 타이핑
    } catch (ClassNotSupportedException e) {
        throw new AssertionError(); // 절대 일어날 수 없는 일
    }
}
```

2. 가변상태를 참조하는 클래스용 clone 메소드
```java
@Override
public Stack clone() {
    try {
        Stack stack = (Stack) super.clone();
        stack.elements = elements.clone();  // 가변상태 필드를 클론 처리
        return stack;
    } catch (ClassNotSupportedException e) {
        throw new AssertionError(); // 절대 일어날 수 없는 일
    }
}
```

3. 가변필드 clone 메소드 호출로 충분하지 않은 경우
- 아래 예시의 경우 buckets의 경우 별도 배열이 복사가 되지만, 각 버킷 내부에 다음 노드를 가리키는 next가 같은 객체를 참조하게되는 문제가 발생한다.
``` java
public class HashTable implements Cloneable {    
    private Entry[] buckets;
    
    private static class Entry {
        final Object key;
        Object value;
        Entry next;

        public Entry(Object key, Object value, Entry next) {
            this.key = key;
            this.value = value;
            this.next = next;
        }
    }

    @Override
    protected HashTable clone() {
        try {
            HashTable result = (HashTable) super.clone();
            result.buckets = buckets.clone();
            return result;
        } catch (CloneNotSupportedException e) {
            throw new AssertionError();
        }
    }
}
```
- 이 경우 문제를 해결하기 위해 아래와 같이 코드를 수정해야한다.
``` java
public class HashTable implements Cloneable {    
    private Entry[] buckets;
    
    private static class Entry {
        final Object key;
        Object value;
        Entry next;

        public Entry(Object key, Object value, Entry next) {
            this.key = key;
            this.value = value;
            this.next = next;
        }

        // 다음 노드에 대한 복사 재귀 호출
        Entry deepCopy() {
           return new Entry(key, value, next == null? null : next.deepCopy()); 
        }
    }

    @Override
    protected HashTable clone() {
        try {
            HashTable result = (HashTable) super.clone();
            result.buckets = new Entry[buckets.length];
            for (int i = 0; i < buckets.length; i++) {
                if (butckets[i] != null)
                    result.buckets[i] = buckets[i].deepCopy();
            }
            return result;
        } catch (CloneNotSupportedException e) {
            throw new AssertionError();
        }
    }
}
```
4. 고수준 API를 사용하는 방법
- 생성자와 고수준 API (예를들면 HashMap의 put 메소드 등)을 이용해 복사하는 방법이다.
- 이 경우 코드는 우아해지지만, 저수준 API 사용하는 것보다 다소 느리다.
- 또한 Cloneable 아키텍처의 기본인 필드 단위 객체 복사를 우회하는 방식이기 때문에 Cloneable 과 어울리지 않는 방식이기도 하다.

#### \# 주의사항
- 생성자에서는 clone 메소드를 호출하면 안된다. (생성자에서는 재정의 가능한 메소드를 호출하면 안됨)
  - 만약 호출하게 되면 하위 클래스는 복제 과정에서 자신의 상태를 교정할 기회를 잃게 됨
- 재정의한 clone 메소드는 CloneNotSupportedException을 던지면 안된다. 그래야 메소드 사용이 용이해진다.
- 상속용 클래스는 Cloneable을 구현해서는 안된다.
  - 방지하려면 Object에 구현되있는것처럼 정상 동작하는 clone 메소드를 구현 후 protected를 지정한다.
  - 또는 아예 clone 메소드를 퇴화시키는 방법도 있다.
- clone메소드 구현 시 스레드 안전이 필요하다면, 동기화를 처리해줘야한다. Object의 clone메소드는 동기화 처리가 되어있지 않으므로, 필요 시 직접 해야한다.

#### \# 요약
1. Cloneable을 구현한 모든 클래스는 clone메소드를 재정의해야한다.
  - 이 때, 접근제한자는 public, 반환 타입은 클래스 자신으로 지정
2. clone메소드는 가장 먼저 super.clone()을 호출한 후, 가변필드에 대한 복사 처리를 해준다.
  - 필요 시 내부 원소에 대해 깊은 복사(deep copy)도 처리한다.
3. 불변객체의 경우 일반적으로 아무 수정을 할 필요 없지만, 일련번호나 고유 ID등의 경우 수정해준다.

#### \# 복사 생성자 or 복사 팩터리를 이용하는 방법
- 위에서 언급한 상황을 반드시 지켜야하는 상황은 생각보다 많지 않다.
- 그렇지 않은 경우 복사 생성자 혹은 복사 팩터리를 구현하는 것이 더 나은 복사방식일수도 있다.
```java
// 복사 생성자
public Data(Data data) {
    ...
}

// 복사 팩터리
public static Data(Data data) {
    ...
}
```
- 이 방식의 경우 Cloneable이 내재한 맹점들을 회피할 수 있고, 불필요한 검사 예외, 형변환 등을 할 필요도 없다.
- 또한 타입으로 인터페이스 타입을 받을 수 있다.
  - 예컨데, 인자를 Map으로 선언하면 HashSet 객체를 복사하여 TreeSet 타입으로 복제가 가능하다. 