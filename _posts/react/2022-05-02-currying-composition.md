---
layout: post
title: '[React] - Currying & Composition '
subtitle: 'react -  Currying & Composition'
categories: react
tags: react currying composition
comments: true
---


함수형 프로그래밍의 백미

# Currying

---

음식점에 주문하는 코드를 짠다고 가정해보자.

1. 음식점을 선택하고
2. 그곳에서 메뉴를 주문하자.

```jsx
const order = store => menu => console.log(`${store} - ${menu}`);
```

함수가 함수를 리턴한다.

```jsx
order('중국집')('짜장면'); // 중국집 - 짜장면
```

ES5로 작성해보자.

```jsx
function order(store) {
	return function(menu) {
		console.log(`${store} - ${menu}`); //클로저를 통해서 store를 접근할 수 있다.
	} 
}
```

코드를 읽을때 밖에서부터 읽고 속에서부터 바깥으로 나와야한다.

```jsx
order('중국집')('짜장면');
order('중국집')('짬뽕');
//중복?

const orderCh = order('중국집');
orderCh('짜장면');
orderCh('짬뽕'); //이렇게 활용가능
```

# Composition

---

함수가 f1, f2, f3 ...가 있다고 쳐보자.

f1 : 풀 네임을 가져오는 함수

f2 : 주소를 가져오는 함수

f3 : 이름을 지우는 함수

f1,f2,f3를 조합함에 따라 데이터를 가져올 수 있음

```jsx
//보통
function userResult(~) {
//f1,f2,f3를 조합해서 짬
//명령형

//상황에 따라 f2를 안부른다? 
// 점점 if문 추가...
}
```

```jsx
const data = componse(
	fullName //f1
	appendAddr, //f2 , f2를 안쓰는 곳에서는 빼고 호출하면 된다.
	removeNames,//f3
)(u);
```

```jsx
const compose = (...fns) => (obj) =>
	fns.reduce((c, fn) => fn(c), obj);
//reduce를 활용해서 composition 활용

//es5로 표현해보자
const fullName = (name) => ({
	...user,
	fullName: `${user.fullName}`
});

const appedAddr = (user) => ({...user, add: '~~~'});

const removeNames = (user) => {
	delete user.fullName,
	return user; 
};

```

위의 Recude를 활용한 compose 함수를 ES5로 바꿔보자

```jsx
function componse() {
	var fns = arguments;
	return function rfn(obj, i) {
			i = i || 0;
			if( i < fns.length)
				return rfn(fns[i](obj), i+1);
			return obj;
	}
}
```