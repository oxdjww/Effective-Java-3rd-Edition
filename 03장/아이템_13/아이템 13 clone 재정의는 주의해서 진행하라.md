# 아이템 13. clone 재정의는 주의해서 진행하라

## Cloneable 인터페이스의 문제점

Cloneable 인터페이스는 복제해도 되는 클래스임을 명시하는 용도의 믹스인 인터페이스(mixin interface)이나, 의도한 목적을 제대로 이루지 못하였다. 가장 큰 문제는 clone 메서드가 선언된 곳이 Cloneable이 아닌 Object이고, 그마저도 protected라는 점이다. 그래서 Cloneable을 구현하는 것만으로는 외부 객체에서 clone 메서드를 호출할 수 없다.

## Cloneable 인터페이스의 실제 역할

Cloneable 인터페이스는 Object의 protected 메서드인 clone의 동작 방식을 결정한다:
- Cloneable을 구현한 클래스의 인스턴스에서 clone을 호출하면 해당 객체의 필드들을 하나하나 복사한 객체를 반환한다.
- 그렇지 않은 클래스의 인스턴스에서 호출하면 CloneNotSupportedException을 던진다.

## 클론 메서드의 일반 규약

1. x.clone() != x는 참이다.
2. x.clone().getClass() == x.getClass()는 참이다.
3. x.clone().equals(x)는 참이지만, 필수는 아니다.
4. 원본 객체에 영향을 주지 않고 복제된 객체의 불변식을 보장해야 한다.

<aside>
💡
즉, clone은 원본 객체를 손상시키지 않으면서 복제본을 생성해야 한다.
</aside>

## clone 메서드 구현 방법

### 1. 불변 객체의 경우

불변 클래스라면 굳이 clone 메서드를 제공하지 않는 것이 좋다. 복사 생성자나 복사 팩터리를 제공하는 방법이 더 나은 선택이다.

```java
// 복사 생성자
public Yum(Yum yum) { ... }

// 복사 팩터리
public static Yum newInstance(Yum yum) { ... }
```

### 2. 가변 객체의 경우

1. super.clone()을 호출하여 얻은 객체는 원본의 완벽한 복제본이다.
2. 모든 필드가 기본 타입이거나 불변 객체를 참조한다면 이 객체는 완벽히 우리가 원하는 상태다 (예외: 일련번호나 고유 ID 값은 수정이 필요할 수 있다).

```java
@Override
public PhoneNumber clone() {
    try {
        return (PhoneNumber) super.clone();
    } catch (CloneNotSupportedException e) {
        throw new AssertionError(); // 일어날 수 없는 일
    }
}
```

<aside>
💡
Cloneable을 구현한 클래스에서는 CloneNotSupportedException이 발생할 수 없으나 검사 예외이므로 처리해야 한다.
</aside>

### 3. 가변 객체를 참조하는 필드가 있는 경우

가변 객체를 참조하는 필드가 있다면 단순히 super.clone()을 호출하는 것만으로는 불완전한 복제본이 만들어진다. 원본과 복제본이 같은 가변 객체를 참조하게 되기 때문이다.

**얕은 복사의 예시 (문제 있음):**
```java
public class Stack implements Cloneable {
    private Object[] elements;
    private int size = 0;
    // ...

    @Override
    public Stack clone() {
        try {
            return (Stack) super.clone(); // 문제 발생! elements 배열을 공유함
        } catch (CloneNotSupportedException e) {
            throw new AssertionError();
        }
    }
}
```

**깊은 복사 구현 (올바른 방법):**
```java
@Override
public Stack clone() {
    try {
        Stack result = (Stack) super.clone();
        result.elements = elements.clone(); // 배열의 clone은 배열의 완벽한 복제본을 반환
        return result;
    } catch (CloneNotSupportedException e) {
        throw new AssertionError();
    }
}
```

### 4. 더 복잡한 가변 객체가 있는 경우

연결 리스트 등의 복잡한 자료구조에서는 각 노드를 복사해야 하는 등 더 깊은 수준의 복사가 필요하다.

```java
@Override
public HashTable clone() {
    try {
        HashTable result = (HashTable) super.clone();
        result.buckets = new Entry[buckets.length];
        
        // 각 버킷에 대해 깊은 복사 수행
        for (int i = 0; i < buckets.length; i++) {
            if (buckets[i] != null)
                result.buckets[i] = buckets[i].deepCopy();
        }
        return result;
    } catch (CloneNotSupportedException e) {
        throw new AssertionError();
    }
}

// Entry 클래스의 deepCopy 메서드 (연결 리스트를 복사)
private static class Entry {
    final Object key;
    Object value;
    Entry next;
    
    // 연결 리스트를 반복적으로 복사
    Entry deepCopy() {
        Entry result = new Entry(key, value, next == null ? null : next.deepCopy());
        return result;
    }
}
```

<aside>
💡
연결 리스트의 재귀적 복사는 스택 오버플로를 일으킬 수 있으므로 반복자를 사용한 반복적 복사 방법도 고려해야 한다.
</aside>

## 주의사항

1. clone 메서드에서는 재정의될 수 있는 메서드를 호출하지 않아야 한다.
2. 상속용 클래스는 Cloneable을 구현해서는 안 된다.
3. 스레드 안전 클래스를 작성할 때는 clone 메서드도 적절히 동기화해야 한다.

<aside>
💡
Cloneable을 구현한 스레드 안전 클래스를 작성할 때는 clone 메서드 역시 적절히 동기화해주어야 한다.
</aside>

## 결론

1. Cloneable을 구현하는 모든 클래스는 clone을 재정의해야 한다.
2. 접근 제한자는 public으로, 반환 타입은 클래스 자신으로 변경해야 한다.
3. super.clone을 호출한 후, 필요한 필드를 전부 적절히 수정해야 한다.
4. 불변 객체를 만들거나 상속을 위한 클래스를 설계한다면 Cloneable을 구현하지 않는 것이 좋다.
5. 복제 기능은 생성자와 팩터리를 이용하는 것이 최선이다. 단, 배열만은 clone 메서드 방식이 가장 깔끔하다.
