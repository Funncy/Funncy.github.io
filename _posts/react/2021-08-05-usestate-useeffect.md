---
layout: post
title: '[React] - useState & useEffect '
subtitle: 'react -  useState & useEffect'
categories: react
tags: react useState useEffect
comments: true
---

react Hook을 통해 function 컴포넌트에서도 state와 lifecycle 처리를 해줄 수 있다.

### useState

---

```jsx
import React, { useState } from 'react';

function Example() {
	// "count"라는 새 상태 변수를 선언합니다
	const [count, setCount] = useState(0);

	return (
		<div>
			<p>You clicked {count} times</p>
			<button onClick={() => setCount(count + 1)}>Click me</button>
		</div>
	);
}
```

class에서만 사용 가능하던 state를 위와 같이 사용이 가능하다.

state변수와 설정을 위한 함수가 2개 생성된다.

### useEffect

---

class 컴포넌트에서 사용하던

- componentDidMount
- componentDidUpdate
- componentWillUnmount

를 함수형 컴포넌트에서 useEffect 하나로 대체하여 사용 가능하다.

```jsx
useEffect(() => {
	function handleStatusChange(status) {
		setIsOnline(status.isOnline);
	}

	ChatAPI.subscribeToFriendStatus(props.friend.id, handleStatusChange);
});
```

기본적으로 useEffect는 콜백함수를 등록해 랜더링 이후에 호출된다.

그러다보니 componentDidMount와 componentDidUpdate를 하나의 함수로 해결이 가능하다.

또한, 매번 랜더링시 새로운 함수를 만들어 useEffect에 등록해준다.

> 이것은 버그를 줄이기 위함이다.
> 변경된 사항을 새로 적용하여 useEffect에 등록한다.

또 만약 종료시에 clean-up 처리가 필요하다면 useEffect에 넣어주는 콜백 함수의 return에 등록해주면 된다.

```jsx
useEffect(() => {
	function handleStatusChange(status) {
		setIsOnline(status.isOnline);
	}

	ChatAPI.subscribeToFriendStatus(props.friend.id, handleStatusChange);
	return () => {
		ChatAPI.unsubscribeFromFriendStatus(props.friend.id, handleStatusChange);
	};
});
```

return으로 등록된 함수는 컴포넌트 해지시에 호출된다.

componentWillUnmount를 대체할 수 있다.

마지막으로 최적화 방법이다.

매번 새로 등록되는 useEffect를 최적화 하려면

```jsx
useEffect(() => {
	function handleStatusChange(status) {
		setIsOnline(status.isOnline);
	}

	ChatAPI.subscribeToFriendStatus(props.friend.id, handleStatusChange);
	return () => {
		ChatAPI.unsubscribeFromFriendStatus(props.friend.id, handleStatusChange);
	};
}, [props.friend.id]); // props.friend.id가 바뀔 때만
```

위와 같이 변경을 해주어야 하는 변수를 같이 넣어주면 그 값이 바뀌었을 때만 새로 등록이 된다.
