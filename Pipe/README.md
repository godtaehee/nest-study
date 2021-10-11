# Pipes

Pipe는 `@Injectable()`데코레이터로 어노테이트된 클래스 입니다.

Pipe는 `PipeTransform` interface를 구현해야합니다.

Pipe는 두가지의 use case가 있습니다.

- transformation: 원하는 데이터형태로 입력데이터를 변형합니다 (예를들어 String에서 Int로)
- validation: 입력데이터를 평가하는데, 유효한 데이터이면 변경없이 패스시키고, 그렇지않으면 데이터가 알맞지 않다고 오류를 냅니다.

두가지 경우 모두 컨트롤러의 argument에서 사용가능합니다.

Nest는 method가 invoked 되기전에 pipe를 배치시키고 파이프는 메서드로 지정된 인수를 동작시킵니다.

어떠한 transformation 혹은 validation이 이때 발생하며, 라우트 핸들러는 변형된 후의 인수와 함께 동작합니다.

> Pipe는 exceptions zone안에서 실행됩니다. 이것이 의미하는 바는 파이프가 예외를 발생시키면  exceptions layer(global exceptions filter 와 다른 exceptions filters)에 의해 핸들링 됩니다. 위와 같은 방식으로, exception이 pipe안에서 throw되면, controller method는 절대 실행되지않으므로 시스템의 경계에 있는 외부 소스로부터 애플리케이션으로 오는 데이터를 Validation하기위한 `best-practice`를 줍니다.
 
## Built-in Pipes

Nest가 제공하는 pipe들은 다음과 같습니다.

- ValidationPipe
- ParseIntPipe
- ParseFloatPipe
- ParseBoolPipe
- ParseArrayPipe
- ParseUUIDPipe
- ParseEnumPipe
- DefaultValuePipe

이 파이프들은 `@nestjs/common` 패키지에서 export 할수있습니다

### ParseIntPipe

ParseIntPipe는 transformation의 예이며, javascript 정수형으로서 파라미터가 이용될수 있게 보장해줍니다. 만약 변형이 실패하면 오류를 던집니다.

이외에도 `ParseBoolPipe`, `ParseFloatPipe`, `ParseEnumPipe`, `ParseArrayPipe`, `ParseUUIDPipe` 들이 있습니다.

## Binding pipes

파이프를 사용하기위해서는, 적절한 context에 pipe class의 instance를 바인딩 할필요가 있습니다.

아래의 `ParseIntPipe`예에서 특정한 route handler와 파이프를 연관짓고 메소드가 호출되기 이전에 실행되는지 확인합니다. 이러한것을 우리는 method parameter 레벨에서 파이프를 바인딩 한다라고 부릅니다.

```javascript
@Get(':id')
async findOne(@Param('id', ParseIntPipe) id: number) {
  return this.catsService.findOne(id);
}
```

이것은 우리에게 2가지를 확신할수 있게 해주는데, 한개는 findOne()메서드가 받는 parameter는 number임이 보장되고, 라우트 핸들러 실행전에 exception이 throw된다라는것을 보장받습니다.

예를들어 다음과 같이 요청이 들어오면

```shell
GET localhost:3000/abc
```

Nest는 아래와 같은 오류를 발생시킵니다.

```javascript
{
  "statusCode": 400,
  "message": "Validation failed (numeric string is expected)",
  "error": "Bad Request"
}
```

우리 라우터 핸들러의 예를보면 `ParseIntPipe`라는 클래스를 pass시키고 있는데 이는 DI를 전적으로 프레임워크에게 맡긴것입니다. pipe와 guard처럼, 우리는 in-place 인스턴스를 통과시킵니다. in-place 인스턴스를 패스시키는것은 우리가 `built-in`파이프의 행동을 옵션을 통해 아래와 같이 커스터마이징 할수있다는 장점이 있습니다.

```javascript
@Get(':id')
async findOne(
  @Param('id', new ParseIntPipe({ errorHttpStatusCode: HttpStatus.NOT_ACCEPTABLE }))
  id: number,
) {
  return this.catsService.findOne(id);
}
```

`Parse*` 파이프들은 쿼리스트링과 request body에도 사용가능합니다.

```javascript
@Get()
async findOne(@Query('id', ParseIntPipe) id: number) {
  return this.catsService.findOne(id);
}
```

아래는 UUID인지 검증하고 string으로 파싱하는데 `ParseUUIDPipe`를 사용합니다.

```javascript
@Get(':uuid')
async findOne(@Param('uuid', new ParseUUIDPipe()) uuid: string) {
  return this.catsService.findOne(uuid);
}
```

> `ParseUUIDPipe()`를 사용할때 3,4,5 버전을 사용하는데 버전을 pipe option을 통해 특정 지울수도 있습니다.
 
