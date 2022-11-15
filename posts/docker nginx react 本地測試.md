---
title: docker nginx react 本地測試
date: 2021-07-13
tags: [react, docker, nginx]
layout: layouts/post.njk
---

前些日子公司要部署了新的專案，所以需要跟 devOps 洽街 react 部署方式，因為 react 是 SPA 所以需要在 nginx 處理 try_file 設定，這樣才會解決 react-router 在 refresh 會找不到畫面的問題

所以先在本地模擬測試一下 nginx 的 config

nginx 可以有兩種方式測試

- 在 mac 裡用 brew install nginx 測試
- 在 docker 裡起 nginx image 測試

在 mac install nginx 似乎略顯複雜(其實是看步驟很多懶得去了解ＸＤ)
所以這邊我就決定來用 docker 起測試

---

# Docker 裡用 nginx 起 index.html

一開始 docker 的基本觀念就不多說拉～

剛好在網路上找到這篇 Docker 建立 Nginx 基礎分享 ，就參考裡面的步驟，先完成用 nginx 起 index.html

## 建立 index.html

就先創立一個簡單的 index.html 拉，裡面內容要打什麼都可以～

下載 nginx image

先從遠端 docker hub 下載 nginx 的 image

```bash
$docker pull nginx
```

這時如果下 `$docker images`
可以看到 nginx 在我們本地的 docker image 裡了

![](/img/20210713/nginx-image.png)

## 創建測試 image

在跟 index.html 的地方新增一個 `dockerfile` ，一樣參考上面那篇網頁的寫法，`dockerfile` 裡面內容如下

```yml
//dockerfile
FROM nginx
COPY index.html /usr/share/nginx/html
COPY nginx.conf /etc/nginx/conf.d/default.conf
```

看這 dcokerfile 就知道我們還缺一個檔案 nginx.conf，一樣在跟 index.html 平行的位置新增 nginx.conf 檔案，內容如下

```text
//nginx.conf
server{
    listen       80 default_server;
    listen       [::]:80 default_server;
    server_name  lou.com;
    root         /usr/share/nginx/html;
    index        index.html;
    charset utf-8;
    access_log /var/log/nginx/access_log;
    error_log /var/log/nginx/error_log;
}
```

### 創建屬於我們測試的 image

先 build 我們的 image

```bash
$docker build -t <image_name>
```

所以例如 image 名稱我們取為 nginx-index-test ，所以指令就這樣寫

```bash
$docker build -t nginx-index-test .
```

在用 `$docker images` 看一下我們剛剛創的 image 有沒有出現

![](/img/20210713/image.png)

## 啟動 container

接下來就是基於剛剛 build 的 image 起 container

```bash
$docker run --name <container_name> -p 8080:80 -d <image_name>
```

所以如果我們取 nginx-index-test-run 為 container 名字 ，所以我們指令就要這樣下

```bash
$docker run --name nginx-index-test-run -p 8080:80 -d nginx-index-test
```

用 `$docker ps -a` 來看看我們的 container 有沒有正常起起來

![](/img/20210713/container.png)

這樣就有起起來拉～

---

# Docket 裡用 nginx 起 react

因為只用在測試 nginx 有沒有可以通，所以就先在本地把 react build 起來
然後我們把 build 的檔案取代原本 index.html 的地方

dockerfile 也要改成這樣，把 /build 整個料夾映射過去

```yml
//dockerfile
FROM nginx
COPY /build /usr/share/nginx/html
COPY nginx.conf /etc/nginx/conf.d/default.conf
```

nginx 這邊新增我們要測試的 try_files 設定檔就好了

```text
server{
    listen       80 default_server;
    listen       [::]:80 default_server;
    server_name  lou.com;
    root         /usr/share/nginx/html;
    index        index.html;
    charset utf-8;
    access_log /var/log/nginx/access_log;
    error_log /var/log/nginx/error_log;
    location / {
        root   /usr/share/nginx/html;
        index  index.html index.htm;
        try_files $uri $uri/ /index.html;
    }
}
```

一樣先 build image

```bash
$docker build -t <image_name>
```

然後再跑 container

```bash
    $docker run --name <container_name> -p 8080:80 -d <image_name>
```

搭拉～這樣就簡單完成在本地起 nginx 測試 react 拉～

---

> 一開始覺得好像很難，但實作起來其實好像真的沒有想像中的這麼難ＸＤ
