# 요약

무작정 public 생성자를 제공하는 습관을 고치고, 정적 팩터리 메서드의 장단점을 고려하여 생성자를 골라서 사용하자.

# 주요 내용

클래스의 인스턴스를 생성하는 대표적인 방식은 `생성자`를 만드는 것이다.

하지만 `정적 팩터리 메서드` 를 통해서 생성자 역할 대신하는 커스텀하는 방식으로, 여러 장점을 가질 수 있다.

```java
// 정적 팩터리 메서드 예시
public class Example {

    int a, b;

    // 생성자
    public Example(int a, int b) {
        this.a = a;
        this.b = b;
    }

    // 정적 팩터리 메서드
    public static Example create(int a, int b) {
        return new Example(a, b);
    }
}

```

## 정적 팩터리 메서드의 장단점

## 장점 5가지.

### 1. 이름 커스텀 가능

- 더욱 직관적이고 특성 묘사 가능.

```java
    public Example(int a,int b){
    this.a=a;
    this.b=b;
    }

// 명확한 메서드 명 
public static Example withAandB(int a,int b){
    return new Example(a,b);
    }
```

### 2. 호출될 때 마다 인스턴스를 새로 생성안해도 된다.

- 인스턴스를 미리 만들어 놓거나 캐싱하여 재활용하는 식으로 불필요한 객체 생성 피함

  ⇒ 메모리 절약으로 성능 향상 (생성비용이 클수록 더욱 차이가 남)

```java
// java.lang.Boolean 
public final class Boolean implements java.io.Serializable,
    Comparable<Boolean>, Constable {

    public static final Boolean TRUE = new Boolean(true);
    public static final Boolean FALSE = new Boolean(false);
    //...

    // 미리 만들어진 Boolean 반환
    public static Boolean valueOf(boolean b) {
        return (b ? TRUE : FALSE);
    }
}
```

- 이렇게 언제 어느 인스턴스를 살아 있게 할지를 통제하는 클래스를 `인스턴스 통제 클래스`라고 함
    - 장점 3가지
        - 싱글톤 가능 (아이템3)
        - 인스턴스화 불가로 만들기 (아이템4)
        - 불편 값 클래스에서 동치인 인스턴스가 하나뿐임을 보장 (아이템17)
            - (a == b일 때만 a.equlas(b)가 성립 등)

### 3. 반환 타입의 하위 타입 객체를 반환 가능

- 조건 등에 따라 반환할 객체의 클래스를 자유롭게 선택하는 **엄청난 유연성** 제공

  ⇒ 구현 클래스를 **공개하지 않고도**, 그 객체가 반환 가능하여 **API 작게 유지** 가능

    - `Collection`프레임워크는 총 45개 유틸리티 구현체를 제공하는데, 클래스를 안만들고 얻기 가능. ( 자바 컴파일러에 미리 정의되어 있기 때문 )
        - 반환되는 클래스가 어떤건지 알 필요 없이 그냥`of()`메서드의 기능이 무엇인지만 알고,`List.of()`의 형태로 사용하면 된다. (**API 사용 난이도
          낮춤**)

    ```java
    // java.util.Collection
    // 자바 8 이후 interface에 정적 메서드 가질 수 있게 되었음 
    public interface List<E> extends Collection<E> {
        static <E> List<E> of() {
            return (List<E>) ImmutableCollections.ListN.EMPTY_LIST;
        }
    }
    ```

### 4. 입력 매개변수에 따라 매번 다른 클래스의 객체를 반환할 수 있다

- 장점 3의 연장선

### 5. 정적 팩토리 메서드 작성 시점에 반환할 객체의 클래스가 없어도 됨

- 장점 3, 4와 비슷한 개념. 언제든 구현체를 바꿔 끼울 수 있음

## 단점 2가지

### 1. 생성자 없이 정적 팩토리 메서드만 있으면 하위 클래스 만들 수 없음

- 상속을 하려면 무조건 public protected 생성자가 필요함
- 상속보단 컴포지션 사용(아이템 18)하도록 유도하고, 불변 타입(아이템 17)로 만들려면 이 제약을 지켜야 하기 때문에 오히려 장점이 될 수 있음.

### 2. 개발자가 찾기 불편할 수 있음

- javadoc로 모아볼 수 없음

## 정적 팩터리 메서드 추천 컨벤션

- `from` : 하나의 매개 변수를 받아서 객체를 생성
    - `Date date = Date.from(instant);`
- `of` : 여러개의 매개 변수를 받아서 객체를 생성
- `valueOf`: from과 of의 더 자세한 버전
    - `BigInteger prime = BigInteger.valueOf(Integer.MAX_VALUE);`
- `getInstance`| `instance`: 인스턴스를 생성. 이전에 반환했던 것과 같을 수 있음.
- `newInstance`| `create`: 새로운 인스턴스를 생성
- `get+[OtherTypeName]` : 다른 타입의 인스턴스를 생성. 이전에 반환했던 것과 같을 수 있음.
- `new+[OtherTypeName]` : 다른 타입의 새로운 인스턴스를 생성.

# 질문 및 추가