---
title: 用 hook clean code
date: 2021-04-09
tags: [react, hook, clean-code]
layout: layouts/post.njk
---

開發時總有一種疑惑，不知道自己這樣自我感覺寫的 code 到底是好 code 還是糞 code，只能參考別人寫的專案或是文件，然後檢討自己的 code

直到最近看到了一篇

[How To Apply SOLID Principles To Clean Your Code in React](https://betterprogramming.pub/how-to-apply-solid-principles-to-clean-your-code-in-react-cdfd5e0a9cea)

突然驚覺，其實用 hook 也可以讓 code 很簡潔有力，所以想說拿之前寫過的一個 [search-github-repos](https://github.com/HiDana/search-github-repos) ( demo 在這 [https://searchgithubrepos.hidana.me](https://searchgithubrepos.hidana.me) ) 來修改成 hook 的方式，讓每個 component 變得更單純

這是原先很長的 Home.tsx component，雖然把 InfiniteScroll 另外拉出一個 layout 後(總覺得沒有寫得很好)，這個元件就…還是沒有很單純，而且那時寫的時候還沒有針對 getReposData 用 useCallback，回頭看真的是有點糟這樣＠＠

所以現在先試試把 fetch data 拉出來，讓這個 component 只處理顯示

```js
//Home.tsx
import React, { ReactElement, useState, useEffect } from "react";
import styled from "styled-components";
import {
  SearchBar,
  GithubRepoCard,
  InfiniteScrollLayout,
  Loading,
} from "components";
import { getSearchReposApi } from "services";
import { repoType } from "types";
export const Home = (): ReactElement => {
  const [searchInfo, setSearchInfo] = useState("");
  const [isLoading, setIsLoading] = useState(false);
  const [reposData, setReposData] = useState<repoType[]>([]);
  const [reposCount, setReposCount] = useState(0);
  const [page, setPage] = useState(1);
  const isAllDisplay = reposCount !== 0 && page * 20 >= reposCount;

  //fetch data
  const getReposData = async (info: string, page: number) => {
    page === 1 && setReposData([]);
    setIsLoading(true);
    try {
      const response = await getSearchReposApi(info, page);
      if (response.kind === "ok") {
        if (page === 1) {
          setReposData(response.data.items);
          setReposCount(response.data.total_count);
        } else {
          setReposData([...reposData, ...response.data.items]);
        }
      }
    } catch (e) {
      console.log(e);
    } finally {
      setIsLoading(false);
    }
  };

  useEffect(() => {
    if (searchInfo === "") {
      setReposData([]);
    } else {
      getReposData(searchInfo, page);
    }
  }, [searchInfo, setReposData, page]);

  return (
    <HomeStyle>
      <SearchBar
        setInfo={(txt: string) => {
          setSearchInfo(txt);
          setPage(1);
        }}
      />
      <InfiniteScrollLayout
        isLoading={isLoading}
        setTouchBottom={() => !isLoading && !isAllDisplay && setPage(page + 1)}
      >
        {reposData.map((data: repoType, i: number) => (
          <GithubRepoCard data={data} key={i} />
        ))}
        {isLoading && <Loading />}
        {isAllDisplay && searchInfo !== "" && "沒u了"}
      </InfiniteScrollLayout>
    </HomeStyle>
  );
};
const HomeStyle = styled.div`
  width: 100%;
  max-width: 600px;
  padding: 20px;
`;
```

先新增一個 useGetRemoteData 的 hook

```js
//useGetRemoteData.ts
import { useState } from "react";
export const useGetRemoteData = () => {
  const [responseData, setResponseData] = useState([]);
  const [isLoading, setIsLoading] = useState(false);
  return { responseData, isLoading };
};
```

然後在 Home 頁面去引入

```js
//Home.tsx
import { useGetRemoteData } from "utils";
...
const { responseData, isLoading } = useGetRemoteData();
console.log("responseData", responseData);
console.log("isLoading", isLoading);
```

ok ，在 Home 就可以接到 useGetRemoteData 的 data

![](/img/20210409/useGetRemoteData-pass-data.png)

然後我們測試，如果在 search input 輸入的東西，是不是可以正確傳過去
先把 searchInfo 帶進去

```js
const { responseData, isLoading } = useGetRemoteData(searchInfo);

//useGetRemoteData.ts
import { useState } from "react";
export const useGetRemoteData = (searchInfo: string) => {
  const [responseData, setResponseData] = useState([]);
  const [isLoading, setIsLoading] = useState(false);
  console.log("searchInfo", searchInfo);
  return { responseData, isLoading };
};
```

ok，也可以成功拿到 search 的 input 內容
接下來把之前在首頁的 api 拉進來

```js
//useGetRemoteData.js
import { useState, useEffect, useCallback } from "react";
import { repoType } from "types";
import { getSearchReposApi } from "services";

export const useGetRemoteData = (searchInfo: string, page: number) => {
  const [reposCount, setReposCount] = useState(0);
  const [responseData, setResponseData] = useState<repoType[]>([]);
  const [isLoading, setIsLoading] = useState<boolean>(false);
  const getReposData = useCallback(
    async (info: string, page: number) => {
      page === 1 && setResponseData([]);
      setIsLoading(true);
      try {
        const response = await getSearchReposApi(info, page);
        if (response.kind === "ok") {
          if (page === 1) {
            setResponseData(response.data.items);
            setReposCount(response.data.total_count);
          } else {
            setResponseData([...responseData, ...response.data.items]);
          }
        }
      } catch (e) {
        console.log(e);
      } finally {
        setIsLoading(false);
      }
    },
    [searchInfo, page]
  );

  useEffect(() => {
    if (searchInfo === "") {
      setResponseData([]);
    } else {
      getReposData(searchInfo, page);
    }
  }, [searchInfo, setResponseData, page, getReposData]);

  return { responseData, reposCount, isLoading };
};
```

歐歐歐歐，把 request 拉出來，讓原本 Home 的 component 變得更單純的去 display 樣式而且更單純呢！

```js
//Home.js
import React, { ReactElement, useState } from "react";
import styled from "styled-components";
import {
  SearchBar,
  GithubRepoCard,
  InfiniteScrollLayout,
  Loading,
} from "components";
import { repoType } from "types";
import { useGetRemoteData } from "utils";
export const Home = (): ReactElement => {
  const [searchInfo, setSearchInfo] = useState("");
  const [page, setPage] = useState(1);
  const { responseData, reposCount, isLoading } = useGetRemoteData(
    searchInfo,
    page
  );
  const isAllDisplay = page * 20 >= reposCount;
  return (
    <HomeStyle>
      <SearchBar
        setInfo={(txt: string) => {
          setSearchInfo(txt);
          setPage(1);
        }}
      />
      <InfiniteScrollLayout
        isLoading={isLoading}
        setTouchBottom={() => !isLoading && !isAllDisplay && setPage(page + 1)}
      >
        {responseData.map((data: repoType, i: number) => (
          <GithubRepoCard data={data} key={i} />
        ))}
        {isLoading && <Loading />}
        {isAllDisplay && searchInfo !== "" && "沒u了"}
      </InfiniteScrollLayout>
    </HomeStyle>
  );
};
const HomeStyle = styled.div`
  width: 100%;
  max-width: 600px;
  padding: 20px;
`;
```

看起來 InfiniteScrollLayout 也可以改成 hook 方式來寫呢！

---

> 依照作者寫的 Single-Responsibility Principle 原則，回頭看看這 component 覺得即使是把 request 拉出來，還是覺得有違和的感覺以及優化的空間，未來有機會再來重構看看吧！
