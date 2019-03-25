# 반공변(contravariant)

- Java: ? super T
- Kotlin: in T

```java
List<? super GirlSinger> any = Lists.newArrayList();

// 걸그룹의 상위 타입은 뭐든 추가 가능
any.add(new GirlsGeneration("태연"));
any.add(new Twice("미나"));

// 걸그룹의 반공변이기 때문에 요소들은 걸그룹의 상위 타입이다.
// 때문에 걸그룹이 아니라 걸그룹의 수퍼타입인 그냥 여자, 그 이상인 사람이 될 수 있다.
// Error. 다시말해 저 리스트가 어떤 구성원을 가지는지 알 수 없다.
GirlSinger a = any.get(0);
```

```kotlin
val cd: MutableList<in GirlSinger> = mutableListOf() // 반공변 리스트

// 역시 걸그룹이면 모두 추가가능
cd.add(GirlsGeneration("태연"))
cd.add(Twice("미나"))

// Error. a 의 타입은 걸그룹의 반공변이기 때문에 위 java 예제와 같다.
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

// 실제 그룹이 소녀시대가 아닌 다른 걸그룹(트와이스)일 수 있기에, 추가할 수 없다.
ab.add(GirlsGeneration("태연"))

// Error. 소녀시대의 요소가 있어도, 컴파일러는 알 수 없다. 걸그룹일지 소녀그룹일지 그 상위인 사람그룹일지 알 수 없다.
val aGirl:GirlsGeneration = ab.get(0) 
```
