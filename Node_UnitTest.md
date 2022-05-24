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

<br>
<br>
<br>

### Test Code

```jsx
import faker from 'faker';
import { TweetController } from '../tweet.js';
import httpMocks from 'node-mocks-http';

describe('TweetController', () => {
    let tweetController;
    let tweetsRepository;
    let mockedSocket;
    beforeEach(() => {
        tweetsRepository = {};
        mockedSocket = { emit: jest.fn() };
        tweetController = new TweetController(
            tweetsRepository,
            () => mockedSocket
        );
    });

    // getTweets
    describe('getTweets', () => {
        it('returns all tweets when username is not provided', async () => {
            const request = httpMocks.createRequest();
            const response = httpMocks.createResponse();
            const allTweets = [
                {text: faker.random.words(3)},
                {text: faker.random.words(3)}
            ];
            tweetsRepository.getAll = () => allTweets;

            await tweetController.getTweets(request, response);

            expect(response.statusCode).toBe(200);
            expect(response._getJSONData()).toEqual(allTweets);
        });

        it ('returns all tweets for the given user when username is provided', async () => {
            const username = faker.internet.userName();
            const request = httpMocks.createRequest({
                query: { username },
            });
            const response = httpMocks.createResponse();
            const userTweets = [
                {text: faker.random.words(3)}
            ];
            tweetsRepository.getAllByUsername = jest.fn(() => userTweets);

            await tweetController.getTweets(request, response);

            expect(response.statusCode).toBe(200);
            expect(response._getJSONData()).toEqual(userTweets);
            expect(tweetsRepository.getAllByUsername).toHaveBeenCalledWith(username);
        });
    });

    // getTweet
    describe('getTweet', () => { 
        let tweetId, request, response, next;

        beforeEach(() => {
            tweetId = faker.random.alphaNumeric(16);
            request = httpMocks.createRequest({
                params: { id: tweetId }
            });
            response = httpMocks.createResponse();
            next = jest.fn();
        });

        it('returns the tweet if tweet exists', async () => {
            const aTweet = { text: faker.random.words(3) };
            tweetsRepository.getById = jest.fn(() => aTweet);

            await tweetController.getTweet(request, response, next);

            expect(response.statusCode).toBe(200);
            expect(response._getJSONData()).toEqual(aTweet);
            expect(tweetsRepository.getById).toHaveBeenCalledWith(tweetId);
        });

        it("returns 404 error if tweet isn't exist", async () => {
            tweetsRepository.getById = jest.fn(() =>  undefined);

            await tweetController.getTweet(request, response, next);

            expect(response.statusCode).toBe(404);
            expect(response._getJSONData()).toMatchObject({
                message: `Tweet id(${tweetId}) not found`
            });
            expect(tweetsRepository.getById).toHaveBeenCalledWith(tweetId);
        });
    });

    // createTweet
    describe('createTweet', () => {
        let newTweet, authorId, request, response, next;
        beforeEach(() => {
            newTweet = faker.random.words(3);
            authorId = faker.random.alphaNumeric(16);
            request = httpMocks.createRequest({
                body: { text: newTweet },
                userId: authorId,
            });
            response = httpMocks.createResponse();
            next = jest.fn();
        });

        it('returns 201 with created tweet object including userId', async () => {
            tweetsRepository.create = jest.fn((text, userId) => {
                return {
                    text,
                    userId,
                };
            });

            await tweetController.createTweet(request, response, next);

            expect(response.statusCode).toBe(201);
            console.log(response._getJSONData());
            expect(response._getJSONData()).toMatchObject({
                text: newTweet,
                userId: authorId,
            });
            expect(tweetsRepository.create).toHaveBeenCalledWith(newTweet, authorId);
        });

        it('should send an event to a websocket channel', async () => {
            tweetsRepository.create = jest.fn((text, userId) => {
                return {
                    text, 
                    userId,
                };
            });

            await tweetController.createTweet(request, response);
            
            expect(mockedSocket.emit).toHaveBeenCalledWith('tweets', {
                text: newTweet,
                userId: authorId,
            });
        });
    });

    // updateTweet
    describe('updateTweet', () => {
        let tweetId, authorId, updatedText, request, response;
        beforeEach(() => {
            updatedText = faker.random.words(3);
            tweetId = faker.random.alphaNumeric(16);
            authorId = faker.random.alphaNumeric(16);
            request = httpMocks.createRequest({
                body: { text: updatedText },
                params: { id: tweetId },
                userId: authorId,
            });
            response = httpMocks.createResponse();
        });

        it('returns 200 with updated tweet object including userId', async () => {
            tweetsRepository.getById = () => ({
                text: faker.random.words(3),
                userId: authorId,
            });
            tweetsRepository.update = (tweetId, newText) => ({
                text: newText
            });

            await tweetController.updateTweet(request, response);

            expect(response.statusCode).toBe(200);
            expect(response._getJSONData()).toMatchObject({
                text: updatedText,
            });
        });

        it('returns 403 and should not update the repository if the tweet does not belong to the user', async () => {
            tweetsRepository.getById = () => ({
                text: faker.random.words(3),
                userId: faker.random.alphaNumeric(16),
            });
            tweetsRepository.udpate = jest.fn();

            await tweetController.updateTweet(request, response);

            expect(response.statusCode).toBe(403);
        });

        it('returns 404 and should not update the repository if the tweet does not exist', async () => {
            tweetsRepository.getById = () => undefined;
            tweetsRepository.update = jest.fn();

            await tweetController.updateTweet(request, response);

            expect(response.statusCode).toBe(404);
            expect(response._getJSONData().message).toBe(`Tweet not found: ${tweetId}`);
        });
    });

    // deleteTweet
    describe('deleteTweet', () => {
        let tweetId, authorId, request, response;
        beforeEach(() => {
            tweetId = faker.random.alphaNumeric(16);
            authorId = faker.random.alphaNumeric(16);
            request = httpMocks.createRequest({
                params: { id: tweetId },
                userId: authorId,
            });
            response = httpMocks.createResponse();
        });

        it('returns 204 and remove the tweet from the repository if the tweet exists', async () => {
            tweetsRepository.getById = () => ({
                userId: authorId,
            });
            tweetsRepository.remove = jest.fn();

            await tweetController.deleteTweet(request, response);

            expect(response.statusCode).toBe(204);
            expect(tweetsRepository.remove).toHaveBeenCalledWith(tweetId);
        });

        it('returns 403 and should not update the repository if the tweet does not belong to the user', async () => {
            tweetsRepository.getById = () => ({
                userId: faker.random.alphaNumeric(16),
            });
            tweetsRepository.remove = jest.fn();

            await tweetController.deleteTweet(request, response);

            expect(response.statusCode).toBe(403);
            expect(tweetsRepository.remove).not.toHaveBeenCalled();
        });

        it('returns 404 and should not update the repository if the tweet does not exist', async () => {
            tweetsRepository.getById = () => undefined;
            tweetsRepository.remove = jest.fn();

            await tweetController.deleteTweet(request, response);

            expect(response.statusCode).toBe(404);
            expect(response._getJSONData().message).toBe(`Tweet not found: ${tweetId}`);
            expect(tweetsRepository.remove).not.toHaveBeenCalled();
        });
    });
});
```

