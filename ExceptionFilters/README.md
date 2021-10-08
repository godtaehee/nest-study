# Exception filters

애플리케이션 전반에 걸쳐서 많은 unhandled exception들에 대해 모든 처리를 할 책임이있는 Nest 자체의 exceptions layer가 있다.

`HttpException`과 이 클래스를 상속하는 모든 sub-class exception 같은 오류를 마주했을때 `global exception filter`를 통해 이러한 filter작용이 일어난다.

이렇게 global exception filter가 알고있는 에러 외의 에러가 발생한다면 다음과 같은 JSON형태의 기본 응답을 발생시킵니다.

```javascript
{
  "statusCode": 500,
  "message": "Internal server error"
}
```

## Throwing standard exceptions

Nest는 기본 내장된 `@nestjs/common` 패키지에 있는 `HttpException` 클래스를 제공한다. 이는 전형적인 HTTP REST/GraphQL API에 기반을 둔 애플리케이션에서 오류 발생시 HTTP에서 공식적으로 지정한 response objects를 반환하는데 아주 좋은 방법이다.

```javascript
@Get()
async findAll() {
  throw new HttpException('Forbidden', HttpStatus.FORBIDDEN);
  // throw new HttpException( response, status );
}
```


우리는 이렇게 enum형식으로 제공되는 HttpStatus의 상태코드를 사용할수있다.

클라이언트는 아래와 같은 응답을 받게될것입니다.

```javascript
{
  "statusCode": 403,
  "message": "Forbidden"
}
```

