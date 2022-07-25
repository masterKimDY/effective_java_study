# 아이템13. clone 재정의는 주의해서 진행하라

----
### Cloneable은 복제해도 되는 클래스임을 명시하는 용도의 믹스인 인터페이스
    - clone 메서드가 선언된 곳은 Object이고 protected접근제한자를 가진다.
    - Cloneable를 구현하는 것만으로는 외부 객체에서 clone 메소드를 호출 할 수 없다.
    - Cloneable메소드가 없는 인터페이스 마커인터페이스 하지만 clone을 호출하기 위해서는 해당 인터페이스가 구현되어야함.

### Object class의 clone method 
`protected native Object clone() throws CloneNotSupportedException;`

### 일반적인 방식의 Clone 메서드 구현
    - Cloneable 을 구현하는 클래스는 clone 메서드가 정확히 동작해야 한다.
    - 일반적인 clone 메서드 규약은 제약이 심하지 않다. Object 클래스의 명세를 살펴보면 다음과 같다.
```
    - x.clone() != x 는 참이여야 한다.
    - x.clone().getClass() == x.getClass() 는 참이여야 한다.
    - x.clone().equals(x) 는 참이여야 한다.
```
- 위의 일반적인 clone 메서드 규약은 기초자료형을 필드로 담고 있을 경우 쉽게 규약을 지킬 수 있다.
- CloneNotSupportedException은 checked 예외을 catch해서 unchecked 예외로 바꿔 던진다.

```java
class Sample implements Cloneable {
    private int foo;
    private String bar;

    public Sample(int foo, String bar) {
        this.foo = foo;
        this.bar = bar;
    }

    @Override public Sample clone() {
        try {
            return (Sample) super.clone();  //형변환 -> 공변타입반환
        } catch (CloneNotSupportedException e) {
            throw new AssertionError(); //일어날수없음.
        }
    }
}
```

### 가변 객체를 참조하는 Class를 복제할때
    - 원본 객체에 아무런 해를 끼치지 않는 동시에 복제된 객체의 불변식을 보장해야한다. (deep copy)


## 결론
    - final 클래스라면 Cloneable을 구현해도 위험지 크지는 않지만, 별다른 문제가 없을때만 드물게 허용해야한다.
    - 기본 원칙은 복제 기능은 생성자와 팩터리를 이용하는게 최고 단 배열은 clone메서드 방식이 가장 깔끔하다.