⭐️ `tweetController`는 *beforeEach*에서 설정해준다.

‼️ **각각의 테스트들이 개별적인 환경을 가지기 위해선 오브젝트들에 대해선 초기화해주는 것이 중요하다** ‼️

✔︎ Dependency Injection을 사용하여 모듈간 디커플링을 시켰기 때문에, 모듈 전체를 mock할 필요는 없다.

`getSocket`에 해당하는 부분은 mock했다. 
→ *controller*의 `getSocket()`에 해당하는 코드에 `emit`이라는 메서드가 있다. 이를 활용하여, `emit`을 *jest function*으로 정의했다.

❕ **여러가지 종류별로 묶기 위해서 describe 안에서도 describe로 각각 묶어준다.**

⭐️ **getTweets**

1. **username이 없는 경우**
    
    ✔︎ `tweetRepository.getAll = () ⇒ allTweets;`
    → **`tweetRepository`라는 빈 오브젝트에 `getAll`이라는 함수를 추가한다.** `getAll`을 호출하였을 때 `allTweets`을 리턴한다.
    
    ✔︎ 모든 데이터가 준비됐을 때, `tweetController`에 있는 `getTweets`함수를 호출하여 준비된 `request`와 `response`를 전달해준다.
    
2. **username이 있는 경우**
    
    ✔︎ request에 username을 포함한 query 키를 추가하여준다.
    
    ✔︎ `tweetRepository.getAllByUsername = jest.fn(() => userTweets);`
    **→ `tweetRepository`라는 빈 오브젝트에** `getAllByUsername`이라는 함수를 추가한다.
    
    ✔︎`expect(tweetRepository.getAllByUsername).toHaveBeenCalledWith(username);`
    **→ `getAllByUsername`이 호출됐는지 확인했다.**
    

⭐️ **getTweet**

‼️ **위와 같이 초기화 상태를 필요로 하므로 beforeEach를 이용했다.**

1. **tweet이 존재할 경우** 
    
    ✔︎ `aTweet`이라는 변수에 *faker*를 사용하여 랜덤한 문자열을 값으로 둔 `text`라는 키를 갖는 오브젝트를 할당한다.
    
    ✔︎ `tweetsRepository.getById = jest.fn(() ⇒ aTweet);`
    **→ getById를 호출했을 때 aTweet을 리턴한다.**
    
    `expect(tweetsRepository.getById).toHaveBeenCalledWith(tweetId);`
    **→ getById를 호출했을 때 tweetId가 불러와짐을 알 수 있다.**
    
2. tweet이 없을 경우
    
    ✔︎ 위와는 반대로 `tweet`이 없을 경우이므로 `getById`를 호출했을 때 `undefined`가 리턴됨을 알 수 있다.
    

<aside>
💡 **아래 다른 메서드들 관련 테스트들도 위 테스트와 같은 방식으로 이뤄졌다.**

</aside>