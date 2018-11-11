# 빌더 패턴

## 객체 생성시에 거슬리는 요인

- 많은 인자
    - 인자의 순서를 실수할 경우가 잦아진다
- 기본값
    - 옵셔널 기본값을 주고 싶지만 주기 어렵다
- 많은 멤버
    - 수많은 멤버를 가진 객체의 경우 생성자가 비대해진다

기본 생성자 호출 뒤 Setter로 하나하나 세팅하는 방법도 있지만 시간의 흐름에 따라 멤버가 초기화되기에 일관성이 해쳐진다. 

그리고 불변객체를 만들 수 없어 스레드, 코딩 안정성이 저하된다.

## 그래서 준비했습니다

### 일반적인 빈즈의 빌더 패턴

```java
public Car {
    private final String name;
    private final int price;
    private final Color color;

    private Car(String name, int price, Color color) {
        this.name = name;
        this.price = price;
        this.color = color;
    }

    public static class Builder {
        private final String name; // 필수값
        private int price = 0; // 기본값
        private Color color = Color.RED; // 기본값

        public Builder(String name) {
            this.name = name;
        }

        public Builder price(int price) {
            this.price = price;
            return this;
        }

        public Builder color(Color color) {
            this.color = color;
            return this;
        }

        public Car build() {
            return new Car(
                this.name, 
                this.price, 
                this.color
            );
        }
    }
}
```

## 사용
```java
Car superMyCar = new Car.Builder('내 애마')
    .price(9999999999)
    .color(Color.INDIGO)
    .build();
```
