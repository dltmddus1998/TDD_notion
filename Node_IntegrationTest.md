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

## Auth 관련 통합 테스트

---

### Full Code

```jsx
// router/auth.js
import express from 'express';
import {} from 'express-async-errors';
import { body } from 'express-validator';
import { validate } from '../middleware/validator.js';
import * as authController from '../controller/auth.js';
import { isAuth } from '../middleware/auth.js';

const router = express.Router();

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
router.post('/signup', validateSignup, authController.signup);

router.post('/login', validateCredential, authController.login);

router.get('/me', isAuth, authController.me);

export default router;
```

```jsx
// controller/auth.js
import jwt from 'jsonwebtoken';
import bcrypt from 'bcrypt';
import {} from 'express-async-errors';
import * as userRepository from '../data/auth.js';
import { config } from '../config.js';

export async function signup(req, res) {
  const { username, password, name, email, url } = req.body;
  const found = await userRepository.findByUsername(username);
  if (found) {
    return res.status(409).json({ message: `${username} already exists` });
  }
  const hashed = await bcrypt.hash(password, config.bcrypt.saltRounds);
  const userId = await userRepository.createUser({
    username,
    password: hashed,
    name,
    email,
    url,
  });
  const token = createJwtToken(userId);
  res.status(201).json({ token, username });
}

export async function login(req, res) {
  const { username, password } = req.body;
  const user = await userRepository.findByUsername(username);
  if (!user) {
    return res.status(401).json({ message: 'Invalid user or password' });
  }
  const isValidPassword = await bcrypt.compare(password, user.password);
  if (!isValidPassword) {
    return res.status(401).json({ message: 'Invalid user or password' });
  }
  const token = createJwtToken(user.id);
  res.status(200).json({ token, username });
}

function createJwtToken(id) {
  return jwt.sign({ id }, config.jwt.secretKey, {
    expiresIn: config.jwt.expiresInSec,
  });
}

export async function me(req, res, next) {
  const user = await userRepository.findById(req.userId);
  if (!user) {
    return res.status(404).json({ message: 'User not found' });
  }
  res.status(200).json({ token: req.token, username: user.username });
}
```

```jsx
// auth.test.js
// 테스트 전! 서버 시작 및 데이터베이스 초기화! 설정
// 테스트 후! 데이터베이스 깨끗하게 청소해놓기
import { sequelize } from '../../db/database.js';
import { startServer, stopServer } from '../../app.js';
import axios from 'axios';
import faker from 'faker';

describe('Auth APIs', () => {
    let server;
    let request;
    // server start
    // 서버는 비동기 -> async/await 사용
    beforeAll(async () => {
        server = await startServer();
        request = axios.create({
            baseURL: 'http://localhost:8080',
            validateStatus: null,
        });
    });

    // 데이터베이스&서버 초기화 
    afterAll(async () => { 
        await sequelize.drop();
        await stopServer(server);
    });

    describe('POST to /auth/signup', () => {
        it('returns 201 and authorization token when user details are valid', async () => {
            const user = makeValidUserDetails();
            const res = await request.post('/auth/signup', user);

            expect(res.status).toBe(201);
            expect(res.data.token.length).toBeGreaterThan(0);
        });

        it('returns 409 when username has already been taken', async () => {
            const user = makeValidUserDetails();
            const firstSignup = await request.post('/auth/signup', user);

            expect(firstSignup.status).toBe(201);

            const res = await request.post('/auth/signup', user);

            expect(res.status).toBe(409);
            expect(res.data.message).toBe(`${user.username} already exists`);
        });

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

        it('returns 400 when password is too short', async () => {
            const user = {
                ...makeValidUserDetails(),
                password: '123',
            };

            const res = await request.post('/auth/signup', user);

            expect(res.status).toBe(400);
            expect(res.data.message).toBe('password should be at least 5 characters');
        });
    });

    describe('POST to /auth/login', () => {
        it('returns 200 and authorization token when user credeintials are valid', async () => {
            const user = await createNewUserAccount();

            const res = await request.post('/auth/login', {
                username: user.username,
                password: user.password,
            });

            expect(res.status).toBe(200);
            expect(res.data.token.length).toBeGreaterThan(0);
        });

        it('returns 401 when password is incorrect', async () => {
            const user = await createNewUserAccount();
            const wrongPassword = user.password.toUpperCase();

            const res = await request.post('/auth/login', {
                username: user.username,
                password: wrongPassword,
            });

            expect(res.status).toBe(401);
            expect(res.data.message).toMatch('Invalid user or password');
        });

        it('returns 401 when username is not found', async () => {
            const someRandomNonExistentUser = faker.random.alpha({ count: 32 });

            const res = await request.post('/auth/login', {
                username: someRandomNonExistentUser,
                password: faker.internet.password(10, true),
            });

            expect(res.status).toBe(401);
            expect(res.data.message).toMatch('Invalid user or password');
        });
    });

    describe('GET /auth/me', () => {
        it('returns user details when valid token is present in Authorization header', async () => {
            const user = await createNewUserAccount();

            const res = await request.get('/auth/me', {
                headers: { Authorization: `Bearer ${user.jwt}` },
            });

            expect(res.status).toBe(200);
            expect(res.data).toMatchObject({
                token: user.jwt,
                username: user.username,
            });
        });
    });

    async function createNewUserAccount() {
        const userDetails = makeValidUserDetails();
        const prepareUserResponse = await request.post('/auth/signup', userDetails);
        return {
            ...userDetails,
            jwt: prepareUserResponse.data.token,
        };
    }
});

function makeValidUserDetails() {
    const fakeUser = faker.helpers.userCard();
    return { 
        name: fakeUser.name, 
        username: fakeUser.username, 
        email: fakeUser.email, 
        password: faker.internet.password(10, true),
    };
}
```

