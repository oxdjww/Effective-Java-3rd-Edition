# 아이템 8. finalizer와 cleaner 사용을 피하라

## 개요

자바는 두 가지 객체 소멸자를 제공한다.

1. `finalizer`
    - 예측할 수 없고, 상황에 따라 위험할 수 있다.
    - 이에 일반적으로 불필요하다.
    - 자바 9에서는 사용 자제 API (`deprecated`)로 지정하고, `cleaner를` 대안으로 소개했다.
2. `cleaner`
    - `finalizer보다는` 덜 위험하지만, 여전히 예측할 수 없고, 느리고, 일반적으로 불필요하다.

> C++의 destructor와는 다른 개념이다
> C++에서의 destructor는 특정 객체와 관련된 자원을 회수하는 보편적 방법이다.
> 자바에서는 접근할 수 없게 된 객체를 Garbage collector가 회수한다.
> C++에서의 destructor는 비메모리 자원을 회수하는 용도로 쓰이지만, 자바에서는 try-with-resources와 try-finally를 사용해 해결한다.

## 문제점

### 1. finalizer와 cleaner는 즉시 수행된다는 보장이 없다.

- 객체에 접근할 수 없게 된 후, 두 함수가 실행되기까지 얼마나 걸릴지 알 수 없다.
- 즉, 제때 실행되어야 하는 작업은 절대 할 수 없다. (ex. 파일 닫기)
- 얼마나 빠르게 수행될지는 전적으로 **Garbage collector algorithm**에 달렸으며 각 구현마다 천차만별이다.

- 자원 회수의 지연 때문에, 원인 불명의 OutOfMemoryError가 발생하며 Application이 의문사하는 경우도 허다하다. *(자원 회수 프로세스의 priority 때문)*
- 일부 객체의 자원 회수 시도를 하지도 못한 채로 프로그램(프로세스)가 종료되기도 한다.

### 2. finalizer와 cleaner는 수행 시점 뿐만아니라 수행 여부조차 보장하지 않는다.

즉, 상태를 영구적으로 수정하는 작업(ex. 데이터베이스)에서는 절대 `finalizer`나 `cleaner`를 사용해선 안 된다.

`System.gc`나 `System.runFinalization`과 같은 메서드는 실행될 가능성을 높여줄 수는 있으나, 보장해주진 않는다.

> 사실 이를 보장해주는 메서드인 `System.runFinalizersOnExit`과 `Runtime.runFinalizersOnExit`가 있었으나, 심각한 결함으로 인해 사용할 수 없다고 한다. (ThreadStop: 아마 모종의 이유로 스레드가 중단되는 문제가 있는 모양이다.)

### 3. finalizer와 cleaner 실행 중 발생한 예외는 무시되며, 발생 시 즉시 종료된다.

보통의 경우의 예외가 발생하면, 스택 추적 내역(`printStackTrace()`)을 출력하지만, 이 상황에선 경고조차 출력하지 않는다.

### 4. finalizer와 cleaner는 성능 문제도 동반한다.

저자는 간단한 개체를 생성하고 Garbage collector가 수거하기까지 **12ns**가 걸린 반면, `finalizer`를 사용하면 **550ns**가 걸렸다. (약 50배 차이)

### 5. finalizer를 사용한 클래스는 finalizer 공격에 노출되어 보안 문제가 발생할 수 있다.

1. 생성자/직렬화 과정에서 예외가 발생한다.
2. 생성되다 만 객체에서 악의적인 하위 클래스의 `finalizer`가 수행될 수 있다.

이렇게 미완성된 객체가 생성되면, 이 객체의 메서드를 호출해 애초에 허용되지 않을 작업을 수행할 수 있다.

단, 객체 생성을 막기 위해 생성자에서 예외를 던지는 것으로 충분하긴 하지만, `finalizer`가 있다면 꼭 그렇지는 않다라고 한다.

물론, **final class**는 서브 클래스 객체를 만들 수 없으니 이 공격에서는 안전하다.
**non-final class**를 `finalizer` 공격으로부터 방어하려면, 아무 코드도 적히지 않은 `finalizer` 메서드를 만들고 이 또한 final로 선언하면 된다.

## 그렇다면 대안은

관리해야 할 자원을 담고 있는 class에서 `AutoCloseable`을 구현해주고, 클라이언트에서 인스턴스 사용을 끝내면 close 메서드를 호출하여 안전하게 종료하면 된다.
close 메서드에서 이 객체는 더이상 유효하지 않음을 필드에 기록하고, 이 객체가 다시 접근된다면 `IllegalStateException`을 던져 유효하지 않은 객체에 접근함을 알려야 한다.

