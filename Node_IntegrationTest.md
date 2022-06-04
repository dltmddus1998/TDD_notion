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