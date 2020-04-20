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