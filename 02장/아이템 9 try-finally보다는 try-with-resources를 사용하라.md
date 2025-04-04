# 아이템 9. try-finally보다는 try-with-resources를 사용하라

## 개요

자바 라이브러리에는 close 메서드를 호출해 직접 닫아줘야 하는 자원들이 많다.

`InputStream`, `OutputStream`, `java.sql.Connection` 등이 좋은 예다.

자원 닫기는 예측할 수 없는 성능 문제로 이어질 수 있기에,

안전망으로 `finalizer`를 활용하고 있다.

하지만 아이템 8에서 봤다시피 `finalizer`는 그리 믿을만하지 못하다.

## 자원의 닫힘 보장 수단: try-finally

전통적으로는 `try-finally`가 자원이 제대로 닫힘을 보장하도록 사용되었다.

`finally` 블록은 `try` 블록에서 예외나 메서드에서 반환되어도 작동함으로써 닫힘을 보장한다.

```java
import java.io.BufferedReader;
import java.io.FileReader;
import java.io.IOException;

public class TopLine {
    static String firstLineOfFile(String path) throws IOException {
        BufferedReader br = new BufferedReader(new FileReader(path));
        try {
            return br.readLine();
        } finally {
            br.close();
        }
    }
}
```

나쁘지는 않아 보인다.

하지만, **자원을 하나만 더 사용**해도 **코드는 매우 지저분**해진다.

```java
import java.io.*;

public class Copy {
    private static final int BUFFER_SIZE = 8 * 1024;

    static void copy(String src, String dst) throws IOException {
        InputStream in = new FileInputStream(src);
        try {
            OutputStream out = new FileOutputStream(dst);
            try {
                byte[] buf = new byte[BUFFER_SIZE];
                int n;
                while ((n = in.read(buf)) >= 0)
                    out.write(buf, 0, n);
            } finally {
                out.close();
            }
        } finally {
            in.close();
        }
    }
}
```

~~참 읽기 싫은 코드가 아닌가 싶다…~~

하지만 이런 지저분함에도 불구하고 미묘한 결점이 존재한다.

## try-finally 문의 결점

예외는 `try` 블록과 `finally` 블록 모두에서 발생할 수 있는 것이다.

```java
static String firstLineOfFile(String path) throws IOException {
      BufferedReader br = new BufferedReader(new FileReader(path));
      try {
          return br.readLine();
      } finally {
          br.close();
      }
  }
```

위 코드가 작동하던 중, 기기에 물리적인 문제가 생긴다면 

`br.readLine()` 메서드가 예외를 던지고, 같은 이유로 `br.close()` 메서드도 실패할 것이다.

그렇다면 어떻게될까?

<aside>
💡

이런 상황이라면 **두 번째 예외(`br.close()`의 예외)가 첫 번째 예외를 완전히 집어삼켜 버린다!**

→ 스택 추적 내역에서 **첫 번째 예외에 관한 정보는 남지 않게 되어, 디버깅이 매우 어려워진다**.

</aside>

  

## 해결책: try-with-resources의 출현(자바 7)

이 구조는 `AutoCloseable` 인터페이스를 구현함으로써 사용할 수 있다.

<aside>
💡

`AutoCloseable`?

: 단순히 `void`를 반환하는 `close()` 메서드 하나만 존재하는 인터페이스이다.

이미 많은 자바 라이브러리와 서드파티 라이브러리들 속 인터페이스가 `AutoCloseable`을 구현하거나 확장하고 있다. 

</aside>

만약에 자원을 닫아야하는 클래스를 작성한다면 반드시 `AutoCloseable`을 구현하도록 하자!

예제를 통해 **사용법**을 살펴보자.

```java
import java.io.BufferedReader;
import java.io.FileReader;
import java.io.IOException;

public class TopLine {
    static String firstLineOfFile(String path) throws IOException {
        try (BufferedReader br = new BufferedReader(
                new FileReader(path))) {
            return br.readLine();
        }
    }
}
```

위 코드는 코드 9-1을 `try-with-resources`를 적용시킨 모습이다.

```java
import java.io.*;

public class Copy {
    private static final int BUFFER_SIZE = 8 * 1024;
    
    static void copy(String src, String dst) throws IOException {
        try (InputStream   in = new FileInputStream(src);
             OutputStream out = new FileOutputStream(dst)) {
            byte[] buf = new byte[BUFFER_SIZE];
            int n;
            while ((n = in.read(buf)) >= 0)
                out.write(buf, 0, n);
        }
    }
}
```

위 코드는 코드 9-2를 `try-with-resource`를 적용시킨 모습이다.

코드만 보았을 때도 **확실히 가독성이 좋아졌을 뿐더러, 짧아졌다.**

**문제를 진단하기도 훨씬 좋은데**

코드 9-3에서 `readLine()`과 `AutoCloseable`을 구현함으로써 숨겨져 작동하는 `close()` 둘 다에서 예외가 발생한다면, **`readLine()`의 예외만 기록된다.**

`close()` 예외는 버려지진 않고 숨겨지는데, 

스택 추적 내역에 `suppressed`라는 꼬리표를 달고 출력된다.

## catch 절의 사용

`try-finally`에서처럼 `try-with-resources`에서도 `catch` 절의 사용이 가능하다.

덕분에 다수의 예외를 `try` 문의 중첩없이 처리해낼 수 있다.

```java
import java.io.BufferedReader;
import java.io.FileReader;
import java.io.IOException;

public class TopLineWithDefault 
    static String firstLineOfFile(String path, String defaultVal) {
        try (BufferedReader br = new BufferedReader(
                new FileReader(path))) {
            return br.readLine();
        } catch (IOException e) {
            return defaultVal;
        }
    }
}
```

## 정리

<aside>
💡

꼭 회수해야 하는 자원을 다룰 때는 `try-finally` 대신, `try-with-resources`를 사용하도록 하자. **어떠한 경우라도 말이다.**

**코드는 더 짧아지고, 분명해지고, 예외 정보도 훨씬 유용해질 것이다!**

</aside>