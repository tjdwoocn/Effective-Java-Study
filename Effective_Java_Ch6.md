## Ch6. 열거타입과 애너테이션
---
<br>

## item 38. 확장할 수 있는 열거 타입이 필요하면 인터페이스를 사용하라
        Emulate extensible enums with interfaces

---

**열거 타입은 확장할 수 없다**

- 열거 타입 자체는 확장할 수 없지만, **인터페이스와 그 인터페이스를 구현하는 기본 열거 타입을 함께 사용해 같은 효과를 낼 수 있음**.

```java
public interface Operation {
    double apply(double x, double y);
}

public enum BasicOperation implements Operation {
    PLUS("+") {
        public double apply(double x, double y) { return x + y; }
    },
    MINUS("-") {
        public double apply(double x, double y) { return x - y; }
    },
    TIMES("*") {
        public double apply(double x, double y) { return x * y; }
    },
    DIVIDE("/") {
        public double apply(double x, double y) { return x / y; }
    };

    private final String symbol;
    BasicOperation(String symbol) { this.symbol = symbol; }
    @Override public String toString() { return symbol; }
}
```
- 우선 BasicOperation은 열거 타입이기 때문에 추가 확장은 불가능
- 하지만 인터페이스 Operation은 가능. 그렇기 때문에 다른 연산을 추가할 때는 아래와 같이 인터페이스를 구현한 새로운 열거 타입을 작성하면 됨

```java
public enum ExtendedOperation implements Operation {
    EXP("^") {
        public double apply(double x, double y) {
            return Math.pow(x, y);
        }
    },
    REMAINDER("%") {
        public double apply(double x, double y) {
            return x % y;
        }
    };

    private final String symbol;
    // 생성자, toString 생략
}
```

- 타입 수준에서도 기본 열거 타입 대신에 확장한 열거 타입을 넘겨서 열거 타입의 모든 원소를 순회하게 할 수 있음

```java
public static void main(String[] args) {
    double x = Double.parseDouble(args[0]);
    double y = Double.parseDouble(args[1]);
    test(ExtendedOperation.class, x, y);
}

private static <T extends Enum<T> & Operation> void test(Class<T> opEnumType, double x, double y) {
    for (Operation op : opEnumType.getEnumConstants()) {
        System.out.printf("%f %s %f = %f%n", x, op, y, op.apply(x, y));
    }
}
```

- 여기서 <T extends Enum<T> & Operation>는 Class 객체가 열거 타입인 동시에 Operation의 하위 타입임을 말함
- 즉, Enum 타입이면서 Operation을 구현한 클래스
- 위와 다르게 한정적 와일드카드 타입을 사용하는 방법도 있음
- 열거 타입의 리스트를 전달하여 한정적 와일드 카드 타입으로 지정

```java
public static void main(String[] args) {
    double x = Double.parseDouble(args[0]);
    double y = Double.parseDouble(args[1]);
    test(Arrays.asList(ExtendedOperation.values()), x, y);
}

private static void test(Collection<? extends Operation> opSet, double x, double y) {
    for (Operation op : opSet) {
        System.out.printf("%f %s %f = %f%n", x, op, y, op.apply(x, y));
    }
}
```

- 한편 열거 타입끼리 구현을 상속할 수 없는 문제 있음
- 열거 타입 간의 공유하는 기능이 늘어나 코드 중복량이 많아진다면 Helper 클래스 또는 메서드로 분리하는게 좋음

    <br>

## item 39. 명명 패턴보다 애너테이션을 사용하라
        Prefer annotations to naming patterns

---

### 명명 패턴의 단점

- JUnit은 3버전까지 테스트 메서드 이름이 test로 시작해야 했음 (명명패턴)
- 그러나 이러한 설정 시 몇몇 단점이 존재
    - 오타가 나면 안되며 오타 발생시 메서드가 무시됨
    - 의도하지 않은 곳에서 사용될 수 있음
        - 적절한 프로그램 요소에서만 사용될거란 보장이 없음
        - 클래스명이지만 메소드로 인식할 수도 있음
    - 프로그램 요소를 매개변수로 전달할 방법이 없음
        - 특정 예외가 발생하는지에 대해서 검사하고 싶을 때, 매개변수를 전달할 방법이 없어서 불편함

- 위의 단점들에 대한 해결책으로는 애너테이션을 사용하는 방법이 있는데 JUnit도 4버전부터는 애너테이션을 도입함


### 마커(marker) 애너테이션
- 아무 매개변수 없이 단순히 대상에 마킹(marking)한다고 하여 **마커 애너테이션**이라고 하고 별다른 처리가 없음
- 대상 코드의 의미는 그대로 둔채 애너테이션에 관심 있는 도구에서 특별한 처리를 할 기회를 줌
- 실제 클래스에 영향은 주지 않지만, 애너테이션에 관심있는 프로그램에 추가 정보를 제공


```java
// 테스트 메서드임을 선언하는 애너테이션
// 매개변수 없는 정적 메서드 전용이다.
@Retention(RetentionPolicy.RUNTIME) // @NewTest가 런타임에도 유지되어야 한다는 뜻
@Target(ElementType.METHOD) // @NewTest가 메서드 선언에서만 사용돼야 한다는 뜻
public @interface NewTest {
}
```
- **메타 애너테이션 (meta-annotation)** : 애너테이션 선언에 다는 애너테이션

- **@Retention(RetentionPolicy.RUNTIME)** (보존 정책)

- **@Target(ElementType.METHOD)** (적용 대상)


