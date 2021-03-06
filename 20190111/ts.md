# 내가 하는 일을 알고 있다.
- function-scoping
- 블럭-스코프 변수 캡쳐
- JavaScript는 일반적으로 a{를 블록 시작으로 파싱하기에 블럭 표현식이 모호할 경우 반드시 괄호로 래핑해야 함
- readonly vs const
    - property vs variable!
- 프로퍼티 초과 검사(Excess Property Checks)

# 자바스크립트의 수퍼셋.
 - 타입 검사가 추가되고, 타입을 위해 이런저런 기능과 제약이 붙은 언어이다
 - MS 에서 개발
 - 클래스와 인터페이스는 자바와 비슷하면서도 다르다.
 - 클래스도 인터페이스처럼 구현만 정의할 수도 있다. 인터페이스는 클래스 뿐 아니라 함수의 인터페이싱도 가능하다.
 - 제네릭과 타입 매개변수도 물론 지원한다
 - 자바의 클래스와 인터페이스는 언어 자체에 포함된 하나의 조각 같다면 타입스크립트의 인터페이스와 클래스 시스템은 타입 검증을 위한 수단같은 느낌.
 - 덕 타이핑이 적극적으로 언어 레벨에 쓰이고 있다.
 - typeof, keyof
 - 할당된 변수에 뭔가를 할당할때 타입체크가 들어간다 (당연한가...)
 
# Type 
## Type Alias 는 자가 참조 가능하다
```typescript
type Tree<T> = {
    value: T;
    left: Tree<T>;
    right: Tree<T>;
}

type LinkedList<T> = T & { next: LinkedList<T> };
```

## 하지만 순환 참조는 안된다

선언의 오른쪽에 나오는 건 불가능.
```typescript
type Cir = Array<Cir>;
```

## 유니온 타입의 식별 (Discriminated Unions)

```typescript
type Shape = Square | Rectangle | Circle;

interface Square {
    kind: "square";
    size: number;
}
interface Rectangle {
    kind: "rectangle";
    width: number;
    height: number;
}
interface Circle {
    kind: "circle";
    radius: number;
}

function area(s: Shape) {
    switch (s.kind) {
        case "square": return s.size * s.size;
        case "rectangle": return s.height * s.width;
        case "circle": return Math.PI * s.radius ** 2;
    }
}
```

## Proxyfy

```typescript
type Proxy<T> = {
    get(): T;
    set(value: T): void;
}

type Proxify<T> = {
    [P in keyof T]: Proxy<T[P]>;
}

class PX<T> implements Proxy<T> {
    constructor(private value: T) {}
    get(): T { return this.value;}    
    set(value: T) { this.value = value; }    
} 

function proxify<T>(o: T): Proxify<T> {
    let result = {} as Proxify<T>;
    for (const k in o) {
        result[k] = new PX(o[k]);
        // result[k] = {
        //     get() {return o[k]},
        //     set(value) {o[k] = value;}
        // };
    }
    return result;
}
```
