# 아이템 24. 멤버 클래스는 되도록 static으로 만들라

## 중첩 클래스 (Nested class)

중첩 클래스엔 네 가지 종류가 있다

1. 정적 멤버 클래스
2. 비정적 멤버 클래스
3. 익명 클래스
4. 지역 클래스

1번을 제외한 나머지는 모두 내부 클래스(inner class)에 해당한다.

## 왜 사용하는가

### 1. 정적 멤버 클래스

일반 클래스와 2가지 차이가 있다.

1. 다른 클래스 안에 선언된다.
2. 바깥 클래스의 private 멤버에도 접근할 수 있다.

또, private으로 선언하면 바깥 클래스에서만 접근할 수 있다.

바깥 클래스와 함께 쓰일 때만 유용한, **public 도우미 클래스**로 쓰인다.

```java
public class Calculator {
    // 정적 멤버 클래스 (도우미 역할)
    public static enum Operation {
        PLUS, MINUS, MULTIPLY, DIVIDE;

        // 연산 수행
        public double apply(double x, double y) {
            return switch (this) {
                case PLUS -> x + y;
                case MINUS -> x - y;
                case MULTIPLY -> x * y;
                case DIVIDE -> x / y;
            };
        }
    }

    // 예시 메소드: 정적 멤버 클래스 사용
    public static double calculate(double x, double y, Operation op) {
        return op.apply(x, y);
    }

    public static void main(String[] args) {
        double result = Calculator.calculate(10, 5, Operation.PLUS);
        System.out.println("10 + 5 = " + result);  // 출력: 10 + 5 = 15.0
    }
}
```

## 2. 비정적 멤버 클래스

단지 static이 붙어있고 없고의 차이지만, 의미상 차이는 큼.

비정적 멤버 클래스의 인스턴스는 바까타 클래스의 인스턴스와 암묵적으로 연결됨.

비정적 멤버 클래스의 메서드에서 this로 바깥 인스턴스의 메서드를 호출하거나 참조를 가져올 수 있음.

```java
OuterClass.this.method();
```

개념적으로 바깥 인스턴스와 독립적으로 존재할 수 있다면, 이를 **정적 클래스 멤버**로 만들어야 함.

이 **비정적 멤버 클래스 인스턴스-바깥 클래스 인스턴스** 관계는 인스턴스화될 때 불변으로 확정됨.

이 관계는 바깥 클래스 인스턴스 메서드에서, 비정적 멤버 클래스(안쪽 클래스)의 생성자를 호출하며 자동으로 만들어지지만, 드물게는 아래와 같이도 쓰인다.

```java
OuterClass.new MemberClass(args);
```

### 어댑터 패턴

이 비정적 멤버 클래스는, 어댑터 정의 시 자주 쓰인다.

> 어떤 클래스의 인스턴스를 감싸 마치 다른 클래스의 인스턴스처럼 보이게 하는 뷰


[반복자를 반환하는 예]

```java
public class MySet<E> extends AbstractSet<E> {
    ... // 생략

    @Override public Iterator<E> iterator() {
        return new MyIterator();
    }

    private class MyIterator implements Iterator<E> {
        ...
    }
}
```

정리하자면, **멤버 클래스에서 바깥 인스턴스에 접근할 일이 없다면** 무조건 `static`을 붙여 **정적 멤버 클래스**로 만들자.

static을 생략하면 바깥 인스턴스로부터의 숨은 참조를 갖게 되어 시간/공간의 낭비가 있음. 가비지 컬렉션이 바깥 인스턴스의 메모리를 수거하지 못 할 수 있음. (참조가 눈에 보이지않아 문제를 찾기도 어려움)

### private 멤버 클래스

바깥 클래스가 표현하는 **객체의 한 부분**을 표현할 때 씀.

아래 예제에서, `Entry`는 외부에서 직접 접근이 불가하며, 내부 메서드에서 사용한다.

모든 엔트리가 Map과 연관되어 있지만 직접적으로 엔트리의 메서드들(`getKey`, `getValue`, `setValue`)은 맵을 직접 사용하지 않음.

