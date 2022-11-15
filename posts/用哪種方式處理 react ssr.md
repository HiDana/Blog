---
title: 用哪種方式處理 react ssr
date: 2020-11-06
tags: [react, ssr, react-snap, prerender]
layout: layouts/post.njk
---

因為 seo 最終要來面對處理 react 的 ssr 了啊！！！

react 令人讚嘆以及詬病的就是他的 render 方式，渲染速度快但 seo 無法好好呈現

有開發過 react 的，一定對這程式碼不陌生

```html
<!DOCTYPE html>
<html lang="zh-TW">
  <head>
    <meta charset="utf-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1" />
    <!-- <link rel="icon" href="%PUBLIC_URL%/favicon.ico" /> -->
    <link rel="manifest" href="%PUBLIC_URL%/manifest.json" />
  </head>

  <body>
    <div id="root"></div>
  </body>
</html>
```

所以其實 html 真的實際只有這樣，剩下的畫面資料等，就全部都丟給 js 來處理

你一定覺得，“嗯，然後呢？為啥這跟 seo 有關？”

# google 爬蟲是怎麼撈我們網頁

我們先來看看 google 爬蟲是怎麼撈我們網頁的

![](/img/20201106/crawler.png)
this image is from [Server side rendering and dynamic rendering with Headless chrome](https://medium.com/@shotap/server-side-rendering-and-dynamic-rendering-with-headless-chrome-f23cdabfae48)

google 爬蟲大致會撈網站兩次

```text
- 第一次

googlebot 會先看整個網站內容，以及網頁的 title、meta、canonical、語意標記等

- 第二次

googlebot 會在很久很久很久很久很久，才會心血來潮執行一下網頁的 js 然看看是不是跟第一次不一樣
```

這邊就會發現，如果網站大多都是依照 js 去做渲染的話，會等 google 爬蟲心血來朝時回頭再來看看你，而這時間通常距離第一次訪問，間隔幾天至數週都有可能，然後才能找得到正確的你

好呦，所以 react 用 js 處理大多事情，真的對於 google 爬蟲是不友善的

難怪我想說，網站上去後 google 檢索看起來大多都正常啊，怎麼會對 seo 有影響，但其實是我們看不到的爬蟲成本，也算在 seo 裡面

---

# 你要用哪種 SSR ?

react 算是一個龐大的社群，所以其實蠻多方式去做 SSR 這件事，這邊主要介紹三種常見的方式

## step by step，自己手刻

如果是自己一步一步架 ssr 的 server 話，在處理 react 的 ssr 主要注意幾件事

1. 頁面的 ssr 渲染
2. react-router 轉換後的頁面炫懶
3. api request
4. 在 server render 可以使用 redux store

其實很推薦看 udemy 超強老師 Stephen Grider 的 [Server Side Rendering with React and Redux](https://www.udemy.com/course/server-side-rendering-with-react-and-redux/) 非常詳細的教建立 react ssr 的方式

## [react-snap](https://github.com/stereobooster/react-snap)

原先看到這方式以為 ssr 只要安裝個套件就可以解決了，心想 “哪泥？！！ react 轉 ssr 就這麼簡單嗎？那我當初花這麼多時間學 nextjs…”，後來仔細往下看後才發現，其實 react-snap 是給`少數頁面的靜態頁面`去做 ssr 處理，原理大致是在 build 的時候把頁面也 build html 出來，不是像以往 react 只產出一個 index.html。所以當頁面一多，比如說有一千多筆產品頁，這樣其實也不適合用 react-snap。

這邊也有一個類似的 [react-screenshots](https://www.npmjs.com/package/react-screenshots) ，但上次維護已經是很久以前，所以同類型還是推薦 [react-snap](https://github.com/stereobooster/react-snap) 拉

## prerender

prerender 提供兩個方式給使用者

### [prerender.io](https://prerender.io)

頁數跟訪問量不多的話，可以使用 prerender 的服務 prerender.io，因為免費額度有限制，所以如果不自己架依然用他們的服務的話，每個繳保護費也是一個方式

### 自己架 [prerender middle](https://github.com/prerender/prerender#-1)

nginx 原先導向 server 是 prerender.io，所以自己架的話，再把 server 位置導自己架的位置就好了

看了這些，應該對於 ssr 有一定基礎想法，但還是會疑惑哪一種會比較適合自己目前專案使用，如果是這樣的話可以參考這張圖

![](/img/20201106/which-ssr.png)

像這次把專案轉型，因為本身網站已經存在，也在遠端運行，更需要在 ssr 時就 request api。再加上本身網站開發時已經買蔥送和牛了，考量人力與成本，最後決定使用 prerender。

目前 prerender 專案準備上線中，預計 seo 影響或許要等兩到三個月，如果未來有真的有顯著的影響，我想那時或許也可以再來寫一篇做了哪些事情的 seo 優化

---

[reference]

[dynamic-rendering-seo-details-need-know](https://ignitevisibility.com/dynamic-rendering-seo-details-need-know/)

[Server side rendering and dynamic rendering with Headless chrome](https://medium.com/@shotap/server-side-rendering-and-dynamic-rendering-with-headless-chrome-f23cdabfae48)

[深入但不淺出，React,Vue SEO 最佳化](https://medium.com/@milkmidi/深入但不淺出-react-vue-seo-最佳化-f146a65886a6)

---

> react 轉換 ssr 終於到測試架構這步了 （（拭汗
