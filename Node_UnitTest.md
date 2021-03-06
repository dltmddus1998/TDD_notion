## ๐งช ๋ฐฑ์๋ ๊ฐ๋ฐ์์์ ํ์คํธ ์ฝ๋

---

> **๋ฐฑ์๋ ๊ฐ๋ฐ์์, ํตํฉ ํ์คํธ(Integration Test)๋ ROI(Retrun Of Investment)์ด๋ค.**
> 

**+) ROI: ํฌ์ํ ๊ฒ ๋๋น ๊ฐ์ฅ ๋์ ๊ฒฐ๊ณผ๊ฐ์ ๋๋นํ  ์ ์๋ค.**

> **๋ฐฑ์๋ ๊ฐ๋ฐ์์ ํตํฉ ํ์คํธ(Integration Test)๋ client๊ฐ ์ฌ์ฉํ๋ ๋ธ์ถ๋ WEB API๋ง์ ํ์คํธ ํ๋ ๊ฒ์ด๋ค.
ex) login API, tweet API**
> 

> **๋ด๋ถ์ ์ผ๋ก ๋ชจ๋  ๋น์ฆ๋์ค ๋ก์ง๊ณผ ๋ชจ๋๋ค์ด ์๋ก ์ํธ์์ฉ์ ์ํ์ฌ ๋์ํ๋์ง, ๋ฐ์ดํฐ๋ฒ ์ด์ค์ ์ ์ฝ๊ณ  ์จ์ง๋์ง ๋ฑ์ ๋ํ ํ๋ฉด์ ์ธ ๊ฒ์ ๋ํ ํ์ธ์ด ๊ฐ๋ฅํ๋ค.**
> 

## โ๏ธ ๋ธ๋ ํ์คํธ ํ๊ฒฝ ์์

---

### node-mocks-http

> **node express ํ๊ฒฝ์์ request/response๋ฅผ mockingํ๋๋ก ๋์์ฃผ๋ ๋ผ์ด๋ธ๋ฌ๋ฆฌ**
> 

1. ํฐ๋ฏธ๋์์ ๋ค์๊ณผ ๊ฐ์ด jest, node-mokes-http๋ฅผ ์ค์นํด์ค๋ค.
    
    `npm i --save-dev jest node-mocks-http`
    
2. jest๋ moduleํ์์ ์ ๊ณตํ์ง ์๊ธฐ ๋๋ฌธ์ commonjs๋ก ๋ณํํด์ฃผ๋ ๋ถ๋ถ์ด ํ์๋ก ํ๋ค.
    
    โ๏ธ `npm i --save-dev @babel/plugin-transform-modules-commonjs`
    
    โ๏ธ `.babelrc` ์ ๋ค์๊ณผ ๊ฐ์ด ์์ฑ
    
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
    
    โ๏ธ `jest --init`์ ํตํด `jest.config.mjs` ์์ฑ
    

## ๐ ํ์คํธ์ฉ ํ๊ฒฝ๋ณ์ ์ค์ ํ๊ธฐ

---

jest ๋์ ์ด์ ์ dotenv ๋์ํ๋๋ก jest.config.mjs์ ์ค์ 

```jsx
export default {
	.
	.
	setupFiles: ['dotenv/config'],
	.
	. 
}
```

ํ์คํธ ์คํ์ ํ๋ก์ ํธ ๊ฒฝ๋ก์ ์๋ .env.test ๋ฅผ ์ฌ์ฉํ๋๋ก ๋ช์. 

```json
// package.json
"scripts": {
  "test": "DOTENV_CONFIG_PATH=./.env.test jest --watchAll",
  "start": "nodemon app"
},
```

<br>
<br>

## Dwitter - ํ์คํธ ์ฝ๋ ์์ฑํ๊ธฐ

---

### Middleware Test Code

