# Providers

`service`, `repository`, `factory`, `helper` 등등의 Nest Class들은 `Provider`로서 취급한다.

`Provider`의 메인 아이디어는 `Dependency`로서 Injected 될수있다 라는것인데

이말의 뜻은 뭐냐면, Provider객체들끼리 서로 다양한관계를 가질수 있다는것이다, 

이렇게 instance 끼리 연결시켜주는것은 Nest Runtime System에 위임할수 있다.

## Custom Providers

Provider를 정의하는 방법은 value로서, class로서 비동기 factory로서 동기 factory로서 사용가능하다.

## Property-based injection

예를들어 최상위 클래스는 `constructor-based`기반의 Injection을 사용하는데 이 클래스에 많은 Provider를 생성자를 통해서 주입받게되면 이 최상위 클래스를 상속받는 하위클래스에서 super로 의존성을 많이 넘겨줘야한다. 이건 좋지 않은방법이다.

```javascript
class SuperClass {
  constructor(private A a, private B b, ...) { }
}

class SubClass extens SuperClass {
  constructor() {
    super(A, B, ...)
  }
}
```