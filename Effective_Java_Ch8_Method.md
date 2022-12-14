## Ch8. 메서드
---
<br>

## item 53. 가변인수는 신중히 사용하라
        Use varargs judiciously

---

- 가변인수 메서드를 호출하면 인수의 개수와 길이가 같은 배열을 만들고 인수들을 만들어진 배열에 저장한 후에 가변인수 메서드에 전달해줌
- 가변인수 메서드는 명시한 타입의 인수를 0개 이상 받을 수 있음
```java
static int sum(int ... args) {
    int sum = 0;
    for (int arg: args){
        sum += args;
    }
    return sum;
}
```

- 인수를 0개도 받을 수 있게 설계하는건 좋지 않음
```java
static int min(int ... args) {
    if (args.length ==0){
        throw new illegalArgumentException('인수가 1개 이상 필요합니다.');
    }
    int min = args[0];
    for (int i = 1; i < args.length; i++){   //잘못구현한 예
        if(args[i] < min>):
            min = args[i];
    }
}
```

- 인수가 1개 이상이어야 할 때는 아래와 같이 가변인수(remainingArgs) 앞에 필수 매개변수(firstArg)를 받도록 함
- 위 코드보다 깔끔한 코드가 됨

```java
static int min(int firstArg, int... remainingArgs) {
    int min = firstArg;
    for (int arg : remainingArgs) {
        if (arg < min) {
            min = arg;
        }
    }
    return min;
}
```

**성능에 민감한 상황이라면 가변인수가 걸림돌이가 될 수 있음**

- 가변인수는 성능에 해가 될 수 있기 때문에 사용할 때는 신중해야 함 
- 가변인수 메서드가 호출될 때마다 **배열을 새로 할당하고 초기화하기 때문**
<br>
- 따라서 아래와 같은 패턴으로 변경할 수도 있음

**성능 최적화의 이점은 없지만 가변인수의 유연성이 필요할 때 선택할 수 있는 패턴**

```java
public void foo() {}
public void foo(int arg1) {}
public void foo(int arg1, arg2) {}
public void foo(int arg1, arg2, arg3) {}
public void foo(int arg1, arg2, arg3, int... restArg) {} // 단 5%로만 사용
```

- 메서드 호출의 95%가 3개 이하의 인수를 사용한다고 가정
- 그렇기 때문에 가변인수는 5%의 호출을 담당

### 정리
**가변인수는 성능에 문제가 있을 수 있다. 신중히 사용하자.**
**인수 개수가 일정하지 않은 메서드를 정의해야 한다면 가변인수가 반드시 필요**
**메서드를 정의할 때 필수 매개변수는 가변인수 앞에 두고, 가변인수를 사용할 때는 성능 문제까지 고려**

<br>

## item 54. null 이 아닌, 빈 컬렉션이나 배열을 반환하라
        Return empty collections or arrays, not nulls
---

- 아래와 같이 컬렉션이 빈 경우 null을 반환하는 메서드를 자주 볼 수 있는데,

```java
// 서버에서 null 반환 시
private final List<Cheese> cheesesInStock = ...;
public List<Cheese> getCheeses() {
    return cheesesInStock.isEmpty() ? null : new ArrayList<>(cheesesInStock);   // nono
}

// 클라에서는 방어 로직을 구현해야 함
List<Cheese> cheeses = show.getCheeses();
if (cheeses != null && cheeses.contains(Cheeses.STILTON)
    System.out.println('이렇게 예외 조건을 걸어줘야 함'))

```
- 컬렉션이나 배열 등이 비었을 때 null 을 반환하는 메서드를 사용할때면 위와 같이 방어 코드를 추가해줘야 함
- 때론, 빈 컨테이너를 할당할때에도 비용이 들어 null을 반환하는 쪽이 낫다는 주장이 있지만 두가지 면에서 틀린 주장임
    1. 성능 분석 결과 성능 저하의 주범이라고 확인되지 않는 한 이정도의 성능 차이는 신경쓸 수준이 못됨
    2. 빈 컬렉션과 배열은 굳이 새로 할당하지 않고도 반환할 수 있음
    ```java
    public List<Cheese> getCheeses(){
        return new ArrayList<>(cheesesInStock); // 빈 컬렉션을 반환하는 방법
    }
    ```

- 그럼에도 눈에 띄는 성능 저하가 발생한다면, 매번 같은 빈 '불변' 컬렉션을 반환하면 됨

```java
// 매번 새로운 빈 컬렉션 대신 같은 빈 불변 컬렉션 반환
public List<Cheese> getCheeses() {
    return cheesesInStock.isEmpty() ? Collections.emptyList() : new ArrayList<>(cheesesInStock);
}
```

