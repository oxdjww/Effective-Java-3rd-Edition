# 아이템 11. equals를 재정의하려거든 hashCode도 재정의하라

## `equals`를 재정의한 클래스 모두에서 `hashCode`도 재정의해야한다.

그렇지 않는다면 `hashCode` 일반 규약을 어기게 되어 인스턴스를 **`HashMap`이나 `HashSet`의 원소로 사용할 때 문제를 일으킬 것**이다!

## `hashCode` 일반 규약

- `equals` 비교에 사용되는 정보가 변경되지 않았다면, 애플리케이션이 실행되는 동안 그 객체의 `hashCode` 메서드는 몇 번을 호출해도 일관되게 항상 같은 값을 반환해야 한다.

<aside>
💡

단, 애플리케이션을 다시 실행한다면 달라져도 상관없다.

</aside>

- `equals(Object)`가 두 객체를 같다고 판단했다면, 두 객체의 `hashCode`는 똑같은 값을 반환해야 한다.
- `equals(Object)`가 두 객체를 다르다고 판단했더라도, 두 객체의 `hashCode`가 서로 다른 값을 반환할 필요는 없다.

<aside>
💡

단, 다른 객체에 대해서는 다른 값을 반환해야 해시테이블이 좋아진다.

</aside>

이 중 중요한 규약은 두 번째 규약이다. 

코드를 통해서 `hashCode`를 재정의하지 않았을 때 결과를 확인해보자!

```java
public final class PhoneNumber {
    private final short areaCode, prefix, lineNum;

    public PhoneNumber(int areaCode, int prefix, int lineNum) {
        this.areaCode = rangeCheck(areaCode, 999, "area code");
        this.prefix   = rangeCheck(prefix,   999, "prefix");
        this.lineNum  = rangeCheck(lineNum, 9999, "line num");
    }

    @Override public boolean equals(Object o) {
        if (o == this)
            return true;
        if (!(o instanceof PhoneNumber))
            return false;
        PhoneNumber pn = (PhoneNumber)o;
        return pn.lineNum == lineNum && pn.prefix == prefix
                && pn.areaCode == areaCode;
    }

    public static void main(String[] args) {
        Map<PhoneNumber, String> m = new HashMap<>();
        m.put(new PhoneNumber(707, 867, 5309), "제니");
        System.out.println(m.get(new PhoneNumber(707, 867, 5309)));
    }
}
```

위의 코드를 보면 `PhoneNumber`의 필드 값이 똑같기에 “제니”가 출력되리라고 생각되지만,

실제로는 `null`이 출력된다.

이는 논리적으로 동치인 두 객체일지라도 `hashCode`를 재정의하지 않았기에 

`HashMap`에서 발생하는 증상이다.

그렇다면 `hashCode`를 어떻게 재정의해야할까?

## `hashCode`의 구현(최악의 구현)

```java
@Override
public int hashCode(){
	return 42;
}
```

위 코드는 모든 객체에게 같은 `hashCode`를 반환하므로 위의 문제는 해결될것이다.

> 다만, 모든 객체에게 똑같은 값만 반환하므로 해시테이블의 버킷 하나에 모든 객체가 담겨 **O(1)**이어야 할 테이블이 **O(n)**의 성능으로 작동한다.
> 

> **좋은 해시 함수는 서로 다른 인스턴스들을 32비트 정수 범위에 균일하게 분배**해야한다.
> 

좋은 `hashCode`를 작성하는 요령에 대해 알아보자.

## 좋은 hashCode 작성법

1. **int 변수 `result`를 선언한 후 값 `c`로 초기화**한다.

<aside>
💡

※ `c`: 해당 객체의 첫 번째 핵심 필드를 단계 2.a 방식으로 계산한 해시코드

※ 핵심 필드: `equals` 비교에 사용되는 필드

</aside>