## finalizer와 cleaner의 용도

그렇다면 두 기능의 용도는 어떻게 될까

1. 자원의 소유자가 `close` 메서드를 호출하지 않는 것에 대비한 안전망 역할
    - 즉시 호출되리라는 보장은 없지만, 나중에라도 수거하는 것에 대한 보험
    - 자바 라이브러리의 일부 클래스에서는 안전망 역할 `finalizer`를 지원한다. (`FileInputStream`, `FileOutputStream`, `THreadPoolExecutor`)
2. 네이티브 피어(native peer)와 연결된 객체에서 사용
    > 네이티브 피어란: 일반 자바 객체가 네이티브 메서드를 통해 기능을 위임한 네이티브 객체
    - 가비지 컬렉터는 이 존재를 알지 못 하기에, 이는 `finalizer`나 `cleaner`가 하는 것이 적합하다.
    - 단, 성능 저하를 감당할 수 있고 네이티브 피어가 심각한 자원을 가지고 있지 않을때만 해당된다.

### 사용하기 까다로운 cleaner

`cleaner`를 사용할지 말지는 순전히 내부 구현 방식에 따라 나뉜다.
예제를 보자.

```java
public class Room implements AutoCloseable {
    private static final Cleaner cleaner = Cleaner.create();

    private static class State implements Runnable {
        int numJunkPiles; // 방 안의 쓰레기 수

        State(int numJunkPiles) {
            this.numJunPiles = numJunkPiles;
        }

        // close나 cleaner 메서드가 호출한다.
        @Override public void run() {
            System.out.println("방 청소");
            numJunkPiles = 0;
        }
    }

    // 방의 상태. cleanable과 공유함.
    private final State state;

    // cleanable 객체. 수거 대상이 되면 방을 청소한다.
    private final Cleaner.Cleanable cleanable;

    public Room(int numJunkPiles) {
        state = new State(numJunkPiles);
        cleanable = cleaner.register(this, state);
    }

    @Override public void close() {
        cleanable.clean();
    }
}
```

**static nested class**인 State는 `cleaner`가 방을 청소할 때 수거할 자원들을 담고 있다.
(`numJunkPiles` field)

`cleanable` 객체는 Room 생성자에서 `cleaner`에 Room과 State를 등록되며, run 메서드는 `cleanable`에 의해 딱 한 번만 호출된다.

run 메서드가 호출되는 상황은 아래와 같다.

1. Room의 close 메서드가 호출될 때
    - close 메서드에서, Cleanable의 clean을 호출하면 이 메서드 안에서 run이 호출된다.
2. `cleaner`가 State의 run을 호출한다.
    - Garbage collector가 Room을 회수할 때까지 client에서 close를 호출하지 않는다면

또, 내부 구현 시 State 인스턴스가 **절대** Room 인스턴스를 참조하지 않도록 유의해야 한다. (순환참조)

이렇게 단지 안전망의 기능만 하게 `cleaner`를 사용할 수 있다.
클라이언트가 모든 Room 인스턴스를 생성할 때 `try-with-resources` 블록으로 감쌌다면 이 내용은 전혀 필요하지 않다.

```java
public class Adult {
    public static void main(String[] args) {
        try (Room myRoom = new Room(7)) {
            System.out.println("안녕~");
        }
    }
}
```

실행 결과

```bash
$ 안녕~
$ 방 청소
```

하지만 이 경우는 실행 결과가 다르다.

```java
public class Teenage {
    public static void main(String[] args) {
        new Room(99);
        System.out.println("아무렴");
    }
}

```

```bash
$ 아무렴
```

예측할 수 없는 상황이 벌어진 것이다. `cleaner`의 명세를 보면,
> `cleaner`의 동작은 구현하기 나름이다. 청소가 이뤄질지는 보장하지 않는다.

main 메서드에 `System.gc()`를 호출하여 프로그램 종료 직전에 **"방 청소"** 를 출력할 수도 있으나, 그렇지 않은 경우도 있다는 것이다. (우선순위 스케줄링 등의 이유로 인해)

## 정리

`cleaner(finalizer)`는 안전망 역할이나, 중요하지 않은 네이티브 자원 회수에만 사용하자. (불확실성과 성능 저하를 감수하며..)
