---
layout: post
title: '[Flutter] - Life Cycle 생명주기 '
subtitle: 'Flutter life cycle'
categories: flutter
tags: flutter lifecycle
comments: true
---

# 생명주기

---

![이미지](https://Funncy.github.io/assets/img/flutter/2021-03-09-life-cycle-01.png 'flutter life cycle')

출처 : [https://www.youtube.com/watch?v=8nZ7kRPsHSA](https://www.youtube.com/watch?v=8nZ7kRPsHSA)

## Stateless Widget

---

상태가 없는 위젯으로 생성자 이후에 바로 build함수가 호출된다.

> Build() 함수 밖에서는 BuildContext를 접근 할 수 없다.

## Stateful Widget

---

State가 있는 위젯으로 생성자 이후에 createState()를 통해 Stater가 생성된다.

### initState

---

State초기 함수로 초기 셋팅을 위한 함수이다.

생성시에 1번 호출되며 rebuild를 하여도 호출되지 않는다. (새로고침 해주자)

> Async await가 불가능하며
> BuildContext에 접근이 불가능하다.

### didChangeDependencies

---

initState와 똑같이 생성시 초기 1번만 호출된다.

차이점은 BuildContext에 접근이 가능하다.

### setState

---

rebuild를 요청하는 함수이다.
state의 값을 변경 할 수있다.

### didUpdateWidget

---

부모 Widget에서 rebuild를 하여 현재 Widget이 다시 그려질때

과거의 Widget과 현재 Widget을 비교하는 함수이다.
State값을 비교하여 rebuild를 막거나 실행 시킬 수 있다.

### dispose

---

initState와 쌍으로 쓰이며 Controller, Stream 등 메모리 누수가 발생 할 수 있는 것들을
해제해주는 영역이다.

화면이 종료될때 호출된다.
