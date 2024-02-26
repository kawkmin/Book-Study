# 요약

DI를 사용해야하는 이유 환기

# 주요 내용

정적 유틸리티(아이템 4)나 싱글턴(아이템 3) 클래스를 만들 때, 필드에 다른 객체를 의존하면 모듈간의 결합도가 커져 유지보수가 힘들고(유연하지 않음), 테스트도 힘듬

```java
// 정적 유틸리티 클래스
public class SpellChecker {

    private static final Lexicon dictionary = ...;

    private SpellChecker() {
    }    // 객체 생성 방지

    public static boolean isValid(String word) { ...}

    public static List<String> suggestions(String type) { ...}
}
```

```java
// 싱글턴 클래스
public class SpellChecker {

    private static final Lexicon dictionary = ...;

    private SpellChecker(...) {
    }

    public static SpellChecker INSTANCE = new SpellChecker(...);

    public boolean isValid(String word) { ...}

    public List<String> suggestions(String type) { ...}
}
```

- 만약 여기서 다른 dictionary를 사용해야 한다면? 필드를 수정해야함 (OCP 원칙 위반)

따라서 의존 객체 주입으로 유연하고 테스트성이 좋은 설계를 하자 (+ 불변 보장)

```java
public class SpellChecker {

    private final Dictionary dictionary;

    // 생성자를 이용해 외부로부터 자원(객체) 주입
    // 외부의 자원에 관계없이 객체를 생성할 수 있다.
    public SpellChecker(Dictionary dictionary) {
        this.dictionary = Objects.requireNonNull(dictionary);
    }

    public boolean isValid(String word) { ...}

    public List<String> suggestions(String type) { ...}
}
```

이런 방식으로 팩터리 메서드 패턴(GOF)과 DI 패턴이 있고, Spring에서는 아주 편하게 DI를 해준다

# 질문 및 추가