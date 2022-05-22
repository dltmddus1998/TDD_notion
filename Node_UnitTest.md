## 🧪 백엔드 개발에서의 테스트 코드

---

> **백엔드 개발에서, 통합 테스트(Integration Test)는 ROI(Retrun Of Investment)이다.**
> 

**+) ROI: 투자한 것 대비 가장 높은 결과값을 대비할 수 있다.**

> **백엔드 개발에서 통합 테스트(Integration Test)는 client가 사용하는 노출된 WEB API만을 테스트 하는 것이다.
ex) login API, tweet API**
> 

> **내부적으로 모든 비즈니스 로직과 모듈들이 서로 상호작용을 잘하여 동작하는지, 데이터베이스에 잘 읽고 써지는지 등에 대한 표면적인 것에 대한 확인이 가능하다.**
> 

## ⚙️ 노드 테스트 환경 셋업

---

### node-mocks-http

> **node express 환경에서 request/response를 mocking하도록 도와주는 라이브러리**
> 

1. 터미널에서 다음과 같이 jest, node-mokes-http를 설치해준다.
    
    `npm i --save-dev jest node-mocks-http`
    
2. jest는 module타입을 제공하지 않기 때문에 commonjs로 변환해주는 부분이 필요로 하다.
    
    ✔︎ `npm i --save-dev @babel/plugin-transform-modules-commonjs`
    
    ✔︎ `.babelrc` 에 다음과 같이 작성
    
    ```json
    {
        "env": {
            "test": {
                "plugins": [
                    "@babel/plugin-transform-modules-commonjs"
                ]
            }
        }
    }
    ```
    
    ✔︎ `jest --init`을 통해 `jest.config.mjs` 생성
    

## 🖇 테스트용 환경변수 설정하기

---

jest 동작 이전에 dotenv 동작하도록 jest.config.mjs에 설정

```jsx
export default {
	.
	.
	setupFiles: ['dotenv/config'],
	.
	. 
}
```

테스트 실행시 프로젝트 경로에 있는 .env.test 를 사용하도록 명시. 

```json
// package.json
"scripts": {
  "test": "DOTENV_CONFIG_PATH=./.env.test jest --watchAll",
  "start": "nodemon app"
},
```

<br>
<br>

## Dwitter - 테스트 코드 작성하기

---

### Middleware Test Code

> **외부 라이브러리, 데이터베이스에 접근하는 것은 테스트 코드의 속도를 느려지게 하는 요인이 될 수 있으므로 이 부분들은 모두 mock 처리해준다.**
> 

```jsx
// middleware/auth.js
import jwt from 'jsonwebtoken';
import { config } from '../config.js';
import * as userRepository from '../data/auth.js';

const AUTH_ERROR = { message: 'Authentication Error' };

export const isAuth = async (req, res, next) => {
  const authHeader = req.get('Authorization');
  if (!(authHeader && authHeader.startsWith('Bearer '))) {
    return res.status(401).json(AUTH_ERROR);
  }

  const token = authHeader.split(' ')[1];
  jwt.verify(token, config.jwt.secretKey, async (error, decoded) => {
    if (error) {
      return res.status(401).json(AUTH_ERROR);
    }
    const user = await userRepository.findById(decoded.id);
    if (!user) {
      return res.status(401).json(AUTH_ERROR);
    }
    req.userId = user.id; // req.customData
    req.token = token;
    next();
  });
};
```

