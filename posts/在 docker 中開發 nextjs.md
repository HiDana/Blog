---
title: 在 docker 中開發 nextjs
date: 2020-03-10
tags: [code, js, nextjs, docker]
layout: layouts/post.njk
---

在部署時，有時就會整合後發現根本地開發完全不一樣的情況

就被後端面帶微笑地提醒，是不是以後開發前端時就要在 docker 裡呢？

無奈我就是拖延症加上怕麻煩患者，直到現在才好好把它研究起來

之前有稍微測試跟 react 的整合，可能是當初是菜逼八，怎麼測就是沒辦法即時 rerender

每改一次 code 就要手動 refresh 頁面？！！又菜又廢的我就棄坑了

但最後自己挖的坑還是要自己跳，所以這次依舊撿回來研究如何跟 nextjs 整合

所以這次測試主功能是

- 改 code 後會自動重新渲染

```text
//file structure

    ├── frontend
        ├── ...
        ├── dockerfile
        └── next.config.js
    └── docker-compose.yml
```

`next.config.js`

主要是這邊去自動 check 然後觸發渲染

```js
next.config.js:
    module.exports = {
      webpackDevMiddleware: (config) => {
        // Solve compiling problem via vagrant
        config.watchOptions = {
          poll: 1000,   // Check for changes every second
          aggregateTimeout: 300,   // delay before rebuilding
        };
        return config;
      }
    };
```

`dockerfile`

```yml
FROM node

WORKDIR /usr/src/app

COPY . .

RUN npm install

CMD [ "npm", "run", "dev" ]
```

`docker-compose.yml`

```yml
version: "3"

services:
  frontend:
    build:
      context: ./frontend
      dockerfile: dockerfile
    volumes:
      - ./frontend:/usr/src/app
      - /usr/src/app/node_modules
    ports:
      - 3000:3000
```

`command`

```bash
$ docker-compose up -d
```

就可以跟開發一樣在 3000 port 看到囉！

---

研究後發現其實沒有想像中的這麼困難

大概就是當初菜逼八心態特容易放棄

> 人生就是不停的挖坑棄坑再填坑

---

[reference]

[Nextjs not recompiling with Docker in development](https://github.com/zeit/next.js/issues/6417)
[Next.js + Docker. Made easy.](https://dev.to/kumar_abhirup/next-js-docker-made-easy-2bok)
