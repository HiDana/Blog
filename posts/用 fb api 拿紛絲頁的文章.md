---
title: 用 fb api 拿紛絲頁的文章
date: 2020-11-20
tags: [fb, qraph-api, js]
layout: layouts/post.njk
---

野薑官網想從 fb 粉絲頁撈文章，呈現在官網上，如下圖

![gingerdesign-fb-news](/img/20201120/gingerdesign-fb-news.png)

~~說是因為寫 blog 文章好累，所以要用 fb 的 po 文來產生檢索量~~

fb 提供一個叫 graph api (圖形化介面) 的端口，可以搭配每個粉絲團或是個人為節點的 access-token 來拿資料

這樣也是對的拉，畢竟 fb 當初被人詬病就是隱私不夠，如果用爬蟲去模擬爬也是要先登入自己的帳戶才能爬

所以使用 graph api 拿到的資料前提是 **跟你有關**

---

# graph-api 起手式

先至 [fb developer / My app](https://developers.facebook.com/apps/) 新增一個 app，因為我們這邊跟粉絲頁有關，所以會跟自己的粉絲串

因為之前在做 fb social 時需要 appId 所以已經先申請好了

![gingerdesign-myapp](/img/20201120/gingerdesign-myapp.png)

然後就可以去 fb 提供的 [graph-api explorer](https://developers.facebook.com/tools/explorer) 去測試我們要的 api 格式

[graph-api explorer 使用教學文件](https://developers.facebook.com/docs/graph-api/explorer/)

![graph-api_explorer](/img/20201120/graph-api_explorer.png)

因為沒有 token 所以一個 api 都沒辦法 submit 取得

選擇要測試的 facebook app，理應就是剛剛在 My app 創立的 app，或是原本已經存在的 app
因為我是粉絲頁，所以選擇 User or Page → Get Page Accsee Token

![get-token](/img/20201120/get-token.png)

然後就有一連串的認證，最後會回來這頁面，就可以看到 access token 已經有了

## 測試 graph-api_explorer

簡單用一個測試

```bash
GET /me
```

![gingerdesign_me](/img/20201120/gingerdesign_me.png)

可以看到剛剛 user or page 如果這邊是選 user token 的話，response 就會是自己不會是粉絲頁

![dana_me](/img/20201120/dana_me.png)

要找粉絲頁的文章，所以就在 me 的後面找，他其實會有下拉選單建議選項，可以直接點選就好

![choose-posts](/img/20201120/choose-posts.png)

![choose-posts_result](/img/20201120/choose-posts_result.png)

好呦，這邊是在 Graph API Explorer 測試成功的，那如果是在 web 接 api 要怎麼寫呢？

```bash
GET https://graph.facebook.com/me/published_posts?access_token=[your_access_token]
```

嗯嗯嗯，用 postman 也有測試成功呢

![postman-test](/img/20201120/postman-test.png)

把 me 改成剛剛拿到的 id 也是可以的

```bash
GET https://graph.facebook.com/[page-id]/published_posts?access_token=[your_access_token]

GET https://graph.facebook.com/2068xxxxxxxxx/published_posts?access_token=[your_access_token]
```

![postman-test_withId](/img/20201120/postman-test_withId.png)

好拉，基礎使用 qraph-api 應該有初步概念了

---

# qraph-api

因為野薑這次主要是想拿 po 過的文章，所以這次重點著重在 [page post](https://developers.facebook.com/docs/graph-api/reference/page-post/)

## Page Post

### field

原先拿到的 data 其實非常簡單，但如果還想要其他欄位怎麼辦？
其實可以在 field 定義

```bash
GET https://graph.facebook.com/me/published_posts?fields=id,created_time,message,permalink_url,shares＆access_token=[your_access_token]
```

![api-fileds](/img/20201120/api-fileds.png)

嗯嗯，看起來吐出來的 data 是我們想要的呢

其中 shares 的 key 會是有分享才會出現

![api-fileds_results](/img/20201120/api-fileds_results.png)

### likes & comments

那如果想拿到該文章的 like 跟 comment 數呢？

[Page Post Likes](https://developers.facebook.com/docs/graph-api/reference/page-post/likes/)

[Page Post Comments](https://developers.facebook.com/docs/graph-api/reference/page-post/comments/)

```bash
GET https://graph.facebook.com/me/published_posts?fields=likes.summary(true),comments.summary(true)＆access_token=[your_access_token]
```

因為 likes 裡面的 data 會一個一個列出來，因為我們只需要總數，所以也可以加上 `.limit(0)` 讓 data 回傳是空陣列

```bash
GET https://graph.facebook.com/me/published_posts?fields=likes.limit(0).summary(true),comments.limit(0).summary(true)＆access_token=[your_access_token]
```

![api-likes&comments](/img/20201120/api-likes&comments.png)

最後是如果文章有照片，要怎麼拿到呢？
full_picture (string)
Full size picture from attachment

```bash
GET https://graph.facebook.com/me/published_posts?fields=full_picture＆access_token=[your_access_token]
```

ok 會拿到的是一張照片的 link

![api-fullPicture](/img/20201120/api-fullPicture.png)

![api-fullPicture_result](/img/20201120/api-fullPicture_result.png)

照片在 edges 上還有一個參數 picture
picture (string)
The picture scraped from any link included with the post.

```bash
GET https://graph.facebook.com/me/published_posts?fields=picture＆access_token=[your_access_token]
```

回傳一樣是 link ，但就是拿到比較小的照片，看起來很似合作預先載入？！

![](/img/20201120/api-picture.png)

---

# 申請永久的 access token

在測試時，可以看到其實 token 也有期效性

## 申請 long-lived access token

點擊 open in access token tool

![extend-access-token_1](/img/20201120/extend-access-token_1.png)

會到 Access Token Debugger 頁面

點擊 Extend Access Token，然後取得 long-lived access token 大約會時間是 60 天

![extend-access-token_2](/img/20201120/extend-access-token_2.png)

## 申請永久的 Access Token

60 天期限，代表每 60 天要去重申請一次，雖然比較安全，但好像有點麻煩

這邊有一種方式是繞過去申請永久的 access token，感覺很偏方，但好像也不是不可以使用 😂

至 my app 的產品底下，點 products 旁邊的 + ，選擇 messager

![](/img/20201120/add-product.png)

在 setting 裡找到 access tokens ，點 add or remove pages

![](/img/20201120/add-permanent-access-token.png)

經過一連串認證後，會在該區塊看到綁定的畫面，點擊 generate token ，會產生只能看到一次的 access token，因為是永久性的，所以這 token 也要好好保存

![generate-token](/img/20201120/generate-token.png)

去 token debugger 頁面看，可以發現已經變成是永久的

![permanent-access-token](/img/20201120/permanent-access-token.png)

---

# api request 限制

另外要注意其實有每小時 200 次 query 的限制

Application-Level Rate Limiting

> About Application-Level Rate Limiting
> Rate limiting defines limits on how many API calls can be made within a specified time period. Application-level rate limits apply to calls made using any access token other than a Page access token and ads APIs calls. The total number of calls your app can make per hour is 200 times the number of users. Please note this isn't a per-user limit. Any individual user can make more than 200 calls per hour, as long as the total for all users does not exceed the app maximum

---

因為 access-token 跟 200 次的 request 限制，想了想，其實這邊還是要從後端去拿資料然後存在自己的資料庫比較好，畢竟 access-token 還是不要露在前端會比較好，要不別人知道你的 token 一直攻擊，不就 200 次很快就沒了？！！

另外如果前端是 ssr 的話，至少 api request 會在 node server 就先處理完，在 request api 的 access-token 就不太會外漏給用戶，但要注意 api response 的下一頁 link 會自動帶 access_token ，所以這邊也要注意使用！

---

[reference]

[Graph API 是什麼東東?](https://www.tonylin.idv.tw/dokuwiki/doku.php/facebook:basic:graphapi)

[Facebook Graph API get all likes from a post](https://stackoverflow.com/questions/9955493/facebook-graph-api-get-all-likes-from-a-post)

[/{object-id}/likes](https://developers.facebook.com/docs/graph-api/reference/v9.0/object/likes)

[How to get Likes Count when searching Facebook Graph API with search=xxx](https://stackoverflow.com/questions/17755753/how-to-get-likes-count-when-searching-facebook-graph-api-with-search-xxx)

[即時文章 API](https://developers.facebook.com/docs/instant-articles/api?locale=zh_TW#list-all)

[快速取得 FB 粉絲專頁永久存取權杖(Access Token)](https://www.wfublog.com/2018/12/fb-fanpage-access-token-forever.html)

---

> 看起來很親民的 api 結果規矩一堆？！！ 😂😂😂

> 耶～～寫完這篇，野薑官網 [最新消息 News](https://gingerdesign.com.tw/news) 也熱騰騰上線囉~