```jsx
// middleware/test/auth.test.js
import httpMocks from 'node-mocks-http';
import { isAuth } from '../auth.js';
import faker from 'faker';
import jwt from 'jsonwebtoken';
import * as userRepository from '../../data/auth.js';

jest.mock('jsonwebtoken');
jest.mock('../../data/auth.js');

describe('Auth Middlewear', () => {
    // Fail
    it('returns 401 for the request without Authorization header', async () => {
        const request = httpMocks.createRequest({
            method: 'GET',
            url: '/tweets',
        });
        const response = httpMocks.createResponse();
        const next = jest.fn();

        await isAuth(request, response, next);

        expect(response.statusCode).toBe(401);
        expect(response._getJSONData().message).toBe('Authentication Error');
        // 401 error시에 다음 로직으로 넘어가지 않도록!! -> next() x
        // expect(next).toHaveBeenCalled(0)
        expect(next).not.toBeCalled();
    });

    it('returns 401 for the request with unsupported Authorization header', async () => {
        const request = httpMocks.createRequest({
            method: 'GET',
            url: '/tweets',
            headers: { Authorization: 'Basic' },
        });
        const response = httpMocks.createResponse();
        const next = jest.fn();

        await isAuth(request, response, next);

        expect(response.statusCode).toBe(401);
        expect(response._getJSONData().message).toBe('Authentication Error');
        expect(next).not.toBeCalled();
    });

    it('returns 401 for the request with invalid token', async () => {
        const token = faker.random.alphaNumeric(128); 
        const request = httpMocks.createRequest({
            method: 'GET',
            url: '/tweets',
            headers: { Authorization: `Bearer ${token}` },
        });
        const response = httpMocks.createResponse();
        const next = jest.fn();
        jwt.verify = jest.fn((token, secret, callback) => {
            callback(new Error('bad token'), undefined);
        });

        await isAuth(request, response, next);

        expect(response.statusCode).toBe(401);
        expect(response._getJSONData().message).toBe('Authentication Error');
        expect(next).not.toBeCalled();
    });

    it('returns 401 when cannot find a user by id from the JWT', async () => {
        const token = faker.random.alphaNumeric(128); 
        const userId = faker.random.alphaNumeric(32);
        const request = httpMocks.createRequest({
            method: 'GET',
            url: '/tweets',
            headers: { Authorization: `Bearer ${token}` },
        });
        const response = httpMocks.createResponse();
        const next = jest.fn();
        jwt.verify = jest.fn((token, secret, callback) => {
            callback(undefined, { id: userId });
        });
        userRepository.findById = jest.fn((id) => Promise.resolve(undefined));

        await isAuth(request, response, next);

        expect(response.statusCode).toBe(401);
        expect(response._getJSONData().message).toBe('Authentication Error');
        expect(next).not.toBeCalled();
    });

    // Success
    it('passes a request with valid Authorization header with token', async () => {
        const token = faker.random.alphaNumeric(128); 
        const userId = faker.random.alphaNumeric(32);
        const request = httpMocks.createRequest({
            method: 'GET',
            url: '/tweets',
            headers: { Authorization: `Bearer ${token}` },
        });
        const response = httpMocks.createResponse();
        const next = jest.fn();
        jwt.verify = jest.fn((token, secret, callback) => {
            callback(undefined, { id: userId });
        });
        userRepository.findById = jest.fn((id) => Promise.resolve({id}));

        await isAuth(request, response, next);

        expect(request).toMatchObject({ userId, token });
        expect(next).toHaveBeenCalledTimes(1);
    });
})
```

```jsx
// jest.config.mjs
.
.
.
// An array of glob patterns indicating a set of files for which coverage information should be collected
collectCoverageFrom: ['**/*.{js,jsx}', '!**/node_modules/**'],
.
.
.
```

<aside>
💡 확장자가 js, jsx인 모든 파일에 대해서 coverage를 확인할 수 있게 하는 collectCoverageFrom 옵션을 설정해준다. 
단, node_modules 안에 있는 파일들에 대해선 coverage를 보지 않게 따로 설정해준다.

</aside>

<img src="https://user-images.githubusercontent.com/73332608/168229409-f7a761ba-c023-4b48-990d-0a46154226fe.png" width="800" height="480">

<br />
<br />

### Middleware Test Code - Validator

```tsx
// middleware/validator.js
import { validationResult } from 'express-validator';

export const validate = (req, res, next) => {
  const errors = validationResult(req);
  if (errors.isEmpty()) {
    return next();
  }
  return res.status(400).json({ message: errors.array()[0].msg });
};
```

