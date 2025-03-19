# 아이템 10. equals는 일반 규약을 지켜 재정의하라

## 개요

인스턴스를 비교할 때 사용하는 `equals`는 간단해보이지만, 정말 자기 자신인 경우만 같게 취급한다. 이에 아래와 같은 상황에서는 `equals`를 재정의(override)하지 않는 것이 최선이라고 한다.

1. 각 인스턴스가 본질적으로 고유하다.
2. 인스턴스의 논리적 동치성을 검사할 일이 없다.
3. 상위 클래스에서 재정의한 equals가 하위 클래스에서도 동일 용도로 쓰인다.
4. 클래스가 `private`이거나 `package-private`이고 `equals 메서드`를 호출할 일이 없다.
    - 위험을 철저히 회피하고 싶다면, 아래와 같이 구현하는 것도 방법이다.
        ```java
        @Override public boolean equals(Object o) {
            throw new AssertionError(); // 호출 금지!
        }
        ```
## 재정의가 필요할 때

객체 식별성(`object identity`: 두 객체가 물리적으로 같은 것)이 아니라, 논리적 동치성을 비교하는 것이 필요할 때이다.

주로 값 클래스들이 여기에 해당한다. (`Integer`, `String` 처럼 값을 표현하기 위한 클래스)

두 값 객체를 `equals`로 비교하는 프로그래머들은, 물리적이 아닌 논리적으로 같은 값(value)를 갖는 것인지 확인하고 싶을 것이다. 이 경우, **Map의 키와 Set의 원소**로 사용할 수도 있게 된다.

물론 값이 같은 인스턴스가 둘 이상 만들어지지 않음이 보장된다면, 물리적 동치성과 논리적 동치성이 같은 의미를 갖게 된다.

## equals 규악

`Object` 명세에 적힌 일반 규약을 살펴보자.

equals 메서드는 동치관계(`equivalence relation`)를 구현하며, 다음을 만족한다.

### 반사성

> null이 아닌 모든 참조 값 x에 대해, x.equals(x)는 true

` x = x `

객체는 자기 자신과 같아야 한다.

### 대칭성

> null이 아닌 모든 참조 값 x, y에 대해, x.equals(y)가 true 👉 y.equals(x)가 true

` x = y 이면 y = x`

두 객체는 서로에 대한 동치 여부에 동일하게 답해야 한다.

```java
public final class CaseInsensitiveString {

    private final String s;

    public CaseInsensitiveString(String s) {
        this.s = Objects.requireNonNull(s);
    }

    @Override
    public boolean equals(Object o) {
        if (o instanceof CaseInsensitiveString)
            return s.equalsIgnoreCase(((CaseInsensitiveString) o).s);
        if (o instanceof String)
            return s.equalsIgnoreCase((String)o);
        return false;
    }
}
```
위와 같이, 대소문자를 동일하게 취급하는 custom string 클래스가 있다.

```java
CaseInsensitiveString cis = new CaseInsensitiveString("Polish");
Strings = "polish";
```

cis.equals(s) 는 true를 반환한다.
하지만, String의 equals는 CaseInsensitiveString의 존재를 모르기에, s.equals(cis)는 false를 반환하여, 대칭성을 명백히 위반한다.

```java
List<CaseInsensitiveString> list = new ArrayList<>();
list.add(cis);
```

이 다음에, list.contains(s)를 호출한다면? false가 나온다. 하지만 이는 구현하기 나름이라는 것을 참고하자.

즉, custom string 클래스와, String을 연동하겠다는 것이 불가능하다는 의미이다. 이 경우엔 같은 자료형의 객체끼리만 비교가 가능하다.

**즉, equals 규약을 어기면 그 객체를 사용하는 다른 객체들이 어떻게 반응할지 알 수 없다.**

### 추이성

> null이 아닌 모든 참조 값 x, y, z에 대해, x.equals(y)가 true && y.equals(z)가 true 👉 x.equals(z)가 true

` x = y, y = z 이면, x = z 이다.`

상위 클래스에는 없는 새로운 필드를 하위 클래스에 추가하는 상황을 가정하자. (equals 비교에 영향을 줌)

