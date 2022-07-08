# 아이템4.  인스턴스화를 막으려거든 private 생성자를 사용하라

----
### 유틸리티 클래스
    - 정적 메소드와 정적 필드만을 담은 클래스
    - 유틸리티 클래스는 인스턴스로 만들어 쓰려고 설계한게 아니다. 

```java
public final class UtilityClass {
    private UtilityClass() {
        //인스턴스화 방지용
        throw new UnsupportedOperationException("인스턴스 생성불가!");
    }

    public static void foo() {
        System.out.println("UtilityClass foo");
    }
}
```

## 결론
    - 유틸리티 클래스 생성시에는 private 생성자를 명시적으로 만들어 인스턴스를 생성불가하게 한다.
    - class에도 final 키워드가 없어도 생성자가 private라 상속불가하다. 명시적으로 붙여주는것도 좋을듯하다.
