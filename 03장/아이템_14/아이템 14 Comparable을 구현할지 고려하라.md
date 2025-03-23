# 아이템 14 Comparable을 구현할지 고려하라

##  compareTo란?

`Object`의 메서드가 아니며, 두 가지만 제외하면 `Object`의 `equals`와 같다.

1. compareTo는 단순 동치성 비교에 더해 순서까지 비교 가능
2. 구현하고 나면, `Arrays.sort(arr)` 처럼 손쉽게 정렬 가능

예로, `String`이 `Comparable`을 구현했기 때문에 String 객체들은 알파벳 순서로 손쉽게 정렬이 가능하다.

## compareTo 메서드의 일반 규약

객체가 주어진 객체보다 작으면 음, 같으면 0, 크면 양의 정수를 반환. (비교할 수 없는 타입이면 `ClassCastException`)

### 1. Comparable을 구현한 클래스는, 모든 x,y에 대해 sgn(x.compareTo(y)) == -sgn(y.compareTo(x))여야 한다.
### 2. Comparable을 구현한 클래스는, 추이성을 보장해야 한다. **(x.compareTo(y) > 0 && y.compareTo(z) > 0) 이면 x.compareTo(z) > 0이다.**
### 3. (x.compareTo(y) == 0) == (x.equals(y))여야 한다. 크기가 같은 객체들끼리는 어떤 객체와 비교해도 항상 같아야 한다.

이 규약을 지키지 못 하면, 비교를 활용하는 클래스에서 활용될 수 없다. (`TreeSet`, `TreeMap`, `Collections`, `Arrays` 등)

이상의 세 규약은 `compareTo` 메서드로 수행하는 동치성 검사도 `equals`와 똑같이 **반사성**, **대칭성**, **추이성**을 충족해야 함을 뜻한다.

> 그렇기에 기존의 주의점과 똑같이, 확장한 클래스에서 새로운 값 컴포넌트를 추가하면 compareTo 규약을 지킬 방법이 없다. 대신 독립된 클래스를 두고, 이 클래스에 원래 클래스를 가리키는 필드를 두어 내부 인스턴스를 반환하는 '뷰' 메서드를 제공하면 된다. 이렇게 하면 바깥 클래스에서 우리가 원하는 compareTo 메서드를 구현할 수 있다.

### 4. compareTo 메서드로 수행한 동치성 테스트의 결과가 equals와 결과가 같아야 한다.

이를 잘 지키면 `compareTo`로 줄지은 순서와, `equals`의 결과가 일관되게 한다.

## compareTo 메서드 작성 요령

- `Comparable`은 타입을 인수로 받는 제네릭 인터페이스이므로, compareTo의 인수 타입은 컴파일 타임에 정해진다. (형변환할 필요 없음)
- 각 필드가 동치임을 비교하는 게 아니라, 순서를 비교한다. 객체 참조 필드를 비교하려면, `compareTo` 메서드를 재귀적으로 호출한다.
  - `Comparable`을 구현하지 않은 필드나 표준이 아닌 순서로 비교한다면, `Comparator`를 사용한다.

### Comparator 사용 예: 아이템 10 CaseInsensitiveString

```java
public final class CaseInsensitiveString implements Comparable<CaseInsensitiveString> {
    public int compareTo(CaseInsensitiveString cis) {
        return String.CASE_INSENSITIVE_ORDER.compare(s, cis.s);
    }
}
```

자바에서 제공하는 compare를 사용한다. 다만, `Comparable<CaseInsensitiveString>`을 구현하여 `CaseInsensitiveString의` 참조는 `CaseInsensitiveString` 참조와만 비교할 수 있다는 뜻이다.

> 추가: 박싱 타입 비교 시, ">", "<"를 사용하지 말고 Long.compare, Integer.compare, Short.compare, Float.compare, Double.compare 등을 사용하자.

### Comparator 인터페이스의 비교자 생성 메서드(comparator construction method)

자바 8부터, 메서드 연쇄 방식으로 비교자를 생성할 수 있다. (간결하지만, 성능 저하가 있음)

예제를 보자.

```java
private static final Comparator<PhoneNumber> COMPARATOR = comparingInt((PhoneNumber pn) -> pn.areaCode)
.thenComparingInt(pn -> pn.prefix)
.thenComparingInt(pn -> pn.lineNum);

public int compareTo(PhoneNumber pn) {
    return COMPARATOR.compare(this. pn);
}
```
`PhoneNumber` 클래스에서 `areaCode`, `prefix`, `lineNum` 세 필드를 기준으로 정렬하고 싶을 때, 각각의 필드에 대해 `comparingInt`와 `thenComparingInt` 메서드를 연달아 호출하여 하나의 비교자를 만들 수 있다.

이 비교자는 먼저 `areaCode`를 비교하고, 같으면 `prefix`, 그다음 `lineNum`을 비교하는 방식으로 작동한다. 이렇게 생성한 비교자를 `compareTo` 메서드에서 사용하면, `Comparable` 인터페이스를 효율적으로 구현할 수 있다.

`Comparator`는 이렇게 수많은 보조 생성 메서드들로 중무장하고 있다. (`comparingInt`, `thenComparingInt`)

### 객체 참조용 비교자 생성 메서드

comparing이라는 정적 메서드는 2개가 다중정의되어 있다.<br>
(줄글로 이해가 어려우니 예제 코드와 함께 설명. 단순히 정렬 비교자를 같이 쓰느냐 마느냐의 차이이다.)

키 추출자(Function)를 인수로 받아, 해당 키의 **자연적 순서(natural order)**를 기준으로 비교자를 생성한다.
→ 예: `Comparator.comparing(Person::getName)` — 이름의 알파벳 순으로 정렬

키 추출자(Function)와 키에 대한 **비교자(Comparator)**를 함께 인수로 받아, 주어진 비교자를 사용하여 비교자를 생성한다.
→ 예: `Comparator.comparing(Person::getName, String.CASE_INSENSITIVE_ORDER)` — 대소문자 무시하고 이름 정렬

### 부차 비교자를 위한 메서드
thenComparing이라는 인스턴스 메서드는 3개가 다중정의되어 있다.

비교자(`Comparator`) 하나를 인수로 받아, 해당 비교자를 부차 비교자로 사용한다.
→ 예: `.thenComparing(Comparator.comparing(Person::getAge))` — 이름이 같을 경우 나이로 정렬

키 추출자(Function) 하나를 인수로 받아, 해당 키의 자연적 순서를 기준으로 부차 비교자를 생성한다.
→ 예: `.thenComparing(Person::getAge)` — 나이 오름차순으로 보조 정렬

키 추출자(Function)와 해당 키에 대한 **비교자(Comparator)**를 함께 인수로 받아, 주어진 비교자를 기준으로 부차 비교자를 생성한다.
→ 예: `.thenComparing(Person::getAge, Comparator.reverseOrder())` — 나이 내림차순으로 보조 정렬

## 정리

**순서를 고려해야 하는 값 클래스를 작성한다면, Comparable 인터페이스를 반드시 구현하자.**

**compareTo 메서드에서 필드 값 비교할 때 "<" ,">"는 쓰지 말자.** <br>
**-> 대신 박싱된 기본 타입 클래스가 제공하는 정적 compare 메서드, Comparator 인터페이스가 제공하는 비교자 생성 메서드를 사용하자.**