---
title: react code-split
date: 2020-10-23
tags: [react, code-split, react-loadable, react.lazy, lazy-load]
layout: layouts/post.njk
---

在 google 跑分時總是會出現，xxx.chunk.js 太肥囉，要該減肥一下呦，順便去除沒有用到的東西

然後基於你是 react 框架，如果不使用 SSR 推薦你使用 `React.lazy()` 或是處理 code-split 可以使用 `loadable-components`

![google-shootcut](/img/20201023/google-suggest.png)

好，所以我們的目標是把大大的 `main.xxx.chunk.js` code-split 拆分掉

---

## react bundle analyzer

### source-map-explorer

在處理 bundle 之前，先安裝可以看清楚 bundle 畫面的 package
create-react-app 官方([Analyzing the Bundle Size](https://create-react-app.dev/docs/analyzing-the-bundle-size/))是用 [source-map-explorer](https://www.npmjs.com/package/source-map-explorer)

在外面看到大部分人都用 [webpack-bundle-analyzer](https://github.com/webpack-contrib/webpack-bundle-analyzer) 但這篇文章([Revisit the decision to allow webpack analyzer tools](https://github.com/facebook/create-react-app/issues/4563))似乎是有說他有標籤標記錯誤的問題，但是裡面也有很多人舉例說為啥他會愛 `webpack-bundle-analyzer` 勝過於 source-map-explorer 的原因 😂

```bash
$npm i -D source-map-explorer
```

在 `package.json` 新增一條 script

```json
//package.json
"scripts": {
  "analyze": "source-map-explorer 'build/static/js/*.js'",
}
```

在執行 build 跟 analyze

```bash
$npm run build
$npm run analyze
```

也可以改寫成這樣

```json
"analyze": "npm run build && npm run analyze-bundle",
"analyze-bundle": "source-map-explorer 'build/static/js/*.js'"
```

這樣只要跑一行指令就好了

```bash
$npm run analyze
```

![source-map-explorer](/img/20201023/source-map-explorer-1.png)

### webpack-bundle-analyzer

~~不 eject CRA 狀態下用 [webpack-bundle-analyzer](https://github.com/webpack-contrib/webpack-bundle-analyzer)~~

```bash
$npm i -D webpack-bundle-analyzer
```

```json
//package.json
{
  "scripts": {
    "start": "react-scripts start",
    "build": "react-scripts build --stats && webpack-bundle-analyzer build/bundle-stats.json -m static -r build/bundle-stats.html -O",
    "test": "react-scripts test"
  }
}
```

測試了很久都沒有成功，後來找到文件說

```text
Using webpack-bundle-analyzer has been deprecated an the flag `—-stats` has been removed in CRA v3. You should be using source-map-explorer instead by running: `npm i -g source-map-explorer`and `source-map-explorer 'build/static/js/*.js'`.
The following paragraph is deprecated:
```

原本可以用 `--stats` 來生出 `bundle-stats.json` 但看起來 CRA 就是要推廣 `source-map-explorer` 整個拿掉沒辦法在不 eject 狀態下用惹ＱＱ

---

## react-loadable

react 在做 code-split 也可以針對 react-router 做處理

react-router 官方文件是推薦使用 `loadable-component` 但看仿間大多人都使用 `react-loadable`

```bash
$npm i react-loadable
```

```js
import Loadable from "react-loadable";
import Loading from "./my-loading-component";

const LoadableComponent = Loadable({
  loader: () => import("./my-component"),
  loading: Loading,
});

export default class App extends React.Component {
  render() {
    return <LoadableComponent />;
  }
}
```

### Dynamic imports 寫法

這邊預設 function 是 export default ，但我通常都只有 export const 所以這邊 import 要改寫成

```js
const LoadableComponent = Loadable({
  loader: () => import("./my-component").then(({ Component }) => Component),
  loading: Loading,
});
```

但測試後發現，怎麼 chunck 沒有分開？！！原因是因為我都在同一個 index.js 通通 export 所以他一樣會包在一起

發現這篇文章也有一樣的問題
[Code splitting with create-react-app and react-loadable is not working](https://stackoverflow.com/questions/51152419/code-splitting-with-create-react-app-and-react-loadable-is-not-working)

![pages-index](/img/20201023/pages-index.png)

因為在該 `index.js` 頁面就已經全部都合在一起了，所以要把這邊 `index.js` mark 掉，然後 import 分開寫，如果 `index.js` 沒有 mark 掉，即使引入分開寫，也一樣會包在一起不會分開歐

### named chunks.js

如果想要 name chunks.js 檔案的話，可以這樣做設定

```js
const Policies = Loadable({
  loading: PageLoading,
  loader: () =>
    import(
      /* webpackChunkName: "policies" */
      "pages/Policies"
    ).then((module) => module.Policies),
});
```

原本是 2.chunk.js 就會命名稱 policies.chunk.js

![no-webpackChunkName](/img/20201023/webpackChunkName-1.png)
![use-webpackChunkName](/img/20201023/webpackChunkName-2.png)

create-react-app 的 webpack 有設定這功能，如果沒有反應的話要去 wepback 的 config 檔，新增下面的設定

```json
output: {
  path: path.join(__dirname, './../public'),
  filename: 'bundle.js',
  publicPath: '/',
  chunkFilename: '[name].[chunkhash].js'
},
```

### result

原本 main chunk 427.04 kb 占 38%

code-split 後 342.34kb 占 27.3%

origin
![origin](/img/20201023/origin.png)

after
![after code-split](/img/20201023/after.png)

---

## React.lazy

[react Code-Splitting](https://zh-hant.reactjs.org/docs/code-splitting.html)

如果只用 react 原生的 lazy 也可以達到 code-split 的效果

```js
import React, { Suspense, lazy } from "react";
import { BrowserRouter as Router, Switch, Route, Link } from "react-router-dom";
import { Home } from "./components/Home";

export const Routes = () => {
  return (
    <Router>
      <Suspense fallback={<div>Loading...</div>}>
        <Switch>
          <Route
            path="/about"
            component={lazy(() =>
              import(/* webpackChunkName: "about" */ "./components/About")
            )}
          />
          <Route path="/" component={Home} />
        </Switch>
      </Suspense>
    </Router>
  );
};
```

### trouble shooting

用 react lazy 的話，他只接受 `const MyComponent = lazy(() => import('./MyComponent'))` 的格式

![react.lazy-err](/img/20201023/react-lazy-err.png)

所以要 export default

❌

```js
export const About = () => {
  return (
    <div>
      About<Link to="/">Home</Link>
    </div>
  );
};
```

⭕️

```js
let About;
export default About = () => {
  return (
    <div>
      About<Link to="/">Home</Link>
    </div>
  );
};
```

---

[reference]

- bundle

  [Hidden Features of `create-react-app`](https://medium.jonasbandi.net/hidden-features-of-create-react-app-52db3a17acc0)

  [--stats flag is gone in v3?](https://github.com/facebook/create-react-app/issues/6904)

  [Solution: webpack-bundle-analyzer without ejecting.](https://github.com/facebook/create-react-app/issues/3518)

- loadable

  [React Loadable — SSR and Code-Splitting](https://medium.com/@slashtu/react-loadable-ssr-and-code-splitting-ede5b31baf35)

  [使用 React-loadable 在 React 應用中實現 Code Splitting](https://kknews.cc/zh-tw/code/2klp2pe.html)

  [Code Splitting in React using React Loadable | Load code On demand](https://medium.com/@imranhsayed/code-splitting-in-react-using-react-loadable-load-code-on-demand-1726c5192387)

  [React 中使用 Dynamic Import](https://medium.com/itsoktomakemistakes/react-中使用-dynamic-import-3bfed937669e)

- dynamic import

  [Dynamic imports](https://javascript.info/modules-dynamic-imports)

  [ES6: Conditional & Dynamic Import Statements](https://stackoverflow.com/questions/35914712/es6-conditional-dynamic-import-statements)

  [react-loadable import from multiple class exported js](https://stackoverflow.com/questions/50137801/react-loadable-import-from-multiple-class-exported-js)

- webpackChunkName

  [Name webpack chunks from react-loadable](https://stackoverflow.com/questions/46584486/name-webpack-chunks-from-react-loadable)

---

> 把已開發好的專案新增 code-split 真的比一開始開發就加上去來的還有跳戰性啊～！！
