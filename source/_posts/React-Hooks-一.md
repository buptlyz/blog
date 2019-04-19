---
title: React Hooks(一)
date: 2019-04-19 21:09:41
tags: React Hooks
---

## Hooks

* `useState`
* `useEffect`
* `useContext`

## Rules of Hooks

* 只在top level调用Hooks，不要在loops、conditions或者嵌套函数中调用
* 只在React函数组件中调用Hooks，不要在普通Javascript函数中调用（或者在你自定义的Hooks中调用）。

## 自定义Hooks

可以解决之前需要通过借助`heigher-order-components`和`render props`才能解决的组件间状态逻辑的复用。

一个叫`useFriendStatus`的Hook：

```jsx
import React, { useState, useEffect } from 'react';

function useFriendStatus(friendID) {
  const [isOnline, setIsOnline] = useState(null);

  function handleStatusChange(status) {
    setIsOnline(status.isOnline);
  }

  useEffect(() => {
    ChatAPI.subscribeToFriendStatus(friendID, handleStatusChange);
    return () => {
      ChatAPI.unsubscribeFromFriendStatus(friendID, handleStatusChange);
    };
  });

  return isOnline;
}
```

使用`useFriendStatus`：

```jsx
// FriendStatus.js
function FriendStatus(props) {
  const isOnline = useFriendStatus(props.friend.id);

  if (isOnline === null) {
    return 'Loading...';
  }
  return isOnline ? 'Online' : 'Offline';
}

// FriendListItem.js
function FriendListItem(props) {
  const isOnline = useFriendStatus(props.friend.id);

  return (
    <li style={{ color: isOnline ? 'green' : 'black' }}>
      {props.friend.name}
    </li>
  );
}
```

这些组件间的状态是完全独立的。Hooks复用状态逻辑，而不是状态本身。每次调用自定义Hook都会生成完全独立的状态。
