---
title: 高質感 terminal - hyper
date: 2021-04-28
tags: [terminal, hyper, tools]
layout: layouts/post.njk
---

Hyper 標榜是新一代超好看的 terminal ，官方網站 👉🏻 [Hyper](https://hyper.is)

安裝 Hyper 很簡單，直接從官方網站下載就好拉

跟 iTerm2 一樣，可以先安裝超好用的 oh-my-zsh，但如果是從 iTerm2 切過來的，就不用跑安裝 zsh 的功能

## 安裝 zsh

首先要先看電腦本身有沒有 zsh

```bash
$zsh version
```

如果沒有的話可以安裝

Redhat / Centos

```bash
$yum install zsh
```

Debian / Ubuntu

```bash
$ apt-get install zsh
```

## 安裝 Oh-My-ZSH

通過 curl 安裝

```bash
$sh -c "$(curl -fsSL https://raw.githubusercontent.com/robbyrussell/oh-my-zsh/master/tools/install.sh)"
```

通過 wget 安裝

```bash
$ sh -c "$(wget https://raw.githubusercontent.com/robbyrussell/oh-my-zsh/master/tools/install.sh -O -)"
```

---

## 修改字體

一打開就發現字體很小，所以第一件事先來改字體

如果按快捷鍵 `cmd + ,` 會發現跳出一個 .js 的 config，沒看錯，這就是我們要修改的 config

```bash
$vim ~/.hyper.js
```

裡面有 fontSize 值，直接修改存檔就可以囉

```css
fontsize: 14;
```

# 修改 title

hyper default 的 title 是 default 值加上當前路徑，如果有很多個分頁就看不太出來差別

![](/img/20210428/title-origin.png)

參考這篇用 ozh 來設定

[What does terminal / hyperterm display in the tab title? Is it possible to customize this?](https://stackoverflow.com/questions/58239654/what-does-terminal-hyperterm-display-in-the-tab-title-is-it-possible-to-custo)

```bash
$vim ~/.zshrc
```

在最下面貼上

```bash
case $TERM in
    xterm*)
    precmd () {print -Pn "\e]0;%~\a"}
    ;;
esac
```

記得要把 DISABLE_AUTO_TITLE 打開，預設是 mark 起來的

```bash
DISABLE_AUTO_TITLE="true"
```

再跑

```bash
$source ~/.zshrc
```

![](/img/20210428/title-path.png)

雖然說比清楚一點，但似乎路徑一長也就分辨不清處理＠＠

這時候把路徑位置改成 `\033]0;${PWD##*/}\007`

```bash
    case $TERM in
      xterm*)
        precmd () {print -Pn "\033]0;${PWD##*/}\007"}
        ;;
    esac
```

即使有很多頁也可以很清出看得出來位置呢～

![](/img/20210428/title-current.png)

---

# 安裝 plugin

安裝 plugin 前要先點擊這按鈕

```bash
Install Hyper CLI command in PATH
```

![](/img/20210428/plugin.png)

發現之前在 iterm2 用 ozh 安裝的 theme 就自動移過來拉

![](/img/20210428/plugin-display.png)

## 新增透明度

[hyper-opacity](https://github.com/lucleray/hyper-opacity)

```bash
$hyper i hyper-opacity
```

然後去 hyper.js 新增 opacity 的設定就可以囉

```js
//hyper.js
module.exports = {
  config: {
    // rest of the config
    opacity: {
      focus: 0.9,
      blur: 0.5,
    },
  },
  // rest of the file
};
```

---

> 哇，好久沒有寫工具文了～