```java
public class MyMap {
    // 키-값 쌍을 표현하는 내부 클래스 (외부에서 접근 못하게 private static)
    private static class Entry {
        String key;
        String value;

        Entry(String key, String value) {
            this.key = key;
            this.value = value;
        }

        String getKey() { return key; }
        String getValue() { return value; }
        void setValue(String value) { this.value = value; }
    }

    // 엔트리 배열로 간단한 맵 구현
    private Entry[] entries = new Entry[10];
    private int size = 0;

    public void put(String key, String value) {
        entries[size++] = new Entry(key, value);
    }

    public String get(String key) {
        for (int i = 0; i < size; i++) {
            if (entries[i].getKey().equals(key)) {
                return entries[i].getValue();
            }
        }
        return null;
    }

    public static void main(String[] args) {
        MyMap map = new MyMap();
        map.put("apple", "사과");
        map.put("banana", "바나나");

        System.out.println(map.get("apple"));  // 출력: 사과
        System.out.println(map.get("banana")); // 출력: 바나나
    }
}
```

멤버 클래스가 공개된 클래스의 `public` or `protected` 멤버라면, `static`이냐 아니냐가 굉장히 중요해진다.

멤버 클래스 역시 공개 API가 되어 향후에 `static`을 붙이면 호환성이 깨진다.

*바깥 인스턴스 없이 내부 인스턴스를 만들 수 있기 때문이다.*

## 3. 익명 클래스

- 바깥 클래스의 멤버가 아니며, 이름이 없음.
- 쓰이는 시점에 선언과 동시에 인스턴스가 만들어짐.
- 보통 인터페이스나 추상 클래스를 구현할 때 1회용으로 사용.
- 다른 클래스 상속 or 여러 인터페이스 구현 불가
- instanceof 검사 시 이름이 없어서 확인 불가
- 간단한 로직에만 사용해야 가독성 유지

```java
import java.awt.event.ActionEvent;
import java.awt.event.ActionListener;
import javax.swing.JButton;
import javax.swing.JFrame;

public class MyWindow {
    public static void main(String[] args) {
        JFrame frame = new JFrame("익명 클래스 예제");
        JButton button = new JButton("Click Me");

        // 여기서 익명 클래스 사용!
        button.addActionListener(new ActionListener() {
            @Override
            public void actionPerformed(ActionEvent e) {
                System.out.println("버튼이 눌렸어요!");
            }
        });

        frame.add(button);
        frame.setSize(200, 100);
        frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
        frame.setVisible(true);
    }
}
```

## 4. 지역 클래스

- 메서드 내부에서 정의하는 클래스
- 이름이 있는 클래스이지만, 지역변수처럼 메서드 안에서만 유효
- 바깥 클래스의 인스턴스를 참조할 수 있음
- 정적 멤버는 가질 수 없음

```java
public class Greeting {
    public void sayHello(String name) {
        // 지역 클래스 선언
        class Message {
            public void hello() {
                System.out.println("Hello, " + name);
            }
        }

        // 지역 클래스 사용
        Message msg = new Message();
        msg.hello();
    }

    public static void main(String[] args) {
        Greeting g = new Greeting();
        g.sayHello("정태");
    }
}
```

## 정리

- 네 가지 중첩 클래스가 존재하며 쓰임새가 다르다.

1. 정적 멤버 클래스: 메서드 밖에서 사용해야 하거나, 메서드 안에 정의하기에 길고 **멤버 클래스의 인스턴스 각각이 바깥을 참조하지 않을  경우**
2. 비정적 멤버 클래스: 메서드 밖에서 사용해야 하거나, 메서드 안에 정의하기에 길고 **멤버 클래스의 인스턴스 각각이 바깥을 참조할 경우**
3. 익명 클래스: 인스턴스를 생성하는 지점이 한 곳이며, **해당 타입으로 쓰기에 적합한 클래스나 인터페이스가 있는 경우**
4. 지역 클래스: 인스턴스를 생성하는 지점이 한 곳이며, **해당 타입으로 쓰기에 적합한 클래스나 인터페이스가 없는 경우**