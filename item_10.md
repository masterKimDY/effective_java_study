# 아이템10.  equals는 일반 규약을 지켜 재정의하라

----
### equals를 재정의하지 않는것이 최선인 경우
    - 각 인스턴스가 본질적으로 고유하다. (Thread가 좋은예)
    - 인스턴스 논리적 동치성을 검사할 일이 없다. (Pattern의 인스턴스를 생각해보자)
    - 상위 클래스에서 재정의한 equals가 하위 클래스에도 딱 들어맞는다
    - 클래스가 private이거나 패키지-private이면 equals 메서드를 호출할 일이 없다.

### equals를 재정의해야 할 떄
    - 논리적 동치성을 확인해야 하는데, 상위 클래스의 equals가 논리적 동치성을 비교하도록 재정의되지 않았을때
    - 주로 값 클래스들이 여기에 해당한다.

### equals 메서드를 재정의할때는 일반 규약을 따라야한다 (Object 명세에 적힌 규약)

    - 반사성 : null이 아닌 모든 참조 값 x에 대해 x.equals(x)는 참이다.
    - 대칭성 : null이 아닌 모든 참조 값 x에 대해 x,y에 대해 x.equals(y) == y.equals(x)
    - 추이성 : null이 아닌 모든 참조 값 x에 대해 x,y,z에 대해 x.equals(y) == y.equals(z) == x.equals(z)
    - 일관성 : null이 아닌 모든 참조 값 x에 대해 반복해서 호출해도 항상 동일한 값을 반환한다.
    - null-아님 : on-null reference value x, x.equals(null) should return false.

### equals 구현 절차
* == 연산자를 사용해 입력이 자기 자신의 참조인지 확인한다.
    * 성능 최적화용으로 equals가 복잡할 때 같은 참조를 가진 객체에 대한 비교를 안하기 위함.
* instanceof 연산자로 입력이 올바린 타입인지 확인.
    * 올바르지 않다면 false를 반환.
    * equals중에서는 같은 interface를 구현한 클래스끼리 비교하는 경우도 있음.
* 입력을 올바른 타입으로 형변환 한다.
    * 앞서 instanceof 연산을 수행했기 때문에 100% 성공.
* 입력 객체와 자기 자신의 대응되는 '핵심' 필드들이 모두 일치하는지 하나씩 검사.
    * 모두 일치하면 true, 하나라도 다르면 false를 반환.
    * interface 비교시 필드정보를 가져오는 메서드가 interface에 정의되어있어야하고, 구현체에서 메서드를 재정의 해야함.
* float, double을 제외한 기본타입은 ==을 통해 비교하고 참조(reference) 타입은 equals를 통해 비교.
    * float, double은 Float.compare(float, float)와 Double.compare(double, double)로 비교.
    * 배열의 모든 원소가 핵심 필드라면 Arrays.equals를 사용한다.
* null도 정상값으로 취급하는 참조타입 필드도 있음.
    * Objects.equals(obj, obj)를 이용해 NullPointerException 발생을 예방.
* 최상의 성능을 내기 위해 다음과 같은 방법이 있다.
    * 다를 확률이 높은 필드부터 비교한다.
    * 비교하는 비용이 작은 것을 먼저 수행한다.

### equals를 다 구현했다면 세가지만 자문해보자
    - 대칭적인가? 추이성이있는가 일관적인가?

## 결론
    - 꼭 필요한 경우가 아니면 equals를 재정의 하지 말자
    - 재정의해야 할 때는 그 클래스의 핵심 필드 모두를 빠짐없이, 5가지 규약을 확실히 지켜가며 비교해야한다.
