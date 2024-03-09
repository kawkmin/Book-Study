# item 10 equals는 일반 규약을 지켜 재정의하라

# 요약

꼭 필요하지 않으면 eqauls 재정의는 사용하지 말자. 만약 사용할거라면 5가지 규약을 잘 지키자.

# 주요 내용

주의하지 않는 `Object` `equals`재정의는 심각한 오류가 날 수 있다.

### 재정의를 하지 않는 것이 최선인 경우들

1. **각 인스턴스가 본질적으로 고유할 때.**
    - 값을 표현하는게 아니라 동작하는 개체를 표현하는 클래스 등  (`Thread`등)
2. **인스턴스의 '논리적 동치성'을 검사할 일이 없을 때.**
    - 예를들어 `Pattern`은 `equals`를 재정의하여 각 인스턴스가 같은 정규표현식을 나타내는지 검사할 수 있지만, 그럴 일이 없다면 굳이 재정의할 필요가 없다.
3. **상위 클래스에서 재정의한 equals가 하위 클래스에도 딱 들어맞을 때.**
    - 예를들어 `Set`의 구현체들은 모두 `AbstractSet`이 구현한 `equals`를 상속 받아서 쓴다.
        - 서로 같은 `Set`인지 구분하기 위해서 사이즈, 내부 값들을 비교하면 되기 때문.
4. **클래스가 private이거나 package-private이고 equals 메서드를 호출할 일이 없을 때.**
    - 당연한 것. 하지만 실수로라도 사용을 하고싶지 않으면 재정의로 에러 던지게 구현.

**equals 재정의는 논리적 동치성을 확인할 때만 한다. (Enum, 싱글톤 등 제외)**

이 때 규약을 지키는게 중요하다. 만약 지키지 못한다면 심각한 오류 발생.

### 규약들

1. **반사성 : 자기 자신과 같아야함**
    - x.equals(x) == true
2. **대칭성 : 서로에 대한 동치가 같아야함.**
    - x.equals(y) == true && y.equals(x) == true
    - 다른 객체끼리 연동할 때 자주 발생함 (A class, String 연동 등)
3. **추이성 : 3개 이상도 대칭성 유지**
    - x.equals(y) == true && y.equals(z) == true && x.equals(z) == true
    - 추이성을 지키면서 리스코프 치환 원칙을 지킬려면 **상속 대신 컴포지션을 사용하라(아이템 18)**
        - `.. extend Aclass` 대신 필드에 `private Aclass aclass;` 로 사용하라는 뜻.
4. **일관성 : x.equals(y)를 반복해도 값이 변하지 않는다.**
    - equals는 **항시 메모리에 존재하는 객체만**을 사용한 **결정적 계산을 수행**해야 한다.
5. **null 아님 : null 일 때는 무조건 False**
    - x.equals(null) == false
    - `o==null` 로 체크보단 `instanceOf` 로 묵시적 null 검사가 낫다.
        - `instanceof`는 두번째 피연산자와 무관하게 첫번째 피연산자가 null이면 false를 반환하기 때문이다.

**+. hashCode도 재정의 하자 (아이템 11)**

### equals 좋은 예시

```java
@Override
public boolean equals(final Object o){
    // 1. == 연산자를 사용해 입력이 자기 자신의 참조인지 확인한다.
    if(this==o){
    return true;
    }

    // 2. instanceof 연산자로 입력이 올바른 타입인지 확인한다.
    if(!(o instanceof Point)){
    return false;
    }

// 3. 입력을 올바른 타입으로 형변환 한다.
final Piece piece=(Piece)o;

    // 4. 입력 개체와 자기 자신의 대응되는 핵심 필드들이 모두 일치하는지 하나씩 검사한다.

    // float와 double을 제외한 기본 타입 필드는 ==를 사용한다.
    return this.x==p.x&&this.y==p.y;

    // 필드가 참조 차입이라면 equals를 사용한다.
    return str1.equals(str2);

    // null 값을 정상 값으로 취급한다면 Objects.equals로 NullPointException을 예방하자.
    return Objects.equals(Object,Object);
    }
```

# 질문 및 추가