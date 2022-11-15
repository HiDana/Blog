---
title: svgr's typescript setting
date: 2020-10-16
tags: [nextjs, typescript, svg]
layout: layouts/post.njk
---

在製作野薑官網時遇到一個區塊

覺得很適合用 drawline 的 svg 動畫([SVG 筆記：手寫字動畫](https://medium.com/@GlennJong/svg-筆記-手寫字動畫-3e42ec68bfdd)) 來呈現

![](/img/20201016/design-flow.png)

所以必須把 svg 整個 import 近來

在 react 引用圖片是用 import，但是在 nextjs 時卻是用直接引入 `/public` 下面的檔案

因為是 nextjs 框架，所以用了 svgr 作為套件 impot

```bash
$ npm i @svgr/webpack
```

在 next.config.js 修改設定

```js
// next.config.js
module.exports = {
  webpack(config) {
    config.module.rules.push({
      test: /\.svg$/,
      issuer: {
        test: /\.(js|ts)x?$/,
      },
      use: ["@svgr/webpack"],
    });

    return config;
  },
};
```

理論上這裡就可以直接引用 svg 了，svg import 時用 import 這樣寫

```js
import IconSvg from "/images/icon1.svg";
```

畫面有 svg 圖片出現，但是 typescript 會報 `cannot find module` 的錯

![](https://paper-attachments.dropbox.com/s_E7BE94D7CC0417341F804B2BEB6478C767D28962C0332ED21E3DA67B8AA581BB_1590575528213_+2020-05-27+6.32.03.png)

因為本身在 nextjs 裡面有設定 tsconfig.json 的設定檔

```json
{
...
  "include": [
    "next-env.d.ts",
    "types/*.d.ts",
    "**/*.ts",
    "**/*.tsx",
    "**/*.style.tsx"
  ]
}
```

所以要尋找原本 include 的設定 `next-env.d.ts`，並加上 svg 的 typescript 的設定

```js
declare module "\*.svg" {
  import React = require("react");
  export const ReactComponent: React.SFC<React.SVGProps<SVGSVGElement>>;
  const src: string;
  export default src;
}
```

很好，看起來就會正常了

加上 onScroll 等功能，就可以有很棒的手寫感覺

野薑官網已經上線囉，歡迎大家來看看手寫 svg 的感覺

[野薑工作室官方網站](https://gingerdesign.com.tw/)

---

[reference]

[Can't import SVG into Next JS?](https://stackoverflow.com/questions/55175445/cant-import-svg-into-next-js)

[Importing SVG files as React Components in TypeScript](https://duncanleung.com/typescript-module-declearation-svg-img-assets/)

[Two ways to solve: Cannot find SVG module error when compiling Typescript in React application](https://annacoding.com/article/5I7om6mCrW25KeA4ObeyMJ/Two-ways-to-solve:-Cannot-find-SVG-module-error-when-compiling-Typescript-in-React-application)