- 배열의 경우에도 마찬가지로,
- null 반환 대신에 길이가 0인 배열을 반환하면 됨
    - 컬렉션과 동일하게 빈 배열을 매번 새롭게 할당하지 않고 반환하는 방법도 있음

```java
// 길이가 0인 배열 반환
public Cheese[] getCheeses() {
    return cheesesInStock.toArray(new Cheese[0]);
}

// 매번 새로 할당하지 않게 하는 방법
private static final Cheese[] EMPTY_CHEESE_ARRAY = new Cheese[0];

public Cheese[] getCheeses() {
    return cheesesInStock.toArray(EMPTY_CHEESE_ARRAY);
    // 아래와 같이 미리 할당하는 것은 오히려 성능을 떨어뜨릴 수 있음
    // return cheesesInStock.toArray(new Cheese[cheesesInStock.size()]);
}
```
- 단순 성능 개선이 목적이라면 toArray에 넘기는 배열을 미리 할당하는건 추천하지 않는다고 함
    - toArray는 원소가 하나라도 있다면 배열을 새로 생성하고 0개이면 전달받은 배열은 반환함

### 정리
**null이 아닌, 빈 배열이나 컬렉션을 반환하라**

<br>

## item 55. 옵셔널 반환은 신중히 하라
        Return optionals judiciously
---

- 메서드가 특정 조건에서 값을 반환할 수 없을 때가 있는데, 자바 8 전에는 예외를 던지거나 null을 반환했음
- 하지만 예외는 진짜 예외적인 경우에만 사용해야 하며, 예외 생성 시 스택 추적 전체를 캡처하므로 비용도 큼
- 또한, null이 반환 되지 않는다고 확신하지 않는 한 NullPointerException과 같은 별도의 null 처리 코드를 만들어야 함
<br>
- 그러나 자바 8 이후로는 Optional<T> 라는 대안이 등장하였고,
    - null 이 아닌 T 타입 참조를 하나 담거나 또는 아무것도 담지 않을 수 있음
    - 아무것도 담지 않은 Optional은 '비었다'고 말하고, 어떤 값을 담은것은 '비지 않았다'고 함
    - 원소를 최대 1개 가질 수 있는 ‘불변’ 컬렉션이며, 보다 null-safe한 코드를 작성할 수 있게 됨


```java
// 옵셔널을 사용하지 않았을 때
public static <E extends Comparable<E>> E max(Collection<E> c) {
    if (c.isEmpty()) {
        throw new IllegalArgumentException("빈 컬렉션");
    }        

    E result = null;
    for (E e : c) {
        if (result == null || e.compareTo(result) > 0)
            result = Objects.requireNonNull(e);
    }
    return result;
}

// 옵셔널 + 스트림을 사용할 때
public static <E extends Comparable<E>>
        Optional<E> max(Collection<E> c) {
    return c.stream().max(Comparator.naturalOrder());
}
```

### 옵셔널 활용 및 주의사항

#### 기본값을 설정
- 옵셔널을 반환하는 메서드로부터 원하는 값을 받지 못했을 때, 기본 값을 설정할 수 있음

```java
String lastWordInLexicon = max(words).orElse("단어 없음...");
```

#### 항상 값이 채워짐을 가정
- 옵셔널에 항상 값이 있음을 확신할 때 사용해야 함
- 값이 없다면 NoSuchElementException이 발생

```java
Element lastNobleGas = max(Elements.NOBLE_GASES).get();
```

#### 기본값 설정 비용이 큰 경우
- 기본값 설정 비용이 커서 부담이라면 orElseGet을 사용
- 값이 처음 필요할 때 Supplier<T>를 사용하여 생성하므로 초기 설정 비용을 낮출 수 있음

```java
Connection conn = getConnection(dataSource).orElseGet(() -> getLocalConn());
```

#### 주의점
- 컬렉션, 스트림, 배열, 옵셔널 같은 컨테이너를 옵셔널로 감싸면 안된다. 그러니까 Optional<List<T>>를 반환하는 것보다 그저 빈 리스트 List<T>를 반환하는 것이 낫다. 빈 컨테이너를 그대로 반환하면 클라이언트에서는 옵셔널 처리 코드를 만들지 않아도 되기 때문이다.

한편 옵셔널을 컬렉션의 키, 값, 원소 그리고 배열의 원소로 사용하는 것은 좋지 않다. Map에 사용하는 것을 예로 들어보자. 맵 안에 키가 없다는 정의가 2가지가 된다. “키 자체가 없는 경우”와 “키는 있지만 속이 빈 옵셔널인 경우” 이렇게 모호해진다.

### 정리
**성능에 민감한 메서드라면 null을 반환하거나 예외를 던지는 것이 낫다**










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
