# 아이템 31. 한정적 와일드카드를 사용해 API 유연성을 높이라

## 개요

**매개변수화 타입**은 불공변(invariant)이다. 

즉, `List<String>`은 `List<Object>`의 하위 타입이 아니다.

이러한 제약은 타입 안전성을 보장하지만 유연성이 떨어진다.

> **한정적 와일드카드 타입**을 사용하면 API의 유연성을 크게 높일 수 있다.
> 

## 불공변 타입의 제약

다음 스택 예제를 살펴봅시다:

```java
public class Stack<E> {
    public Stack();
    public void push(E e);
    public E pop();
    public boolean isEmpty();
}
```

여기에 스택의 모든 원소를 다른 컬렉션에 옮기는 메서드를 추가해 보자.

```java
// 와일드카드 타입을 사용하지 않은 pushAll 메서드 - 제약이 있음
public void pushAll(Iterable<E> src) {
    for (E e : src)
        push(e);
}
```

이 메서드는 `Iterable<E>` 타입의 원소만 처리할 수 있어서 제약이 있다.

예를 들어 `Stack<Number>`로 선언한 후 `Iterable<Integer>`를 인자로 전달하면 `Integer`는 `Number`의 하위 타입임에도 불구하고 컴파일 오류가 발생합니다.

> 이는 위에서 다뤘듯이 **매개변수화 타입이 불공변**이기 때문이다.
> 

## 한정적 와일드카드를 활용한 해결책

자바는 이런 상황에 대처할 수 있도록 **한정적 와일드카드 타입**이라는 특별한 **매개변수화 타입**을 지원한다.

### extends 사용

```java
// 생산자 매개변수에 와일드카드 타입 적용
public void pushAll(Iterable<? extends E> src) {
    for (E e : src)
        push(e);
}
```

> 와일드카드 타입 **`Iterable<? extends E>`**: `pushAll`의 입력 매개변수 타입은 `E의 Iterable`이 아닌 `E의 하위 타입의 Iterable` 이어야 한다.
> 

이제 `Stack<Number>`에 `Iterable<Integer>`를 인자로 전달할 수 있다.

### super 사용

```java
// 소비자 매개변수에 와일드카드 타입 적용
public void popAll(Collection<? super E> dst) {
    while (!isEmpty())
        dst.add(pop());
}
```

> 와일드카드 타입 `Collection<? super E>`: `popAll`의 입력 매개변수의 타입이 `E의 Collection`이 아니라 `E의 상위 타입의 Collection`이어야 한다.
> 

이제 `Stack<Number>`의 원소를 `Collection<Object>`에 담을 수 있다.

<aside>
💡

즉, 유연성을 극대화하고 싶다면 원소의 생산자나 소비자용 입력 매개변수에 와일드카드 타입을 사용하라!

</aside>

## PECS 원칙 (Producer-Extends, Consumer-Super)

Joshua Bloch가 제안한 **PECS 원칙**은 다음과 같다.

- **Producer-Extends**: 데이터를 생산(읽기)만 한다면 `<? extends T>`를 사용하라.
- **Consumer-Super**: 데이터를 소비(쓰기)만 한다면 `<? super T>`를 사용하라.

## 타입 매개변수와 와일드카드 중 어느 것을 사용해야 할까?

```java
// 비한정적 타입 매개변수를 사용한 메서드
public static <E> void swap(List<E> list, int i, int j)

// 비한정적 와일드카드를 사용한 메서드
public static void swap(List<?> list, int i, int j);
```

public API일 경우 간단한 두 번째(와일드카드)가 나을 것이다.

<aside>
💡

기본 규칙: 메서드 선언에 타입 매개변수가 한 번만 나오면 와일드카드로 대체하라.

</aside>

하지만 이 방법은 문제가 발생한다.

```java
public static void swap(List<?> list, int i, int j) {
    list.set(i, list.set(j, list.get(i)));
}
```

리스트의 타입이 List<?>이므로 null외에 아무값도 넣을 수 없기에 오류 메시지가 발생한다.

이 문제는 **와일드카드 타입의 실제 타입을 알려주는** 도우미 메서드를 사용하여 해결할 수 있다.

```java
public static void swap(List<?> list, int i, int j) {
    swapHelper(list, i, j);
}

// 와일드카드 타입을 실제 타입으로 바꿔주는 도우미 메서드
private static <E> void swapHelper(List<E> list, int i, int j) {
    list.set(i, list.set(j, list.get(i)));
}
```

## 기억해야 할 사항

1. 유연성을 극대화하려면 API의 입력 매개변수 타입에 와일드카드를 사용하라.
2. 생산자(값을 제공하는 매개변수)에는 `<? extends T>`를 사용하라.
3. 소비자(값을 사용하는 매개변수)에는 `<? super T>`를 사용하라.
4. 메서드의 반환 타입에는 와일드카드 타입을 사용하지 마라. 클라이언트 코드에서도 와일드카드 타입을 써야 하는 불상사가 생길 수 있다.
5. 타입 매개변수와 와일드카드에는 공통 부분이 있어 둘 중 하나를 선택해야 하는 경우가 있다. 메서드 선언에 타입 매개변수가 한 번만 나오면 와일드카드를 사용하라.

## 결론

한정적 와일드카드를 적절히 활용하면 API가 훨씬 유연해진다.

그러나 와일드카드 타입을 사용하면 코드가 복잡해질 수 있으므로 신중하게 사용하자.

**PECS 원칙**을 기억하면 어떤 와일드카드 타입을 사용해야 할지 쉽게 결정할 수 있습니다.