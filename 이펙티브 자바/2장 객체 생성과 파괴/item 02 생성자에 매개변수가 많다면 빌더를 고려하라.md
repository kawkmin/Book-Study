# 요약

생성자나 정적 팩터리가 처리해야할 매개변수가 많다면 빌더 패턴을 고려해보자

하지만 책에는 단점이 좀 덜 적혀있는 기분이다. 꼭 써야할 때만 쓰는게 더 좋을 것 같다

# 주요 내용

매개변수에 맞게 생성자를 제공해야할 때, 크게 3가지 패턴으로 제공할 수 있다.

### 1. 점층적 생성자 패턴

말 그대로 상황에 맞게 생성자를 계속 만드는 방식

```java
public class NutritionFacts {

    private final int servingSize;
    private final int servings;
    private final int calories;
    private final int fat;
    private final int sodium;
    private final int carbohydrate;

    public NutritionFacts(int servingSize, int servings) {
        this(servingSize, servings, 0);
    }

    public NutritionFacts(int servingSize, int servings, int calories) {
        this(servingSize, servings, calories, 0);
    }

    public NutritionFacts(int servingSize, int servings, int calories, int fat) {
        this(servingSize, servings, calories, fat, 0);
    }

    public NutritionFacts(int servingSize, int servings, int calories, int fat, int sodium) {
        this(servingSize, servings, calories, fat, sodium, 0);
    }

    public NutritionFacts(int servingSize, int servings, int calories, int fat, int sodium,
        int carbohydrate) {
        this.servingSize = servingSize;
        this.servings = servings;
        this.calories = calories;
        this.fat = fat;
        this.sodium = sodium;
        this.carbohydrate = carbohydrate;
    }
}
```

이런 점층적 생성자 패턴일 때, 특정 생성자를 만들어보자.

```java
public class Main {

    public static void main(String[] args) {
        NutritionFacts cocaCola1 = new NutritionFacts(240, 8, 100, 0, 35, 27);
        NutritionFacts cocaCola2 = new NutritionFacts(240, 8, 100, 0);
    }
}
```

이러면 두 객체가 서로 무슨 차이인지 보이지 않아 가독성이 매우 좋지않고, 실수로 순서가 바뀌면 치명적이다( 컴파일 에러X)

초기 선언하는 것도 점점 장황해진다.

또한 서로 같은 타입이면 특정 생성자를 만들 수 없다 (두 생성자 중 무엇을 사용해야할지 모름)

- 아니면 원치않는 매개변수 받기

```java
    public NutritionFacts(int servingSize,int servings,int calories){
    this(servingSize,servings,calories,0);
    }

// 컴파일 오류 발생!! 원치않는 calories까지 받아야함
public NutritionFacts(int servingSize,int servings,int fat){
    this(servingSize,servings,fat,0);
    }
```

### 2. 자바빈즈 패턴

set을 사용해 생성자를 구현한다.

```java
public class Main {

    public static void main(String[] args) {
        NutritionFacts cocaCola = new NutritionFacts();
        cocaCola.setServingSize(240);
        cocaCola.setServings(8);
        cocaCola.setCalories(100);
        cocaCola.setFat(0);
        cocaCola.setSodium(35);
        cocaCola.setCarbohydrate(27);
    }
}
```

여기에도 문제점이 2가지 있다

1. 객체를 생성하는데 메서드를 여러개 호출해야한다
2. 객체가 완성되기 전까지 일관성이 무너진 상태에 놓이게 된다
    - 일관성이 무너진 상태
        - 디버깅이 어려움
        - Freeze(객체가 완성되기 전까지 사용 불가능)로 해결할 수 있지만 복잡성이 증가한다.

점층적 생성자 패턴의 안전성 + 자바빈즈 패턴의 가독성 ⇒ 빌더 패턴

### 3. 빌더 패턴

Builder를 이용해 일종의 setter처럼 사용하여 매개변수를 초기화하며 build() 메서드를 호출하여 객체를 생성하는 패턴

```java
public class NutritionFacts {

    private final int servingSize;
    private final int servings;
    private final int calories;
    private final int fat;
    private final int sodium;
    private final int carbohydrate;

    private NutritionFacts(Builder builder) {
        servingSize = builder.servingSize;
        servings = builder.servings;
        calories = builder.calories;
        fat = builder.fat;
        sodium = builder.sodium;
        carbohydrate = builder.carbohydrate;
    }

    public static class Builder {

        private final int servingSize;
        private final int servings;
        private int calories = 0;
        private int fat = 0;
        private int sodium = 0;
        private int carbohydrate = 0;

        // 필수 매개변수만을 담은 Builder 생성자
        public Builder(int servingSize, int servings) {
            this.servingSize = servingSize;
            this.servings = servings;
        }

        // 선택 매개변수의 setter, Builder 자신을 반환해 연쇄적으로 호출 가능
        public Builder calories(int val) {
            calories = val;
            return this;
        }

        public Builder fat(int val) {
            fat = val;
            return this;
        }

        public Builder sodium(int val) {
            sodium = val;
            return this;
        }

        public Builder carbohydrate(int val) {
            carbohydrate = val;
            return this;
        }

        // build() 호출로 최종 불변 객체를 얻는다.
        public NutritionFacts build() {
            return new NutritionFacts(this);
        }
    }
}
```

```java
public class Main {

    public static void main(String[] args) {
        NutritionFacts cocaCola = new NutriFacts.Builder(240, 8)
            .calories(100).sodium(35).carbohydrate(30).build();
    }
}
```

이런 패턴으로 가독성도 좋으면서, 상황에 맞는 생성자를 제공할 수 있다.

하지만 Builder를 만들어야해서 생성 비용이 아주 살짝 커지고, 매개변수가 4개 미만일 땐 오히려 복잡할 수 있다 (하지만 클래스의 속성이 추가될 가능성이 크므로 애초에
처음부터 Builder로 하는 것도 고려해보자)

# 질문 및 추가

### 4개 미만일 때 오히려 Builder가 더 복잡해지기 때문에 비추천이라면 Lombok을 이용한 @Builder를 사용하면 추천일까?

```java
public class NutritionFacts {

    private final int servingSize;
    private final int servings;
    private final int calories;
    private final int fat;
    private final int sodium;
    private final int carbohydrate;

    @Builder //빌더 패턴 자동 적용
    private NutritionFacts(Builder builder) {
        servingSize = builder.servingSize;
        servings = builder.servings;
        calories = builder.calories;
        fat = builder.fat;
        sodium = builder.sodium;
        carbohydrate = builder.carbohydrate;
    }

```

[참고 링크](https://multifrontgarden.tistory.com/290)

@Builder로 편하게 만들어도 무변별한 사용 금지인 이유

1. 캡슐화 깨짐 (매개 변수로 속성이 다 보임)
2. 가독성 (생성할 때 개행 많이 잡아먹음)
3. 필수 속성 체크 어려움 (필수 속성을 누락해도 컴파일 오류 발생X)