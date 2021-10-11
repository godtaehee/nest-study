```shell
npm i @nestjs/config
```

> `@nestjs/config` 패키지는 내부적으로 dotenv를 사용한다.
 
설치가 되고난후 우리는 `ConfigModule`을 import 하여 사용할수 있고 이것은 보통 AppModule에 import를 하고 forRoot static메서드를 사용하여 나머지 커스터마이징을 한다.

```javascript
// app.module.ts
import { Module } from '@nestjs/common';
import { ConfigModule } from '@nestjs/config';

@Module({
  imports: [ConfigModule.forRoot()],
})
export class AppModule {}
```

위의 코드가 프로젝트의 루트에 있는 .env파일을 로드하고 파싱한다. process.env에 할당해준다.

이것은 ConfigService에서 사용할수있다.

`forRoot()`메서드는 이러한 환경 변수를 읽어들이는 `get()` 메서드가 있는ConfigService` provider를 등록한다.

> 운영체제 상에서 환경변수가 이미 등록이 되어있다면 운영체제의 환경변수가 우선된다.
 

다른 모듈에서도 Config를 사용해주기위해서는 `isGlobal` 옵션을 사용한다.