> **์ธ๋ถ ๋ผ์ด๋ธ๋ฌ๋ฆฌ, ๋ฐ์ดํฐ๋ฒ ์ด์ค์ ์ ๊ทผํ๋ ๊ฒ์ ํ์คํธ ์ฝ๋์ ์๋๋ฅผ ๋๋ ค์ง๊ฒ ํ๋ ์์ธ์ด ๋  ์ ์์ผ๋ฏ๋ก ์ด ๋ถ๋ถ๋ค์ ๋ชจ๋ mock ์ฒ๋ฆฌํด์ค๋ค.**
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
        // 401 error์์ ๋ค์ ๋ก์ง์ผ๋ก ๋์ด๊ฐ์ง ์๋๋ก!! -> next() x
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
๐ก ํ์ฅ์๊ฐ js, jsx์ธ ๋ชจ๋  ํ์ผ์ ๋ํด์ coverage๋ฅผ ํ์ธํ  ์ ์๊ฒ ํ๋ collectCoverageFrom ์ต์์ ์ค์ ํด์ค๋ค. 
๋จ, node_modules ์์ ์๋ ํ์ผ๋ค์ ๋ํด์  coverage๋ฅผ ๋ณด์ง ์๊ฒ ๋ฐ๋ก ์ค์ ํด์ค๋ค.

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

โญ๏ธ **๊ทธ ์ ์ ํธ์ ์ปจํธ๋กค๋ฌ ๋ฆฌํฉํ ๋งํ๊ธฐ!!**

**์๋ ์ฝ๋**

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

**๋ฆฌํฉํ ๋งํ ์ฝ๋**

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
// Refactoring: routing๋ถ๋ถ์ ํด๋์ค๋ก ๋ฌถ์ด์ฃผ์๋ค
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
// Refactoring: /tweets๊ฒฝ๋ก๋ฅผ router/tweets.js์ tweetsRouter๋ฅผ ํตํด ๊ฐ์ ธ์ค๊ฒ ํ๋ค
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

### ๐ ์ฌ๊ธฐ์ ์ ๊น!!

<aside>
๐ก ํธ์ ์ปจํธ๋กค๋ฌ์ ๋ฆฌํฉํ ๋ง์ ์งํํ ์ด์ ๋ ๋ฌด์์ผ๊น์?
</aside>

<br>

โ๏ธ ๋ชจ๋์์ ์ด๋ค ํน์  ๋ผ์ด๋ธ๋ฌ๋ฆฌ๋ฅผ ๋ฐ๋ก ๊ฐ์ ธ๋ค ์ฐ๊ณ , ๋ค๋ฅธ ๋ชจ๋์ ๋ด๋ถ์์ ๋ฐ๋ก ์์ฑํ๊ฑฐ๋ (โ **์๋ ์ฝ๋์ ๊ฒฝ์ฐ import๋ฅผ ํตํด ๋ฐ์ดํฐ ๋ฑ์ ๊ฐ์ ธ์๋ค**) ํธ์ถํ๋ฉด, **๋ชจ๋๊ฐ์ ๊ฒฐํฉ๋๊ฐ ๋์์ง๋ค**. [**ํ์ดํธํ๊ฒ ์ปคํ๋ง๋์๋ค**]

โ๏ธ ๋ผ์ด๋ธ๋ฌ๋ฆฌ๋ ์ธ๋ถ ๋ชจ๋์ ํจ์์ ์ฌ์ฉ๋ฒ์ด ๋ณ๊ฒฝ๋๋ ๊ฒฝ์ฐ, ๊ทธ๊ฑธ ์ฌ์ฉํ๋ ๋ชจ๋  ๊ณณ์์ ์ฝ๋ ์์ ์ด ์ผ์ด๋์ผ ํ๋ฏ๋ก **ํ์ดํธํ ์ปคํ๋ง**์ ์ง์ํด์ผํ๋ค.

์์์ ๋งํ ๋ถ๋ถ์ ์ง์ํ๋ ๋ฐฉ๋ฒ์ผ๋ก๋, ***Dependency Injection***์ด ์๋ค.

