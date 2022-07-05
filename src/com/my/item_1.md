# 아이템 1 생성자 대신 정적 팩터리 메소드를 고려하라

----

* 클래스의 인스턴스 생성하는 방법
 ### 생성자와 별도로 정적 팩토리 메서드를 제공할 수 있다.
``` java
    Boolean foo = new Boolean(false);   //생성자를 통한 인스턴스 생성
    Boolean bar = Boolean.valueOf(false);   //정적 팩토리 메소드
```

### 5가지 장점
    1. 이름을 가질 수 있다.
        - 메소드 시그니처 만으로는 반환될 객체의 특성을 제대로 설정하지 못한다.
        - 메소드 시그니처가 같은 생성자가 여러개 필요할 경우 정적 팩터리 메소드에 이름을 지어주자.

    2. 호출될 때마다 인스턴스를 새로 생성하지는 않아도 된다.
        - 플라이웨이트 패턴도 이와 비슷한 기법 (인스턴스 수를 제한 할 수있는가)
        - 인스턴스 통제 클래스
        - 열거타입
```java 
    public static Integer valueOf(int i) {
        if (i >= IntegerCache.low && i <= IntegerCache.high)
            return IntegerCache.cache[i + (-IntegerCache.low)];
        return new Integer(i);
    }
```


    3. 반환 타입의 하위타입 객체를 반환할 수 있는 능력이 있다.
        반환할 객체의 클래스를 자유롭게 선택할 수 있는 유연성을 가진다. 이는 인터페이스를 정적 팩터리 메서드의 반환 타입으로 사용하는 인터페이스 기반 프레임워크를 만드는 핵심 기술이기도 하다.

``` java
    public static <E extends Enum<E>> EnumSet<E> noneOf(Class<E> elementType) {
        Enum<?>[] universe = getUniverse(elementType);
        if (universe == null)
            throw new ClassCastException(elementType + " not an enum");

        if (universe.length <= 64)
            return new RegularEnumSet<>(elementType, universe);
        else
            return new JumboEnumSet<>(elementType, universe);
    } 
```

    4. 입력 매개변수에 따라 매번 다른 클래스의 객체를 반환할 수 있다.
        - 참고 : https://siyoon210.tistory.com/152

    5. 정적 팩터리 메소드를 작성하는 시점에는 반환할 객체의 클래스가 존재하지 않아도 된다.
        - 대표적인 서비스제공자 프레임워크 JDBC