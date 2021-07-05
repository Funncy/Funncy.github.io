---
layout: post
title: '[Javascript] -  Typescript '
subtitle: 'javascript typescript'
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

## 1. 기본 타입

- number (숫자, 소수도 가능)
- string
- boolean
- undefined (데이터가 있는지 없는지 모름, null과 다른 느낌, 보편적으로는 null보다 많이 쓰임)
- null (데이터 없음)

```jsx
const age : number = 8;
const name : string = 'hyojun';
const isStudent : boolean = false;

const name: undefined; //💩
const age: number | undefined; //number 혹은 undefined
age = undefined;
age = 1;

const person: null; //💩
const person2: string | null;
```

> 기본적으로 TypeScript도 nullSafety라고 보면 된다.
> string으로 타입을 선언한 곳에는 undefined, null은 들어갈 수 없다.

## 2. 추가 타입

- unknown (알 수 없음, 어떤 데이터 타입도 들어가진다 , 되도록 쓰지 말자 💩)
- any (모든 것, dart의 dynamic 같은 느낌? 💩)
- void (우리가 아는 기본적인 void 형태이다, 변수에서는 쓸 일이 없다 . undefined만 들어감 💩)
- never (절대 반환하지 않음을 뜻한다. Exception이나 무한루프 상태의 함수 타입이다)
- object (어떠한 Object든 모두 가능하다. 💩)

```jsx
const age: unknown = 0; //💩
age = 'he';
age = true;

const name: any = 0; //💩
name = 'true';

//void
function print(): void {
	console.log('hello');
	return;
}

//never
function throwError(): never {
	throw new Error(); //1

	while (true) {
		//2
	}
}
let neverEnding: never; //💩

let obj: Object; //💩 이렇게 사용하지말고 Object의 구체적인 타입을 명시해주자.
obj = {
	name: 'test',
};
obj = {
	animal: 'dog',
};
```

## 3. optional

```tsx
function printName(firstName: String, lastName?: string) {
	console.log(firstName);
	console.log(lastName);
}

printName('hello', 'world');
printName('one');
```

? 기호를 사용해서 변수가 들어올수도 아닐수도 선택적으로 사용가능하게 바꿀 수 있다.

```tsx
function test(lastName?: string); //이렇게도 사용하지만
function test2(lastName: string | undefined); //이렇게도 사용가능하다
test(); //그렇지만 optional이 아닌 |로해주면 무조건 인자를 넣어주어야 한다.
test2(undefined);
```

## 4. default

```tsx
function defaultFun(message: string = 'default message') {
	console.log(message); //값이 들어오지 않을 경우 기본 값을 지정 할 수 있다.
}

defaultFun();
```

## 5. spread

```tsx
//갯수에 상관없이 동일한 parameter를 여러개 받아보고 싶을때
function addNumbers(...numbers: number[]): number {}

addNumbers(1, 2);
addNumbers(1, 2, 3, 4, 5); //이렇게 몇개든 담으면 array로 들어간다.
```

여기서 Typescript의 장점이 나온다.

1. Type Check

   기존 처럼 타입 없이 변수가 할당 가능하나. 뒤에 다른 타입을 넣으려고 하면 Complier에서 Error를 띄워준다.

   : string 과 같이 타입을 정해주면 Type이 설정되어 해당 타입을 따라야 한다.

   > typeof() 와 같은 연산을 할 필요가 없다.

2. parameter 갯수 확인

   기존 Javascript는 Parameter 갯수 체크를 해주지 않는다.

## 6. Array vs Tuple

```tsx
const fruits: string[] = ['apple', 'banana'];
const fruits2: Array<string> = ['apple', 'banana']; //Array로 선언도 가능
function printArray(fruits: readonly string[]) {} //값을 변경 할 수없음

//Tuple => interface, type alias, class로 대체해서 사용하자
// 굳이 사용 할 이유가 없음 , 가독성 떨어짐
let student: [string, number];
student = ['name', 123];
student[0]; //name
student[1]; //123 => 가독성 떨어짐
let [name, age] = student; //이렇게도 사용 가능하지만 그냥 다른 방식 사용 하는게 좋음
```

