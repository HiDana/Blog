---
title: CRA 多環境建置
date: 2021-06-21
tags: [react, CRA, create-react-app, env]
layout: layouts/post.njk
---

在 create-react-app 的官方文件 [Adding Custom Environment Variables](https://create-react-app.dev/docs/adding-custom-environment-variables/) 裡面有很詳細的寫怎麼建立開發 development 跟 production

---

# 在 CRA 建立不同環境

## CRA 目前環境

其實 CRA 本身 react-scripts@0.2.3 以上就有基本環境了
在開發 `npm start` 後，在元件裡下 `process.env` 的 log 就可以看到目前環境變數

![](/img/20210621/origin-env.png)

如果是跑 `npm run build` 後，可以看到 NODE_ENV 就會是 “production”

## 新增變數到 CRA 環境

在根目錄新增 `.env.development` ，然後客製化的變數要在裡面填寫 `REACT_APP_` 開頭的變數定義(皆大寫)

比如說

```text
REACT_APP_API_URL=https://aaa.com
```

![](/img/20210621/react-env_development.png)

可以看到我們剛剛定義的 REACT_APP_API_URL 可以在 `process.env` 裡使用
`process.env.REACT_APP_API_URL` 就可以拿到我們的 url

## HTML 裡引用變量

react-scripts@0.9.0 以上可以直接在 public/index.html 使用
其實到 public/index.html 裡面就可以看到這段

```js
<link rel="icon" href="%PUBLIC_URL%/favicon.ico" />
```

![](/img/20210621/html-useenv.png)

依照目前我們環境變數來定義，所以 PUBLIC_URL 是空字串，就是在本目錄下的 `favicon.ico`

## 那些 .env 可以使用

- `.env`: Default.
- `.env.local`: Local overrides. This file is loaded for all environments except test.
- `.env.development`, `.env.test`, `.env.production`: Environment-specific settings.
- `.env.development.local`, `.env.test.local`, `.env.production.local`: Local overrides of environment-specific settings.

Files on the left have more priority than files on the right:

- `npm start`: `.env.development.local`, `.env.local`, `.env.development`, `.env`
- `npm run build`: `.env.production.local`, `.env.local`, `.env.production`, `.env`
- `npm test`: `.env.test.local`, `.env.test`, `.env` (note `.env.local` is missing)

所以可以看到如果在根目錄新增 .env 就會是全部環境都可以吃到裡面的變數

以上就是官方文件大多就可以知道的內容
但有時候會需要建立不同階段的 production ，比如說 staging → uat → prod
所以接下來要用其他方式來完成我們可以在不同 production 的環境

---

# 在 production 建立不同環境

## 用 sh 處理

比如說要在 staging 環境建立 production
先在根目錄建立 `.env.staging`
在裡面新增我們要在 staging 裡的變數

```text
//.env.staging
REACT_APP_API_URL=https://staging.com
```

在 package.json 重新定義 build

```json
//package.json
    "scripts": {
    ...
    "build": "sh -ac '. .env.staging; react-scripts build'"
    },
```

[Package.json complicated start script with “sh -ac” and .env file for Firebase](https://stackoverflow.com/questions/51123507/package-json-complicated-start-script-with-sh-ac-and-env-file-for-firebase)
這篇有解釋 sh -ac 的意思

```text
Running sh -ac is saying "run the shell command and automatically export all variables assigned during its run" effectively.
```

就可以直接 build ，來看看我們設定的環境會不會一起帶進去

```bash
$npm run build
```

這邊就可以看到 build 出來的東西，藉由 react 建議提供的 serve 起起來

```bash
$serve -s build
```

可以看到我們下 process.env 有吃到我們定義的 API_URL

![](/img/20210621/sh-env.png)

也可以用變數來處理

```json
"scripts": {
    ...
    "build": "sh -ac '. .env.${REACT_APP_ENV}; react-scripts build'",
    "build:stg": "REACT_APP_ENV=staging npm run build"
},
```

![](/img/20210621/sh-env-variable.png)

所以可以看到環境變出會多一個我們設定 `REACT_APP_ENV`

## 用 [env-cmd](https://github.com/toddbluhm/env-cmd) 來處理環境

```bash
$npm i -D env-cmd
```

```json
//package.json
"scripts": {
    ...
    "build:stg": "env-cmd -f .env.staging react-scripts build"
    }
```

![](/img/20210621/env-cmd.png)

結果也會跟下 sh 是一樣的

也可以新增 .env-cmdrc 把所有 env 設定都放在一起

```json
//.env-cmdrc
{
  "stg": {
    "REACT_APP_API_URL": "https://staging.com"
  },
  "prod": {
    "REACT_APP_API_URL": "https://prod.com"
  }
}
```

在 script 指令這樣下

```json
    "scripts": {
      ...
     "build:stg": "env-cmd -e stg react-scripts build",
     "build:prod": "env-cmd -e prod react-scripts build",
    },
```

也可以在在開發時，或是 build 時候作多環境的變數處理呦

---

[reference]

[Adding Custom Environment Variables](https://create-react-app.dev/docs/adding-custom-environment-variables/)

[Create-react-app environments](https://medium.com/@tacomanator/environments-with-create-react-app-7b645312c09d)

[How to set build .env variables when running create-react-app build script?](https://stackoverflow.com/questions/42458434/how-to-set-build-env-variables-when-running-create-react-app-build-script)

[Various ways of handling environment variables in React and Node.js](https://javascript.plainenglish.io/various-ways-of-handling-environment-variables-in-react-and-node-js-5b9ce13aa7b1)

[Managing environment variables in Nodejs and Modern JS apps](https://medium.com/dubizzletechblog/managing-environment-variables-in-nodejs-and-modern-js-apps-608003f4686c)

---

> 赫然發現之前用 config 的方式，真的算是簡易的環境處理 😂
