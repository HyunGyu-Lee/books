# 생성자에 매개변수가 많다면 빌더를 고려하라
#### \#1. 빌더를 사용하는 이유
- 정적 팩토리 메서드에는 public 생성자와 같이, 매개변수가 많은 경우 적절히 대응하기 어렵다는 단점이 있다.
- 기존에는 필수 매개변수 / 선택적 매개변수를 받는 생성자를 늘려가며 작성하는 **점층적 생성자 패턴**이 주로 사용됐다.
- 하지만 점층적 생성자 패턴도 결국은 원치않는 매개변수가 포함될 가능성이 높고, 이런 코드를 사용하는 클라이언트 측 코드가 작성하기 어렵고 읽기 어려워진다.
- 두 번째는 객체를 생성하고 Setter로 값을 설정하는 **자바빈즈 패턴**이다.
- 이 방법은 **점층적 생성자 패턴**에 비해 읽기는 쉽지만 객체 하나 생성을 위해 클라이언트가 여러 메소드를 호출해야하고, 값 설정이 완료되기 전까지는 일관성이 무너진 상태에 놓이게 된다.
- 또한 불변성이 깨지고 스레드 안정성을 위해 추가 작업이 필요하다.

#### \#2. 점층적 생성자 패턴의 안정성과 자바빈즈 패턴의 가독성을 겸비한 빌더 패턴
- 빌더 패턴은 클라이언트가 객체를 직접 만드는 대신, 필수 매개변수만으로 생성자(혹은 정적 팩토리 메소드)를 호출해 빌더 객체를 얻는다.
- 이후 선택적 매개변수를 설정해주고 마지막으로 build 메소드를 호출하여 필요한 객체를 얻어낸다.
- 빌더는 생성할 클래스 안에 정적 멤버 클래스로 만들어두는게 일반적이다.
- 빌더 패턴 예시
```java
public class NutritionFacts {
    private final int servingSize;
    private final int servings;
    private final int calories;
    private final int fat;

    public static class Builder {
        // 필수 매개변수
        private final int servingSize;
        private final int servings;

        // 선택 매개변수 (default 값 초기화)
        private int calories = 0;
        private int fat = 0;

        public Builder(int servingSize, int servings) {
            this.servingSize = servingSize;
            this.servings = servings;
        }

        public Builder calories(int val) {
            this.calories = val;
            return this;
        }

        public Builder fat(int val) {
            this.fat = val;
            return this;
        }        

        public NutritionFacts build() {
            return new NutritionFacts(this);
        }
    }

    private NutritionFacts(Builder builder) {
        this.servingSize = builder.servingSize;
        this.servings = builder.servings;
        this.calories = builder.calories;
        this.fat = builder.fat;
    }
}
```
- 빌더 패턴에서 입력값의 유효성 검사 후 IllegalArgumentException을 사용해 오류를 발생시켜 주면 된다.

#### \#3. 빌더패턴과 계층 설계 클래스
- 빌더 패턴은 계층적으로 설계된 클래스와 함께 쓰기 좋다
- 추상 클래스는 추상 빌더를, 구체 클래스(concrete class)는 구체 빌더를 갖게 한다.
- 추상 클래스 예시
- 아래 예시에서 **self 추상 메소드**는 셀프 타입 관용구(simulated self-type)라 함
```java
public abstract class Pizza {
    public enum Topping { HAM, MUSHROOM, ONION }
    final Set<Topping> toppings;

    abstract static class Builder<T extends Builder<T>> {
        EnumSet<Topping> toppings = EnumSet.noneOf(Topping.class);

        // addTopping 메소드에 형변환 없이 메소드 체이닝을 지원하기 위해 self 이용
        public T addTopping(Topping topping) {
            toppings.add(topping);
            return self();
        }

        abstract Pizza build();

        public abstract T self();
    }

    Pizza(Builder<?> builder) {
        this.toppings = builder.toppings.clone();
    }
}
```
- 하위 클래스 예시
- build 메소드 재정의 시 **자신의 타입을 반환(공변 반환 타이핑)**을 통해 클라이언트 코드에서 형변환을 하지 않아도 되게함
```java
public class NyPizza extends Pizza {
    public enum Size { SMALL, MEDIUM, LARGE }
    private final Size size;
    
    public static class Builder extends Pizza.Builder<Builder> {
        private final Size size;

        public Builder(Size size) {
            this.size = Objects.requireNonNull(size);
        }

        // 하위 클래스의 build 메소드에서 상위 클래스의 정의대신 하위 타입을 반환하는 것을 공변 반환 타이핑이라 함
        @Override
        public NyPizza build() {
            return new NyPizza(this);
        }

        // self 메소드 구현
        @Override
        public Builder self() {
            return this;
        }
    }

    NyPizza(Builder builder) {
        super(builder);
        this.size = builder.size;
    }
}
```

#### \# 결론
- 생성자나 정적 팩토리 메소드가 처리해야할 매개변수가 많다면 빌더 패턴을 선택하는게 낫다.
- 매개변수 중 다수가 필수가 아니거나 같은 타입이면 특히 그렇다.
- 빌더는 점층적 생성자보다 클라이언트 코드를 읽고 쓰기가 훨씬 간편하다.
- 빌더는 자바빈즈보다 훨씬 안전하다.