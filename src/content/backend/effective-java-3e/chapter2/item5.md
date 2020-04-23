# 자원을 직접 명시하지 말고 의존 객체 주입을 사용하라
- 대부분의 클래스는 하나 이상의 자원에 의존한다.
- 예를들어 맞춤법 검사기의 경우 사전에 의존한다.
- 뭔가 자원을 의존하는데 정적 유틸리티나 싱글턴을 사용하여 의존하는 자원을 클래스 내부에 명시하면 유연하지 않고 테스트하기 어렵다.
```java
public class SpellChecker {
    // 자원을 클래스 내부에 직접 명시
    private static final Lexicon dictionary = ...;

    private SpellChecker() {}

    public static boolean isValid(String word) { ... }

    public static List<String> suggestions(String type) { ... }
}
```
- 이러한 방식은 사전을 단 1개만 사용한다고 가정하는거나 다름없어 유연성이 심각하게 저하된다.
- `SpellChecker`가 여러 사전을 사용할 수 있도록 변경하는건 간단하다.
- 단순히 사전을 변경하는 메소드를 추가할 수 도 있지만, 이런 방식은 오류를 내기 쉬우며 멀티 스레드 환경에서 사용할 수 없다.
- 위 이야기들을 종합해보면, **사용하는 자원에 따라 동작이 달라지는 클래스에는 정적 유틸리티나 싱글턴 방식은 적합하지 않다.**
- 이러한 경우에 적합한 방식은 `인스턴스 생성 시 자원을 넘겨주는 방식`이다. (의존 객체 주입의 한 방식)
```java
public class SpellChecker {
    private final Lexicon dictionary;

    public SpellChecker(Lexicon dictionary) {
        this.dictionary = Objects.requireNonNull(dictionary);
    }

    ...
}
```
- 위 방식은 불변을 보장하여 클라이언트가 의존 객체들을 안심하고 공유할 수 있다.
- 이 방식의 응용으로 `Supplier<T>`를 활용해 자원을 생성하는 팩터리를 넘겨줄 수도 있다.
- 의존 객체 주입이 유연성과 테스트를 용이하게 해주지만, 의존성이 많아지면 코드가 복잡해질 수 있다.
  - 이 경우엔 [`Dagger`](https://dagger.dev/users-guide), [`Guice`](https://github.com/google/guice), `Spring` 등 DI 프레임워크를 활용하면 복잡성을 줄일 수 있다.

## 정리
- 클래스가 내부적으로 하나 이상의 자원에 의존하고, 그 자원이 클래스 동작에 영향을 준다면 싱글턴, 정적 유틸리티 클래스 방식은 사용하지 않는것이 좋다.