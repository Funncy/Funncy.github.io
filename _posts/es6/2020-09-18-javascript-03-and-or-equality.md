---
layout: post
title: "[Javascript] -  기초 03. AND, OR performance tip , eqaulity  "
subtitle: "javascript and or equaility "
categories: web
tags: javascript
comments: true
---

## 1. & |

---

우리가 연산을 하다보면 and(&) 와 or(|)을 사용하게 될 때가 많다. 여기서 우리가 알아두면 좋은 점은

- & : 왼쪽부터 하나라도 false가 나오면 뒤에는 보지 않는다. (어차피 false이기 때문에)
- | : 왼쪽부터 하나라도 true가 나오면 뒤에는 보지 않는다. (어차피 true이기 때문에)

### 즉, 되도록 간단한 변수 부터 그리고 연산, 마지막으로 함수 실행 순으로 배치해라

```jsx
var value1 = false;
var value2 = 4 < 2;

function value3() {
  /// wasting time...
  return true;
}

console.log(`or: ${value1 | value2 | value3()}`);
```

## 2. equal

---

javascript에는 loose equal 과 strict equal이 있다.

loose equal은 자동으로 형변환을 해서 equal을 연산하기 때문에

문자와 숫자 '5' == 5가 true가 나온다.

하지만 strict equal로 진행하면 '5' === 5 는 타입이 다르기 때문에 false가 나온다.

```jsx
const stringFive = "5";
const numberFive = 5;

console.log(stringFive == numberFive); //true
console.log(stringFive === numberFive); //false

//object equality by reference
const ellie1 = { name: "ellie" };
const ellie2 = { name: "ellie" };
const ellie3 = ellie1;
console.log(ellie1 == ellie2); //false
console.log(ellie1 === ellie2); //false
console.log(ellie1 === ellie3); //true
```
