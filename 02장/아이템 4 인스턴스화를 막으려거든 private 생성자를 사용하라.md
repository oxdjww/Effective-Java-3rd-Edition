# 아이템 4. 인스턴스화를 막으려거든 private 생성자를 사용하라

## 배경

인스턴스가 필요 없는, 정적 메서드와 정적 필드만을 담은 클래스를 정의하는 경우가 있다.
객체지향적 프로그래밍에는 적합하지 않다고 볼 수 있으나, 종종 쓰임새가 있다.

1. `java.lang.Math`, `java.util.Arrays` 처럼 기본 타입 값이나, 배열 관련 메서드들을 모아 놓는 경우
2. `java.util.Collections`처럼 특정 인터페이스를 구현하는 객체를 생성해주는 정적 메서드들을 모아 놓는 경우
3. final 클래스와 관련된 메서드들을 모아놓을 때
    ```
    public final class StringUtils {

        // private 생성자: 인스턴스화 방지
        private StringUtils() {
            throw new UnsupportedOperationException("StringUtils cannot be instantiated.");
        }

        // 정적 메서드
        public static String reverse(String input) {
            StringBuilder reversed = new StringBuilder(input);
            return reversed.reverse().toString();
        }

        public static boolean isEmpty(String input) {
            return input == null || input.isEmpty();
        }
    }
    ```

## 인스턴스를 만들 수 없게 하라

생성자를 명시하지 않으면, 컴파일러가 자동으로 매개변수가 없는 기본 생성자를 만들어준다.
이를 방지하기 위해,

- 추상 클래스로 만들기: 이 방법은 단순히 서브 클래스에서 상속하면 인스턴스화가 가능하다.

하지만, private 생성자를 추가하면 클래스의 인스턴스화를 막을 수 있다.

```
public class UtilityClass {
        private UtilityClass() { ... }
}
```

또, 위 방식은 상속을 불가능하게 하는 효과도 있다.
(서브 클래스의 생성자는 슈퍼 클래스의 생성자를 반드시 호출하는데, `private method`에 접근 할 수 없다.)