```tsx
import httpMocks from 'node-mocks-http';
import faker from 'faker';
import * as validator from 'express-validator';
import { validate } from '../validator.js';

jest.mock('express-validator');

describe('Validator Middlewear', () => {
    // Success
    it('calls next if there are no validation errors', () => {
        const request = httpMocks.createRequest();
        const response = httpMocks.createResponse();
        const next = jest.fn();
        validator.validationResult = jest.fn(() => ({ isEmpty: () => true }));

        validate(request, response, next);

        expect(next).toBeCalled();
    })
    it('returns 400 if there are validation errors', () => {
        const request = httpMocks.createRequest();
        const response = httpMocks.createResponse();
        const next = jest.fn();
        const errorMessage = faker.random.words(3);
        validator.validationResult = jest.fn(() => ({ 
            isEmpty: () => false, 
            array: () => [{ msg: errorMessage }] 
        }));

        validate(request, response, next);

        expect(next).not.toBeCalled();
        expect(response.statusCode).toBe(400);
        expect(response._getJSONData().message).toBe(errorMessage);
    });
});
```

<br>
<br>
<br>

### Tweet Controller Test Code

⭐️ **그 전에 트윗 컨트롤러 리팩토링하기!!**

**원래 코드**

```jsx
// controller/tweet.js
import * as tweetRepository from '../data/tweet.js';
import { getSocketIO } from '../connection/socket.js';

export async function getTweets(req, res) {
  const username = req.query.username;
  const data = await (username
    ? tweetRepository.getAllByUsername(username)
    : tweetRepository.getAll());
  res.status(200).json(data);
}

export async function getTweet(req, res, next) {
  const id = req.params.id;
  const tweet = await tweetRepository.getById(id);
  if (tweet) {
    res.status(200).json(tweet);
  } else {
    res.status(404).json({ message: `Tweet id(${id}) not found` });
  }
}

export async function createTweet(req, res, next) {
  const { text } = req.body;
  const tweet = await tweetRepository.create(text, req.userId);
  res.status(201).json(tweet);
  getSocketIO().emit('tweets', tweet);
}

export async function updateTweet(req, res, next) {
  const id = req.params.id;
  const text = req.body.text;
  const tweet = await tweetRepository.getById(id);
  if (!tweet) {
    return res.status(404).json({ message: `Tweet not found: ${id}` });
  }
  if (tweet.userId !== req.userId) {
    return res.sendStatus(403);
  }
  const updated = await tweetRepository.update(id, text);
  res.status(200).json(updated);
}

export async function deleteTweet(req, res, next) {
  const id = req.params.id;
  const tweet = await tweetRepository.getById(id);
  if (!tweet) {
    return res.status(404).json({ message: `Tweet not found: ${id}` });
  }
  if (tweet.userId !== req.userId) {
    return res.sendStatus(403);
  }
  await tweetRepository.remove(id);
  res.sendStatus(204);
}
```

**리팩토링한 코드**

```jsx
// controller/tweet.js
export class TweetController {
  constructor(tweetRepository, getSocket) {
    this.tweets = tweetRepository;
    this.getSocket = getSocket;
  }

  async getTweets(req, res) {
    const username = req.query.username;
    const data = await (username
      ? this.tweets.getAllByUsername(username)
      : this.tweets.getAll());
    res.status(200).json(data);
  }

  async getTweet(req, res, next) {
    const id = req.params.id;
    const tweet = await this.tweets.getById(id);
    if (tweet) {
      res.status(200).json(tweet);
    } else {
      res.status(404).json({ message: `Tweet id(${id}) not found` });
    }
  }

  async createTweet(req, res, next) {
    const { text } = req.body;
    const tweet = await this.tweets.create(text, req.userId);
    res.status(201).json(tweet);
    this.getSocket.emit('tweets', tweet);
  }
  
  async updateTweet(req, res, next) {
    const id = req.params.id;
    const text = req.body.text;
    const tweet = await this.tweets.getById(id);
    if (!tweet) {
      return res.status(404).json({ message: `Tweet not found: ${id}` });
    }
    if (tweet.userId !== req.userId) {
      return res.sendStatus(403);
    }
    const updated = await this.tweets.update(id, text);
    res.status(200).json(updated);
  }
  
  async deleteTweet(req, res, next) {
    const id = req.params.id;
    const tweet = await this.tweets.getById(id);
    if (!tweet) {
      return res.status(404).json({ message: `Tweet not found: ${id}` });
    }
    if (tweet.userId !== req.userId) {
      return res.sendStatus(403);
    }
    await this.tweets.remove(id);
    res.sendStatus(204);
  }
}
```

