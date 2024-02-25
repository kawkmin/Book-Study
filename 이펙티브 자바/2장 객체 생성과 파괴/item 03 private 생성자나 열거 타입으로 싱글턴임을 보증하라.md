# 요약

Enum을 열거형으로만 쓰지말고 싱글턴이 필요할 때 제일 먼저 고려해보자

# 주요 내용

싱글턴을 만드는 방식은 크게 3가지 방식이 있다. (싱글턴에 대해서 다루진 않을거임)

### 1. public 필드 방식

미리 static final로 인스턴스를 만든 후 public으로 열고, 생성자를 private로 막아놓는 방식

```java
public class Elvis {

    public static final Elvis INSTANCE = new Elvis(); //public

    private Elvis() { ...}

    public void leaveTheBuilding()
}
```

가장 흔히 사용했던 싱글턴 사용 방식이라 직관적이고 간결함

### 2. 정적 팩터리 방식

1에서 선언한 public 필드도 private로 막고, 정적 팩터리 방식으로만 인스턴스 반환하는 방식

```java
public class Elvis {

    private static final Elvis INSTANCE = new Elvis(); //private

    private Elvis() { ...}

    public static Elvis getInstance() {
        return INSTANCE;
    }

    public void leaveTheBuilding()
}
```

장점

1. 싱글턴이 아니게 바꿀 수 있음
    - 상황에 맞는 다른 인스턴스를 넘기면됨
2. 제네릭 싱글턴 팩터리로 만들 수 있음 (아이템 30)
3. 정적 팩터리의 메서드 참조를 공급자로 사용할 수 있음 (아이템 43, 44)
    - Elvis::getInstance()를 Supplier<Elivis>로 사용

### 위 두 가지 방식의 공통 문제점

1. 사용하지 않더라도 인스턴스가 생성 ⇒ 메모리 낭비
2. 직렬화 할 때, 인스턴스가 하나 더 생기는 문제 발생.
    - 직렬화 스펙상 readResolve 메서드 제공 해야함(아이템 89) 아니면 역직렬화 할 때 새로운 인스턴스 생성됨

### 3. Enum

대부분의 상황에서 원소가 하나뿐인 열거 타입이 **가장 좋은 방법**

```java
public enum Elvis {
    INSTANCE;

    public String getName() {
        return "Elvis";
    }

    public void leaveTheBuilding() { ...}
}

    // Elvis의 인스턴스는 INSTANCE 하나뿐임
    String name = Elvis.INSTANCE.getName();
```

간결하고, 복잡한 직렬화 상황이나 리플렉션 공격에서도 제 2의 인스턴스가 생기는일이 없어짐.

하지만 Enum 이외의 다른 상위 클래스를 상속한다면 사용 불가능.

# 질문 및 추가