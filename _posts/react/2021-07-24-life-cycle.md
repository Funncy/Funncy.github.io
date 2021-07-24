---
layout: post
title: '[React] - LifeCycle API '
subtitle: 'react -  lifecycle api'
categories: react
tags: react lifecycle
comments: true
---

## 컴포넌트 초기 생성

---

### 1. constructor

```tsx
constructor(props) {
	//새로 만들어질때 호출
  super(props);
}
```

### 2. componentDidMount

```tsx
componentDidMount() {
  // 외부 라이브러리 연동: D3, masonry, etc
  // 컴포넌트에서 필요한 데이터 요청: Ajax, GraphQL, etc
  // DOM 에 관련된 작업: 스크롤 설정, 크기 읽어오기 등
}
```

## 컴포넌트 업데이트

---

### 1. getDerivedStateFromProps

```tsx
static getDerivedStateFromProps(nextProps, prevState) {
  // 여기서는 setState 를 하는 것이 아니라
  // 특정 props 가 바뀔 때 설정하고 설정하고 싶은 state 값을 리턴하는 형태로
  // 사용됩니다.
  /*
  if (nextProps.value !== prevState.value) {
    return { value: nextProps.value };
  }
  return null; // null 을 리턴하면 따로 업데이트 할 것은 없다라는 의미
  */
}
```

### 2. shouldComponentUpdate

```tsx
shouldComponentUpdate(nextProps, nextState) {
  // return false 하면 업데이트를 안함
  // return this.props.checked !== nextProps.checked
  return true;
}
```

- 아무리 Virtual DOM이라고 해도 쓸데없이 reRendering되는 것을 막아주기 위한 함수

### 3. getSnapshotBeforeUpdate

```tsx
getSnapshotBeforeUpdate(prevProps, prevState) {
    // DOM 업데이트가 일어나기 직전의 시점입니다.
    // 새 데이터가 상단에 추가되어도 스크롤바를 유지해보겠습니다.
    // scrollHeight 는 전 후를 비교해서 스크롤 위치를 설정하기 위함이고,
    // scrollTop 은, 이 기능이 크롬에 이미 구현이 되어있는데,
    // 이미 구현이 되어있다면 처리하지 않도록 하기 위함입니다.
    if (prevState.array !== this.state.array) {
      const {
        scrollTop, scrollHeight
      } = this.list;

      // 여기서 반환 하는 값은 componentDidMount 에서 snapshot 값으로 받아올 수 있습니다.
      return {
        scrollTop, scrollHeight
      };
    }
  }

  componentDidUpdate(prevProps, prevState, snapshot) {
    if (snapshot) {
      const { scrollTop } = this.list;
      if (scrollTop !== snapshot.scrollTop) return; // 기능이 이미 구현되어있다면 처리하지 않습니다.
      const diff = this.list.scrollHeight - snapshot.scrollHeight;
      this.list.scrollTop += diff;
    }
  }
```

1. render()
2. getSnapshotBeforeUpdate
3. 실제 DOM 변경
4. componentDidUpdate

DOM 변경 전의 상태를 componentDidUpdate에 넘겨주는 역할을 한다.

### 4. componentDidUpdate

```tsx
componentDidUpdate(prevProps, prevState, snapshot) {

}
```

- 이 함수는 render() 이후에 호출 된다.
- state와 props가 바뀐 상태이다. 그러므로 이전 상태를 확인 할 수 있다.

## 컴포넌트 제거

---

### 1. componentWillUnmount

```tsx
componentWillUnmount() {
  // 이벤트, setTimeout, 외부 라이브러리 인스턴스 제거
}
```

## 컴포넌트 에러 체크

---

### 1. componentDidCatch

```tsx
componentDidCatch(error, info) {
  this.setState({
    error: true
  });
}
```

이 함수의 특징은 자신의 render() 함수에서 에러는 못잡지만

자식 컴포넌트의 error는 잡을 수 있다.

에러를 확인하여 분기 가능하다.