### ⚙️ 사전 설정

```jsx
describe('Auth APIs', () => {
    let server;
    let request;
    // server start
    // 서버는 비동기 -> async/await 사용
    beforeAll(async () => {
        server = await startServer();
        request = axios.create({
            baseURL: 'http://localhost:8080',
            validateStatus: null,
        });
    });

    // 데이터베이스&서버 초기화 
    afterAll(async () => { 
        await sequelize.drop();
        await stopServer(server);
    });
.
.
.
```

✔︎ 테스트 시작 전 서버를 시작해야하므로 `app.js`에서 선언해준 `startServer()` 메서드를 실행해준다.

✔︎ 각각의 테스트가 돌아갈 때마다 서버와 데이터베이스가 초기화돼야하므로 해당 테스트가 끝난 후(`afterAll()`) 데이터베이스를 비워주고(`sequelize.drop()`) 서버를 멈춘다(stopServer(server)).

### 👩‍👩‍👧‍👦 Signup

```jsx
describe('POST to /auth/signup', () => {
  it('returns 201 and authorization token when user details are valid', async () => {
			// 1
      const user = makeValidUserDetails();
      const res = await request.post('/auth/signup', user);
			// 2
      expect(res.status).toBe(201);
      expect(res.data.token.length).toBeGreaterThan(0);
  });

  it('returns 409 when username has already been taken', async () => {
      const user = makeValidUserDetails();
      const firstSignup = await request.post('/auth/signup', user);

      expect(firstSignup.status).toBe(201);

      const res = await request.post('/auth/signup', user);

      expect(res.status).toBe(409);
      expect(res.data.message).toBe(`${user.username} already exists`);
  });
.
.
.
function makeValidUserDetails() {
    const fakeUser = faker.helpers.userCard();
    return { 
        name: fakeUser.name, 
        username: fakeUser.username, 
        email: fakeUser.email, 
        password: faker.internet.password(10, true),
    };
}
```

**`makeValidUserDetails()`**

→ 임의의 회원정보를 만들어주는 메서드 선언. 

→ 공통적으로 사용되므로 코드 중복을 막기 위해서 메서드 사용.

`**201**`

1. 임의의 회원 `user`를 만들어서 이를 이용해 `/auth/signup`의 경로로 요청을 보내 `res`에 할당해준다.
2. 회원가입이 될 경우, 해당 응답에 들어가 있는 토큰의 길이가 최소 1 이상이므로 다음과 같이 표현해준다.

`**409**`

✍️ 위와 같은 방식이다.

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

### 🙋‍♀️ Login

```jsx
describe('POST to /auth/login', () => {
        it('returns 200 and authorization token when user credeintials are valid', async () => {
            const user = await createNewUserAccount();

            const res = await request.post('/auth/login', {
                username: user.username,
                password: user.password,
            });

            expect(res.status).toBe(200);
            expect(res.data.token.length).toBeGreaterThan(0);
        });

        it('returns 401 when password is incorrect', async () => {
            const user = await createNewUserAccount();
            const wrongPassword = user.password.toUpperCase();

            const res = await request.post('/auth/login', {
                username: user.username,
                password: wrongPassword,
            });

            expect(res.status).toBe(401);
            expect(res.data.message).toMatch('Invalid user or password');
        });

        it('returns 401 when username is not found', async () => {
            const someRandomNonExistentUser = faker.random.alpha({ count: 32 });

            const res = await request.post('/auth/login', {
                username: someRandomNonExistentUser,
                password: faker.internet.password(10, true),
            });

            expect(res.status).toBe(401);
            expect(res.data.message).toMatch('Invalid user or password');
        });

				async function createNewUserAccount() {
	        const userDetails = makeValidUserDetails();
	        const prepareUserResponse = await request.post('/auth/signup', userDetails);
	        return {
	            ...userDetails,
	            jwt: prepareUserResponse.data.token,
	        };
	    }
    });
```

**`createNewUserAccout()`**

→ `makeValidUserDetails()`를 이용하여 만든 임의의 회원정보를 이용하여 회원가입한 것을 `prepareUserResponse`에 할당해준 후 토큰정보인 `jwt`를 포함한 정보들을 반환하는 메서드를 선언해준다.

**`401` - password incorrect**

✔︎ 잘못된 비밀번호(`wrongPassword`)를 `request.post()`에 담아서 보내면 `401` 에러가 발생한다.

**`401` - username not found**

✔︎ `username`이 일치하지 않은 경우이므로, 무작위로 사용자명을 할당해준 후 이를 `request.post()`에 담아서 보내면 `401` 에러가 발생한다.

### 🔨 회원정보 찾기

```jsx
describe('GET /auth/me', () => {
    it('returns user details when valid token is present in Authorization header', async () => {
        const user = await createNewUserAccount();

        const res = await request.get('/auth/me', {
            headers: { Authorization: `Bearer ${user.jwt}` },
        });

        expect(res.status).toBe(200);
        expect(res.data).toMatchObject({
            token: user.jwt,
            username: user.username,
        });
    });
});
```

✔︎ 올바르게 회원 정보를 `get`했을 때 `res.data`안에 토큰과 유저명 정보가 담겨있음은 개발 코드를 통해 알 수 있다.
