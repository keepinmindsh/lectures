# NULL에 대해서 
- JVM 언어 전쟁 
    - 그루비, jRuby, Jython (동적타이핑,스트립팅) -> Scala, Clojure (함수형 프로그래밍) -> Ceylon, Kotlin(널 안정성)


- null 참조 
    - 레코드 핸들링 : 객체 지향의 시초가 된 논문
    - 특별한 값이 없음을 나태려고 null을 도입했고 이 값을 사용하려고 할 대 오류를 내도록 설계함 
    - 두 참조값이 null일 때 두 참조는 동일하다고 판단함.


- 자바의 null 참조 
    - 의미가 모호함 : 초기화되지 않음, 저으이되지 않음, 값이 없음 
    - 모든 참조의 기본 상태 ( 값 )
    - 모든 참조는 null 가능

대부분의 에러 중에 상위를 차지하는 것이 Null Pointer임. 


# Null을 안전하게 다루는 방법 

### 단정문 ( Assertion )

- assert 식1 ;
- assert 식1 : 식2;

- 부울식인 식1의 거짓이면 AssersionError 발생 
- 식2는 AssetionError에 포함될 상세 정보를 만드는 생성식 
- 공개 메서드에는 사용하지 말아야함 
- -enableassertions 또는 -ea 옵션 활성화 

```java

// 아래의 구문은 운영시에는 기본적으로 무시가 되기 때문에 운영에서는 사용을 위해서 옵션 설정이 필요함. 
class Sample {
  private void setRefreshInterval(int interval){
    assert interval > 0 && interval <= 1000/MAX_REFRESH_RATE : interval;
  }
}

```

### 자바 기본 장치 : java.util.Objects

- 자바8
  - isNull(Object obj)
  - nonNull(Object obj)
  - requireNonNull(T obj)
  - requireNonNull(T obj, String message)
  - requireNonNull(T obj, Supplier<String> messageSupplier)

- 자바9
  - requireNonNullElse(T obj, T defaultObj)
  - requireNonNullElseGet(T obj, Supplier<? extend T> supplier)


### Java.util.Optional 

- 절대로 optional 변수와 **반환 값에 null을 사용하지 마라** 
- Optional에 값이 들어 있다는 걸 확신하지 않는 한 Optional.get()을 쓰지 말라
- Optional.isPresent()이나 Optional.get()외 API를 가능한 사용하라. 
- Optional에서 여러 메서드를 연속해서 호출하고 값을 얻기 위해서 Optional을 생성하는 건 권장할만하지 않다. 
- Optional로 값을 처리하는 중에 그 안에 중간 값을 처리하기 위해서 또 다른 Optional이 사용되면 너무 복잡해진다. 
- Optional을 **필드, 메서드 매개변수, 집합 자료형에 쓰지 말라** 
- 집합 자료형(List, Set, Map)을 감싸는데 Optional을 쓰지말고 빈 집합을 사용해라.

### null을 잘 쓰는 법 

- **API(매개변수)에 null을 최대한 쓰지 말아라.** 
  - null로 지나치게 유연한 메서드를 만들지 말고 명시적인 메소드를 만들어라. 
  - null을 반환하지 말라 
    - 반환 값이 꼭 있어야 한다면 null을 반환하지 말고 예외를 던져라. 
    - 빈 반환 값은 빈 컬렉션이나 'Null 객체'를 활용하라. 
    - 반환 값이 없을 수도 있다면 Optional을 반환하라. 
  - 선택적 매개 변수는 null 대신 다형성 ( 메서드 추가 정의; overload )를 사용해서 표현하라.
- **사전조건과 사후 조건을 확인하라. 계약에 의한 설계 ( design by contract )**
  - API 소비자와 제공자 사이의 지켜져야 할 규약을 지켜야할 계약으로 여기는 설계 방법 
  - 형식적 규왕 외에 사전 조건과 사후 조건과 유지 조건을 포함 
  - 베트르링 마이어 ( Bertrand Meyer ) - 에펠(Eiffel) 프로그래밍 언어 제작 
  - 개방-폐쇄 원칙의 상위 개념
  - 자바의 계약에 의한 설계 
    - Interface + Java Doc 
    - 사전조건 = 보호절 ( Guard Clause )
      - 단정문
      - Objects의 메서드 
      - IllegalArgumentException, NullPointerException
    - 자바 라이브러리 
      - 스프링 Assert 클래스 
      - 구아바 Preconditions 클래스 
      - valid4j + hamcrest
        - http://www.valid4j.org/
      - AssertJ Preconditions 클래스 
      - Bean Validation 
      - Cofoja
        - https://github.com/nhatminhle/cofoja
