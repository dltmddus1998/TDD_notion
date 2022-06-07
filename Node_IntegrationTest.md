## 🏭 app.js Refactoring

---

```jsx
// app.js
import express from 'express';
import 'express-async-errors';
import cors from 'cors';
import morgan from 'morgan';
import helmet from 'helmet';
import tweetsRouter from './router/tweets.js';
import authRouter from './router/auth.js';
import { config } from './config.js';
import { initSocket, getSocketIO } from './connection/socket.js';
import { sequelize } from './db/database.js';
import { TweetController } from './controller/tweet.js';
import * as tweetRepository from './data/tweet.js';

const corsOption = {
  origin: config.cors.allowedOrigin,
  optionsSuccessStatus: 200,
};

export async function startServer() {
  const app = express();
  app.use(express.json());
  app.use(helmet());
  app.use(cors(corsOption));
  app.use(morgan('tiny'));

  app.use('/tweets', tweetsRouter(new TweetController(tweetRepository, getSocketIO)));
  app.use('/auth', authRouter);

  app.use((req, res, next) => {
    res.sendStatus(404);
  });

  app.use((error, req, res, next) => {
    console.error(error);
    res.sendStatus(500);
  });

  // await을 붙이면 sync()에서 문제가 생길시 바로 에러를 던짐.
  await sequelize.sync();

  console.log('Server is started....');
  const server = app.listen(config.port);
  initSocket(server);
  return server;
}

export async function stopServer(server) {
  // 그냥 아래처럼 하면  sequelize.close에서 문제가 생겼을 때 에러를 던질수 없음 -> 비동기적으로 표현하자
  // server.close(() => {
  //   sequelize.close();
  // });
  return new Promise((resolve, reject) => {
    server.close(async() => {
      try {
        await sequelize.close();
        resolve();
      } catch(err) {
        reject(err);
      }
    })
  })
}
```

자동으로 시작하는 코드가 필요하므로 index.js 생성

```jsx
// index.js
import { startServer } from './app.js';

startServer();
```

package.json에서 main과 npm start부분 변경

```jsx
"main": "index.js",
"scripts": {
	"start": "nodemon index"
} 
```

<br>

# 노드-통합테스트

### 🍯 **꿀팁**

‼️ **Validate 관련 테스트를 진행할 때 사용하는 API**

[Globals · Jest](https://jestjs.io/docs/api#testeachtablename-fn-timeout)

```tsx
// router/auth.js VALIDATE
.
.
.
const validateCredential = [
  body('username')
    .trim()
    .notEmpty()
    .withMessage('username should be at least 5 characters'),
  body('password')
    .trim()
    .isLength({ min: 5 })
    .withMessage('password should be at least 5 characters'),
  validate,
];

const validateSignup = [
  ...validateCredential,
  body('name').notEmpty().withMessage('name is missing'),
  body('email').isEmail().normalizeEmail().withMessage('invalid email'),
  body('url')
    .isURL()
    .withMessage('invalid URL')
    .optional({ nullable: true, checkFalsy: true }),
  validate,
];
.
.
.
```

```tsx
// auth.test.js VALIDATE
test.each([
    { missingFieldName: 'name', expectedMessage: 'name is missing' },
    { 
        missingFieldName: 'username', 
        expectedMessage: 'username should be at least 5 characters'},
    {
        missingFieldName: 'email',
        expectedMessage: 'invalid email'
    },
    {
        missingFieldName: 'password',
        expectedMessage: 'password should be at least 5 characters'
    },
])(`returns 400 when $missingFieldName field is missing`, async ({ missingFieldName, expectedMessage }) => {
    const user = makeValidUserDetails();
    delete user[missingFieldName];
    const res = await request.post('/auth/signup', user);

    expect(res.status).toBe(400);
    expect(res.data.message).toBe(expectedMessage);
});
```

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/25567121-ccb5-4a79-ad5e-d66526ad2919/Untitled.png)

✔︎ `test.each`라는 API를 사용하여 `router/auth.js`에서 회원정보 검증을 위해 코딩한 부분에 대해 테스트 코드를 작성했다.

`test.each((table)(`<name>`, () => {}))`

```tsx
// auth.test.js
it('returns 400 when password is too short', async () => {
  const user = {
      ...makeValidUserDetails(),
      password: '123',
  };

  const res = await request.post('/auth/signup', user);

  expect(res.status).toBe(400);
  expect(res.data.message).toBe('password should be at least 5 characters');
});
```

✔︎ 비밀번호가 5자리 미만인 경우에 관한 부분은 따로 테스트 코드를 작성해줬다.