โ ํด๋น ํด๋์ค/๋ชจ๋ ์์์ ํ์ํ  ๋๋ง๋ค ์์ฑํ๊ฑฐ๋ ์ง์ ์ ์ผ๋ก ์ ๊ทผํ๋ ๊ฒ์ด ์๋๋ผ, **์ธ๋ถ ๋ผ์ด๋ธ๋ฌ๋ฆฌ/๋ชจ๋์ ํ๋จ๊ณ ๊ฐ์ธ๋ ํด๋์ค๋ฅผ ๋ง๋ค๊ณ , ๊ทธ ์ธ๋ถ ์์กด์ฑ์ ๋ฐ์์ ์ฃผ์ํด์ฃผ๋ ๊ฒ์ด๋ค.**

๐ ์ด๋ฅผ ํตํด, ์๋ฐ์ดํธ๊ฐ ํ์ํ ๊ฒฝ์ฐ, ๊ทธ ์ธ๋ถ ์์กด์ฑ ๋ด๋ถ๋ง ์๋ฐ์ดํธ ํด์ฃผ๋ฉด ๋๋ ๊ฒ์ด๋ค. [**๋์ปคํ๋ง**]

โญ๏ธ **์๋ฐ์คํฌ๋ฆฝํธ์์ mock์ผ๋ก ๊ฐํธํ๊ฒ ๋ฎ์ด์ฐ๊ธฐ๊ฐ ๋์ง๋ง, ์ด๋ฅผ ๋จ์ฉํ  ๊ฒฝ์ฐ(๋ชจ๋๊ฐ์ ๊ฒฐํฉ๋๊ฐ ๋์๋ฐ, ์ฝ๋ ๊ฐ์  ์์ด ๋์ถฉ ๋ฎ์ด ์ฐ๋ ๋ฑ...) ๋์ค์ ์ ์ง๋ณด์๊ฐ ํ๋ค์ด์ง๋ค. 
 ๋ฐ๋ผ์, ์ธ๋ถ ๋ผ์ด๋ธ๋ฌ๋ฆฌ๋ ์ธ๋ถ ๋ชจ๋ ์ ์ฒด๋ฅผ mockํ๋ ํ์คํธ ์ฝ๋๋ ์ง์ํด์ผํ๋ค.**

 โผ๏ธย **[์ถ์ฒ]by ellie of Dream Coding โผ๏ธ**

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

โญ๏ธ `tweetController`๋ *beforeEach*์์ ์ค์ ํด์ค๋ค.

โผ๏ธ **๊ฐ๊ฐ์ ํ์คํธ๋ค์ด ๊ฐ๋ณ์ ์ธ ํ๊ฒฝ์ ๊ฐ์ง๊ธฐ ์ํด์  ์ค๋ธ์ ํธ๋ค์ ๋ํด์  ์ด๊ธฐํํด์ฃผ๋ ๊ฒ์ด ์ค์ํ๋ค** โผ๏ธ

โ๏ธ Dependency Injection์ ์ฌ์ฉํ์ฌ ๋ชจ๋๊ฐ ๋์ปคํ๋ง์ ์์ผฐ๊ธฐ ๋๋ฌธ์, ๋ชจ๋ ์ ์ฒด๋ฅผ mockํ  ํ์๋ ์๋ค.

`getSocket`์ ํด๋นํ๋ ๋ถ๋ถ์ mockํ๋ค. 
โ *controller*์ `getSocket()`์ ํด๋นํ๋ ์ฝ๋์ `emit`์ด๋ผ๋ ๋ฉ์๋๊ฐ ์๋ค. ์ด๋ฅผ ํ์ฉํ์ฌ, `emit`์ *jest function*์ผ๋ก ์ ์ํ๋ค.

