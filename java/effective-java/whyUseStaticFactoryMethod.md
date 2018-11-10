# 객체 생성 정적 팩토리 메서드를 쓰는게 왜 유리한가

## 생성자 기본지식

java 에서의 보통의 객체 생성은 new 를 사용한다

```java
List<String> strList = new ArrayList<>();
```

공개된(public) 생성자를 new 연산자와 함께 호출해서 객체를 생성한다.

이 과정에서 알아둘 생성자에 대한 몇가지 기본적인 지식이 있다

- class 정의 시 생성자를 정의하지 않았다면 기본적으로 인자가 없는 생성자가 언어레벨에서 지원된다.
- Subclassing 된 class 라면 자식 생성자 호출 시 부모의 생성자도 연쇄적으로 호출한다. 생성자 체이닝이라고도 부른다.
- 어떤 클래스가 기본 생성자가 아닌 생성자가 정의되었다면 Subclass 도 반드시 생성자를 정의해야 한다. 그리고 그 생성자에는 super로 명시적 체이닝을 해줘야 한다

생성자 호출은 모듈에는 Spring 등의 [DI Framework](https://en.wikipedia.org/wiki/Dependency_injection#Dependency_injection_frameworks) 에 일반 객체는 [Builder 패턴](https://johngrib.github.io/wiki/builder-pattern/) 을 사용하게 되면 거의 쓸일이 없어진다.

하지만 독자 레이어링 시에는 간간히 호출하는 경우가 있기에 알아두면 좋을 것 같다.

## 정적 팩토리 메서드

이 책에서 생성자 대신 정적 팩토리 메서드 생성을 추천하는 이유로 5가지를 들고 있다.

### 이름을 가질 수 있다

객체 생성의 목적은 상황에 따라 다르다. 그리고 그 결과가 어떤 값이 될련지는 생성자 정의에 따라 달라지지만 `new XXX` 로 호출하는 표현에는 명시적으로 나와있지 않다.

이름을 가지게 되면 해당 생성의 목적을 이름에 표현할 수 있어 가독성이 증가한다.

### 객체 생성 제한이 가능하다

객체 생성에 제한을 걸거나, 불변 객체를 매번 반환하는 등의 요령을 쓸 수 있다.
상황에 따라 신규 생성인지 기존 객체 반환인지도 정할 수 있다.

### SubClass 를 반환할 수 있다

상속 계층과 타입 상 하위 객체를 반환할 수 있고, 사용자는 그걸 모른채로 사용할 수 있다.

특정 버전의 class 정적 팩토리 메서드가 하위클래스 A를 반환하다가, 역시 Sub-Classing 된 B 를 반환하더라도 Client 코드에서는 그걸 신경쓰지 않고 작업이 가능하다.

기본적인 생성자를 사용한다면 구현체 자체가 바뀌었기에 Client 코드의 변경이 불가피해진다.

### 입력 매개변수에 따라 다른 타입으로 반환 가능하다

위 `SubClass 를 반환할 수 있다` 랑 비슷한 이점이다. 매개변수에 따라 선택적인 Sub-Classing 된 객체를 반환할 수 있다.

### 실제 구현체 클래스가 없어도 된다

```java
class Car {
    public static Car newTruck() {
        Class<?> truckClz = Class.forName("me.javarouka.vehicle.Truck");
        return truckClz.newInstance();
    }

    public static Car newBus() {
        Class<?> busClz = Class.forName("me.javarouka.vehicle.Bus");
        return busClz.newInstance();
    }
} 
```

위 코드처럼 처음 클래스가 로더에 존재하지 않아도 동적으로 현재 코드가 실행되는 클래스 로더에 해당 클래스 (예제에서는 com.vehicle.Truck, com.vehicle.Bus) 들을 로딩한다.

그리고 그 클래스의 인스턴스를 생성해서 반환할 수 있다.

> 추가적으로 Class.forName 으로 로딩된 클래스들은 static 구문을 수행한다 . 보통 JDBC 3.x 이하를 써본 사람은 많이 본 코드일 것이다

## 정적 팩토리 메서드의 단점

책에서는 2가지를 소개하고 있지만 큰 단점으로 보이지 않는다.

- Sub-Classing 어려움
- 문서화

하지만 둘다 큰 단점으로 보이진 않는다.
오히려 이시대의 사회악처럼 되어버린 상속을 방지하는 부가효과(?) 가 있고, 문서화는 코드 표현과 네이밍으로 대체할 수 있기 때문이다.

주로 쓰는 네이밍들은 다음과 같다.

- from
    - 매개변수 하나인 메서드
- of
    - 매개변수가 N개인 메서드
- getInstance
    - 인스턴스를 생성한다. 앞서 반환한 객체와 같을 수도 있다.
- newInstance
    - 인스턴스를 생성한다. 앞서 반환한 객체와는 항상 다르다.
- getSomeType
    - 인스턴스를 생성하지만 자신의 타입이 아닌 다른 타입 (SomeType) 으로 반환한다. 다른 내용은 `getInstance` 와 같다
- newSomeType
    - 인스턴스를 생성하지만 자신의 타입이 아닌 다른 타입 (SomeType) 으로 반환한다. 다른 내용은 `newInstance` 와 같다