```java
public class Point {
    
    private final int x;
    private final int y;
    
    public Point(int x,int y){
        this.x=x;
        this.y=y;
    }
    @Override
    public boolean equals(Object o){
        if(!(o instanceof Point))
            return false;
        Point p=(Point)o;
        return p.x==x&&p.y==y;
    }
}
```
위 코드에서 확장하여, 아래 하위 클래스를 정의하였다.

```java
public class ColorPoint extends Point{
    private final Color color;

    public ColorPoint(int x, int y, Color color) {
        super(x, y);
        this.color = color;
    }
}
```

부모 `Point 클래스`의 `equals`를 그대로 물려 받으면, `color` 멤버변수에 대한 비교가 없기에, **추이성**을 지킬 수 없다.

```java
@Override
public boolean equals(Object o){
    if(!(o instanceof  ColorPoint))
        return false;
    return super.equals(o)&&((ColorPoint)o).color==color;
}
```

위와 같이 비교하여도, 아래를 보면,

```java
public static void main(String[] args) {

    Point point=new Point(1,2);
    ColorPoint cp=new ColorPoint(1,2,Color.BLUE);
    System.out.println(point.equals(cp));
    System.out.println(cp.equals(point));
}
```

대칭성을 위반하게 된다.

그렇기에, 또 다른 방법으로는 `ColorPoint`와 `Point`의 비교만 예외를 두어 처리하면 어떻게 될까?

```java
@Override
public boolean equals(Object o){
    if(!(o instanceof Point))
        return false;

    if(!(o instanceof ColorPoint))
        return o.equals(this);

    return super.equals(o)&&((ColorPoint)o).color==color;
}

// 예제 코드
ColorPoint p1 = new ColorPoint(1, 2, Color.RED);
Point p2 = new Point(1, 2);
ColorPoint p3 = new ColorPoint(1, 2, Color.BLUE);
// p1 == p2, p2 == p3, p1 != p3
```

이렇게 될 경우 추이성을 만족하지 못 한다.

한계를 인정하고, `instanceof` 검사를 `getClass` 검사로 바꾼다면, 규악도 지키고, 값 추가도 되며 구체 클래스를 상속할 수 있는 것처럼 보인다.

```java
@Override
public boolean equals(Object o) {
    if (o == null || o.getClass() != getClass())
        return false;
    Point p = (Point) o;
    return p.x == x && p.y == y;
}
```

같은 클래스의 객체와 비교할 때만 `true`를 반환하지만, 이는 리스코프 치환 원칙에 위배된다.

> 리스코프 치환 원칙: 부모 객체와 자식 객체가 있을 때 부모 객체를 호출하는 동작에서 자식 객체가 부모 객체를 완전히 대체할 수 있다는 원칙

즉, `ColorPoint`와 같은 `Point`의 하위 클래스의 인스턴스들은 어디서든지 `Point`와 동일한 활용을 할 수 있어야 하지만 불가능하다는 것이다.

> Collection의 내부에서 인스턴스들을 비교하는 연산이 존재하는데, 이 연산에서도 equals를 호출할 때 상위 클래스를 상속 받은 하위 클래스의 인스턴스들을 모조리 false 처리하는 현상이 일어나게 된다.

#### 해결책?

그렇다면, 상속 대신 컴포지션으로 해결할 수 있다. (아이템 18)

`Point`를 상속하는 대신, `Point`를 `ColorPoint`의 `private` 필드로 두고 `view method`(아이템 06)를 `public`으로 정의하면 된다.

```java
public class ColorPoint {
    private final Color color;
    private final Point point

    public ColorPoint(int x, int y, Color color) {
        point = new Point(x,y);
        this.color = Objects.requireNonNull(color);
    }

    public Point asPoint(){
        return point;
    }
    
    @Override
    public boolean equals(Object o){
        if(!(o instanceof  ColorPoint))
            return false;
        ColorPoint cp=(ColorPoint) o;
        
        return cp.point.equals(point)&&cp.color.equals(color);
    }
}
```

자바 라이브러리에도 이런 경우가 종종 있다. `java.sql.Timestamp`는 `java.util.Date`를 확장한 후 `nanoseconds` 필드를 추가했다. 물론 그 결과로 대칭성을 위배하며 `Date` 객체와 한 컬렉션에 넣고 섞어 사용하면 엉뚱하게 동작할 수 있다. (설계의 실수이다.)