โย **์ฌ๋ฌ๊ฐ์ง ์ข๋ฅ๋ณ๋ก ๋ฌถ๊ธฐ ์ํด์ describe ์์์๋ describe๋ก ๊ฐ๊ฐ ๋ฌถ์ด์ค๋ค.**

โญ๏ธ **getTweets**

1. **username์ด ์๋ ๊ฒฝ์ฐ**
    
    โ๏ธ `tweetRepository.getAll = () โ allTweets;`
    โ **`tweetRepository`๋ผ๋ ๋น ์ค๋ธ์ ํธ์ `getAll`์ด๋ผ๋ ํจ์๋ฅผ ์ถ๊ฐํ๋ค.** `getAll`์ ํธ์ถํ์์ ๋ `allTweets`์ ๋ฆฌํดํ๋ค.
    
    โ๏ธ ๋ชจ๋  ๋ฐ์ดํฐ๊ฐ ์ค๋น๋์ ๋, `tweetController`์ ์๋ `getTweets`ํจ์๋ฅผ ํธ์ถํ์ฌ ์ค๋น๋ `request`์ `response`๋ฅผ ์ ๋ฌํด์ค๋ค.
    
2. **username์ด ์๋ ๊ฒฝ์ฐ**
    
    โ๏ธ request์ username์ ํฌํจํ query ํค๋ฅผ ์ถ๊ฐํ์ฌ์ค๋ค.
    
    โ๏ธ `tweetRepository.getAllByUsername = jest.fn(() => userTweets);`
    **โ `tweetRepository`๋ผ๋ ๋น ์ค๋ธ์ ํธ์** `getAllByUsername`์ด๋ผ๋ ํจ์๋ฅผ ์ถ๊ฐํ๋ค.
    
    โ๏ธ`expect(tweetRepository.getAllByUsername).toHaveBeenCalledWith(username);`
    **โ `getAllByUsername`์ด ํธ์ถ๋๋์ง ํ์ธํ๋ค.**
    

โญ๏ธ **getTweet**

โผ๏ธ **์์ ๊ฐ์ด ์ด๊ธฐํ ์ํ๋ฅผ ํ์๋ก ํ๋ฏ๋ก beforeEach๋ฅผ ์ด์ฉํ๋ค.**

1. **tweet์ด ์กด์ฌํ  ๊ฒฝ์ฐ** 
    
    โ๏ธ `aTweet`์ด๋ผ๋ ๋ณ์์ *faker*๋ฅผ ์ฌ์ฉํ์ฌ ๋๋คํ ๋ฌธ์์ด์ ๊ฐ์ผ๋ก ๋ `text`๋ผ๋ ํค๋ฅผ ๊ฐ๋ ์ค๋ธ์ ํธ๋ฅผ ํ ๋นํ๋ค.
    
    โ๏ธ `tweetsRepository.getById = jest.fn(() โ aTweet);`
    **โ getById๋ฅผ ํธ์ถํ์ ๋ aTweet์ ๋ฆฌํดํ๋ค.**
    
    `expect(tweetsRepository.getById).toHaveBeenCalledWith(tweetId);`
    **โ getById๋ฅผ ํธ์ถํ์ ๋ tweetId๊ฐ ๋ถ๋ฌ์์ง์ ์ ์ ์๋ค.**
    
2. tweet์ด ์์ ๊ฒฝ์ฐ
    
    โ๏ธ ์์๋ ๋ฐ๋๋ก `tweet`์ด ์์ ๊ฒฝ์ฐ์ด๋ฏ๋ก `getById`๋ฅผ ํธ์ถํ์ ๋ `undefined`๊ฐ ๋ฆฌํด๋จ์ ์ ์ ์๋ค.
    

<aside>
๐ก **์๋ ๋ค๋ฅธ ๋ฉ์๋๋ค ๊ด๋ จ ํ์คํธ๋ค๋ ์ ํ์คํธ์ ๊ฐ์ ๋ฐฉ์์ผ๋ก ์ด๋ค์ก๋ค.**

</aside>