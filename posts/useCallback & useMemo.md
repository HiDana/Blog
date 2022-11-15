---
title: useCallback & useMemo
date: 2021-02-11
tags: [react, fb, hooks]
layout: layouts/post.njk
---

在 react 16 後，其實 useState 跟 useEffect 很好上手，很快就可以知道在 fn component 哪裡可以使用，但一直沒有好好深入了解 useCallback 跟 useMemo 以及他們的應用場景，直到最近開始研究 react 效能優化等問題，才好好面對這兩個好用的方法

### 使用場景

因為在網路上的文章，大多都是說，“沒用到的就不用 render ，所以這時候要用 useCallback 做 memoized”

內心就有無限個 os : “啊？”

# useCallback

官方文件 [useCallback](https://zh-hant.reactjs.org/docs/hooks-reference.html#usecallback) 很簡單只有這樣舉例

```js
const memoizedCallback = useCallback(() => {
  doSomething(a, b);
}, [a, b]);
```

> `useCallback` 回傳一個 memoized 的 callback，僅在依賴改變時才會更新。

恩…看完還是不太理解他的應用場景

[從 Hooks 開始，讓你的網頁 React 起來](https://www.tenlong.com.tw/products/9789864345083) 作者 的 [30 天系列 - [Day 20 - 即時天氣] 在 useEffect 中使用呼叫需被覆用的函式 - useCallback 的使用](https://ithelp.ithome.com.tw/articles/10225504) 是這麼說的

> 「如果某個函式不需要被覆用，那麼可以直接定義在 `useEffect` 中，但若該方法會需要被共用，則把該方法提到 `useEffect` 外面後，記得用 `useCallback` 進行處理後再放到 `useEffect` 的 dependencies 中」。

所以其實之前在做 useEffect 時就有遇到這類問題，當把共用的 function (假設是叫 fetchData) 從 useEffect 往外拉，讓畫面初始 load data 跟 button onClick to load data 都可以用同一隻 function。但這時就會出現 miss dependency 的 err ，跟作者一樣，當我們直接把 function name 填到， useEffect 的 dependency 後，就會出現無限迴圈

之前一直很疑惑為什麼，而看完 [你所不知道的 JS ｜範疇與 Closures，this 與物件原型](https://www.tenlong.com.tw/products/9789864760497) 就可以很清楚知道，因為 refence 不同，所以

```js
{} === {} //false
```

這樣 useEffect 一直覺得這個程式碼都一樣的 fetchData 不等於那個 fetchData，所以一直被觸發

這時用 useCallback 就可以解決這問題，讓他都指向同樣的位置

```js
const fetchDataCallback = useCallback(() => {
    fetchData(a,b){
    ...
    }
}, [a, b]);
```

然後在 useEffect 可以讓他加入到 dependency

```js
useEffect(() => {
  etchDataCallback();
}, [etchDataCallback]);
```

### useCallback in child component

> 當傳遞 callback 到已經最佳化的 child component 時非常有用，這些 child component 依賴於引用相等性來防止不必要的 render（例如，`shouldComponentUpdate`）

這篇 [如何錯誤地使用 React hooks useCallback 來保存相同的 function instance](https://as790726.medium.com/如何錯誤地使用-react-hooks-usecallback-來保存相同的-function-instance-7744984bb0a6)
其實就有列舉如何用 useCallback 去讓 子 component 不要一直重複 render

應該還會需要一個篇幅來講解這功能，靜待接下來的文章囉～

---

# useMemo

恩，官方文件更短的去舉例 [useMemo](https://zh-hant.reactjs.org/docs/hooks-reference.html#usememo)

```js
const memoizedValue = useMemo(() => computeExpensiveValue(a, b), [a, b]);
```

> 回傳一個 memoized 的值。傳遞一個「建立」function 及依賴 array。`useMemo` 只會在依賴改變時才重新計算 memoized 的值。這個最佳化可以避免在每次 render 都進行昂貴的計算。
> 要謹記傳到 `useMemo` 的 function 會在 render 期間執行。不要做一些通常不會在 render 期間做的事情。例如，處理 side effect 屬於 `useEffect`，而不是 `useMemo`。

所以其實就是可以藉由 dependency 去避免 function 在每次 render 都被觸發，而是有需要時才會去觸發。

看起來 useMemo 就是我們要的效能優化呢，但官方文件也有這樣註明

> 你可以把 `useMemo` 作為效能最佳化的手段，但請不要把它當作成語意上的保證。在將來，React 可能會選擇「忘記」某些之前已 memorize 的值並在下一次 render 時重新計算，例如，為已離開螢幕的 component 釋放記憶體。先撰寫沒有 `useMemo` 也可執行的代碼 — 然後再加入它來做效能最佳化。

所以其實是有提升，但也會有釋放再重新計算的可能性。

但其實也要注意，如果只單純的字串處理，如果用 useMemo 反而是浪費效能

```js
const result = useMemo(() => `foo: ${foo}`, [foo]);
```

但如果是要 map 陣列，就很適合使用

```js
// Assume this returns an Array of 3000 records
const menuItemRows = useMemo(
  () =>
    thousandsOfMenuItems.map((menuItem) => (
      <MenuItemRow key={menuItem.uuid} name={menuItem.name} />
    )),
  [thousandsOfMenuItems]
);
```

這時候又看到官網的一個 function

```js
    useCallback(fn, deps) 相等於 useMemo(() => fn, deps)。
```

誒？所以 useCallback 跟 useMemo 只有宣告的方式不一樣嗎？

---

# useCallback vs useMemo

參考別人的文章是這麼說的

> In other words, `useCallback` gives you referential equality between renders for functions. And `useMemo` gives you referential equality between renders for values.

> The difference is that `useCallback` returns its function when the dependencies change while `useMemo` calls its function and returns the result.

> `useCallback` returns its function uncalled so you can call it later, while `useMemo` calls its function and returns the result.

所以其實 useCallback 主要是用在調用 function 這部分，然後 useMemo 是來調用其函數的 result(values)

---

[reference]

[什麼時候該使用 useMemo 跟 useCallback](https://medium.com/ichef/什麼時候該使用-usememo-跟-usecallback-a3c1cd0eb520)

[useCallback vs useMemo](https://medium.com/@jan.hesters/usecallback-vs-usememo-c23ad1dc60)

[When to useMemo and useCallback](https://kentcdodds.com/blog/usememo-and-usecallback)

---

> 因為官網說 The following Hooks are either variants of the basic ones from the previous section, or only needed for specific edge cases. Don’t stress about learning them up front. 結果一不小心就真的 no stress to start study 😂

> 終於開始放假拉，新年快樂～