## 7. Type Alias

C언어의 typedef 처럼 내가 type을 선언해서 사용할 수 있다.

```tsx
type Text = string;
const name: Text = 'name';

//object를 type으로 선언할 수 있다.
type Student = {
	name: string;
	age: number;
};
const student: Student = {
	name: 'hyojun',
	age: 29,
}; //위에서 정의한 타입에 맞춰서 구조를 만들어줘야 한다.

//이렇게 값 자체를 타입으로 지정 할 수 있다.
type Name = 'name';
const name: Name = 'name'; //'name'말고 다른 값을 넣을 수 없다 => union과 조합해서 사용해보자.
```

## 8. Union

```tsx
type Direction = 'left' | 'right' | 'up' | 'down';

function move(direction: Direction) {}

move('left'); //자동완성 기능도 지원해준다. vscode
```

서버 통신 예제:

```tsx
type SuccessState = {
	response: {
		body: string;
	};
};
type FailState = {
	reason: string;
};
type LoginState = SuccessState | FailState;
function login(): LoginState {
	return {
		response: {
			body: 'logged in!',
		},
	};
}
```

하지만 저렇게 Union 타입의 데이터가 들어간다면 내부에서는 어느 형태의 타입인지 알아보기 힘들 수 있다.

그 부분은 Discriminated Union으로 해결해보자.

## 9. Discriminated Union

공통된 내부 변수를 접근해서 분기를 시켜주는 방식이다.

```tsx
type SuccessState = {
	result: 'success';
	response: {
		body: string;
	};
};
type FailState = {
	result: 'fail';
	reason: string;
};
type LoginState = SuccessState | FailState;

function printLoginState(state: LoginState) {
	if(state.result === 'success') { //두 Type의 result는 공통 사항임으로 이렇게 접근이 가능하다.
		console.log(`${state.response.body});
	} else {
		console.log(`${state.reason});
	}
}
```

## 10. Intersection Type

Discriminated Union이 OR과 같이 공통사항으로 접근했다면 Intersection은 AND와 같은 방식이다.

```tsx
type Human = {
	name: string;
	score: number;
};
type Worker = {
	employeeId: number;
	work: () => void;
};

function doWork(person: Student & Worker) {
	//& 연산한 타입의 데이터가 모두 존재하는 Object가 들어와야 한다.
	console.log(person.name, person.employeedId, person.work());
}

doWork({
	name: 'hyojun',
	score: 100,
	employeeId: 10,
	work: () => {},
};
```

## 11. Enum

다른 언어와 다르게 Javascript에서는 enum을 지원해주지 않는다. typescript에서는 구현을 해 놓았지만 다른 방식을 사용하는것을 추천한다.

```tsx
//JavaScript
const DAYS_ENUM = Object.freeze({ MONDAY: 0, TUESDAY: 1 });
//freeze로 선언하면 수정이 불가능하다.

//TypeScript
enum Days {
	Monday,
	Tuesday,
}
console.log(Days.Monday);
const day: Days = Days.Monday;
day = 10; //complie error가 발생하지 않는다. 숫자면 다 들어가진다.
//그래서 다른 방식으로 사용하는걸 추천한다.
type DaysOfWeek = 'Monday' | 'Tuesday' | 'Wednesday'; //이런방식으로 사용하는걸 추천한다.
```

## 13. Interface

---

```jsx
const person = {
	name: 'hyojun',
	age: 28,
	gender: 'male',
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
	name: 'hyojun',
	age: 28,
	gender: 'male',
};

const sayHi = (person: Human) => {
	console.log(`Hello ${person.name} ${person.age} ${person.gender}`);
};

sayHi(person);
```

위와 같이 interface로 정의해놓고 사용하면 미리 complier가 error를 체크해준다.

> 참고로 interface는 javascript로 transcomplie되지 않는다.
> transcomplie을 위해서라면 class를 사용하자.

## 14. class

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
