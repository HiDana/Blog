---
title: 用手機原生分享網頁 - navigator.share
date: 2021-02-28
tags: [web, social-media, social-share, fb, line, twitter]
layout: layouts/post.njk
---

在 228 連假，剛好有時間可以來實驗一直很想試試看的一個功能

“用手機原生系統做網頁分享”

![](/img/20210228/ios-share.png)

其實在手機上注意到這功能已經有一陣子了，因為非常貼合每個使用者的情境，而且可以很方便的就做分享，重點是 “不會跳出網頁” ！！讓使用者可以無負擔分享，並連續在網頁繼續瀏覽

比如說，過往分享是點擊後，會開起 line 或 fb app 再做分享，這時用戶如果發現其他人的訊息或是有趣的 po 文，這樣回到原本網頁的機率就變低了…間接的引響 SEO 跳出率…

總之這功能非常適合 blog 網站、產品網站、貼文網站…各種需要做分享，而且用戶大多是使用手機訪問的網站

---

但這分享功能一開始只開放給 app 呼叫，所以 web 最初是沒有這功能，不過從去年突然發現，誒？好像網頁也開始開放了！！就找了幾篇文章確認一下功能，打算找機會來試試看

# [Navigator.share](https://developer.mozilla.org/zh-CN/docs/Web/API/Navigator/share)

其實程式碼很簡單就是去呼叫 `navigator.share` 然後把你要帶的東西放進去就好了

```js
navigator.share({
  title: document.title,
  text: "Hello World",
  url: "https://developer.mozilla.org",
});
```

可以看到桌機其實沒有支援度很高，但是手機其實大多都有支援了～

但還是要記得先做 `if(navigator?.share)` 判斷再觸發 `navigator` 分享

[can i use navigator.share](https://caniuse.com/?search=navigator.share)
![](/img/20210228/caniuse_share.png)

不過這分享功能目前是使用手機原生的分享，所以其實在桌機上分享效果沒有這麼好

![](/img/20210228/desktop-safari.png)

用 safari 桌機點擊測試，發現跳出來的東西其實不怎麼吸引人後續做分享，而且跟我們社群分享的需求完全無關，所以在桌機還是用原本以前的作法來使用會比較直觀

![](/img/20210228/desktop-share.png)

以前社群分享做法，可參考 [social share buttons(fb、line、twitter)](<https://blog.hidana.me/social%20share%20buttons(fb%E3%80%81line%E3%80%81twitter)/>)

目前測試手機上(ios 14.4) 在 safari/chrome/firefox 都可以良好使用

![](/img/20210228/ios-share-safari.png)

---

> [野薑官網](https://gingerdesign.com.tw/blog/%E5%93%AA%E4%B8%80%E5%80%8B%E9%87%8E%E8%96%91%E6%9C%8D%E5%8B%99%E6%9C%80%E9%81%A9%E5%90%88%E6%88%91%EF%BC%9F) 也上此分享功能囉～歡迎大家來試用看看
