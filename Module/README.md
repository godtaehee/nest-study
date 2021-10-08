# Modules

Module은 `@Module()` 데코레이터로 Annotated된 클래스이다. `@Module()`데코레이터는 Nest가 Application Structure를 구성하기위한 메타데이터를 제공해준다.

각각의 Application은 `root module`이라는 모듈을 적어도 한개는 가지고있다.

Root Module은 `application graph`를 설계하기위해 사용하는 시작지점이다.

이때 `application graph`는 Nest가 module을 resolve하고 provider의 관계들과 의존성을 resolve하는데 사용되는 내부 데이터 구조이다.


## Module Property

`@Module` 데코레이터는 다음과같은 객체의 속성들을 갖는다.

### providers

Nest Injector에 의해 instantiated될 providers또는 적어도 해당 모듈내에서는 공유가 되야하는 providers를 말한다.

### controllers

이 모듈에서 instantiated되야할 controller의 집합을 말한다.

### imports

해당 모듈에서 반드시 필요한 export된 providers를 import한 리스트이다.

### exports

이 모듈에 의해 제공되는 providers의 부분집합 혹은 이 모듈을 import하여 다른 모듈에서 사용가능하게 하기위해 사용하는 속성

> Module은 provider를 기본적으로 캡슐화한다. 이것은 현재 직접적으로 모듈에 속해있거나 import된 모듈을 export하지 않으면 Inject 할수없다 라는 뜻이다. 그러므로, module로부터 export된 provider들은 모듈의 공용 인터페이스 혹은 API로 간주될수있다.

## Feature modules

`CatsController`와 `CatsService`는 같은 애플리케이션 도메인에 속한다.

위의 컨트롤러와 서비스는 매우 관련되어있으며, 이러한 것들을 `Feature Module`로 묶는데 적합하다. 구체적인 기능을 위한 코드의 상관관계를 조직화 하여 코드를 조직화하고 명확한 경계를 짓게 해준다.

SOLD원칙과 함께 개발하여 복잡성을 관리하고 특히 팀의 성장 혹은 애플리케이션에 사이즈에관한것들도 관리해준다.

관련된 것들(Controller, Provider)을 모듈에 적어주고 최종적으로는 app.module에 import 해주어야한다.