# Generics

- 제네릭은 타입 파라미터 정의 문법이다.
- 제네릭이 정의된 타입은 타입 파라미터에 구체적인 타입을 정의해야만 인스턴스화가 가능하다.
- inline 과 reified 와 같이 쓸 경우 제네릭 타입이 소거되지 않고 실체화된다

# 반공변(contravariant)

- Java: ? super T
- Kotlin: in T

```java
List<? super GirlSinger> any = Lists.newArrayList();

// 걸그룹의 하위 타입은 뭐든 추가 가능
any.add(new GirlsGeneration("태연"));
any.add(new Twice("미나"));

// Error. 걸그룹의 반공변이기 때문에 요소들은 걸그룹을 상위로 하는 타입이다.
// 다시말해 저 리스트가 어떤 구성원을 가지는지 알 수 없다.
// 극단적으로 오브젝트의 리스트일수도 있다. 따라서 다음 문은 오류.
GirlSinger a = any.get(0);

// 이해가 안된다면 다음 구문을 보자
// any 의 타입은 GirlSinger의 반공변이기에 뭐든 올 수 있다.
// 그러므로 다음 구문은 적법하다.
List<? super GirlSinger> any = Lists.newArrayList(1,2,3,4);

// Error. 컴파일러가 어떻게 캐스팅 코드를 넣어야 할지 모르게 된다.
GirlSinger b = any.get(0);
```

```kotlin
val cd: MutableList<in GirlSinger> = mutableListOf() // 반공변 리스트

// 역시 걸그룹이면 모두 추가가능
cd.add(GirlsGeneration("태연"))
cd.add(Twice("미나"))

// Error. a 의 타입은 걸그룹의 반공변(걸그룹을 상위로 하는 타입)이기 때문에 위 java 예제와 같다.
val a:GirlSinger = cd.get(0)
```

# 공변(covariant)

- Java: ? extends T
- Kotlin: out T

```java
List<? extends GirlSinger> girls = Lists.newArrayList(
        new GirlsGeneration("태연"),
        new Twice("미나") 
);

// 걸그룹의 공변이기에 요소들이 확실하게 걸그룹의 하위 타입이라는걸 알 수 있다.
GirlSinger girl = girls.get(0);

// Error. 소녀시대인지, 트와이스인지 제 3의 걸그룹인지 확정할 수 없기에 함부로 특정적인 신규 멤버를 넣을 수 없다
girls.add(new GirlsGeneration("서현"));
```

```kotlin
val ab: MutableList<out GirlSinger> = mutableListOf() // 공변 리스트

val aGirl:GirlSinger = ab.get(0) 

// Error. 실제 그룹이 소녀시대가 아닌 다른 걸그룹(트와이스)일 수 있기에, 추가할 수 없다.
ab.add(GirlsGeneration("태연"))
```

## 다른 예제
https://gist.github.com/javarouka/a9deb84708dacec5e39aff5ca1533a01