HttpException는 2가지의 파라미터를 받는다.
1. JSON 으로 정의된 response body혹은 string
2. [HTTP status code](https://developer.mozilla.org/en-US/docs/Web/HTTP/Status)

이때 JSON객체는 다음과 같은 프로퍼티를 가집니다
1. `statusCode`: status에 제공된 statusCode
2. `message`: status에 기반한 짧은 HTTP 에러메시지

message를 override 하고싶다면 JSON response 파마리터에 string 값을 주면됩니다.

전체의 JSON response body를 override하고 싶다면, response 파라미터에 object를 넣어주면 Nest가 serialize하여 응답 바디로 내보내준다.

status 파라미터에 알맞은 HTTP Code만 사용하기위해 HttpStatus의 enum값을 아래와같이 넣는것을 추천한다.

```javascript
@Get()
async findAll() {
  throw new HttpException({
    status: HttpStatus.FORBIDDEN,
    error: 'This is a custom message',
  }, HttpStatus.FORBIDDEN);
}
```

위의 라우터에 대한 응답바디는 아래와 같이 나온다.

```javascript
{
  "status": 403,
  "error": "This is a custom message"
}
```

## Custom exceptions

많은 경우에 exception을 custom하고싶을경우가 있는데, Nest의 기본내장 HTTP exception을 사용하면 되긴한다. 하지만 그래도 custom exception을 사용하고 싶다면 `HttpException`클래스를 상속하는 커스텀 exception을 만들수 있다. 이렇게 상속하게되면 Nest가 자동으로 우리가 만든 커스텀 에러를 인지하고, error response로서 관리를 해준다.
아래는 실제 예이다.

```javascript
export class ForbiddenException extends HttpException {
  constructor() {
    super('Forbidden', HttpStatus.FORBIDDEN);
  }
}
```

`ForbiddenException`은 기본 `HttpException`을 상속하므로 기본 내장 exception handler와 다르지않게 잘 작동한다.

이렇게 만든 exception은 controller에서 다음과 같이 사용할수 있다.

```javascript
@Get()
async findAll() {
  throw new ForbiddenException();
}
```

## Built-in HTTP exceptions

Nest는 `HttpException`을 상속한 많은 표준 에러들을 제공한다.

이것들은 모두 `@nestjs/common`패키지에 있으며 가장 공통된 HTTP exception들을 제공한다

- BadRequestException
- UnauthorizedException
- NotFoundException
- ForbiddenException
- NotAcceptableException
- RequestTimeoutException
- ConflictException
- GoneException
- HttpVersionNotSupportedException
- PayloadTooLargeException
- UnsupportedMediaTypeException
- UnprocessableEntityException
- InternalServerErrorException
- NotImplementedException
- ImATeapotException
- MethodNotAllowedException
- BadGatewayException
- ServiceUnavailableException
- GatewayTimeoutException
- PreconditionFailedException

## Exception filters

내장(built-in)된 exception filter가 자동적으로 많은 오류 케이스에 대해서 핸들링 해주지만, 우리는 exceptions layer를 전적으로 우리가 관리하게끔 하고싶을때도 있다. 만약 에러에 Logging처리를 해준다던지, response body의 스키마를 유동적으로 다르게 해주고싶을때 Exception filter를 사용한다. 이러한 flow of control을 우리가 전적으로 컨트롤하여 client에게 우리가 원하는 형태의 response를 보내줄수 있다.

이렇게 구현을 하기위해서 우리는 Platform 기반의 `Request`와 `Response`에 접근할 필요가 있다. Request객체를 통해서 우리는 `url`과 `logging information`을 추출할수 있으며 `response.json()` 메서드를 사용하여 직접 Response를 제어할수 있다. 아래는 실제 예이다.

```javascript
import { ExceptionFilter, Catch, ArgumentsHost, HttpException } from '@nestjs/common';
import { Request, Response } from 'express';

@Catch(HttpException)
export class HttpExceptionFilter implements ExceptionFilter {
  catch(exception: HttpException, host: ArgumentsHost) {
    const ctx = host.switchToHttp();
    const response = ctx.getResponse<Response>();
    const request = ctx.getRequest<Request>();
    const status = exception.getStatus();

    response
      .status(status)
      .json({
        statusCode: status,
        timestamp: new Date().toISOString(),
        path: request.url,
      });
  }
}
```

> 모든 exception filter들은 `ExceptionFilter<T>` 인터페이스를 구현해야 하며, `catch(exception: T, host: ArgumentsHost)`메서드를 필수로 구현해야한다. 이때 T는 exception의 타입을 가리킨다.

`@Catch(HttpException)` 데코레이터는 exception filter에 필수적으로 필요한 metadata들을 바인딩한다.

위의 예 같은 경우는 Nest에게 이 filter는 `HttpException`타입의 exception을 찾고있으며 그 외에는 아니다라는것을 알려준다.

`@Catch()` 데코레이터는 파라미터가 한개이거나 여러개 일수도 있다.

## Arguments host

catch() 메서드의 `exception` 파라미터를 보면 가장 최근 진행된 exception object를 말한다.

`host`파라미터는 `ArgumentsHost`객체이다.

`ArgumentsHost` 객체는 [execution context chapter](https://docs.nestjs.com/fundamentals/execution-context)에서 살펴볼 강력한 utility 객체이다.

위의 코드 예제에서 우리는 ArgumentHost를 원래의 요청 핸들러(exception이 최초 발생하는 controller)로 전달되는 `Request`와 `Response`객체의 참조를 포함하기위해 사용했다.

> 이러한 레벨의 추상화는 `ArgumentsHost` 함수가 모든 컨텐스트에서 작용하기 때문이다. (예를들어, 우리가 지금 사용하고 있는 HTTP server context에서도, Microservice와 WebSocket에서도 사용가능하다.) `execution context`챕터에서 우리는 ArgumentsHost의 강한 힘과 함께 어떠한 execution context에도 적절한 `underlying arguments`에 접근하는 방법을 배운다. 이것은 모든 contexts에 동작하는 generic exception filter를 작성하도록 도운다.
 
 ## Binding filters

```javascript
@Post()
@UseFilters(new HttpExceptionFilter())
async create(@Body() createCatDto: CreateCatDto) {
  throw new ForbiddenException();
}
```

`@Catch()` 어노테이션과 비슷하게 `@UseFilters()`데코레이터를 사용하였는데 한개 혹은 여러개의 파라미터를 가질수 있다. 아래와 같이 Instance 대신 Instantiate를 프레임워크에 전적으로 맞기고 DI를 해줄수있게 class로도 줄수있다.

```javascript
@Post()
@UseFilters(HttpExceptionFilter)
async create(@Body() createCatDto: CreateCatDto) {
  throw new ForbiddenException();
}
```

> class를 사용하는것이 instance를 사용하는것보다 메모리 사용량을 줄여준다는 측면에서 더 좋다. 전체 모듈을 통틀어서 Nest가 instance를 재사용 해주기 때문이다.
 
Exception filter는 `method-scoped`, `controller-scoped`, `global-socped`로도 사용가능하다.

글로벌 스코프로 주고싶을땐 아래와 같이한다. 근데 별로 추천하지않는다.

```javascript
// main.ts
async function bootstrap() {
  const app = await NestFactory.create(AppModule);
  app.useGlobalFilters(new HttpExceptionFilter());
  await app.listen(3000);
}
bootstrap();
```

Global-scoped filter들은 애플리케이션 전반에 걸쳐 모든 컨트롤러, 모든 라우터에게 쓰입니다.

DI 관점에서보면, 모듈의 밖에서 등록된 global filter들은 모듈의 밖에서 작업들이 다 끝났기 때문에 DI를 할수 없습니다.

이게 무슨말이냐면, 지금 GlobalFilter는 Module의 밖(main.ts)에서 진행되기때문에 DI를 할 Dependency로 등록이 안되니 DI를 할수가 없다는것이다. 이러한 문제를 해결하기위해 아래와 같이 모듈의 provider에게 filter를 적용하는것을 추천한다.

```javascript
import { Module } from '@nestjs/common';
import { APP_FILTER } from '@nestjs/core';

@Module({
  providers: [
    {
      provide: APP_FILTER,
      useClass: HttpExceptionFilter,
    },
  ],
})
export class AppModule {}
```

> Filter DI를 위해 위의 예처럼 하게되면 모듈에 대한 Filter적용임에도 불구하고 이건 결국 global로 적용한것과 마찬가지이다. 왜냐하면 AppModule에다가 해주고 있기 때문이다. 그러면 앱모듈이 아닌 이러한 필터가 적용될 모듈을 직접 선택해야한다. custom provider를 register함에 있어서 useClass만 방법이 있지는 않다.
 
 
## Catch everything

모든것을 Catch처리 해주기위해서는 (exception type도 상관없음) `@Catch()`처럼 catch 데코레이션의 파라미터를 비워주면된다. 아래와같이 말이다.

```javascript
import {
  ExceptionFilter,
  Catch,
  ArgumentsHost,
  HttpException,
  HttpStatus,
} from '@nestjs/common';

@Catch()
export class AllExceptionsFilter implements ExceptionFilter {
  catch(exception: unknown, host: ArgumentsHost) {
    const ctx = host.switchToHttp();
    const response = ctx.getResponse();
    const request = ctx.getRequest();

    const status =
      exception instanceof HttpException
        ? exception.getStatus()
        : HttpStatus.INTERNAL_SERVER_ERROR;

    response.status(status).json({
      statusCode: status,
      timestamp: new Date().toISOString(),
      path: request.url,
    });
  }
}
```

## Inheritance

우리의 애플리케이션에 맞게 exception을 전적으로 컨트롤 하고싶을 수 있다.

하지만 어떠한 요소를 override하거나 기본으로 제공하는 global exception filter를 상속하고 싶은 경우가 있을수 있다.

base filter에 에러처리를 위임하기위해서는, 아래와같이 `BaseExceptionFilter`라는것을 상속하고 `catch()`메서드를 상속하여 호출하면된다.

```javascript
import { Catch, ArgumentsHost } from '@nestjs/common';
import { BaseExceptionFilter } from '@nestjs/core';

@Catch()
export class AllExceptionsFilter extends BaseExceptionFilter {
  catch(exception: unknown, host: ArgumentsHost) {
    super.catch(exception, host);
  }
}
```

> BaseExceptionFilter를 상속한 Method-scoped와 Controller-scoped 필터는 new를 통해서 인스턴스화가 되지 않아야한다. framework가 자동으로 해주기 때문이다.
 
 위의 구현은 위 전략에 접근하기위한 단순한 쉘에 불과하다.
 
우리가 생각한 비지니스 로직을 여기에 추가할수있다.