- 정리: 
        - **명명패턴보다 자바가 제공하는 애너테이션 타입을 사용하는 것을 권장**
        - **다른 개발자가 코드에 추가 정보를 제공할 수 있는 도구를 만드는 일을 한다면, 적당한 애너테이션 타입도 함께 정의해 제공 권장**


## item 40. @Override 애너테이션을 일관되게 사용하라
        Consistently use the Override annotation
---

### @Override를 사용했을 때 장점
- @Override는 메서드 선언에만 달 수 있으며, 상위 타입의 메서드를 재정의했음을 뜻함
- 일관되게 사용시 여러 악명높은 버그들을 예방해줌

<br>

- Overriding을 Overloading로 잘못 작성할 수 있는 오류를 방지 할 수 있고,
- 잘못 작성 했을 경우, 컴파일시 컴파일러가 잘못된 부분을 명확히 알려줌 (@Override 를 달면 컴파일 오류가 발생하여 미연에 방지할 수 있음)
- 대부분의 IDE에서 의도한 재정의를 상위 클래스의 메서드를 재정의하려는 모든 메서드에 @Override 애너테이션을 달자
- 컴파일 오류의 보완재 역할

<br>

**@Override를 다는 습관을 들이면 시그니처가 올바른지 재차 확신할 수 있음**

 <br>

### @Override를 작성하지 않아도 되는 예외 경우

```java
import java.util.HashSet;
import java.util.Set;

public class Item40 {
    private final char first;
    private final char sescond;

    public Item40(char first, char second) {
        this.first = first;
        this.second = second;
    }

    // @Override
    // public boolean equals(Item40){
    public boolean equals(Object o){
        if (!(o instanceof Item40)){
            return false;
        }
        Item40 i = (Item40) o;
        return i.first == this.first && i.second == this.secod;
    }

    public int hashCode() {
        return 31 * first + second;
    }

    public static void main(String[] args) {
        Set<Item40> b = new HashSet<> args();
        for (int i = 0; i < 10; i++) {
            for (char ch = 'a'; ch <= 'z'; ch++) {
                b.add(new Item40(ch, ch));
            }
            System.out.println(b.size());
        }
    }
}
```

- equals, hashCode 도 재정의 했지만 실해함녀 26개가 아닌 260개가 나오는데, Obejct 의 equals를 재정의 하려면 매개변수도 Object여야 하는데 그렇지 않아 다중 정의가 됨
- 상위 클래스의 메서드를 재정의하려는 모든 메서드에 @override 애너테이션을 달자


**재정의한 모든 메서드에 @Override 애너테이션을 의식적으로 달면 문제시 컴파일러가 바로 알려주어 실수/에러를 줄일 수 있음**
**예외는 하나다. 구체 클래스에서 상위 클래스의 추상 메서드를 재정의한 경우엔 이 애너테이션을 달지 않아도 됨 (단다고 해서 문제가 되진 않음)**


## item 41. 정의하려는 것이 타입이라면 마커 인터페이스를 사용하라
        Use marker interfaces to define types

- **마커 인터페이스(marker interface)**: 아무 메서드도 갖고 있지 않고 단지 자신을 구현하는 클래스가 특정 속성을 갖는 것을 표현해주는 인터페이스
    - 예를 들어 Serializable 인터페이스가 있는데, 이를 구현한 클래스의 인스턴스는 직렬화(Serialization)할 수 있다고 알려주며 ObjectOutputStream을 통해 사용할 수 있음

```java
public interface Serializable {
    }
```
**실제로 아무런 메서드도 담고있지 않음**
<br>

### 마커 인터페이스의 장점
- 1. 클래스의 인스턴스를 구분하는 타입으로 사용할 수 있고, 타입이기 때문에 오류를 컴파일 타임에 잡을 수 있음
    - 반대로 마커 애너테이션을 사용했다면 런타임 시점에 오류를 확인할 수 있음

- 2. 적용 대상을 더 정밀하게 지정할 수 있음 
    - 마커 애너테이션의 경우 적용 대상(@Target)을 elementType.TYPE 으로 선언했다면 클래스, 인터페이스, enum 그리고 애너테이션 모두에 설정할 수 있음 즉, 더 세밀하게 제한하지 못한다는 뜻.
    - 마커 인터페이스의 경우는 마킹하고 싶은 클래스 또는 인터페이스에서만 마커 인터페이스를 구현(인터페이스라면 확장)하기만 하면 됨
<br>

### 마커 애너테이션의 장점
- 거대한 애너테이션 시스템의 지원을 받음
- 애너테이션을 적극 활용하는 프레임워크에서는 애너테이션을 사용하는게 일관성에 좋음

<br>

### 사용해야할 때
- 클래스와 인터페이스 외의 프로그램 요소(모듈, 패키지, 필드, 지역변수)들에 마킹해야할 때 애너테이션을 쓸 수 밖에 없음
    - 클래스와 인터페이스만이 인터페이스를 구현하거나 확장할 수 있기 때문
<br>

- 마커를 **클래스나 인터페이스**에 적용해야 한다면,
    - "이 객체를 매개변수로 받는 메서드를 작성할 일이 있을까?" 라고 자문하고 
    - "그렇다" 라고 한다면 **마커 인터페이스**를 사용
<br>

- 반대로 메서드를 작성할 일은 절대 없다고 확신한다면 **마커 애너테이션**이 나은 선택임
<br>

- 애너테이션을 활발히 활용하는 프레임워크에서 사용하는 마커라면 **마커 애너테이션** 사용

