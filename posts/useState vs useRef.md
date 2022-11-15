---
title: useState vs useRef
date: 2021-03-16
tags: [react, hooks]
layout: layouts/post.njk
---

在開始使用 hooks 後，其實蠻常使用 useState 或是 useRef，但沒有好好去探討他們之間的差異

來做一個簡單的貓貓新增功能

![](/img/20210316/AddCat.png)

先用 useState 來寫，並放一個 log 來監測這 component 的 render

```js
//AddCat.js
import React, { useState } from "react";
export const AddCat = () => {
  const [catsName, setCatsName] = useState("");
  return (
    <div>
      {console.log("AddCat-render")}
      <input
        placeholder="貓貓名稱"
        onChange={(e) => setCatsName(e.target.value)}
      />
      <button>新增貓貓</button>
      <h5>{catsName}</h5>
    </div>
  );
};
```

因為每一次 onChange 就會觸發一次 setState，然後觸發一次頁面 render，所以效能來說其實這樣很不好

![](/img/20210316/AddCat-useState.png)

然後換 useRef 來寫，這邊可以很明顯地看到，setState 不是從 onChange 監聽然後賦值，而是在綁定 onClick 後，用 useRef 拿到 input 裡面的值

```js
import React, { useState } from "react";
export const AddCat = () => {
  const valueRef = React.useRef();
  const [catsName, setCatsName] = useState();
  return (
    <div>
      {console.log("AddCat-render")}
      <input placeholder="貓貓名稱" ref={valueRef} />
      <button onClick={() => setCatsName(valueRef.current.value)}>
        新增貓貓
      </button>
      <h5>{catsName}</h5>
    </div>
  );
};
```

![](/img/20210316/AddCat-useRef.png)

這邊可以很明顯地看到，只有兩此 render，一次是第一次渲染，第二次是我們按下 btn 後新增的渲染

但可以很明顯地看到，如果要即時看到 h5 裡的渲染成果，用 onChange 去監聽然後一直 setState，然如果其實只需要看到一次 h5 裡的結果，那就可以放在 btn 的 onClick 後用 useRef 來取值再 setState

---

[reference]

[React: useState or useRef?](https://stackoverflow.com/questions/56455887/react-usestate-or-useref)

[useRef vs useState: Should we re-render or not?](https://www.codebeast.dev/usestate-vs-useref-re-render-or-not/)

---

> 看起其實不是他們之間兩個的差別，而是在使用這兩個的時機，要如何使用才會讓頁面渲染最省力才是這篇探討的重點 😂