- **null의 범위를 지역에 제한하라.**
  - 기본 문제 해결 원칙 : 큰 문제는 제어 가능한 작은 문제로 나누어 정복하고 다시 통합한다. 
  - 상태와 비슷하게 null도 지역적으로 제한할 경우 큰 문제가 안된다. 
  - 클래스와 메서드를 작게 만들어라 
  - 설계가 잘된 코드에서는 널의 위험도 약해진다. 

- (상태와 같이) null 의 범위릘 지역(클래스, 메서드)에 제한하라. 
- 초기화를 명확히 하라. 

### null 안전한 언어들 

- null을 안전하고 쉽게 다루게 해주는 엘비스 연산자 
  - C# : null 조건 연산자 ( ?. 과 ?[] )
  - 그루비(Groovy) : def name = person?.name
  - 코틀린 : ?. 과 ?
  - 스위프트 : Optional Chaining & 가드 (Guard) 뿐 

- null이 예외인 언어 
  - 코틀린 : null 가능 타입과 non-null 타입 
  - 스위프트 : Optional 
  - C# 8.0
  
### JSR-308 타입 어노테이션 

- 선언부가 아닌 타입 지저 위치에 어노테이션 추가 기능 
- 어노테이션 프로세싱을 통한 빈약한 자바 타입 시스템을 강화 
- 초안 제출 2006/10/17, 최종안 승인 2014/2/18 자바 8에 추가 
- 워싱턴대 마이클 에슬스트 교수 주도 
- Checker Framework와 동시에 진행 

### Checker Framework 

- null 안정성 확인   
  @Nullable, @NonNull, @PolyNull
- Map키, 잠금, 순차 자료형(배열, List 등) 색인 값, 정규식, 문자열 형식, 단위 등 다수 확인 기능 제공 
- 자작 확인 기능 추가 가능 
- 특정 환경이나 IDE 독립적


- 기본 null 정책 
  - 과도한 어노테이션 사용 예정 
  - 기본 @NonNull  
    필드, 매개변수, 반환 값등 
  - 예외적 @Nullable  
    지역변수, 타입 캐스크 등 
  - 패키지, 클래스 수정 정책 설정  
    @DefaultQualifier

```java

package bong.lines.checker;

public class CheckSample {
    public static void main(String[] args) {
        
        DataForChecker dataForChecker = DataForChecker.builder()
                .name1("For Test1")
                .build();


        System.out.println("dataForChecker.getName1() = " + dataForChecker.getName1());
        
    }
}


```

```java

package bong.lines.checker;

import lombok.Builder;
import lombok.Getter;
import org.springframework.lang.NonNull;

@Builder
@Getter
public class DataForChecker {
    private final @NonNull String name1;
    private final @NonNull String name2;
    private final @NonNull String name3;

}

```

- Result 

```shell

> Task :checkframework:CheckSample.main() FAILED
Exception in thread "main" java.lang.NullPointerException: name2 is marked non-null but is null
	at bong.lines.checker.DataForChecker.<init>(DataForChecker.java:7)
	at bong.lines.checker.DataForChecker$DataForCheckerBuilder.build(DataForChecker.java:7)
	at bong.lines.checker.CheckSample.main(CheckSample.java:8)

Execution failed for task ':checkframework:CheckSample.main()'.
> Process 'command 'C:/Program Files/JetBrains/IntelliJ IDEA 2020.3.2/jbr/bin/java.exe'' finished with non-zero exit value 1

* Try:
Run with --stacktrace option to get the stack trace. Run with --info or --debug option to get more log output. Run with --scan to get full insights.


```

- 패키지, 클래스 수준 기본 정책 설정 
  - @DefaultQualifier 
  - 패키지 ( package-info.java ) 나 클래스 전체의 기본 정책 설정 

```java

@DefaultQualifier(value= NonNull.class, locations = TypeUseLocation.LOCAL_VARIABLE)
package bong.lines.defaultqualifier;

```

```java

import org.springframework.lang.NonNull;

@DefaultQualifier(value= NonNull.class, locations = TypeUseLocation.FIELD)
public class DateForDefaultQualifier {
    Object nullableField = null;
    @NonNull Object nonNullField = new Object();
}

```

### 자동 타입 개선(Automatic type refinement)

- 단순한 정적 타입 확인이 아닌 코드 흐름과 실행 결과를 반영 
- 코드로 null 확인을 한 경우 @nonNull로 취급
- 메서드 내부로 제한

```java
package bong.lines.checker;

import java.io.Console;

import static java.util.Objects.nonNull;

public class AutomaticTypeRefinement {
    public static void main(String[] args) {
        Console console = System.console();

        char[] value = nonNull(console) ? console.readPassword() :  new char[0];
    }
}

```

# 강연 영상 

> https://www.youtube.com/watch?v=vX3yY_36Sk4
