---
title: React-router 在 github page 的問題
date: 2019-08-13
tags: [code, js, react, react-router, github, githubPage]
layout: layouts/post.njk
---

最近想把 react 網頁部署在 github page 上，並使用 custom domain，卻遇到了在子頁面重新 refresh 會有 404 的問題

![](/img/20190813/1.png)

在這邊先了解一下 react-router 的運作方式

```text
訪問 test.hidana.me → 跟 server 要東西 → 獲得 index.html → 點擊某個換頁按鈕 -> 運行 js 做頁面轉換(CSR) → 換置頁面到 test.hidana.me/secFloor
```

我們頁面換置到 test.hidana.me/secFloor 是在 client-side 自己使用的，沒有刷新頁面，因此完全沒有訪問到 server

但當我們在 `test.hidana.me/secFloor` refresh 或是把這網址另外開始窗訪問，但 server 完全不認識這東西，因此這網頁沒有 react-router 的 js ，所以就會是 404

網路上有很多解法，有 client 端解，也有從 server 端解，這邊主要提如何用前端簡單的處理這問題

## 方法一，用 Hash History

當訪問時 讓網址呈現 test.hidana.me/#/secFloor 這樣在跟 server 請求時，就只會用 test.hidana.me 請求，而 react-router 處理剩下的 #/secFloor

缺點

- 網址很醜
- 就 Search Engine Optimization(SEO)而言，網站是由一個頁面組成，幾乎沒有任何內容。

## 方法二，設定 basename

> 此方法參考網路上文章，並未實際測試過

在 react-router 的 BrowserRouter 加上 basename 就可以解決這問題，但 process.env 裡面是空的

```js
export default function Routes() {
  const store = configureStore({ history });
  return (
    <Provider store={store}>
      <BrowserRouter history={history} basename={process.env.PUBLIC_URL}>
        <Switch>
          <Route exact path="/" component={App} />
          <Route path="/todo" component={Todo} />
          <Route component={() => <div>404 Not found </div>} />
        </Switch>
      </BrowserRouter>
    </Provider>
  );
}
```

因為使用 create-react-app 做靜態專案，所以會有類似問題

這邊就可以在 build 時把 PUBLIC_URL 綁進去

```json
"build": "set PUBLIC_URL=<url> && node scripts/build.js",
```

## 方法三，用 index.html 取代 404.html

![](/img/20190813/2.png)

[Using create-react-app on Netlify](https://hugogiraudel.com/2017/05/13/using-create-react-app-on-netlify/)

```json
//package.json
{
  "build": "react-scripts build && cp build/index.html build/404.html"
}
```

把 index.html 覆蓋 404.html，讓 404.html 偽裝成 index.html 裡面涵蓋著被 hash 過的 js(e.g. main.8626537e.js).

我想的大概流程應該是這樣

原本有問題的網頁，如果從根目錄開始點擊，是可以完整呈現的，流程如下

```text
訪問 test.hidana.me → 點擊某個按鈕到下一層 → 目前網頁在 test.hidana.me/secFloor
```

但如果 refresh 後，就會導向 404

```text
訪問 test.hidana.me → 點擊某個按鈕到下一層 → 目前網頁在 test.hidana.me/secFloor → 在當前網頁 refresh → 導向 404 告訴你找不到
```

所以這邊用 index.html 複製過去到 404 讓他模擬，好像是從 index 來的方式，並成功得到 hash 過的 js 檔案可以用

```text
訪問 test.hidana.me → 點擊某個按鈕到下一層 → 目前網頁在 test.hidana.me/secFloor → 在當前網頁 refresh → 導向 404 告訴你找不到(但模擬成 index.html) -> 成功取得 test.hidana.me/secFloor
```

> 這次遇到的問題，成功用 index.html 複製 404.html 解決掉，react 真的是到處都是好玩的坑啊！

---

參考文章

[Deploying a create-react-app with routing to GitHub pages](https://levelup.gitconnected.com/deploying-a-create-react-app-with-routing-to-github-pages-f386b6ce84c2)

[React-router problem with gh-pages](https://medium.com/@Dragonza/react-router-problem-with-gh-pages-c93a5e243819)

[BrowserRouter](https://reacttraining.com/react-router/web/api/BrowserRouter)

[Gh-pages deployment problems with react-router](https://github.com/facebook/create-react-app/issues/1765)

[Using create-react-app on Netlify](https://hugogiraudel.com/2017/05/13/using-create-react-app-on-netlify/)

[React-router urls don't work when refreshing or writing manually](https://stackoverflow.com/questions/27928372/react-router-urls-dont-work-when-refreshing-or-writing-manually)

[Fixing the "cannot GET /URL" error on refresh with React Router and Reach Router (or how client side routers work)](https://tylermcginnis.com/react-router-cannot-get-url-refresh/)

[解決 React/React route 部屬在 GitHub Pages 上遇到的問題](https://thecodingday.blogspot.com/2018/12/reactreact-routegithub-pages.html)
