# 아이템 13. clone 재정의는 주의해서 진행하라

## Cloneable 인터페이스의 문제점

`Cloneable` 인터페이스는 복제해도 되는 클래스임을 명시하는 용도의 믹스인 인터페이스(mixin interface)이나, 의도한 목적을 제대로 이루지 못하였다. 

<aside>
💡

**믹스인 인터페이스란?**

: 객체지향언어에서 다른 클래스에서 **‘사용’** 할 목적으로 만들어진 클래스.

일반적으로 **특정 기능을 추가하는 역할**을 하며, 이를 구현하는 **클래스에 기능을 주입**하는 방식으로 활용된다.

Ex) `Comparable` 인터페이스

</aside>

> **왜일까?**
> 

가장 큰 문제는 `clone` 메서드가 선언된 곳이 `Cloneable`이 아닌 `Object`이고, 그마저도 `protected`라는 점이다. 

→ **Cloneable을 구현하는 것만으로는 외부 객체에서 clone 메서드를 호출할 수 없다.**

## `Cloneable` 인터페이스의 실제 역할

`Cloneable` 인터페이스는 `Object`의 `protected` 메서드인 `clone`의 동작 방식을 결정한다.

- **`Cloneable`을 구현한 클래스의 인스턴스:** `clone`을 호출하면 해당 객체의 필드들을 하나하나 복사한 객체를 반환한다.
- **그렇지 않은 클래스의 인스턴스:** `clone`을 호출하면 `CloneNotSupportedException`을 던진다.

## 클론 메서드의 일반 규약

1. x.clone() != x는 참이다.
2. x.clone().getClass() == x.getClass()는 참이다.
3. x.clone().equals(x)는 참이지만, 필수는 아니다.
4. 원본 객체에 영향을 주지 않고 복제된 객체의 불변식을 보장해야 한다.

<aside>
💡

즉, clone은 원본 객체를 손상시키지 않으면서 복제본을 생성해야 한다!

</aside>

## clone 메서드 구현 방법

### 1. 가변 상태를 참조하지 않는 객체의 경우

**가변 상태를 참조하지 않는다는 건** 모든 필드가 기본 타입이거나 불변 객체를 참조한다는 것이다.

```java
@Override public PhoneNumber clone() {
    try {
        return (PhoneNumber) super.clone();
    } catch (CloneNotSupportedException e) {
        throw new AssertionError();  // 일어날 수 없는 일이다.
    }
}
```

1. `super.clone()`을 호출한다.

→ 이렇게 얻은 객체는 원본의 완벽한 복제본이며, 우리가 원하는 상태라고 볼 수 있다.

※ 예외: 일련번호나 고유 ID 값은 수정이 필요할 수 있다.

1. **공변 반환 타이핑**(covariant return typing)을 사용하자.

→ 클라이언트의 수고를 덜어준다.

1. `PhoneNumber` 클래스 선언에 `Cloneable`을 구현한다고 추가하자.
2. `Cloneable`을 구현한 클래스에서는 `CloneNotSupportedException`이 발생할 수 없으나 **검사 예외**이므로 처리해야 한다.

<aside>
💡

즉, 원래는 CloneNotSupportedException이 **비검사 예외(unchecked exception**)이어야 했다는 뜻이다.

</aside>

### 2. 가변 객체를 참조하는 필드가 있는 경우

가변 객체를 참조하는 필드가 있다면 단순히 `super.clone()`을 호출하는 것만으로는 불완전한 복제본이 만들어진다. **원본과 복제본이 같은 가변 객체를 참조하게 되기 때문**이다.

**얕은 복사의 예시 (문제 있음):**

```java
private Object[] elements;

...

@Override public Stack clone() {
    try {
        return (Stack) super.clone();
    } catch (CloneNotSupportedException e) {
        throw new AssertionError();
    }
}
```

<aside>
💡

그럼 어떻게 해야할까?

</aside>

`Stack`의 `clone` 메서드가 제대로 동작하려면 스택 내부 정보를 복사해야 한다.

가장 쉬운 방법은 **`elements` 배열의 `clone`을 재귀적으로 호출**해주는 것이다.

**깊은 복사 구현 (올바른 방법):**

```java
private Object[] elements;

...

@Override public Stack clone() {
    try {
        Stack result = (Stack) super.clone();
        result.elements = elements.clone();
        return result;
    } catch (CloneNotSupportedException e) {
        throw new AssertionError();
    }
}
```

### 3. 더 복잡한 가변 객체가 있는 경우

**연결 리스트** 등의 복잡한 자료구조에서는 각 노드를 복사해야 하는 등 더 깊은 수준의 복사가 필요하다.

