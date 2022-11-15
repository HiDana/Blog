---
title: 前端角色權限管理 RBAC - casl
date: 2021-04-20
tags: [react, rbac, casl]
layout: layouts/post.njk
---

每次做後台時，就會遇到一個問題 → 不同身份可以使用不同功能

過去角色比較簡單，功能比較單純，所以大多就直接自己寫完，直到最近專案需要一個強健的後台，所以就要來找找好用的 RBAC（Role based Access Control） 套件

看了一些關於權限的文章，在前端大概處理權限會有幾個方向

- 判斷 router？還是判斷 component?
- 判斷角色？還是判斷可不可以使用這個功能

## 判斷 router？還是判斷 component?

這邊就要看需求了，如果需求跟 router 走，比如說一個角色可以使用這頁面裡的所有功能，或是不可以使用頁面裡的所有功能，不會有這角色在這頁面裡，有些功能可以用有些功能不能用，那就很適合跟著 router 走，這樣判斷就寫在 router 那邊， react-router 有 PrivateRoute 來處理基本需求的權限控管需求。如果不是使用 router 那就以功能來做判斷，也就是你在寫一個功能時，要先去做權限判斷。

## 判斷角色？還是判斷可不可以使用這個功能

其實這個問題很簡單，就是拿變動性最小的來去做判斷。如果是角色，角色會有新增、修改、刪除角色等需求，而且角色裡面也會有底下權限的變動，比如說今天元件或是功能是這樣寫

```js
if (userA || userB) {
  //can do something
}
```

那今天新增一個 userC 也可以使用該功能，是不是就要去各元件或是功能下去做修改？ 所以角色應該可以當作一個整理的統稱，而每個需要判斷的功能要用 “可不可以使用這功能” 來做判斷。

這樣講其實有點抽象，可以看下面 casl 等套件的使用判斷方式

```js
ability.can("read", "BlogPost");
```

就是去判斷這個目前這個 ability 可不可以在 BlogPost 使用 read

### 跟 RABC 有關的套件

下面這幾個套件，以目前星星數，由上而下排列，每個都有個別的特色，大家就拿最適合自己專案的來使用就好了吧！

- [casl](https://github.com/stalniy/casl)
- [role-acl](https://github.com/tensult/role-acl#readme)
- [easy-rabc](https://github.com/DeadAlready/easy-rbac#readme)
- [rabc](https://www.npmjs.com/package/rbac)
- [role-based-access-control](https://github.com/umair-khanzada/role-based-access-control)

---

# [casl](https://casl.js.org/v5/en)

這邊先試試目前是星星最多的 casl ，裡面也有針對前端 react 的 project，而且還有很詳細的使用文件 😂

| Project        | Description               | Supported envinronemnts                       |
| -------------- | ------------------------- | --------------------------------------------- |
| @casl/ability  | CASL's core package       | nodejs 8+ and ES5 compatible browsers (IE 9+) |
| @casl/mongoose | integration with Mongoose | nodejs 8+                                     |
| @casl/angular  | integration with Angular  | IE 9+                                         |
| @casl/react    | integration with React    | IE 9+                                         |
| @casl/vue      | integration with Vue      | IE 11+ (uses `WeakMap`)                       |
| @casl/aurelia  | integration with Aurelia  | IE 11+ (uses `WeakMap`)                       |

### [@casl/react](https://github.com/stalniy/casl/blob/master/packages/casl-react)

```bash
$npm install @casl/react @casl/ability
```

這邊測試權限管理，大概會有兩個面向

1. 定義用戶權限
2. 取得用戶是否可以使用該功能

## 定義用戶權限

參考 casl 文件，可以先創建一個定義權限的功能

casl 有三個可以定義權限的方式

- using `defineAbility` function
- using `AbilityBuilder` class
- using `JSON` objects

using `AbilityBuilder` class

下面用 `AbilityBuilder` class 的方式示範

```js
export const defineAbilitiesFor = () => {
  const { can, rules } = new AbilityBuilder(Ability);
  can(["read", "update"], ["BlogPost", "BlogComment"]);
  return new Ability(rules);
};
```

除了可以個別寫

```js
can("read", "BlogPost");
can("update", "BlogPost");
can("read", "BlogComment");
can("update", "BlogComment");
```

也可以像上面一樣合起來寫

```js
can(["read", "update"], ["BlogPost", "BlogComment"]);
```

using `JSON` objects

如果不用 AbilityBuilder 用 json 來定義也蠻直觀的

```js
import { Ability } from "@casl/ability";

export const defineAbilitiesFor = () =>
  new Ability([
    {
      action: "read",
      subject: "BlogPost",
    },
    {
      inverted: true,
      action: "delete",
      subject: "BlogPost",
      conditions: { published: true },
    },
  ]);
```

而 json 格式須依照他的 RawRule 走

```js
interface RawRule {
  action: string | string[]
  subject?: string | string[]
  /** an array of fields to which user has (or not) access */
  fields?: string[]
  /** an object of conditions which restricts the rule scope */
  conditions?: any
  /** indicates whether rule allows or forbids something */
  inverted?: boolean
  /** message which explains why rule is forbidden */
  reason?: string
}
```

## 取得用戶是否可以使用該功能

使用方式其實很簡單，就是直接把剛剛定義的 defineAbilitiesFor 抓進來，然後對他詢問 ` .can(``"``do-somthing``"``, ` ` "``feature``"``) `

```js
import React from "react";
import { defineAbilitiesFor } from "./defineAbilitiesFor";

import React from "react";
import { defineAbilitiesFor } from "./defineAbilitiesFor";
export const CheckAbilities = () => {
  const ability = defineAbilitiesFor("user"); //這邊 "user" 可以帶不同角色，或是 userInfo
  const access_read = ability.can("read", "BlogPost");
  const access_update = ability.can("update", "BlogPost");
  console.log("access_read", access_read); //true
  console.log("access_update", access_update); //true
  return <div>CheckAbilities</div>;
};
```

上面就是 casl 最簡易的使用方式

casl 本身也有一個 react 的 [example CASL React Todo](https://codesandbox.io/s/github/stalniy/casl-examples/tree/master/packages/react-todo)

裡面用 react context 來跟 casl 綁定，使用時也可以用 @casl/react 的語法使用

- `do` - name of the action (e.g., `read`, `update`). Has an alias `I`
- `on` - checked subject. Has `a`, `an`, `this` aliases
- `field` - checked field

```js
export default ({ post }) => (
  <Can I="read" this={post} field="title">
    Yes, you can do this! ;)
  </Can>
);
```

把 component 上面的英文唸起來蠻有趣的

```text
Can I read this post.
```

---

[reference]

[Role based access control in React-Redux apps](https://medium.com/@perfectsudh/role-based-access-control-in-react-redux-apps-8454d4ca1a3b)

[Public, private, and role-based routes in React](https://javascript.plainenglish.io/role-based-authorization-role-based-access-control-v-2-in-react-js-cb958e338f4b)

[How to Role Based Access Control (RBAC) ?](https://dev.to/thearvindnarayan/how-to-role-based-access-control-rbac-2935)

---

> 因為只有初步測試這 RBAC 方式，所以這篇寫得蠻淺的，未來如果有使用後有心得，就可以再來寫一篇 RBAC2 了 😂
