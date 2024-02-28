# item 07 다 쓴 객체 참조를 해제하라

# 요약

메모리 누수는 겉으로 잘 보이지도 않는, 아무도 모르는 사이에 성능을 낮추는 원인이다. 그래서 애초에 코드를 작성할 때 부터 메모리 누수가 일어나지 않도록 잘 짜는 것이 중요하다.

# 주요 내용

객체를 참조하고 있으면 GC에서 회수를 하지 못해 메모리를 계속써야한다.

하지만 앞으로 계속 안쓸 객체를 계속 참고하고있으면?? **메모리 누수**가 발생한다.

이를 방지하기 위해 다 쓴 객체 참조는 해제해야한다.

여러 예시를 통해 알아보자.

### 1. 메모리 직접 관리하는 클래스

다음은 객체를 저장하는 Stack을 구현한 것이다.

```java
public class Stack {

    private Object[] elements;
    private int size = 0;
    private static final int DEFAULT_INITIAL_CAPACITY = 16;

    public Stack() {
        elements = new Object[DEFAULT_INITIAL_CAPACITY];
    }

    public void push(Object e) {
        ensureCapacity();
        elements[size++] = e;
    }

    public Object pop() {
        if (size == 0)
            throw new EmptyStackException();
        return elements[--size];
    }

    /**
     * 원소를 위한 공간을 적어도 하나 이상 확보한다.
     * 배열 크기를 늘려야 할 때마다 대략 두 배씩 늘린다.
     */
    private void ensureCapacity() {
        if (elements.length == size)
            elements = Arrays.copyOf(elements, 2 * size + 1);
    }

}
```

여기서 pop() 부분을 살펴보면 `elements[—size]` 로 배열의 가르키는 size(index)만 줄이지 객체를 참조 해제한 것이 아니라, 계속해서 누수가 일어난다.

따라서 다음과 같이 변경하면, 메모리 누수 방지!

```java
public Object pop(){
    if(size==0)
    throw new EmptyStackException();
    Object result=elements[--size];
    elements[size]=null; // 다 쓴 참조 해제
    return result;
    }
```

**‘ 자기 메모리를 직접 관리하는 클래스라면 메모리 누수에 주의해야 한다’**

### 2. 캐시

객체 참조를 캐시에 넣고, 객체를 다 쓴 뒤에도 한참 놔두면 그만큼 메모리가 누수됨

```java
Object key1=new Object();
    Object value1=new Object();

    Map<Object, List> cache=new HashMap<>();
    cache.put(key1,value1);
```

만약 key1이 사라지면 cache에 저장된 것이 의미가 없어지고, 메모리가 누수된다.

캐시 외부에서 키를 참조하는 동안만 엔트리가 살아 있는 캐시가 필요하면 `WeakHashMap` 을 사용하면 다 쓴 엔트리가 자동으로 삭제되어 메모리 누수 방지 가능. (
WeakReference)

하지만 대부분 엔트리 유효 기간을 다 알고 캐시를 만들지 않음 ⇒ 엔트리 가치 떨어뜨리고 청소하는 방식? 으로 사용하기도 함

### 3. 콜백

클라이언트가 콜백을 등록만 하고 해지하지 않으면, 콜백이 계속 쌓여감.

⇒ 콜백을 `weakHashMap` 로 저장하면 GC가 즉시 수거해감!

# 질문 및 추가