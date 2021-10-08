# Middleware

Nest의 미들웨어는 기본적으로 `express`의 미들웨어랑 같다. 아래의 내용은 express 공식문서의 middleware 설명이다.

- 미들웨어는 다음과 같은 일을 할수있다.
  - 어떤 코드든 실행가능
  - request와 response 객체에 변화를 줄수있다.
  - request-response 사이클을 끝낼수 있다.
  - 스택안에서 다음 미들웨어를 호출할수 있다.
  - 만약 가장 최근의 미들웨어가 request-response cycle을 끝내지 않을거라면 다음 미들웨어로 넘어가기위해 반드시 `next()`를 호출해줘야한다. 그렇지않으면 request한 client가 무한정 기다리게 된다.

미들웨어를 만드려면 `@Injectable()`데코레이터를 사용하고 `NestMiddleware` 인터페이스를 아래와 같이 구현해야한다.

```javascript
// logger.middleware.ts
import { Injectable, NestMiddleware } from '@nestjs/common';
import { Request, Response, NextFunction } from 'express';

@Injectable()
export class LoggerMiddleware implements NestMiddleware {
  use(req: Request, res: Response, next: NextFunction) {
    console.log('Request...');
    next();
  }
}
```

## Middleware 적용

`@Module()` 데코레이터안에는 Middleware를 정의할수 없다. 그 대신, module class에 `configure()` 함수를 사용하여 미들웨어를 적용시킬수 있다. 미들웨어를 사용하는 모듈은 `NestModule`인터페이스를 구현해야한다. 아래는 AppModule 레벨의 미들웨어를 구현한것이다.

```javascript
// app.module.ts
import { Module, NestModule, MiddlewareConsumer } from '@nestjs/common';
import { LoggerMiddleware } from './common/middleware/logger.middleware';
import { CatsModule } from './cats/cats.module';

@Module({
  imports: [CatsModule],
})
export class AppModule implements NestModule {
  configure(consumer: MiddlewareConsumer) {
    consumer
      .apply(LoggerMiddleware)
      .forRoutes('cats');
  }
}
```

forRoutes의 파라미터를 통해서 객체를 전달할수도 있는데 아래와 같이 RequestMethod와 path를 정할수도 있다. 또한 여러개의 객체, 여러개의 컨트롤러 또한 취급할수 있다.

apply함수는 여러개의 미들웨어도 파라미터로 받을수있다.

```javascript
import { Module, NestModule, RequestMethod, MiddlewareConsumer } from '@nestjs/common';
import { LoggerMiddleware } from './common/middleware/logger.middleware';
import { CatsModule } from './cats/cats.module';

@Module({
  imports: [CatsModule],
})
export class AppModule implements NestModule {
  configure(consumer: MiddlewareConsumer) {
    consumer
      .apply(LoggerMiddleware)
      .forRoutes({ path: 'cats', method: RequestMethod.GET });
  }
}
```

> configure함수는 asynchronouse로서 동작도 가능하다.

### Middleware Exclude

```javascript
consumer
  .apply(LoggerMiddleware)
  .exclude(
    { path: 'cats', method: RequestMethod.GET },
    { path: 'cats', method: RequestMethod.POST },
    'cats/(.*)',
  )
  .forRoutes(CatsController);
```

### functional middleware

LoggerMiddleware class를 가만히 보면 콜솔로그 하나있는 아주 간단한 미들웨어인데 이것을 굳이 클래스로 만들 필요가 없다는생각이든다. 따라서 미들웨어를 클래스가 아닌 함수로서도 사용을 가능하게 Nest에서 아래와 같이 지원을 해준다. 앱 모듈에 사용방법은 같다.

```javascript
import { Request, Response, NextFunction } from 'express';

export function logger(req: Request, res: Response, next: NextFunction) {
  console.log(`Request...`);
  next();
};
```