### 일관성

> null이 아닌 모든 참조 값 x, y에 대해, x.equals(y)를 반복해서 호출하여도 항상 true만 혹은 false만 반환한다.

두 객체가 같다면 수정하지 않는 한 영원히 같아야 한다.

가변 객체는 비교 시점에 따라 다를 수도, 같을 수도 있지만 불변 객체는 한번 다르면 끝까지 달라야 한다.

`equals`는 항시 **"메모리에 존재하는 객체"**만을 사용하여 정의해야 한다. `java.net.URL`의 `equals`는, URL과 매핑된 호스트의 IP 주소를 이용해 비교하는데, 이 IP주소는 네트워크를 통해 들어오므로 항상 같다고 보장될 수 없다. (설계 오류)

### null-아님
> null이 아닌 모든 참조 값 x에 대해, x.equals(null)은 false

모든 객체가 null과 같지 않아야 한다. `o.equals(null)`이 true를 던지는 상황은 상상하기 어렵지만, `NullPointerException`을 던지는 코드는 흔할 것이다. 이 일반 규약은 이런 경우도 허용하지 않는다.

수많은 클래스가 다음 코드처럼, 입력이 null인지 검토한다.

```java
// 명시적 null 검사 - 필요 없다
@Override public boolean equals(Object o) {
    if (o == null)
        return false;
    ...
}
```
이렇게 직접적으로 null과 비교하는 것보단,

```java
@Override
public boolean equals(Object o) {
  if(!(o instanceof MyType)) { 
      return false;
  }
  MyType myType = (MyType) o;
}
```

이처럼 묵시적으로 null 검사하는 편이 낫다. `instanceof`는 첫 피연산자가 null이면 `false`를 반환하는 특징을 이용한 것이다.

## 단계별 정리

`equals` 메서드 구현법을 단계적으로 정리하자.

1. == 연산자를 사용해 입력이 자기 자신의 참조인지 확인한다.
2. `instanceof` 연산자로 입력이 올바른 타입인지 확인한다.
3. 입력을 올바른 타입으로 형변환한다.
4. 입력 객체와 자기 자신의 대응되는 '핵심' 필드들이 모두 일치하는지 하나씩 검사한다.
    - `float, double` 제외한 기본 타입 필드는 ==로 비교하고, 참조 타입 필드는 각각의 `equals`로, `float, double` 필드는 각각 정적 메서드인 `Float.compare()`와 `Double.compare()`로 비교한다.
    - 배열을 비교할 때는 `Arrays.equals()`를 사용하자.
    - null도 정상 값으로 취급하는 참조 타입 필드이면 `Object.equals()`를 통해 **NPE**를 예방하자.
    - 최상의 성능을 바란다면 다를 가능성이 더 큰 필드 or 비교 비용이 비교적 싼 필드를 먼저 비교하자.
5. `equals`를 다 구현헀다면 세 가지만 자문해보자. 대칭적인가? 추이성이 있는가? 일관적인가?

추가로..

6. `equals`를 재정의할 땐 `hashCode`도 반드시 재정의하자 (아이템 11)
7. 너무 복잡하게 해결하려 들지 말자.
    - 필드 동치성만 검사해도 `equals` 규악을 어렵지 않게 지킬 수 있다.
    - **alias는** 비교하지 않는 게 좋다.

8. Object 외의 타입을 매개 변수로 받는 equals 메서드는 선언하지 말자.
    ```java
    // 잘못된 예
    public boolean equals(MyClass o) {
        ...
    }
    ```
9. AuthValue 프레임워크를 적극 활용하자.

구글이 만든 @AutoValue를 적극 활용하여, 우리가 구현하고자 하는 코드와 거의 유사하게 구현되므로 실수하지 말고 잘 사용하자.

## 총정리

꼭 필요한 경우가 아니라면 `equals`를 재정의하지 말자. 많은 경우의 `Object`의 `equals`가 대부분의 비교를 수행해준다.

재정의를 해야 할 때는 그 클래스의 핵심 필드를 빠짐없이, 다섯 가지 규약을 확실히 지켜가며 비교하자.