## Effective Java 3/E 스터디

<img src="https://github.com/user-attachments/assets/1bad42e2-141c-4ac8-b0e0-ec007a458e1a" style="width: 200px; height: auto;">


### 참여자

| <a href="https://github.com/oxdjww">태태 | <a href="https://github.com/Dimo-2562"> 모리 |
| --- | --- |
| <img src="https://github.com/oxdjww.png" width="100"> | <img src="https://github.com/Dimo-2562.png" width="100"> |

### 일정

- `2025.3.09(SUN)` ~ 

### 규칙

- 매일 아래와 같은 규칙으로 스터디를 진행한다.
    1. [태태/모리]는 1 아이템을 정독, 정리, 본인 `branch`에 정리 내용을 올리고 `pull request`를 넣는다.
    2. [모리/태태]는 그 다음날까지 `pull request`를 확인 후 리뷰를 남기고 `merge` 후, 역할을 바꾸어 1번을 반복한다.
- 본인의 역할을 수행하지 않은 참여자는 벌금 `5,000`을 즉시 납부한다.

### 목차

- [x] 1. 생성자 대신 정적 팩터리 메서드를 고려하라 (모리)  
- [x] 2. 생성자에 매개변수가 많다면 빌더를 고려하라 (태태)  
- [x] 3. private 생성자나 열거 타입으로 싱글턴임을 보증하라 (모리)  
- [x] 4. 인스턴스화를 막으려거든 private 생성자를 사용하라 (태태)  
- [x] 5. 자원을 직접 명시하지 말고 의존 객체 주입을 사용하라 (모리)  
- [x] 6. 불필요한 객체 생성을 피하라 (태태)  
- [x] 7. 다 쓴 객체 참조를 해제하라 (모리)  
- [x] 8. finalizer와 cleaner 사용을 피하라 (태태)  
- [x] 9. try-finally보다는 try-with-resources를 사용하라 (모리)  
- [x] 10. equals는 일반 규약을 지켜 재정의하라 (태태)  
- [x] 11. equals를 재정의하려거든 hashCode도 재정의하라 (모리)  
- [x] 12. toString을 항상 재정의하라 (태태)  
- [x] 13. clone 재정의는 주의해서 진행하라 (모리)  
- [x] 14. Comparable을 구현할지 고려하라 (태태)  
- [x] 15. 클래스와 멤버의 접근 권한을 최소화하라 (모리)  
- [x] 16. public 클래스에서는 public 필드가 아닌 접근자 메서드를 사용하라 (태태)  
- [ ] 17. 변경 가능성을 최소화하라 (모리)  
- [ ] 18. 상속보다는 컴포지션을 사용하라 (태태)  
- [ ] 19. 상속을 고려해 설계하고 문서화하라. 그러지 않았다면 상속을 금지하라 (모리)  
- [ ] 20. 추상 클래스보다는 인터페이스를 우선하라 (태태)  
- [ ] 21. 인터페이스는 구현하는 쪽을 생각해 설계하라 (모리)  
- [ ] 22. 인터페이스는 타입을 정의하는 용도로만 사용하라 (태태)  
- [ ] 23. 태그 달린 클래스보다는 클래스 계층구조를 활용하라 (모리)  
- [ ] 24. 멤버 클래스는 되도록 static으로 만들라 (태태)  
- [ ] 25. 톱레벨 클래스는 한 파일에 하나만 담으라 (모리)  
- [ ] 26. 로 타입은 사용하지 말라 (태태)  
- [ ] 27. 비검사 경고를 제거하라 (모리)  
- [ ] 28. 배열보다는 리스트를 사용하라 (태태)  
- [ ] 29. 이왕이면 제네릭 타입으로 만들라 (모리)  
- [ ] 30. 이왕이면 제네릭 메서드로 만들라 (태태)  
- [ ] 31. 한정적 와일드카드를 사용해 API 유연성을 높이라 (모리)  
- [ ] 32. 제네릭과 가변인수를 함께 쓸 때는 신중하라 (태태)  
- [ ] 33. 타입 안전 이종 컨테이너를 고려하라 (모리)  
- [ ] 34. int 상수 대신 열거 타입을 사용하라 (태태)  
- [ ] 35. ordinal 메서드 대신 인스턴스 필드를 사용하라 (모리)  
- [ ] 36. 비트 필드 대신 EnumSet을 사용하라 (태태)  
- [ ] 37. ordinal 인덱싱 대신 EnumMap을 사용하라 (모리)  
- [ ] 38. 확장할 수 있는 열거 타입이 필요하면 인터페이스를 사용하라 (태태)  
- [ ] 39. 명명 패턴보다 애너테이션을 사용하라 (모리)  
- [ ] 40. @Override 애너테이션을 일관되게 사용하라 (태태)  
- [ ] 41. 정의하려는 것이 타입이라면 마커 인터페이스를 사용하라 (모리)  
- [ ] 42. 익명 클래스보다는 람다를 사용하라 (태태)  
- [ ] 43. 람다보다는 메서드 참조를 사용하라 (모리)  
- [ ] 44. 표준 함수형 인터페이스를 사용하라 (태태)  
- [ ] 45. 스트림은 주의해서 사용하라 (모리)  
- [ ] 46. 스트림에서는 부작용 없는 함수를 사용하라 (태태)  
- [ ] 47. 반환 타입으로는 스트림보다 컬렉션이 낫다 (모리)  
- [ ] 48. 스트림 병렬화는 주의해서 적용하라 (태태)  
- [ ] 49. 매개변수가 유효한지 검사하라 (모리)  
- [ ] 50. 적시에 방어적 복사본을 만들라 (태태)  
- [ ] 51. 메서드 시그니처를 신중히 설계하라 (모리)  
- [ ] 52. 다중정의는 신중히 사용하라 (태태)  
- [ ] 53. 가변인수는 신중히 사용하라 (모리)  
- [ ] 54. null이 아닌, 빈 컬렉션이나 배열을 반환하라 (태태)  
- [ ] 55. 옵셔널 반환은 신중히 하라 (모리)  
- [ ] 56. 공개된 API 요소에는 항상 문서화 주석을 작성하라 (태태)  
- [ ] 57. 지역변수의 범위를 최소화하라 (모리)  
- [ ] 58. 전통적인 for 문보다는 for-each 문을 사용하라 (태태)  
- [ ] 59. 라이브러리를 익히고 사용하라 (모리)  
- [ ] 60. 정확한 답이 필요하다면 float와 double은 피하라 (태태)  
- [ ] 61. 박싱된 기본 타입보다는 기본 타입을 사용하라 (모리)  
- [ ] 62. 다른 타입이 적절하다면 문자열 사용을 피하라 (태태)  
- [ ] 63. 문자열 연결은 느리니 주의하라 (모리)  
- [ ] 64. 객체는 인터페이스를 사용해 참조하라 (태태)  
- [ ] 65. 리플렉션보다는 인터페이스를 사용하라 (모리)  
- [ ] 66. 네이티브 메서드는 신중히 사용하라 (태태)  
- [ ] 67. 최적화는 신중히 하라 (모리)  
- [ ] 68. 일반적으로 통용되는 명명 규칙을 따르라 (태태)  
- [ ] 69. 예외는 진짜 예외 상황에만 사용하라 (모리)  
- [ ] 70. 복구할 수 있는 상황에는 검사 예외를, 프로그래밍 오류에는 런타임 예외를 사용하라 (태태)  
- [ ] 71. 필요 없는 검사 예외 사용은 피하라 (모리)  
- [ ] 72. 표준 예외를 사용하라 (태태)  
- [ ] 73. 추상화 수준에 맞는 예외를 던지라 (모리)  
- [ ] 74. 메서드가 던지는 모든 예외를 문서화하라 (태태)  
- [ ] 75. 예외의 상세 메시지에 실패 관련 정보를 담으라 (모리)  
- [ ] 76. 가능한 한 실패 원자적으로 만들라 (태태)  
- [ ] 77. 예외를 무시하지 말라 (모리)  
- [ ] 78. 공유 중인 가변 데이터는 동기화해 사용하라 (태태)  
- [ ] 79. 과도한 동기화는 피하라 (모리)  
- [ ] 80. 스레드보다는 실행자, 태스크, 스트림을 애용하라 (태태)  
- [ ] 81. wait와 notify보다는 동시성 유틸리티를 애용하라 (모리)  
- [ ] 82. 스레드 안전성 수준을 문서화하라 (태태)  
- [ ] 83. 지연 초기화는 신중히 사용하라 (모리)  
- [ ] 84. 프로그램의 동작을 스레드 스케줄러에 기대지 말라 (태태)  
- [ ] 85. 자바 직렬화의 대안을 찾으라 (모리)  
- [ ] 86. Serializable을 구현할지는 신중히 결정하라 (태태)  
- [ ] 87. 커스텀 직렬화 형태를 고려해보라 (모리)  
- [ ] 88. readObject 메서드는 방어적으로 작성하라 (태태)  
- [ ] 89. 인스턴스 수를 통제해야 한다면 readResolve보다는 열거 타입을 사용하라 (모리)  
- [ ] 90. 직렬화된 인스턴스 대신 직렬화 프록시 사용을 검토하라 (태태)  
