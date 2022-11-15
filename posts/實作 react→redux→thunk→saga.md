---
title: 實作 react→redux→thunk→saga
date: 2021-03-08
tags: [react, redux, redux-thunk, redux-saga]
layout: layouts/post.njk
---

一開始寫 react 後就知道 redux 狀態管理的方式，所以也很快就引入到專案使用，直到最近就突然想到，好像沒有很好的去理解他的演變ＸＤ

這邊就觀念就簡單帶過去，主要是在簡易的實現實作跟比較差異，其中也有連結很多很棒的文章，想看更多可以參考文章連結～

---

# redux

一開始主要是去處理跨多層頁面的傳值，所以選擇 redux 來做狀態管理。

可以參考 超Ｑ神人 的文章

[React 與他的快樂小夥伴 Redux -基礎教學](https://medium.com/enjoy-life-enjoy-coding/react-及-redux-間的日常-1-基本使用-215436d14430)

[React 與他的快樂小夥伴 Redux -事件處理（Handling events）](https://medium.com/enjoy-life-enjoy-coding/react-與他的快樂小夥伴-redux-事件處理-handling-events-8dd35545f7b9)

底下就來用 `新增貓貓的功能` 來當範例吧ＸＤ

```bash
$ npm i redux react-redux
```

```js
//action.js
export const ADD_CAT = "ADD_CAT";
export const getMoreCat = (name) => {
  return {
    type: ADD_CAT,
    payload: {
      cat: name,
    },
  };
};

//reducer.js
import { ADD_CAT } from "./action";

const initState = { list: ["橘貓"] };
const catReducer = (state = initState, action) => {
  switch (action.type) {
    case ADD_CAT:
      console.log("cat payload", action.payload);
      return { ...state, list: [...state.list, action.payload.cat] };
    default:
      return state;
  }
};
export default catReducer;
```

要記得中央 store 要把剛剛貓貓的 reducer 引入

```js
//store.js
import { createStore } from "redux";
import catReducer from "./reducer";

let store = createStore(catReducer);

export default store;
```

App.js 也不要忘了帶入剛剛設定的 store

```js
//app.js
import { Provider } from "react-redux";
import store from "./redux/store";
import Cat from "./Cat"; //發起 dispatch 的貓貓元件
function App() {
  return (
    <Provider store={store}>
      <div className="App">
        <Cat />
      </div>
    </Provider>
  );
}
export default App;
```

最後在一般的 component connect 我們的 redux

```js
//Cat.js
import React from "react";
import { connect } from "react-redux";
import { getMoreCat } from "./redux/action";
const Cat = ({ list, getMoreCat, getMoreCatApi }) => {
  console.log("cat list", list);
  return (
    <div>
      <botton onClick={() => getMoreCat("虎斑貓")}>增加貓貓</botton>
    </div>
  );
};
const mapStateToProps = (state) => ({ list: state.list });
const mapDispatchToProps = (dispatch) => {
  return {
    getMoreCat: (name) => dispatch(getMoreCat(name)),
  };
};
export default connect(mapStateToProps, mapDispatchToProps)(Cat);
```

好拉，現在如果我們要從遠端 request 貓貓 api 怎麼辦

這邊用 the cat api : GET [https://api.thecatapi.com/v1/categories](https://api.thecatapi.com/v1/categories)

btw 剛剛 redux 是用 array<string>，去做範例，但是這邊 api request 回傳是物件，所以 store list 內容會不一樣，若要轉換可以自行去處理

並新增一個 axios 的 request

```bash
    $npm i axios
```

```js
//request.js
import axios from "axios";

export const fetchGetCats = async () => {
  try {
    const { data } = await axios.get("https://api.thecatapi.com/v1/categories");
    return data;
  } catch (error) {
    console.log("error", error);
  }
};
```

在 action 新增 `getMoreCatApi` 然後把剛剛的 api request 放進去

```js
//這是錯誤使用方式
//acion.js
export const getMoreCatApi = async () => {
  try {
    const response = await fetchGetCats();
    return {
      type: ADD_CAT,
      payload: {
        cat: response,
      },
    };
  } catch (error) {
    console.log("error");
  }
  const response = "api貓貓";
};
```

別忘了新增一個 增加遠端貓貓 的 button 連接剛剛我們定義的 `getMoreCatApi`

先在 mapDispatchToProps connect 我們定義的 getMoreCatApi

`getMoreCatApi: () => dispatch(getMoreCatApi()),`

然後新增 "增加遠端貓貓的" button

`<button onClick={() => getMoreCatApi()}>增加遠端貓貓</button>`

貓貓元件如下

```js
//Cat.js
import React from "react";
import { connect } from "react-redux";
import { getMoreCat, getMoreCatApi } from "./redux/action";

const Cat = ({ list, getMoreCat, getMoreCatApi }) => {
  console.log("list", list);
  return (
    <div>
      <button onClick={() => getMoreCat("虎斑貓")}>增加貓貓</button>
      <br />
      <button onClick={() => getMoreCatApi()}>增加遠端貓貓</button>
    </div>
  );
};
const mapStateToProps = (state) => ({ list: state.list });
const mapDispatchToProps = (dispatch) => {
  return {
    getMoreCat: (name) => dispatch(getMoreCat(name)),
    getMoreCatApi: () => dispatch(getMoreCatApi()),
  };
};
export default connect(mapStateToProps, mapDispatchToProps)(Cat);
```

登愣，會出現異步的問題 err

![](/img/20210308/async-error.png)

搭拉，這時候 thunk 就派上用場了拉

---

# thunk

在使用 thunk 前，可以先看看 [詳解 Redux Middleware](https://max80713.medium.com/詳解-redux-middleware-efd6a506357e) ，了解一下 middleware 在 redux 是扮演什麼角色

我覺得他這圖片解釋得非常好，middleware 在 action 被指派後，在 reducer 之前，可以去做額外處理

![image from https://max80713.medium.com/詳解-redux-middleware-efd6a506357e](/img/20210308/middleware.png)

那就開始使用 redux-thunk 囉～

```bash
    $npm i redux-thunk
```

而要變動的地方也不多，先在 store 引入 thunk 的 middleware

記得要在 createStore 裡把 thunk 加入到 redux 的 middleware 裡

```js
//store.js
import { createStore, applyMiddleware } from "redux";
import thunk from "redux-thunk";
import catReducer from "./reducer";

const store = createStore(catReducer, applyMiddleware(thunk));
export default store;
```

action 則藉由 dispatch callback 去呼叫剛剛定義過的 getMoreCat

```js
return async (dispatch) => {
...
dispatch(getMoreCat(response));
...
};

```

因此 action 會是長這樣

```js
//action.js
export const getMoreCatApi = () => {
  console.log("get more cats with api");
  return async (dispatch) => {
    try {
      const response = await fetchGetCats();
      dispatch(getMoreCat(response));
    } catch (error) {
      console.log("error");
    }
  };
};
```

這邊應該可以看到有成功藉由 thunk 的去處理 api 的異步處理

![](/img/20210308/thunk-success.png)

---

# saga

其實 thunk 就可以解決 redux 異步問題，但又出現 redux-saga 出現，為了解決持續增長導致難以維護的 action

開始使用 saga 囉～

```bash
$npm i redux-saga
```

先處理好 store 與 saga 的 middleware

```js
import createSagaMiddleware from "redux-saga";
import { createStore, applyMiddleware } from "redux";
import catReducer from "./reducer";
import rootSaga from "./saga";

const sagaMiddleware = createSagaMiddleware();
const store = createStore(catReducer, applyMiddleware(sagaMiddleware));
sagaMiddleware.run(rootSaga);

export default store;
```

saga 這邊適用 generator function 使用 yield 一步一步下去，監聽 FETCH_DATA 的動作，然後定義等 api response 回來後再用原本定義的 ADD_CAT 去跟 reducer 溝通新增貓貓

```js
//saga.js
import { call, put, takeEvery } from "redux-saga/effects";
import { ADD_CAT } from "./action";
import { fetchGetCats } from "../api/request";

export const FETCH_DATA = "FETCH_DATA";

function* fetchData() {
  const data = yield call(() => fetchGetCats());
  yield put({ type: ADD_CAT, payload: data });
}

function* mySaga() {
  yield takeEvery(FETCH_DATA, fetchData);
}

export default mySaga;
```

可以看到 Cat 這邊是直接 dispatch 到 saga 去做 FETCH_DATA 處理

```js
//Cat.js
import React from "react";
import { connect } from "react-redux";
import { getMoreCat } from "./redux/action";
import { FETCH_DATA } from "./redux/saga";

const Cat = ({ list, getMoreCat, getMoreCatApi }) => {
  console.log("list", list);
  return (
    <div>
      <button onClick={() => getMoreCat("虎斑貓")}>增加貓貓</button>
      <br />
      <button onClick={() => getMoreCatApi()}>增加遠端貓貓</button>
    </div>
  );
};
const mapStateToProps = (state) => ({ list: state.list });
const mapDispatchToProps = (dispatch) => {
  return {
    getMoreCat: (name) => dispatch(getMoreCat(name)),
    getMoreCatApi: () => {
      dispatch({ type: FETCH_DATA });
    },
  };
};
export default connect(mapStateToProps, mapDispatchToProps)(Cat);
```

ok 了拉，可以看到藉由 saga 的去處理 api 的異步處理

![](/img/20210308/saga-success.png)

這篇 saga 寫法是參考超Ｑ神人的 [Redux Saga | Redux 界的非同步救星 - 基本用法](https://medium.com/enjoy-life-enjoy-coding/redux-saga-redux-界的非同步救星-基本用法-d38ce3e64308)，所以想要更了解觀念也可以看看這篇

---

redux-thunk 與 redux-saga 的差異

| redux-thunk                          | redux-saga                |
| ------------------------------------ | ------------------------- |
| 使用上較為簡單                       | 程式複雜度較高            |
| 持續增長 Action 容易過於龐大難以維護 | 切分較為乾淨，較容易維護  |
| 較適合入門者學習                     | 建議先了解 thunk 後再切換 |

可以說 redux thunk 的 fetch api 事件處理是這樣流程

```text
    頁面點擊 新增遠端貓貓 按鈕 →
    action 裡運用 thunk 去做異步處理 →
    reducer 去 parse 資料
```

而 redux saga 的 fetch api 事件處理是這樣流程

```text
    頁面點擊 新增遠端貓貓 按鈕 →
    在 saga 去做異步處理 →
    reducer 去 parse 資料
```

可以看到 saga 很明顯把異步處理另外抽出來做處理了，所以應該會比較易讀跟好維護吧
（燦笑）

---

> 兩年前寫一篇 thunk 跟 saga 的比較也只寫到一半 😂，如今終於 run 完整個流程了。不過沒有寫過 saga 的 test ，感覺未來也可以試試，感覺挺有趣的～
