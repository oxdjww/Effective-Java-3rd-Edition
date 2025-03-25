# 아이템 16. public 클래스에서는 public 필드가 아닌 접근자 메서드를 사용하라

## 예시

```java
class Point {
    public double x;
    public double y;
}
```

- 이런 클래스는 데이터 필드에 접근할 수 있어, 캡슐화 이점을 적용할 수 없음.
- 불변성 보장 불가.

그렇기에 아래와 같이 **접근자**와 **변경자(mutator)**를 두도록 수정해야 한다.

```java
class Point {
    private double x;
    private double y;

    public Point(double x, double y) {
        this.x = x;
        this.y = y;
    }

    public double getX() { return x; }
    public double getY() { return y; }

    public void setX(double x) { this.x = x; }
    public void setY(double y) { this.y = y; }
}
```

- 패키지 바깥에서 접근할 수 있는 클래스라면, 접근자를 제공하여 유연성을 얻을 수 있음.
- 그렇지만, `package-private(default)` 클래스나 `nested private` 클래스라면 데이터 필드를 노출해도 큰 의미가 없음.

> 자바 플랫폼 라이브러리에서, public 클래스의 필드를 직접 노출하여 성능 문제가 발생한 클래스가 있다. (Dimension)

## public 필드가 불변이라면?

단점은 조금 줄어들지만, 그래도 좋은 생각이 아니다.

- API를 변경하지 않고는 표현 방식을 바꿀 수 없음. (즉, 변수를 사용하는 함수에 의존성이 생김)

### 예제

```java
    public class Person {
    public String birthdate;  // "YYYY-MM-DD" 형식의 생일
}
```

이 클래스는 `birthdate`를 퍼블릭 필드로 공개했기 때문에, 외부에서 이렇게 직접 사용할 수 있음:

```java
Person p = new Person();
p.birthdate = "1990-01-01";
System.out.println(p.birthdate.substring(0, 4));  // 연도 추출
```

이제 만약 내부적으로 `birthdate`를 `LocalDate` 같은 객체로 바꾸고 싶다면?

```java
public class Person {
    public LocalDate birthdate;  // 내부 표현 방식 바뀜
}
```

이렇게 바꾸면 기존에 `String`으로 처리하던 외부 코드들이 전부 에러 발생. **즉, API가 바뀐 것이고, 외부 코드에도 영향을 미침.**

**✅ 개선안**

```java
public class Person {
    private LocalDate birthdate;  // 내부 표현 숨김

    public String getBirthdate() {
        return birthdate.toString();  // 외부에는 여전히 String으로 보이게
    }

    public void setBirthdate(String dateStr) {
        this.birthdate = LocalDate.parse(dateStr);
    }
}
```
이렇게 하면 나중에 내부 표현 방식이 바뀌더라도 `get/set` 메서드만 고치면 되니까, 외부 API는 그대로 유지 가능.

- 필드를 읽을 때 부수 작업을 수행할 수 없음. (로깅, 비즈니스 로직 수행 등등 불가능. 그저 멤버 변수에 접근해서 읽기만.)

## 결론

- `public` 클래스는, 절대 가변 필드를 직접 노출하지 말자. (`public` 필드)
- 불변 필드라면 노출해도 덜 위험하지만, 안심할 수 없음
- 하지만 `default` 클래스나 `private` 중첩 클래스라면 (불변/가변 상관없이) 필드를 노출하는 편이 나을 때도 있다.