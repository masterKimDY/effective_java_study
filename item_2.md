# 아이템2.  생성자에 매개변수가 많다면 빌더를 고려하라

----
### 점층적 생성자 패턴
    - 필수 매개변수만 받는 생성자, 필수 매개변수 + 선택 1,2,3,4..... 형태로 생성자를 늘려가는 방식
    - 선택 매개변수가 많아지면 그만큼 생성자도 추가해야함.
    - 매개변수가 많아지면 클라이언트 코드를 작성하거나 읽기 어려움.

### 자바빈즈패턴
    - 참고 : https://docstore.mik.ua/orelly/java-ent/jnut/ch06_02.htm
    - 객체 하나를 만들려면 메서드를 여러 개 호출해야하고, 객체가 완전히 생성되기 전까지는 일관성이 깨진다.
    - 불변으로 만들수도 없음 setter로 수정가능

### 빌더패턴
    - 필수 매개변수만으로 생성자 (아니면 정적 팩토리 메소드)를 호출해 빌더 객체를 얻는다.
      그런다음 빌더 객체가 제공하는 setter 메소드들로 원하는 선택 매개변수들을 셋팅한다.
      마지막으로 매개변수가 없는 build 메소드를 호출해 불변객체를 얻는다.
    - 롬복 빌더 참고 : https://projectlombok.org/features/Builder
    - 롬복 실험실에 있는 fluent API for getters and setters. 
        참고 : https://projectlombok.org/features/experimental/Accessors

### 계층적으로 설계된 클래스와 빌더패턴
    - 각 계층의 클래스에 관련 빌더를 멤버로 정의하자
    - 추상 클래스는 추상 빌더를 구체 클래스는 구체 빌더를 갖게 한다.

```java
public abstract class Pizza{
   public enum Topping { HAM, MUSHROOM, ONION, PEPPER, SAUSAGE }
   final Set<Topping> toppings;
   
   abstract static class Builder<T extends Builder<T>> {
      EnumSet<Topping> toppings = EnumSet.noneOf(Topping.class);
      public T addTopping(Topping topping) {
         toppings.add(Objects.requireNonNull(topping));
         return self();
      }

      abstract Pizza build();

      // 하위 클래스는 이 메서드를 overriding하여 this를 반환하도록 해야 한다.
      protected abstract T self();
   }

   Pizza(Builder<?> builder) {
      toppings = builder.toppings.clone();
   }
}

public class NyPizza extends Pizza {
   public enum Size { SMALL, MEDIUM, LARGE }
   private final Size size;

   public static class Builder extends Pizza.Builder<Builder> {
      private final Size size;

      public Builder(Size size) {
         this.size = Objects.requireNonNull(size);
      }

      @Override public NyPizza build() {
         return new NyPizza(this);
      }

      @Override protected Builder self() { return this; }
   }

   private NyPizza(Builder builder) {
      super(builder);
      size = builder.size;
   }
}

public class Calzone extends Pizza {
   private final boolean sauceInside;

   public static class Builder extends Pizza.Builder<Builder> {
      private boolean sauceInside = false;

      public Builder sauceInside() {
         sauceInside = true;
         return this;
      }

      @Override public Calzone build() {
         return new Calzone(this);
      }

      @Override protected Builder self() { return this; }
   }

   private Calzone(Builder builder) {
      super(builder);
      sauceInside = builder.sauceInside;
   }
}
```

`Pizza.Builder` 클래스는 재귀적 타입 한정을 이용하는 제네릭 타입이다. **여기에 추상 메서드인 `self`를 더해 하위 클래스에서는 형변환 하지 않고도 메서드 연쇄를 지원할 수 있다.** self 타입이 없는 자바를 위한 이 우회 방법을 시뮬레이트한 셀프 타입(simulated self-type) 관용구라 한다.

`Pizza`의 각 하위 클래스 빌더가 정의한 `build`메서드는 해당하는 구체 하위 클래스를 반환하도록 선언한다.  
`NyPizza.Builer`는 `NyPizza`를 반환하고, `Calzone.Builder`는 `Calzone`를 반환한다.  
하위 클래스 메서드가 상위 클래스의 메서드가 정의한 반환 타입(`Pizza`)이 아닌, 그 하위 타입을 반환하는 기능을 공변 반환 타이핑이라고 한다.  
이 기능을 이용하면 클라이언트가 형변환에 신경 쓰지 않고도 빌더를 사용할 수 있다.

##### 계층적 빌더를 사용하는 클라이언트 코드

```java
public class Main {
    public static void main(String[] args) {
        NYPizza pizza = new NYPizza.Builder(SMALL)
                .addTopping(SAUSAGE)
                .addTopping(ONION)
                .build();

        Calzone calzone = new Calzone.Builder()
                .addTopping(HAM)
                .sauceInside()
                .build();
    }
}
```
빌더를 이용하면 가변인수 매개변수를 여러 개 사용할 수 있다. `addTopping` 메서드가 이렇게 구현된 예다.  
빌더 하나로 여러 객체를 순회하면서 만들 수 있고, 빌더에 넘기는 매개변수에 따라 다른 객체를 만들 수도 있다.

### 빌더 패턴의 단점

**성능** 객체를 만들려면 그에 앞서 빌더부터 만들어야 한다. 물론 빌더 생성 비용이 크진 않지만, 성능에 민감한 상황에서는 문제가 될 수 있다.

**장황** 점층적 생성자 패턴보다는 코드가 장황해 매개변수가 4개 이상은 되어야 값어치를 한다.

&nbsp;

## Lombok @Builder

lombok으로 @Builder 애노테이션을 붙이면 Builder 패턴을 생성해준다.

```java
@Builder
public class NutritionFacts {
    private final int servingSize;
    private final int servings;
    private final int calories;
    private final int fat;
    private final int sodium;
    private final int carbohydrate;
	
		public static void main(String[] args) {
			NutritionFacts.builder()
								.servingSize()
								.servings()
								.calories()
								.fat()
								.sodium()
								.carbohydrate()
								.build();
		}
}
```

(기본 생성자는 만들어주지 않음)

setter없이 필요한 매개변수 값을 set한 후에 build하여 thread-safe하게 사용할 수 있다.


## 결론

**생성자나 정적 팩터리가 처리해야 할 매개변수가 많다변 빌더 패턴을 선택하는 게 더 낫다.**

매개변수 중 다수가 필수가 아니거나 같은 타입이면 특히 더 그렇다.

빌더는 점층적 생성자보다 클라이언트 코드를 읽고 쓰기가 훨씬 간결하고, 자바빈즈보다 훨씬 안전하다.