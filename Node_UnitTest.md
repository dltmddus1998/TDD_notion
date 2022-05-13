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
