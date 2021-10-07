# Controller

[Docs Link](https://docs.nestjs.com/controllers)

- 컨트롤러가 `response`를 다루는데 두가지의 방법이 있다.

## Standard (recommended)

`JavaScript object`나 `array`를 반환하게되면 자동적으로 `JSON`으로 `serialized`된다.

하지만 `Primitive type`(string, number, boolean)을 반환하게되면 `serialize`없이 그냥 값 자체를 반환한다.

어떤 요청이든 무조건 `status code`가 200을 반환하지만 `POST`만 201을 반환한다. Handler 레벨의 `@HttpCode(...)` 데코레이터를 사용하여 이러한 상태코드를 쉽게 바꿀수 있다.

## Library-specific

우리는 `@Res()`데코레이터를 사용하여 주입받은 `library-specific`(e.g., Express) **response object**를 사용할수도 있다.

이렇게 함으로서 우리는 native한 response 객체의 method들을 다룰수 있게되고, 예를들어 `Express`에서 `response.status(200).send()`를 사용하는것을 말한다.


### 주의사항

> Nest는 Handler가 `@Res()`와 `@Next()`를 사용하는것을 감지하고 Library-specific(e.g, Express)의 response, request를 사용하고 있다고 알려준다. Standard와 Library-specific이 동시에 사용되면 자동적으로 Standard가 disable된다. 그럼 더이상 예상하는대로 애플리케이션이 동작하지않을것이다. 쿠키를 설정해주기위해 response object를 주입받고 그걸 그대로 둘거면, passthrough option을 true로 해줘야한다. @Res({passthrough: true})처럼 말이다.

## Request Object

![Screen Shot 2021-10-07 at 10 25 23 PM](https://user-images.githubusercontent.com/44861205/136393267-4168fc05-ccaa-4819-8c42-72de7018580a.png)

## DTO(Data Transfer Object)

> DTO is an object that defines how the data will be sent over the network.
 
DTO는 TypeScript의 인터페이스 혹은 단순한 클래스로 정의가 가능하다.

### !!!!! 중요 !!!!!!!

인터페이스와 클래스중 Nest는 클래스를 DTO 스키마를 정의할때 사용하길 권장한다.

그 이유는 class는 ES6에서 제공하는 표준이기때문에 컴파일된 자바스크립트에서 실제 객체로서 보존이된다.

하지만 인터페이스는 사라져 없어지기때문에, Nest가 런타임때 불러올수가 없다.

이게 왜 중요하냐면, Pipe같은 feature들이 런타임에 이러한 MetaData에 접근할수있는 가능성이 있기때문이다.

## Library-specific approach

```javascript
import { Controller, Get, Post, Res, HttpStatus } from '@nestjs/common';
import { Response } from 'express';

@Controller('cats')
export class CatsController {
  @Post()
  create(@Res() res: Response) {
    res.status(HttpStatus.CREATED).send();
  }

  @Get()
  findAll(@Res() res: Response) {
     res.status(HttpStatus.OK).json([]);
  }
}
```

위에서 언급한 Controller를 사용하는데 있어서 `Standard`와 `Library-specific`이 있는데, 둘이 동시에 사용되면 Standard가 Disable된다고 했다.

물론 Library-specific이 더 유연성있게 쓸수도 있지만(헤더를 조작하거나, Express에서 사용했던 API를 그대로 사용하고싶다거나 등등..) 이거는 조심히 사용해야한다.

### 단점

- Platform에 의존적이다.
  - 이게 무슨말이냐면 underlying libraries(기반이 되는 library 예를들면 Express, Fastify)들마다 response객체에서 제공해주는 API가 다르기때문에 내가 만약에 Express Response객체를 실컷사용하고있었는데 Fastify로 플랫폼을 바꿔야할때가 오게되면 Express response객체를 Fastify response객체로 다 바꿔줘야한다.

- Test를 하기 어렵다
  - 다른 response객체를 목킹해야하고 번거롭다 Nest에서는 자체적으로 제공하는 Test하기 좋은 다양한 툴들을 제공한다.