```jsx
// router/tweets.js
// Refactoring: routing부분을 클래스로 묶어주었다
import express from 'express';
import 'express-async-errors';
import { body } from 'express-validator';
import * as tweetController from '../controller/tweet.js';
import { isAuth } from '../middleware/auth.js';
import { validate } from '../middleware/validator.js';

const router = express.Router();

const validateTweet = [
  body('text')
    .trim()
    .isLength({ min: 3 })
    .withMessage('text should be at least 3 characters'),
  validate,
];

export default function tweetsRouter(tweetController) {
  // GET /tweet
  // GET /tweets?username=:username
  router.get('/', isAuth, tweetController.getTweets);

  // GET /tweets/:id
  router.get('/:id', isAuth, tweetController.getTweet);

  // POST /tweeets
  router.post('/', isAuth, validateTweet, tweetController.createTweet);

  // PUT /tweets/:id
  router.put('/:id', isAuth, validateTweet, tweetController.updateTweet);

  // DELETE /tweets/:id
  router.delete('/:id', isAuth, tweetController.deleteTweet);
}
```

```jsx
// app.js
// Refactoring: /tweets경로를 router/tweets.js의 tweetsRouter를 통해 가져오게 한다
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

const app = express();

const corsOption = {
  origin: config.cors.allowedOrigin,
  optionsSuccessStatus: 200,
};
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

sequelize.sync().then(() => {
  console.log('Server is started....');
  const server = app.listen(config.port);
  initSocket(server);
});
```

### 👀 여기서 잠깐!!

<aside>
💡 트윗 컨트롤러의 리팩토링을 진행한 이유는 무엇일까요?
</aside>

<br>

✔︎ 모듈에서 어떤 특정 라이브러리를 바로 가져다 쓰고, 다른 모듈을 내부에서 바로 생성하거나 (→ **원래 코드의 경우 import를 통해 데이터 등을 가져왔다**) 호출하면, **모듈간의 결합도가 높아진다**. [**타이트하게 커플링되었다**]

✔︎ 라이브러리나 외부 모듈의 함수와 사용법이 변경되는 경우, 그걸 사용하는 모든 곳에서 코드 수정이 일어나야 하므로 **타이트한 커플링**은 지양해야한다.

위에서 말한 부분을 지양하는 방법으로는, ***Dependency Injection***이 있다.

→ 해당 클래스/모듈 안에서 필요할 때마다 생성하거나 직접적으로 접근하는 것이 아니라, **외부 라이브러리/모듈을 한단계 감싸는 클래스를 만들고, 그 외부 의존성을 밖에서 주입해주는 것이다.**

🖊 이를 통해, 업데이트가 필요한 경우, 그 외부 의존성 내부만 업데이트 해주면 되는 것이다. [**디커플링**]

⭐️ **자바스크립트에서 mock으로 간편하게 덮어쓰기가 되지만, 이를 남용할 경우(모듈간의 결합도가 높은데, 코드 개선 없이 대충 덮어 쓰는 등...) 나중에 유지보수가 힘들어진다. 
 따라서, 외부 라이브러리나 외부 모듈 전체를 mock하는 테스트 코드는 지양해야한다.**

 ‼️ **[출처]by ellie of Dream Coding ‼️**

