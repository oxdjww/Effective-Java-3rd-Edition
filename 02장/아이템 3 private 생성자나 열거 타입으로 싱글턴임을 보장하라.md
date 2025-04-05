# 아이템 3. private 생성자나 열거 타입으로 싱글턴임을 보장하라.

## 싱글턴이란?

**인스턴스를 오직 하나**만 생성할 수 있는 클래스를 의미한다.

Ex) `stateless` 객체 or 설계상 유일해야 하는 시스템 컴포넌트

단점

- 이를 사용하는 클라이언트를 테스트하기가 어려워질 수 있다.

→ 싱글턴 인스턴스를 가짜(mock) 구현으로 대체할 수 없기 때문이다.

## 싱글턴의 생성 방식

총 3가지가 존재하는데 보통 1, 2번을 많이 사용한다고 한다.

### 1. private 생성자 + public static final 필드(field)의 사용

```java
public class Elvis {
    public static final Elvis INSTANCE = new Elvis();

    private Elvis() { }

    public void leaveTheBuilding() {
        System.out.println("Whoa baby, I'm outta here!");
    }
}
```

```java
public static void main(String[] args) {
		Elvis elvis = Elvis.INSTANCE;
		elvis.leaveTheBuilding();
}
```

이 경우에 `private` 생성자는 `public static final`필드를 초기화할 때 딱 한 번만 호출된다.

<aside>
💡

이렇게 함으로서 클라이언트가 인스턴스 객체의 수를 조정할 수 없게 된다!

</aside>

**장점**

- 해당 클래스가 싱글턴임이 API 명백히 드러난다.
- 간결하다.

### 2. private 생성자 + public static 정적 팩터리 메서드

```java
public class Elvis {
    private static final Elvis INSTANCE = new Elvis();
    private Elvis() { }
    public static Elvis getInstance() { return INSTANCE; }

    public void leaveTheBuilding() {
        System.out.println("Whoa baby, I'm outta here!");
    }
}
```

```java
public static void main(String[] args) {
    Elvis elvis = Elvis.getInstance();
    elvis.leaveTheBuilding();
}
```

1번 케이스와 다르게 `private`생성자를 호출하는 `field`를 `private`로 지정했다.

대신 `public static` 정적 팩터리 메서드(`getInstance()`)를 제공하여 항상 같은 객체의 참조를 반환시킨다.

**장점**

- (마음이 바뀌면) API를 그대로 둔 채로 **싱글턴이 아니게 변경**할 수 있다.

> Ex. 호출하는 스레드별로 다른 인스턴스를 넘겨준다던가 하는 방식.
> 
- 정적 팩터리를 **제너릭 싱글턴 팩터리**로 만들 수 있다.(아이템 30)
- 정적 팩터리의 **메서드 참조를 공급자(supplier)로 사용**할 수 있다.

> `Elvis::getInstance` 대신 `Supplier<Elvis>`로 사용하는 방식(아이템 43, 44)
> 

<aside>
💡

공급자는 나중에 아이템 43, 44를 통해서 알아보도록 하자.

</aside>

위의 장점이 필요하지 않다면 1번 방식인 `public static final` 필드 방식이 좋다.

### 두 가지 방식의 유의할 점

**첫째. 예외가 존재한다.**

1, 2번 방식의 경우 권한이 있는 클라이언트가 리플렉션 API(아이템 65)인 `AccessibleObject.setAccessible`을 사용하면 `private` 생성자를 호출할 수 있다.

이러한 케이스도 방지하고 싶다면,

생성자에 코드를 추가하여 두 번째 객체가 생성될 때 예외를 던지게 하도록 하자.

**둘째. 직렬화하려면 단순히** `Serializable`**을 구현한다고 선언하는 것만으로는 부족하다.**

```java
// 싱글턴임을 보장해주는 readResolve 메서드
private Object readResolve(){
	// '진짜' Elvis를 반환하고, 가짜 Elvis는 가비지 컬렉터에 맡긴다.
	return INSTANCE;
}
```

1. 모든 인스턴스 필드를 일시적(`transient`)라고 선언하고, 

> → 직렬화되지 않도록 막는다.
> 
1. `readResolve` 메서드를 제공해야 한다.

> → 역직렬화 시 싱글턴 인스턴스를 반환한다.
> 

<aside>
💡

위와 같은 과정을 거치는 이유?

: 직렬화 후 역직렬화 시 **새로운 객체가 생성**되므로 싱글턴이 깨지기 때문이다!

</aside>

### 3. 원소가 하나인 열거 타입의 선언

```java
public enum Elvis {
    INSTANCE;

    public void leaveTheBuilding() {
        System.out.println("기다려 자기야, 지금 나갈께!");
    }
}
```

```java
public static void main(String[] args) {
    Elvis elvis = Elvis.INSTANCE;
    elvis.leaveTheBuilding();
}
```

저자는 사실 이 방식이 제일 좋은 방법이라고 한다!

이 방법은 public 필드 방식과 비슷하지만,

**더 간결**하고,

**추가 노력 없이 직렬화**할 수 있고,

심지어 아주 복잡한 직렬화 상황이나 리플렉션 공격에서도 **제2의 인스턴스가 생기는 일을 완벽히 막아준다.**

<aside>
💡

단, 만들려는 싱글턴이 Enum외의 클래스를 상속해야 한다면 적용할 수 없다는 점을 유의하자.

</aside>