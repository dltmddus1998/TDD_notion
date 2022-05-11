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