```java
static class HashTable implements Cloneable {
    private Entry[] buckets;

    private static class Entry {
        final Object key;
        Object value;
        Entry next;

        public Entry(Object key, Object value, Entry next) {
            this.key = key;
            this.value = value;
            this.next = next;
        }
    }

    @Override
    public HashTable clone() throws CloneNotSupportedException {
        HashTable result = (HashTable) super.clone();
        result.buckets = buckets.clone();
        return result;
    }
}
```

<aside>
💡

위 코드는 잘 작동할까?

</aside>

**정답은 NO.**

위와 같은 코드는 `HashTable`은 적절하게 복제되었을지 몰라도,

필드인 `buckets` 내부에 있는 객체들은 여전히 복제되기 이전의 객체들을 가리키고 있을 것이다. 

```java
static class HashTable implements Cloneable {
    private Entry[] buckets;

    private static class Entry {
        final Object key;
        Object value;
        Entry next;

        public Entry(Object key, Object value, Entry next) {
            this.key = key;
            this.value = value;
            this.next = next;
        }

        Entry deepCopy() {
            Entry result = new Entry(key, value, next);
            for(Entry p = result; p.next != null; p = p.next) {
                p.next = new Entry(p.next.key, p.next.value, p.next.next);
            }
            return result;
        }
    }

    @Override
    public HashTable clone() throws CloneNotSupportedException {
        HashTable result = (HashTable) super.clone();
        result.buckets = new Entry[buckets.length];

        for (int i=0; i<buckets.length; i++) {
            if(buckets[i] != null) {
                result.buckets[i] = buckets[i].deepCopy();
            }
        }

        return result;
    }
}
```

deepCopy를 **재귀**를 활용하여 구현한 코드이다.

버킷이 길지 않다면 잘 작동한다.

<aside>
💡

**만약 버킷이 길다면?**

연결 리스트의 재귀적 복사는 스택 오버플로를 일으킬 수 있으므로 **반복자**를 사용한 반복적 복사 방법도 고려해야 한다.

</aside>

<aside>
💡

`put(key, value)` 메서드를 활용해 구현하는 방법도 있지만, 속도가 느리기도 하고 전체 `Cloneable` 아키텍쳐와 어울리지 않다고 한다. 

궁금하다면 찾아보길 바란다!

</aside>

## 주의사항

1. `clone` 메서드에서는 재정의될 수 있는 메서드를 호출하지 않아야 한다.

→ 위의 `put(key, value)` 메서드를 활용한다면 `final`이거나 `private`를 써야한다.

1. `public`인 `clone`메서드는 `thorws`절을 던지지 않는 것이 사용에 편리하다. 
2. 상속용 클래스는 `Cloneable`을 구현해서는 안 된다.
3. 스레드 안전 클래스를 작성할 때는 `clone` 메서드도 적절히 동기화해야 한다.

## 결론

1. `Cloneable`을 구현하는 모든 클래스는 `clone`을 재정의해야 한다.
2. 접근 제한자는 `public`으로, 반환 타입은 클래스 자신으로 변경해야 한다.
3. `super.clone`을 호출한 후, 필요한 필드를 전부 적절히 수정해야 한다.
4. 불변 객체를 만들거나 상속을 위한 클래스를 설계한다면 `Cloneable`을 구현하지 않는 것이 좋다.

## 근데… 꼭 써야할까?

Cloneable을 이미 구현한 클래스를 확장한다면 clone 메서드가 제기능을 하도록 위와같이 수정해줘야한다.

하지만, 만약 그렇지 않다면 **복사 생성자와 복사 팩터리**가 더 나은 방식이다.

<aside>
💡

**복사 생성자?**

: 단순히 자신과 같은 클래스의 인스턴스를 인수로 받는 생성자.

```java
public Yum(Yum otherYum){ ... };
```

</aside>

<aside>
💡

**복사 팩터리?**

: 복사 생성자를 모방한 정적 팩터리.

```java
public static Yum newInstance(Yum otherYum){ ... };
```

</aside>

### 두 방식의 장점

1. 언어 모순적이고 위험천만한 객체 생성 매커니즘을 사용하지 않는다.
2. 엉성하게 문서화된 규약에 기대지 않는다.
3. 정상적인 final 필드 용법과도 충돌하지 않는다.
4. 불필요한 검사 예외를 던지지 않는다.
5. 형변환도 필요하지 않다.
6. 해당 클래스가 구현한 ‘인터페이스’ 타입의 인스턴스를 인수로 받을 수 있다.

<aside>
💡

예외)

배열의 clone 메서드는 깔끔한 방식이므로 사용해도 좋다고 한다.

</aside>

## 정리하며

`Cloneable`에 대해 열심히 설명해주시며 결론적으로 복사 생성자와 복사 팩터리를 쓰라고하시니 좀 김이 빠지긴 하지만, 그래도 더 편리한 방법이 있다는 것에 감사하게 된다!
