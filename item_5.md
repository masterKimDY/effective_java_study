# 아이템5.  자원을 직접 명시하지 말고 의존 객체 주입을 사용하라

----

- 사용하는 자원에 따라 동작이 달라지는 클래스에는 정적 유틸리티 클래스나 싱글턴 방식이 적합하지 않다.
- 인스턴스를 생성할때 생성자에 필요한 자원을 넘겨주는 방식이 의존 객체 주입.


##### 정적 유틸리티를 잘못 사용한 예 - 유연하지 않고 테스트하기 어렵다.
```java
public class SpellChecker {
	private static final Lexicon dictionary = ...; //의존성
	private SpellChecker() {} // 객체 생성 방지 

	public static boolean isValid(String word) { ... } 
	public static List<String> suggestions(String typo) { ... } 
}
```

##### 싱글턴을 잘못 사용한 예 - 유연하지 않고 테스트하기 어렵다.
```java
public class SpellChecker { 
	private final Lexicon dictionary = ...; 

	private SpellChecker(...) { } 
	public static SpellChecker INSTANCE = new SpellChecker(...);

	public boolean isValid(String word) { ... } 
	public List<String> suggestions(String typo) { ... }
}
```
 - 위에 두가지 방식은 사전을 단 하나만 사용한다.
 - 직접 명시되어 고정되어있는 변수는 테스트를 하기 힘들다.
 - 사용하는 자원에 따라 동작이 달라지는 클래스에는 정적 유틸리티 클래스나 싱글턴 방식이 적합하지 않음.

## 의존 객체 주입
##### 의존 객체 주입은 유연성과 테스트 용이성을 높여준다.
```java
public class SpellChecker { 
	private final Lexicon dictionary; 

	private SpellChecker(Lexicon dictionary) { 
    		this.dictionary = Objects.requireNonNull(dictionary); 
 	} 

	public boolean isValid(String word) { ... } 
	public List<String> suggestions(String typo) { ... } 
}
```
- 불변을 보장하여 여러 클라이언트가 의존 객체들을 안심하고 공유할 수 있기도 하다.
- 의존 객체 주입은 생성자, 정적팩터리(아이템1), 빌더(아이템2) 모두에 똑같이 응용된다.

---
### 팩터리 메서드 패턴

생성자에 자원 팩터리를 넘겨주는 방식
 - 팩터리란 호출할 때마다 특정 타입의 인스턴스를 반복해서 만들어주는 객체
   
    즉, 팩터리 메서드 패턴을 구현한것

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

##### 한정적 와일드카드 타입을 사용해 팩터리의 타입 매개변수를 제한한다.
```java
Mosaic create(Supplier<? extends Tile> tileFactory) { ... }
```


#### 의존성 주입을 사용하면 `유연성`과 `테스트용이성`을 개선해준다.
```java
public class SpellChecker { 
	private final Lexicon dictionary; 

	private SpellChecker(Supplier<Lexicon> dictionary) { 
    		this.dictionary = Objects.requireNonNull(dictionary); 
 	} 

	public boolean isValid(String word) { ... } 
	public List<String> suggestions(String typo) { ... } 

	public static void main(String[] args) {
		Lexicon lexicon = new KoreanDictionary();
		SpellChecker spellChecker = new SpellChecker(() -> lexicon);
		spellChecker.isValid("good");
	}
}

interface Lexicon{
    
}

class KoreanDictionary implements Lexicon {
    
}
class TestDictionary implements Lexicon {
    
}
```

## 결론
 - 클래스가 내부적으로 하나 이상의 자원에 의존하고, 그 자원이 클래스 동작에 영향을 준다면 싱글턴과 정적 유틸리티 클래스는 사용하지 않는 것이 좋다.  
이 자원들을 클래스가 직접 만들게 해서도 안 된다. 대신 필요한 자원을 (혹은 그 자원을 만들어주는 팩터리를) 생성자에 (혹은 정적 팩터리나 빌더에) 넘겨주자.  
의존 객체 주입이라 하는 이 기법은 클래스의 유연성, 재사용성, 테스트 용이성을 기막히게 개선해준다.