1. 해당 객체의 나머지 핵심 필드 `f` 각각에 대하여 다음 작업을 수행한다.
    1. **해당 필드의 해시코드 `c`를 계산**한다.
        
        1)기본 타입 필드: `Type.hashCode(f)`
        
        <aside>
        💡
        
        ※ `Type`: 해당 기본 타입의 박싱 클래스
        
        </aside>
        
        2)참조 타입 필드(`equals`가 필드의 `equals`를 재귀적으로 호출할 때)
        
        :  필드의 `hashCode`를 재귀적으로 호출
        
        3)필드가 배열이라면
        
        : 핵심 원소 각각을 별도 필드처럼 다룬다.
        
        <aside>
        💡
        
        핵심 원소가 없다면? → 단순한 상수(0)를 사용
        
        모든 원소가 핵심 원소라면? → `Arrays.hashCode` 사용
        
        </aside>
        
    2. 단계 2.a에서 계산한 **해시코드 `c`로 `result`를 갱신한다.**
    
    ```java
    result = 31 * result + c;
    ```
    
2. **`result`를 반환**한다.

코드 작성이 끝나면 단위 테스트를 작성해 확인하도록 하자!

<aside>
💡

해시 코드를 계산할 필요가 없는 것

- 파생 필드: 다른 필드로부터 계산해낼 수 있음
- equals 비교에 사용되지 않은 필드: **반드시** 제외할 것
</aside>

## 31을 곱하는 이유

`31 * result`는 필드를 곱하는 순서에 따라 `result` 값이 크게 달라지게 한다.

또한 31은 **홀수이면서 소수**이다.

- 홀수인 점: 오버플로가 발생해도 정보가 남아있다.
- 소수인 점: 전통적인 이유

→ `31 * i`는 `(i << 5) -  i` 로 치환되어 최적화된다.

그럼 이제 이걸 예제에 적용해보자.

## 좋은 hashCode 예제

```java
@Override 
public int hashCode() {
    int result = Short.hashCode(areaCode);
    result = 31 * result + Short.hashCode(prefix);
    result = 31 * result + Short.hashCode(lineNum);
    return result;
}
```

이 방법은 best는 아니지만 꽤 훌륭하다.

자바 플랫폼 라이브러리에서 제공하는 메서드와 비교해도 손색이 없다.

해시 코드 충돌을 더 줄이고 싶다면 com.google.cmmon.hash.Hashing을 참고하자.

## 짧은 hashCode 메서드(성능 이슈)

```java
@Override public int hashCode() {
		return Objects.hash(lineNum, prefix, areaCode);
}
```

Objects에서 제공하는 hash 메서드를 활용하는 방법이다.

이 방법은 속도는 느린데,

- 입력 인수를 담기 위한 배열이 만들어질뿐더러
- 입력 중 기본 타입이 있다면 박싱과 언박싱도 거쳐야한다.

<aside>
💡

성능에 민감하지 않은 상황에서만 사용하도록 하자.

</aside>

## 로직의 성능을 향상시키는 방법

만약 클래스가 불변이고 해시코드를 계산하는 비용이 크다면, **캐싱**을 고려해볼 수 있다.

만약 객체가 **해시의 키로 사용될 것 같다면?**

→  **인스턴스가 만들어질 때** 해시코드를 계산하자.

해시의 **키로 사용되지 않는 경우라면?**

→ **지연 초기화(lazy initialization)** 전략도 좋다.

> 단, 지연 초기화를 사용할 땐 스레드 안정성도 고려하자.
> 

```java
private int hashCode; // 자동으로 0으로 초기화된다.

@Override public int hashCode() {
    int result = hashCode;
    if (result == 0) {
        result = Short.hashCode(areaCode);
        result = 31 * result + Short.hashCode(prefix);
        result = 31 * result + Short.hashCode(lineNum);
        hashCode = result;
    }
    return result;
}
```

## 주의할 점

### 1. 성능을 높이려고 해시코드를 계산할 때 핵심 필드를 생략하지 마라.

equals 비교에서 사용한 핵심 필드들은 해시 테이블을 고루 퍼뜨려주기 때문에 ****반드시 생략하지 마라.

### 2. hashCode가 반환하는 값의 생성 규칙을 API 사용자에게 자세히 공표하지 마라.

1. 그래야 클라이언트가 이 값에 의지하지 않게 되고,
2. 추후에 계산 방식을 바꿀 수도 있다.

자바 라이브러리에는 `String`과 `Integer` 등 자세히 공표된 게 많은데, 이건 바람직하지 않다.

## 정리하며

- `equals`를 재정의하면 반드시 `hashCode`도 재정의하자.
- `hashCode` 규약을 지켜서 재정의하라.
- 프레임워크나 IDE에서 제공하는 구현 기능을 사용하는 것도 편리한 방법이다.