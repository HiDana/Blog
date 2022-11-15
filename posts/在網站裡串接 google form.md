---
title: 在網站裡串接 google form
date: 2020-07-02
tags: [google, form, react]
layout: layouts/post.njk
---

要把 google form 嵌入到網站有好幾種方式

一種是用 iframe 嵌入進去
點擊點擊左上方傳送，並選擇嵌入的 html，直接把這 iframe 貼進自己的 code 就好

![](/img/20200702/1.png)

這種做法好處是很快速，壞處就是他只是把 google 表單整個整個嵌進去我們自己的網站，就沒辦法跟網站本身的 style 很好的融入

所以我們這邊主要說另外一種簡單的做法

## 用自己的 form 串接 google 表單

我們主要要拿到兩種值

- google 表單的 post url
- input 元件自己的 name

```html
<form
  action="https://docs.google.com/forms/u/0/d/e/aaa/formResponse"
  method="post"
>
  <label>name *</label>
  <input type="text" placeholder="name" name="entry.1851193727" required />

  <label>email*</label>
  <input
    type="email"
    placeholder="email address"
    name="entry.123456789"
    required
  />
  <button type="submit">寄送</button>
</form>
```

開啟對外的表單，就是要給別人填寫的那個網址，然後找到 form 的 tag ，裡面 action 就是我們要得值，大概是長成這個樣子

`https://docs.google.com/forms/u/0/d/e/<google 表單的id>/formResponse`

![](/img/20200702/2.png)

再來是 每個 name 的 entry，之前舊版的 google 表單 entry 值比較好找，直接選 input 的欄位就很明顯的叫做 entry.xxx 新版的就比較複雜

![](/img/20200702/3.png)

位置會在該框框的 data-params 裡，裡面大概就是組成這 block 的資訊，input 的 name 會在 title 後面的第一個常數 id ，大約就是紅匡的位置的 id

ok，到這裡點擊寄送按鈕，理論上可以在 google 表單的回覆得到我們填寫的值了

## 進階，串接自製的完成頁面

上面做到這邊以功能面是滿足了，但是有一個缺點，送出完成會後被自動跳轉到 google 的 thankyou page，這樣 seo 就會跟你說用戶在填寫表單這頁，跳轉率超高之類的

這邊有幾個做法可以解決不要跳轉到 google 的感謝頁面，而是我們自製的感謝頁面

1. 用 api 處理

做法大概就是針對 url post form data，然後去偵測 response 是 success 還是 error

為了延續我們上面原本用 form submit 的做法，這邊就不多說第一種做法

2. 創一個假的 iframe，讓 form target 指向該 iframe

在這邊記得要把 iframe 加上 style `display:none;` 把 iframe 隱藏

```html
<iframe
    name="hiddenFrame-wish"
    width="0"
    height="0"
    onLoad={() => setSuccess()}
  ></iframe>
  <form
  action="https://docs.google.com/forms/u/0/d/e/aaa/formResponse"
  method="post"
  target="hiddenFrame-wish"
>
  <label>name *</label>
  <input type="text" placeholder="name" name="entry.1851193727" required />

  <label>email*</label>
  <input
    type="email"
    placeholder="email address"
    name="entry.123456789"
    required
  />
  <button type="submit">寄送</button>
</form>
```

然後自己網站切換成成功頁面的觸發放在 iframe 的 onLoad 上，就可以不跳轉的完成表單傳送
