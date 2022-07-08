# 아이템3.  private 생성자나 열거 타입으로 싱글턴임을 보증하라

----
### 싱글턴 (singleton) 
    - 인스턴스를 오직하나만 생성 할 수있는 클래스
    - 전형적인 예로 함수와 같은 무상태 객체나 설계상 유일해야 하는 시스템 컴포넌트

### 클래스를 싱글턴으로 만들면 이를 사용하는 클라이언트를 테스트하기가 어려워 질 수 있다.
    - 타입을 인터페스로 정의한 다음 그 인터페이스를 구현해서 만든 싱글턴이 아니라면 싱글턴 인스턴스를 mock객체로 대체 불가.

### public static final 필드 방식의 싱글턴
``` java
public class Elvis {
    //클라이언트에서는 여기로만 접근가능 최초에 1번만 호출됨
    public static final Elvis INSTANCE = new Elvis();
    
    private Elvis() {
        //생성자를 감춘다. 하지만 리플렉션을 이용해서 호출은 가능
    }

    public void leaveTheBuilding() {        
    }
}
```
#### 장점
    - 해당 클래스가 싱글턴임이 API에 드러난다.
    - public static 필드가 final이니 절대로 다른 객체를 참조불가.
    - 간결하다.

### static factory method (정적팩토리메소드) 방식의 싱글턴
``` java
public class Elvis {
    private static final Elvis INSTANCE = new Elvis();  //감춘다 getInstance()로만 접근가능 
   
    private Elvis() {
        //생성자를 감춘다. 하지만 리플렉션을 이용해서 호출은 가능
    }
    
    public static Elvis getInstance() {
        return INSTANCE;    //  항상 같은 객체의 참조를 반환
    } 

    public void leaveTheBuilding() {        
    }
}
```

#### 장점
    - API를 바꾸지 않고도 싱글턴이 아니게 변경할 수 있다는점 예)스레드별로 다른 인스턴스를 반환
    - 원한다면 정적 팩터리를 제네릭 싱글턴팩토리로 만들수있다.
    - 정적 팩토리의 메소드 참조를 공급자(supplier)로 사용할 수 있다.
    - 위 장점이 필요없으면 public static final 필드 방식이 낫다.
  
#### Supplier는 인자를 받지 않고 Type T 객체를 리턴하는 함수형 인터페이스
``` java
@FunctionalInterface
public interface Supplier<T> {

    /**
     * Gets a result.
     *
     * @return a result
     */
    T get();
}
```

### 아이템 87에서 다시 알아보자 
### 직렬화된 인스턴스를 역직렬화 할때 새로운 인스턴스가 만들어진다?
    - 위 2가지 방식으로 만든 싱글턴 클래스를 직렬화하려면 Serializable 인터페이스를 선언하는 것으로는 부족하다.
    - 모든 인스턴스 필드를 transient 선언한다.
    - readResolve 메소드를 추가한다.


### enum으로 싱글턴 만드는 방법
```java
public enum Elvis {
    INSTANCE;
    
    public void leaveTheBuilding() {} 
}
```

#### 장점
    - public 필드 방식과 비슷하지만 더 간결하고 추가 노력없이 직렬화할 수 있다.
    - 복잡한 직렬화 상황이나 리플렉션 공격에서도 제2의 인스턴스가 생기는 일을 완벽히 막아준다.
    - 상속이 필요한 경우는 사용불가.



## 결론
    1. 생성자만를 private로 만들고 외부에서 인스턴스 생성을 차단한다. (리플렉션으로 생성하는경우도 고려한다.)
    2. 외부에서 접근할때는 아래 2가지 방법을 고려한다. 
       public static final  
       static final method
    - 위 2가지 방법은 직렬화된 인스턴스를 역직렬화할때 마다 새로운 인스턴스가 만들어질수있다. 
    - enum을 활용하면 직렬화 상황이나 리플렉션 공격에도 안전하다? 


### 싱글턴패턴 참고 
참고: https://readystory.tistory.com/116
