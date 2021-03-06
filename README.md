# nestJS

- nestJS는 이미 만들어진 기능을 제공하는 프레임워크
- nestJS의 src에는 controller.spec, controller, module, service, main이 존재함(main의 이름은 고정)
- main의 bootstrap() 함수는

```JS
const app = await NestFactory.create(AppModule);
await app.listen(3000);
```

- AppModule을 통해서 서버를 구동시킴.
- AppModule은 app.module.ts이고, app.module.ts에는 데코레이터라는 함수가 존재함.

```JS
@Module({
  imports: [],
  controllers: [AppController],
  providers: [AppService],
})
```

- nestJS에서는 데코레이터 함수를 많이 사용함. (익숙해져야 함)
- 데코레이터 함수는 클래스 위의 함수이고, 클래스를 위해 움직인다고 생각하면 됨.

#### 'Hello, Nest!' 출력 과정

- app.module.ts -> app.controller.ts -> app.service.ts =>

### app.module.ts

- 모든 것의 루트 모듈
- 모듈이란 어플리케이션의 일부분
  - 한가지 역할을 하는 앱, (e.g. 인증을 담당하는 어플리케이션 users 모듈))
  - 인스타그램이라면 photos, videos 모듈 등
- app.module.ts에서는 controller와 provider를 사용하는데

#### controller : url을 가져오고 함수를 실행하는 역할 (express의 router 같은 역할)

- controller에서 @Get() 데코레이터는 express.js의 app.get router 역할을 함.

```JS
@Get('/hello')
sayHello(): string {
  return 'Hello everyone';
}
```

- 이렇게 /hello get url의 router를 가지고 'Hello everyone'을 리턴하는 Get 데코레이터 함수를 쉽게 만들 수 있음
- 주의할 점은 데코레이터와 함수 사이에 빈칸이 있으면 안됨.

### service

- 위 코드처럼 controller에서 그대로 string을 리턴해도 문제가 없는데 왜 service가 필요할까?

* nestJS는 Controller를 비즈니스 로직과 구분지음
* controller는 url을 가져오는 역할만 함. (+ 함수를 실행하는 정도)
* 나머지 비즈니스 로직은 service에서 수행함.
* service는 일반적으로 실제 Function을 가지는 부분
* controller의 Appservice의 정의로 이동하면

```JS
@Injectable()
export class AppService {
  getHello(): string {
    return 'Hello Nest!';
  }
}
```

- 이런 class가 존재하는데, 이 class 안에 getHello() 함수가 들어있음.
- 그렇기 때문에 controller의 sayHello 함수는

```JS
// controller appService안의 함수 호출
@Get('/hello')
  sayHello(): string {
    return this.appService.getHi();
  }

// service 클래스 안에 추가
getHi(): string {
    return 'Hi Nest!';
  }
```

- 이렇게 사용하는 게 올바른 사용법임.

---

- cmd 창에 nest를 입력하면 nestJS 프레임워크에서 사용할 수 있는 것들이 나옴
- generate 커맨드로 nestJS의 거의 모든 것을 생성할 수 있음.

#### nest g co : generate로 controller를 생성함 => 후에 이름을 물어보는데 'movies'로 생성

- 생성하면 src/movies 폴더가 생성되고 app.module.ts의 controllers에 MoviesController가 자동 추가되어있음

```JS
@Controller('movies')
export class MoviesController {

  @Get()
  getAll() {
    return "This will return all movies";
  }
}
```

- movies.controller에 위 get router를 추가해줘도 http://localhost:3000 주소에서 404 에러가 발생함.
- @Controller('movies') 이 부분이 컨트롤러를 위한 url을 생성하게 되어 (url의 엔트리 포인트를 컨트롤하는 부분)
- http://localhost:3000/movies 로 들어가야 함.

```JS
@Get("/:id")
  getOne(@Param("id") id: string){
    return `This will return ${id} movies`;
  }
```

- localhost:3000/movies/1 => id에 대한 값을 같이 url로 받으면 실행됨.
- @Get 데코레이터 안에 getOne 함수를 생성하고 인자값으로
- 파라미터 데코레이터, @Param("id")를 넣어주면 NestJS는 url에 있는 id paramter를 원한다고 요청하는 것임

- url로 넘어온 id부분을 string 형식의 id라는 이름으로 받겠다는 뜻이고 뒤의 id: string은 url에서 넘어온 id값을 id라는 변수로 재사용한다는 것

#### nestJS는 필요한 파라미터를 원할 때 직접 요청해서 사용해야 함.

- @Param("id") 파라미터 데코레이터를 통해서 id를 요청해서 가져와서 사용할 수 있고

```JS
@Post()
  create(@Body() movieData) {
    console.log(movieData);
    return "This will create a movie";
  }
@Get("search")
search(@Query('year') searchingYear: string) {
  return `We are searching for a movie made after: ${searchingYear}`
}
```

- @Body() Body 데코레이터를 통해서 body 데이터를 받아올 수 있음
- @Query('year') 쿼리스트링 데이터 데코레이터 (/search?year=2000 요청 이면 2000 받아옴)

- 또한 json 형식의 Data를 그대로 return해도 json을 이해하고 그대로 출력해줌.

#### nest g s movies : generate로 movies service 생성

- module.ts의 Provider에 자동으로 추가

- /entities/movie.entity.ts에 가짜 Movie DB를 만들고
- service에서 private movies: Movie[] = []; 로 가져옴
- service에서 각 비즈니스 로직을 처리하고 결과값을 return해서 controller로 보냄

---

- npm i class-validator, class-transformer

```JS
// main.ts
// 유효성 검사용 파이프 생성
  app.useGlobalPipes(
    new ValidationPipe({
      whitelist: true,  // 아예 dto의 class-validator에 접근하지 못함 (보안성 증가)
      forbidNonWhitelisted: true, // 잘못된 값에 대해 알려줌
      transform: true,
      // 유저가 보낸 Input 값을 원하는 실제 타입으로 변환해줌 (querystring은 string으로 오기때문에 id를 string으로 타입 지정 후에 parseInt로 int로 형변환해, 사용했는데 number로 타입 지정을 해도 에러가 발생하지 않음)
    })
  );

```

- 위 코드를 추가해서 유효성 검사를 해주는 파이프를 생성 후
- create-movie.dto.ts에서

```JS
export class CreateMovieDTO {

  @IsString()
  readonly title: string;
  @IsNumber()
  readonly year: number;
  @IsString({ each: true })
  readonly genres: string[];
}
```

- 각각 데코레이터를 통해 유효성검사를 해주면 맞지 않는 Input값을 보낼 때 검증을 통해 에러를 발생시켜줌

### npm i mapped-types

- 타입을 변환시키고 사용할 수 있게 하는 패키지
- DTO를 변환시켜줌

- nestJS는 타입스크립트를 통해서 보안 뿐 아니라 유효성 검사를 통해서 서버를 보호함.

---

app.module.ts의 Controllers와 Providers의 MovieController, MovieService를 지운 후

### nest g mo movies

- movies 모듈 생성
- app.module.ts의 import에 자동으로 업데이트 됨.
- movies.module.ts의 controllers와 providers에 각각 MovieController, MovieService 추가

- movies.controller.ts에서 req, res도 사용할 수 있음
- getAll((@Req() req, @Res() res))
- express위에서 돌아가기 때문에 가능하지만 fastify나 다른 프레임워크를 사용할수도 있기떄문에 req, res는 추천하지 않음.
