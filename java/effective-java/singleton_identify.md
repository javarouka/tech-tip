# Serialize

## Serialize?

JVM 메모리 - Heap or Stack - 에 있는 객체 데이터를 바이트 형태로 변환하는 기술. 이 역방향 변환을 Deserialize 라고 한다.

Java 에서의 [Primitive Data Types](https://docs.oracle.com/javase/tutorial/java/nutsandbolts/datatypes.html)은 별도의 처리 없이도 Serialize 가 가능하다. 다만 Object 형식의 객체는 [java.io.Serializable](https://docs.oracle.com/javase/9/docs/api/java/io/Serializable.html) 을 구현해야 Serialize 대상이 된다.

```java
import java.io.Serializable;

/**
 * Serializable 를 구현해서 Serialize 가능하게 한다.
 */
public class Sleep implements Serializable {
    
    private int duration;

    public Sleep(int duration) {
        this.duration = duration;
    }
}
```

Serialize 에는 [java.io.ObjectOutputStream](https://docs.oracle.com/javase/9/docs/api/java/io/ObjectOutputStream.html) 을 사용한다

```java
Sleep sleep = new Sleep(1000);

try (
    ByteArrayOutputStream arraySerializer = new ByteArrayOutputStream();
    ObjectOutputStream objectSerializer = new ObjectOutputStream(arraySerializer);
) {
    // 실제 Serialize 수행
    objectSerializer.writeObject(sleep);

    // 배열로 저장
    persistSerialized(arraySerializer.toByteArray());
} 
catch(IOException serializeFailed) {
    handleSerializeFail(serializeFailed);
}
```

## serialVersionUID

Serialize/Deserialize 간에 Serialize 대상의 고유 번호 표시이다.

Serialize 인터페이스를 상속한 클래스는 가급적 이 번호도 추가하는게 좋다.

`serialVersionUID`는 명시적인 선언 없이도 자동으로 생성되나 없을 경우 [특정 알고리즘을 사용](https://docs.oracle.com/javase/10/docs/specs/serialization/class.html#inspecting-serializable-classes) 하여 클래스의 해시값으로 지정된다.

Serialize 는 상당히 까다롭게 동작한다.

클래스가 같더라도 Serialize/Deserialize 시에 serialVersionUID 값이 다르다면 InvalidClassException 이 던져진다

자동 생성된 serialVersionUID 는 클래스의 해시값이기에 클래스 구조에 변화가 생기면 다를 수밖에 없다. 또한, 명시적인 선언이 있어도 데이터 타입이 변경될 경우 오류가 발생한다. 이 경우에는 InvalidClassException 이 던져진다.

## Pre-Process Methods

Serialize/Deserialize 중 Singleton 등의 인스턴스 제한, 알수없는 데이터에 대한 Deserialize 검증 등의 특수한 처리가 필요할 경우가 종종 있다.

이럴 경우에 대해 Serialize 전처리 메서드들이 준비되어 있다

```java
void writeObject(java.io.ObjectOutputStream out) throws IOException
void readObject(java.io.ObjectInputStream in) throws IOException, ClassNotFoundException;
void readObjectNoData() throws ObjectStreamException;
Object writeReplace() throws ObjectStreamException;
Object readResolve() throws ObjectStreamException;
```

### writeObject/readObject
해당 class 에 override 하면 Serialize/Deserialize 시 호출된다. 보통은 해당 클래스의 상태에 전처리를 할때 사용한다. 

특정 데이터를 writeObject 시에 추가한뒤 readObject 시에 다시 읽거나, 외부 시스템으로부터 받은 수상한 데이터에 대한 방어 목적으로도 사용할 수 있다. 객체에 특정 서명을 추가하거나 해서 어느정도의 보안도 적용 가능해진다.

만일 이 메서드에서 NotSerializableException 을 던지게 되면 Serialize 가 불가능하게 된다

```java
/**
 * place 필드는 transient 로 실제 직렬화 대상이 아니지만
 * writeObject/readObject 구현으로 추가로 직렬화에 포함시켰다.
 */
public class Sleep implements Serializable {

	private int duration;
	private transient String place;

    public Sleep(int duration, String place) {
        this.duration = duration;
        this.place = place;
    }

	private void writeObject(ObjectOutputStream out) throws IOException {
        out.defaultWriteObject(); // static, transient 필드를 제외하고 현재 객체에서 데이터를 읽는다.
        out.writeObject(this.place);
    }

    private void readObject(ObjectInputStream in) throws IOException, ClassNotFoundException {
        in.defaultReadObject(); // static, transient 필드를 제외하고 현재 객체로 데이터를 읽는다. 
        this.place = (String) in.readObject();
    }
}
```

### readObjectNoData

https://stackoverflow.com/questions/7445217/java-when-to-add-readobjectnodata-during-serialization/7445415

### writeReplace/readResolve 

이 메서드를 구현해서 원래 Serialize/Deserialize 대상 객체를 변경할 수 있다. 
직렬화에 대한 프록시 기능이나 인스턴스 수 제한에 유용하다.

링크 하나 공유한다.

[Serialization and magic methods](http://thecodersbreakfast.net/index.php?post/2011/05/12/Serialization-and-magic-methods)