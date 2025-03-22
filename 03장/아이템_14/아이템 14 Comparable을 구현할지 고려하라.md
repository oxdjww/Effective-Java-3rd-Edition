# 아이템 14 Comparable을 구현할지 고려하라

##  compareTo란?

`Object`의 메서드가 아니며, 두 가지만 제외하면 `Object`의 `equals`와 같다.

1. compareTo는 단순 동치성 비교에 더해 순서까지 비교 가능
2. 구현하고 나면, `Arrays.sort(arr)` 처럼 손쉽게 정렬 가능

예로, String이 Comparable을 구현했기 때문에 String 객체들은 알파벳 순서로 손쉽게 정렬이 가능하다.

## compareTo 메서드의 일반 규약

1. 이 객체가 주어진 객체보다 작으면 음, 같으면 0, 크면 양의 정수를 반환. (비교할 수 없는 타입이면 ClassCastException)
2. 