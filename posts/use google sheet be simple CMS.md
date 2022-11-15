---
title: use google sheet be simple CMS
date: 2019-05-08
tags: [code, js, google, google excel, google script]
layout: layouts/post.njk
---

前些日子接了一個 case，因為只有一個工程師加上一個設計，卻要完成有後台的網站

因此討論後決定用 wordpress + google sheet

以前寫程式時，很排斥 wordpress 覺得這都是個速成的東西，讓專業變成不專業

但實際研究過後，發現真的是很速成的東西呢！！許多 server 的平台都有提供一鍵完成，只要付平台費就可以獲得一個有前台跟後台的網站，某方面來說真的讓專案開發時程可以大幅縮短

拉回正題，因為專案需求所以我們需要滿足以下幾個需求

1. 客戶使用 google sheet 完成前台頁面的變化

2. google sheet 可以有個 api 讓 wordprss 可以 request

原先預想 運用 google sheet 可以直接用 “發佈到網路”，如果這樣使用 ajax 引用的話會有 CORS 的問題

![](/img/20190508/1.png)

_注意_
有些網路上會說 <s>透過 Google Sheet API 拿回來的資料預設會帶有 CORS Header，所以不受跨 domain 存取限制，很方便吧！</s>

但僅限於舊版 google sheet 能這樣做，現在的版本都不行了

```
I've confirmed that this is a bug in the Drive API, but it is only affecting the exportLinks for New Google Sheets file. Old Google Sheets and other Google file types should work correctly. I've raised it with the team, but it's unclear how long it will take to fix. Unfortunately there are no simple workarounds, and the only solution would be to introduce a server-side component that would download the file and then pass it along to the client.
```

但如果真的想要只用 google sheet 做 api 可以用 [Tabletop javascript](https://medium.com/@jaejohns/how-to-use-google-sheets-as-your-website-database-b0f2f13d0396) 做引入，就不回有 CORS 的問題了

---

因為直接從 google sheet 下來的資料其實有太多不是我們想要的，如果使用 wordpress 做 request 的話，會額外花太多精力在洗資料，所以最後決定使用 [google app script](https://developers.google.com/apps-script/)  來做 google sheet 的轉接

```
google app script 在接 google 相關服務其實都蠻方便的，比如說 google drive 或是 google calendar 等
```

![](/img/20190508/3.png)

完成 sheet 的編輯後，在雲端硬碟新增一個 google app script，如果沒有就點擊 更多… 尋找

_用 google app script 不用 "發佈到網路"_

![](/img/20190508/2.png)

google app script 是吃 doGet() 開始的

![](/img/20190508/4.png)

```js
function doGet(e) {
  // Change Spread Sheet url
  var ss = SpreadsheetApp.openByUrl("google sheet 的 URL");

  // Sheet Name, Chnage Sheet1 to Users in Spread Sheet. Or any other name as you wish
  var sheet = ss.getSheetByName("該 sheet 的 name (下方該表單名稱)");

  return getUsers(sheet);
}

function getUsers(sheet) {
  var jo = {};
  var dataArray = [];

  // collecting data from 3nd Row , 1st column to last row and last column
  var rows = sheet
    .getRange(3, 1, sheet.getLastRow() - 1, sheet.getLastColumn())
    .getValues();

  for (var i = 0, l = rows.length; i < l; i++) {
    var dataRow = rows[i];
    var record = {};
    record["secondary_category"] = dataRow[2];
    record["name"] = dataRow[3];

    dataArray.push(record);
  }

  jo.user = dataArray;

  var result = JSON.stringify(jo);
  return ContentService.createTextOutput(result).setMimeType(
    ContentService.MimeType.JSON
  );
}
```

因為 google app script 並非 100% 支援 ES6 的語法，所以有寫還是要用舊語法來寫

![](/img/20190508/5.png)

再來把 google app script 對外發佈

發佈 -> 部署為網路應用程式

專案版本，點擊新增，選擇新增才會更新 google app script 的對外 api 資訊
應用程式存取權只用者，更改為 “任何人，甚至是匿名使用者”

![](/img/20190508/6.png)

點擊 “部署”

![](/img/20190508/7.png)

給予授權

![](/img/20190508/8.png)

就會獲得該網路應用的網址

![](/img/20190508/9.png)

簡單的使用 postman 測試，是有成功拿到我們要的 google sheet 資料跟 json 格式

---

其實挺早就知道 google sheet 可以當一個簡單的後台來做，網路上也有很多的服務也是用 google sheet 做後台編輯，例如 [timeline.js](https://timeline.knightlab.com)

實際拿來專案上使用到是第一次，但發現很多人其實在於類似 excel 這種表格掌握度，是比一個陌生的後台來得還快上手，所以運用 google sheet 做一個簡易的 CMS ，其實客戶接受度也算是高的

---

參考文章

[No 'Access-Control-Allow-Origin' header for exportLink](https://stackoverflow.com/questions/26823456/no-access-control-allow-origin-header-for-exportlink)

[No CORS header on GET of exportLink of New Google Sheets](https://issuetracker.google.com/issues/36759302)

[How to use Google Sheets As Your Website Database](https://medium.com/@jaejohns/how-to-use-google-sheets-as-your-website-database-b0f2f13d0396)

[Get Sheet Done — using Google Spreadsheets as your data backend](https://medium.freecodecamp.org/get-sheet-done-using-google-spreadsheets-as-your-data-backend-650ba23dc6d9)

[google sheets api](https://developers.google.com/sheets/api/reference/rest/)

[利用-Google-試算表-Google-Sheet-作為外部資料來源](https://kuro.tw/posts/2018/08/27/利用-Google-試算表-Google-Sheet-作為外部資料來源/)

[利用 Google 試算表當小型資料庫 (4)使用 SQL 語法讓搜尋功能更強大](https://www.wfublog.com/2016/11/google-4-sql.html)

[Google 試算表當資料庫並取得 API](https://www.ioa.tw/article/7-Google%20試算表當資料庫並取得%20API.html)

[Vedio:Get Google Sheet Data in JSON Format using App Script code](https://www.youtube.com/watch?v=uJDLT8nh2ps)
