---
layout: post
title: "[Javascript] -  Typescript "
subtitle: "javascript typescript"
categories: web
tags: javascript
comments: true
---

React를 본격적으로 공부해보기 전에 Typescript를 먼저 정리해보았다.

### 0. 초기 설정

---

```jsx
npm init
```

- 먼저 npm init를 통해 프로젝트를 생성해준다.

그리고 `tsconfig.json` 을 만들어 아래와 같이 설정해주자.

```json
{
  "compilerOptions": {
    "module": "commonjs",
    "target": "ES2015",
    "sourceMap": true,
    "outDir": "dist"
  },
  "include": ["src/**/*"],
  "exclude": ["node_modules"]
}
```

- target : 우리가 컴파일하고자 하는 JS 버전
- sourceMap : typescript와 javascript를 연결해 디버깅을 하도록 도와주는 옵션
- outDir : js 생성 디렉토리
- include : ts 파일 디렉토리
- exclude : ts 컴파일 제외 디렉토리 (node_modules는 외부 라이브러리)

ts 컴파일을 위해 Package.json을 설정해주자.

```json
{
  ...
  "scripts": {
    "start": " node dist/index.js ",
		"prestart" : " tsc"
  },
  ...
}
```

그리고 간단한 JS파일을 작성해서 npm start를 해주면 정상적으로 파일이 생성된다.

### 1. Type

---

```tsx
const name: string = "hyojun",
  age: number = 28,
  gender: string = "male";

const sayHi = (name, age, gender) => {
  console.log(`Hello ${name} ${age} ${gender}`);
};

sayHi(name, age, gender);

export {};
```

간단하게 이름 나이 성별을 출력하는 프로그램을 작성하였다.

여기서 Typescript의 장점이 나온다.

1. Type Check

   기존 처럼 타입 없이 변수가 할당 가능하나. 뒤에 다른 타입을 넣으려고 하면 Complier에서 Error를 띄워준다.

   : string 과 같이 타입을 정해주면 Type이 설정되어 해당 타입을 따라야 한다.

   > typeof() 와 같은 연산을 할 필요가 없다.

2. parameter 갯수 확인

   기존 Javascript는 Parameter 갯수 체크를 해주지 않는다.

### 2. Interface

---

```jsx
const person = {
  name: "hyojun",
  age: 28,
  gender: "male",
};

const sayHi = (name: string, age: number, gender: string) => {
  console.log(`Hello ${name} ${age} ${gender}`);
};

// 그냥 person을 넣을 수 없을까?
sayHi(person.name, person.age, person.gender);
```

우와 같은 내용을 interface를 통해 정리할 수 있다.

```jsx
interface Human {
  name: string;
  age: number;
  gender: string;
}

const person = {
  name: "hyojun",
  age: 28,
  gender: "male",
};

const sayHi = (person: Human) => {
  console.log(`Hello ${person.name} ${person.age} ${person.gender}`);
};

sayHi(person);
```

위와 같이 interface로 정의해놓고 사용하면 미리 complier가 error를 체크해준다.

> 참고로 interface는 javascript로 transcomplie되지 않는다.
> transcomplie을 위해서라면 class를 사용하자.

### 3. class

---

javascript에서의 class는 property를 선언해주지 않아도 사용이 가능했다.

하지만 typescript 에서는 다르다.

- property를 선언해주어야 한다.
- private, protected, public을 선언해줄 수 있다. (javascript 에서는 적용이 되지 않고 complier 용이다.)

```jsx
class Human {
  public name: string;
  public age: number;
  public gender: string;

	// ?을 통해 Null을 허용해 줄 수 있다.
  constructor(name: string, age: number, gender?: string) {
    this.name = name;
    this.age = age;
    this.gender = gender;
  }
}

const hyojun = new Human('hyojun', 28);

const sayHi = (person: Human) => {
  console.log(`Hello ${person.name} ${person.age} ${person.gender}`);
};

sayHi(hyojun);

export {};
```
