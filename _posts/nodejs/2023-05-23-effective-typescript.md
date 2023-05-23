---
layout: post
title: '[Effective Typescript] 주관적 Summary'
subtitle: 'effective typescript'
categories: nodejs
tags: typescript effective
comments: true
---


# Effective Typescript : Summary

### 아이템 3 : 코드 생성과 타입이 관계없음을 이해하기

---

타입스크립트 컴파일러는 2가지 역할을 합니다.

- 구버전의 자바스크립트로 트랜스파일
- 코드의 타입 오류를 체크

위 2가지는 완벽히 독립적입니다. 오류가 있어도 자바스크립트가 문제가 없다면 컴파일이 가능합니다.

<aside>
💡 만약 오류가 있을 때 컴파일 하지 않으려면 tsconfig.json에 noEmitOnError 설정을 하면 됩니다.

</aside>

런타임에는 타입 체크가 불가능합니다. (ex. instanceof 는 런타임 체크)

타입스크립트는 해당 이슈에 대해서 ‘태그’ 기법을 사용합니다.

```jsx
interface Square {
	kind : 'square';
	width : number;
}

interface Rectangle {
	kind : 'rectangle',
	width : number;
	height: number;
}

type Shape = Sqaure | Rectangle;
// kind가 union으로 타입 체크 가능
```

혹은 class를 활용해서 instanceof를 사용할수도 있습니다.

<aside>
💡 여기서 type Shape = Square | Rectangle 은 type으로 참조되지만
instanceof Rectangle 은 값으로 참조됩니다. ⇒ 구분이 중요하다고 합니다. (아이템8에서 계속 이어짐)

</aside>

타입 연산은 런타임에 영향을 주지 않습니다. ( as 키워드는 타입 연산으로 런타임에는 영향을 주지 않습니다.)

런타임의 타입은 선언된 타입과 다를 수 있습니다.

대표적으로 서버통신을 통해 들어온 데이터는 해당 타입이 아닐 수 있습니다.

타입 스크립트는 함수 오버로딩이 불가능합니다.

타입 수준에서만 가능하지만 주의해서 사용해야 합니다. (아이템 50에서)

타입스크립트의 타입은 런타임 성능에 영향을 주지 않습니다.

빌드타임 오버헤드는 있습니다. 

어떤 경우든지 호환성과 성능 사이의 선택은 컴파일 타깃과 언어 레벨의 문제이지 타입과는 무관합니다.

요약

- 코드 생성과 타입 시스템은 무관하다.
- 타입 체커는 성능에 영향을 주지 않는다.
- 타입 오류가 있더라도 컴파일은 가능하다.
- 타입스크립트의 타입은 런타임에 사용이 불가능하다. 런타임에 타입 정보를 유지하려면 일반적으로는 태그된 유니온 과 속성 체크 방법을 사용한다. 혹은 클래스를 활용한다.

### 아이템4 : **구조적 타이핑**에 익숙해지기

---

자바스크립트는 본질적으로 덕 타이핑 언어 기반입니다. 타입스크립트는 이런 동작을 그대로 모델링합니다. 

<aside>
💡 덕 타이핑 이란
객체가 어떤 타입에 부합하는 변수와 메서드를 가질 경우 객체를 해당 타입에 속하는 것으로 간주

</aside>

```jsx
interface Vertor2D {
	x: number;
	y: number;
}

function calculateLength(v: Vector2D) {
	return Math.sqrt(v.x * v.x + v.y * v.y);
}

const v1 : Vector2D = {x:3 , y:4};
calculateLength(v1); // 정상 동작

const v2 = {x:5, y:3 , z:1};
caculateLength(v2); // 정상 동작 = 구조적 타이핑
```

위와 같은 동작으로 인해 타입체커가 문제를 인식하지 않는 경우도 있습니다. (따로 설정을 할 수 있기는 합니다. 아이템 37)

좋든 싫든 타입은 열려 있습니다.

```jsx
function calculateLengthL1(v : Vector3D) {
	let length = 0;
	for(const axis of Object.keys(v)) {
		const coord = v[axis]; //오류 발생 
			//'string'은 'Vector3D'의 인덱스로 사용할 수 없기에 엘리먼트는 암시적으로 'any'타입 입니다.
		
		length += Math.abs(coord);
	}
	return length;
}
```

우리는 위 코드를 보고 Vector3D가 x,y,z 속성만 가진다고 생각 할 수 있습니다 (봉인된 타입)

하지만 **타입스크립트는 열린 타입으로 다른 데이터가 들어 올 수 있고**, 그러므로 위 오류는 올바른 오류입니다.

위와 같은 경우는 루프보다 모든 속성을 직접 구현하는 것이 더 낫고 위 주제는 아이템 54에서 다시 다룹니다.

클래스와 관련된 할당문에서도 구조적 타이핑이 동작합니다.

```jsx
class C { 
	foo : string;
	constructor(foo: string) {
		this.foo = foo;
	}
}

const c = new C('instance of C');
const d: C = {foo : 'object'}; //정상
```

구조적 타이핑을 활용하여 테스트 코드를 쉽게 작성 할 수 있습니다.

```jsx
function getAuthors(database: PostrgresDB) : Author[] {
	const autorRows = database.runQuery('~~~~');
	...
}
```

위 코드에서 PostgresDB를 구조적 타이핑을 활용하여 수정해보겠습니다.

```jsx
interface DB {
	runQuery : (sql: string) => any[];
}

function getAuthors(database: DB) : Author[] {
	const autorRows = database.runQuery('~~~~');
	...
}
```

구조적 타이핑 덕분에, postgresDB가 DB 인터페이스를 구현하는지 명확히 선언할 필요가 없습니다.

```jsx
	
test('getAuthros', () => {
	const authors = getAuthros({
		runQuery(sql: string) {
			return [['test', 'data'] , ['test2' , 'data2']];
		}
	});
	expect(authors).toEqual([
		{first: 'test', last:'data'},
		{first: 'test2', last: 'data2'}
	]);
});
```

요약

- 자바스크립트는 덕 타이핑 기반이고 타입 스크립트는 이를 모델링 하기 위해 구조적 타이핑을 사용함을 이해해야 한다.
- 타입은 봉인되어 있지 않고 열려 있다.
- 클래스 역시 구조적 타이핑 규칙을 따른 다는 것을 명심해야 한다.
- 구조적 타이핑을 활용하면 유닛 테스트를 손쉽게 할 수 있다.

### 아이템 7 : **타입이 값들의 집합**이라고 생각하기

---

코드가 실행되기 전, 타입스크립트가 오류를 체크하는 순간에 ‘타입’을 가지고 있습니다.

타입은 ‘할당 가능한 값들의 집합’이라고 생각하면 됩니다.

가장 작은 집합은 아무 값도 포함하지 않는 **공집합이며 never 타입**입니다.

그 다음으로 작은 집합은 한가지 값만 포함하는 리터럴 타입입니다. (유닛 타입)

2개 3개를 묶으려면 유니온 타입을 사용합니다.

```jsx
type A = 'A';
type B = 'B';
type Twelve = 12;

type AB = 'A' | 'B';
type AB12 = 'A' | 'B' | 12;

const a:AB = 'A';
const c:Ab = 'C'; //error 'C' 형식은 'AB' 형식에 할당 할 수 없습니다.
```

**& 연산자는 두 타입의 인터섹션(교집합)**을 계산합니다.

```jsx
	interface Person {
    name: string;
  }

  interface Lifespan {
    birth: string;
    death: string;
  }
  type Personspan = Person & Lifespan;
  type Personspan2 = Person | Lifespan;
  const person1: Personspan = {
    name : 'test',
    birth: '123',
    death: 'test',
  };
  const person2: Personspan2 = {
    // name: 'test',
    birth: '123', //Person 혹은 Lifespan 중 모두 혹은 하나에는 속해야 한다.
    death: 'test',
  };

  type K = keyof (Person | Lifespan); // never
  type K2 = keyof (Person & Lifespan); // "name" | keyof Lifespan
  type K3 = keyof Person | keyof Lifespan; // "name" | keyof Lifespan
  type K4 = keyof Person & keyof Lifespan; // never 
  type KPerson1 = keyof Personspan; // "name" | keyof Lifespan
  type KPerson2 = keyof Personspan2; // never
```

위와 같은 결과가 나오는 이유는 위 연산자들은 인터페이스의 속성이 아닌, 값의 집합(타입의 범위) 에 적용되기 때문입니다.

```jsx
keyof (A&B) = (keyof A) | (keyof B);
keyof (A|B) = (keyof A) & (keyof B); 
```

extends 키워드를 사용해도 됩니다. **집합이라는 관점에서 Extends는 ‘~의 부분 집합’이라는 의미**로 받아들 일 수 있습니다.

extends는 제네릭 타입에서 한정자로도 쓰일 수 있습니다.

```jsx
function getKey<K extends string>(val: any, key: K) { ... }

getKey({}, 'x'); 
getKey({}, document.title);
getKey({}, 12); //'12'형식의 인수는 'string'형식의 매개변수에 할당될 수 없습니다.
```

배열과 튜플의 관계 역시 명확해집니다.

```jsx
const list = [1,2]; // type : number[]
const tuple : [number, number] = list ; //number[] 타입은 [number,number] 타입의 0,1 속성에 없습니다.

const triple : [number, number, number] = [1,2,3];
const double : [number, number] = triple; // Type '[number, number, number]' is not assignable to type '[number, number]'.
																				  // Source has 3 element(s) but target allows only 2.
```

구조적타이핑으로 triple이 double에 들어갈 수 있을것 같지만 오류가 발생합니다.

여기서 타입스크립트는 숫자의 쌍을 { 0 : number, 1: number}가 아니라 {0: number, 1: number, length : 2} 로 모델링했습니다. 

쌍에서 길이체크는 합리적이며 이보다 나은 방법은 없습니다.

요약:

- 타입을 값의 집합으로 생각하자
- 타입스크립트의 타입은 엄격한 상속 관계가 아니라 겹쳐지는 집합으로 표현된다.
- 한 객체의 추가적인 속성이 타입 선언에 언급되지 않더라도 그 타입에 속할 수 있습니다. (구조적 타이핑)
- 객체 타입에서 &는 A와 B의 모든 속성을 가짐을 의미합니다. |는 둘중 하나의 속성을 가짐을 의미합니다. (모두다 가질수도 있음)

### 아이템 8 : 타입 공간과 값 공간의 심벌 구분하기

---

타입 스크립트의 심벌은 타입 공간과 값 공간 중 한곳에 존재할 수 있습니다. 

그래서 타입스크립트는 코드를 읽을 때 타입인지 값인지 구분하는 것이 중요합니다.

그리고, 연산자 중에서도 타입에서 쓰일때랑 값에서 쓰일때랑 다른 기능을 하기도 합니다.

```jsx
type T1 = typeof p; // type Person

const v1 = typeof p; // "object" (typeof가 런타임에서 실행)
```

모든 값은 타입을 가지지만, 타입은 값을 가지지 않습니다. **Type, interface는 타입공간에만 존재**합니다.

class, Enum은 타입과 값 두가지 모두 사용될 수 있습니다. 

```jsx
class Cylinder {
	radius = 1;
	...
}

function calculateVolume(shape: unknown) {
	if(shape istanceof Cylinder){
		shape //정상, 타입 Cylinder
		shape.radius //정상, 타입 number
	}
}

const v = typeof Cylinder; // "function"
type T = typeof Cylinder; // type Cylinder

declare let fn: T;
const c = new fn(); // type Cylinder
```

속성 접근자 []는 타입으로 쓰일 수 있다.

obj.field와 obj[’field’]는 값은 같을 수 있으나 타입은 다를 수 있다. 속성 접근자로 타입을 선언하려면 무조건 후자를 선택하자.

```jsx
type PersonEl = Person['first' | 'last'] ;//타입은 string
type Tuple = [string, number, Date];
type TupleEl = Tuple[number]; // 타입은 string | number | Date
```

### 아이템 11 : 잉여 속성 체크의 한계 인지하기

---

```jsx
interface Room {
    numDoors : number;
}

const r : Room = {
    numDoors : 4,
    elephant : 'present' // error : 객체 리터럴은 알려진 속성만 지정할 수 있으며...
}
```

위와 같이 타입이 명시된 변수에 **‘그외 속성이 없는지’를 체크하는게 잉여 속성체크하기**이다.

하지만, **구조적 타이핑 관점에서 보면 오류가 발생하지 않아야한다.**

아래 처럼 **임시변수를 도입하면 에러가 사라집니다.**

> 주의해서 임시변수를 사용하자. 타입스크립트는 런타임 예외 케이스를 찾는 것도 먹적이지만, **의도와 다르게 작성된 코드까지 찾으려고 한다**. 오타 등등 (구조적 타이핑에서는 오타 문제가 발생하지 않음)
> 

```jsx
interface Room {
    numDoors : number;
}

const temp = {
    numDoors : 4,
    elephant : 'present' 
}

const r : Room = temp; //에러 발생 아함. 구조적 타이핑으로 들어감 
```

**객체 리터럴을 할당 할 때, 혹은 함수 파라미터로 객체를 넣을때에만 ‘잉여 속성 체크하기’가 실행된다.**

타입 단언문에서는 잉여 속성 체크하기가 실행되지 않음 

```jsx
interface LOPtion1 {
    test2? : string;
    logScale? : boolean;
    areaChart? : boolean;
}

interface LOPtion2 {
    test1 : string;
    test2? : string;
    logScale? : boolean;
    areaChart? : boolean;
}

const opt = {area: 'test'}; 
const opt2 = {test2 : 'test' , area : 'test'};
const o : LOPtion1 = opt; //error : 공통 속성 체크
const o2 : LOPtion1 = opt2;

const opt3 = {test1 : 'test', logscale: true};
const o3 : LOPtion2 = opt3;
```

위 처럼 **모든 속성이 선택**인 경우에는 **공통된 속성이 있는지 체크**하는 별도의 체크를 수행한다.
공통 속성 체크는 임시 변수 할당 후 객체를 넣어도 문제가 발생한다.

단, **공통된 속성이 아예 없거나 (빈 객체) 하나라도 있어야 에러가 발생하지 않는다.** (LOption2는 선택 말고 필수 옵션이 있기때문에 문제가 발생하지 않는다.)

### 아이템 13 : 타입과 인터페이스의 차이점 알기

---

대부분의 경우는 타입을 사용해도 되고, 인터페이스를 사용해도 된다. **일관성을 유지하자.**

타입과 별칭 모두 제너릭이 가능하고, 인터페이스는 타입을, 타입은 인터페이스를 확장 할 수 있다.

다른 점은 다음과 같다

- **인터페이스는 유니온이라는 개념이 없다.**
- **인터페이스는 보강이 가능하다.** 선언적 병합이 된다.
    
    ```jsx
    interface IState {
    	name : string;
    }
    interface IState {
    	age : number;
    }
    
    const man : IState = {
    		name:'Bob',
    		age:20
    };
    ```
    
    프로퍼티가 추가되는 것을 원하지 않는다면 Type을 사용하자.
    
    혹은 **복잡한 타입이라면 Type을 쓰자** (유니온, 인터섹션 등)
    

### 아이템 14 : 타입 연산과 제너릭 사용으로 반복 줄이기

---

**타입도 DRY 원칙을 적용**해서 반복하지 말라고 한다.

가장 기본적으로는 Type에 이름을 붙이고 extends를 통해서 Interface를 확장해서 쓰자.

type의 경우 인터섹션 연산자를 쓸 수 도 있음(일반적이지 않다고 함)

Pick : 특정 타입을 꺼낼때 사용

```jsx
type Pick<T,K extends keyof T> = { [k in K] : T[k] };

type TopNavState = Pick<State, 'userId' | 'pageTitle' | 'recentFiles'>;
```

Partial: 부분적으로 꺼낼때 사용

```jsx
type Partial<T, K extends keyof T> = {[k in K]? : T[k]};
```

ReturnType : 함수 반환값을 타입으로 선언할때 사용

```jsx
type UserInfo = ReturnType<typeof getUserInfo>; 
//주의 사항은 <getUserInfo>가 아니라
//<typeof getUserInfo> 라는 점이다. 적용대상이 값인지 타입인지 정확히 알고 , 구분해서 처리하자.
```

제네릭 타입에서 Extends는 한정자로 쓸 수 있다고 위에서도 배웠다. Extends를 확장보다 부분 집합으로 이해하자.

### 아이템 15 : 동적 데이터에 인덱스 시그니처 사용하기

---

타입스크립트에서 객체의 타입을 선언할때 가장 추상적으로는

```jsx
type Rocket = {[property:string] : string};
```

위와 같이 작성 할 수 있다. (인덱스 시그니처)

하지만 **가능하다면 인터페이스 혹은 레코드 등 되도록 정확한 타입을 입력하자.**

```jsx
type Vec3D = Record<'x' | 'y' | 'z' , number>;
// = {
// x: number;
// y: number;
// z: number;
// }

type ABc = {[k in 'a' | 'b' | 'c'] : k extends 'b' ? string : number};
// = {
// a: number;
// b: string;
// c: number;
// }
```

### 아이템 21 : 타입 넓히기

---

타입스크립트는 변수의 값을 가지고 할당 가능한 집합을 유추한다. 이 과정을 ‘넓히기’라고 부른다.

```jsx
interface Vetor3 {x:number; y:number; z:number}
function getComponent(vector: Vector3, axis : 'x' | 'y' | 'z') {...}

let x = 'x';
let vec = {x:10, y:20, z:30};
getComponent(vec, x); //error 변수 x의 타입을 string으로 추론함.

const y ='y';
getComponent(vec,y); //문제 없음, const 덕분에 타입을 'y'로 추론함.

const mixed = ['x', 1]; //객체의 const는 얕은 const라서 내부에서는 타입 넓히기 진행
```

위와 같은 이슈에서 크게 3가지 방법이 있다.

1. 명시적 타입 구문 제공
2. 추가적인 문맥 제공
3. const 단언문 사용
    
    ```jsx
    const a1 = [1,2,3] ; //number[]
    const a2 = [1,2,3] as const; //readonly [1,2,3] , readonly 주의
    ```
    

### 아이템 22 : 타입 좁히기

---

타입스크립트는 분기문, 예외, instanceof의 문맥을 파악해 타입을 자동으로 좁혀준다.

```jsx
const el = document.getElementByID('foo'); //HTMLElement | null
if(el) {
	console.log(el); //HTMLElement
}
```

다른 방식으로는 tagged Union방식이 있다. (type 변수를 추가)

식별을 돕기 위해 커스텀 함수를 도입 할 수 있다.

```jsx
//사용자 정의 타입 가드
function isInputElement(el : HTMLElement) : el is HTMLInputElement {
	return 'value' in el;
}

function getElementContent(el: HTMLElement) {
	if(isInputElement(el) {
		el; //HTMLInputElement
	}
}	
```

filter를 통해 undefined를 가려내려고 해도 잘 동작하지 않는다. 
다음과 같은 타입 가드 함수를 사용 할 수있다.

```tsx
function isDefined<T>(x : T | undefined): x is T {
	return x !== undefined;
}
```

### 아이템 28 : 유효한 상태만 표현하는 타입을 지향하기

---

유효한 상태와 무효한 상태를 둘 다 표현하는 타입은 혼란을 초래하기 쉽다.
유효한 상태만 표현하는 타입을 지향해야 한다. 코드가 길어지더라도 결국 그게 시간을 절약하고 고통을 줄인다.

```tsx
//나쁜 설계
interface State{
	...
	isLoading: boolean;
	error? : string;
}

//좋은 설계
interface RequestPending {
	state: 'pending';
}
interface RequestError {
	state: 'error';
	error: string;
}
interface RequestSuccess {
	state: 'ok';
	pageText: string;
}
type RequestState = RequestRending | RequestError | RequestSuccess;
```

### 아이템 32 : 유니온의 인터페이스보다는 인터페이스의 유니온을 사용하기

---

```tsx
interface Layer {
	layout : FillLayout | LineLayout | PointLayout;
	paint : FillPaint | LinePaint | PointPaint;
} //위의 경우 의도하지 않은 조합으로 버그나 오류를 발생시킬 수 있음 (유니온의 인터페이스)
```

```tsx
interface FillLayer {
	type: 'fill'; // Tagged 유니온은 자주 쓰이는 패턴이며 타입의 범위를 좁히기에도 유용하다
	layout : FillLayout;
	paint : FillPaint;
}

interface LineLayer {
	type: 'line';
	layout : LineLayout;
	paint : LinePaint;
}

interface PointLayer {
	type: 'point';
	layout : PointLayout;
	paint : PointPaint;
}

type Layer = FillLayer | LineLayer | PointLayer;
```

### 아이템 33 : String 타입보다 더 구체적인 타입 사용하기

---

모든 문자열을 할당 할 수 있는 string 타입보다는 더 구체적인 타입을 사용하는게 좋다 ( Ex) ‘공개’ | ‘비공개’)

객체의 속성 이름을 함수 매개변수로 받을 때는 string 보다 keyof T를 사용하는게 좋다

```tsx
type RecordingType = 'studio' | 'live'
interface Album {
  artist: string;
  title: string;
  releaseDate: Date;
  recordingType: RecordingType;
}

const kindOfBlue : Album = {
  artist : 'Miles Davis',
  title:'Kind of Blue',
  releaseDate: new Date('1959-08-17'),
  recordingType:'studio'
}

type K = keyof Album;

function pluck<T>(records: T[], key: keyof T) :T[keyof T][] {
  return records.map(r => r[key]);
}

const releaseDate = pluck([kindOfBlue], 'releaseDate'); 
// 이렇게 되면 반환타입이 (string | Date)[] 가 된다. (T[keyof T]는 너무 넓은 범위다)
// 더 구체적인 타입을 명시하기 위해 아래처럼 하자

function pluck2<T, K extends keyof T>(records: T[], key : K) : T[K][] {
  return records.map(r=>r[key]);
}

const releaseDate2 = pluck2([kindOfBlue] , 'releaseDate');
// Date[]
```

### 아이템 37 : 공식 명칭에는 상표를 붙이기

---

구조적 타이핑 때문에 가끔 이상한 결과를 낼 수 있다. 
이를 막기 위해서는 공식 명칭 (nominal typing)을 사용하면 된다

tagged Union처럼 brand를 값에 붙여서 사용하면 된다 (타입이 아닌 값의 관점에서 해당 타입을 표현한다)

```tsx
type AbolustePath = string & {_brand : 'abs'};

function listAbsolutePath(path: AbsolutePath) { ... }

function isAbsolutePath(path: string) : path is AbsolutePath {
	return path.startWith('/');
}

//위처럼 하면 '/'로 시작하는 경우는 isAbsolutePath 함수를 통해 _brand값을 심어줄 수 있다
//나머지 함수들에서는 _brand 값이 있는 AbsolutePath타입에서만 동작하게 구현할 수 있다
```

```tsx
type Meters = number & {_barnd : 'meters'};
type Seconds = number &{_brand: 'seconds'};

const meters = (m:number)=>m as Meters;
const seconds = (s:number)=> s as Seconds;

const oneKm = meters(1000); //type Meters
const oneMin = seconds(60); //type Seconds

const tenKm = oneKm * 10; //type number

//숫자도 상표를 붙일 수 있지만, 산술 연산 후에는 상표가 없어지기 때문에 실제로 사용하기는 어려움
```

brand를 붙이는 상표기법은 타입 시스템에서 동작하지만 런타임에 상표를 검사하는 것과 동일한 효과를 얻을 수 있다

### 아이템 42 : 모르는 타입의 값에는 any 대신 unknown을 사용하기

---

1. 함수 반환값의 unknown
    
    함수 반환값으로 any타입을 쓰는 경우 할당 받는 곳에서 타입 선언을 통해서 받는게 이상적이지만, 만약 생략하게 되면 any가 뿌려진다 ( 똥이 뿌려진다 )
    
    대신 unknown 타입을 반환하게 하면 런타임에러를 방지하고 애초에 모든 프로퍼티의 접근이 에러를 발생시킨다. unknwon으로 반환하고 as를 통해 타입 단언으로 처리하자. (unknown은 다른 타입의 변수에 할당 불가, 아래 표 참조)
    
    |  | 다른 타입의 값을 왼쪽 타입에 할당 가능? | 다른 타입의 변수에 왼쪽 타입의 값을 할당가능? |
    | --- | --- | --- |
    | any | 가능 | 가능 |
    | unkown | 가능 | 불가능 |
    | never | 불가능 | 가능 |
    
    집합의 개념에서 any는 문제를 발생시키기 좋다. 
    
2. 변수 선언의 unknown
    
    어떤 값이 있는데 타입을 모르는 경우 unknown을 쓰자.
    
    instanceof 혹은 사용자 정의 타입 가드를 통해 원하는 타입으로 변경할 수 있다.
    
    하지만 타입 범위 좁히기에서 상당히 노력이 필요하고 조심해야한다.
    
    객체인지 확인하고, null인지도 따로 체크해줘야 한다 (typeof ‘object’ === null …)
    
    제네릭도 내부적으로는 타입 단언과 똑같이 동작한다.
    
    제네릭보다 unknown으로 반환하고 사용자가 직접 단언문을 사용해서 타입을 좁히도록 강제하는게 좋다.
    
3. 단언문의 unknown
    
    ```tsx
    let barAny = foo as any as Bar;
    let barUnk = foo as unknown as Bar;
    ```
    
    기능적으로는 위 2개는 동일하다. 하지만 나중에 단언문을 분리하는 경우 (리팩토링 등) unkown으로 선언해놓는것이 안전하다.
    
    - type {} 는 null과 undefined를 제외한 모든 값
    - type object는 모든 비기본형 (true, number, string 등 제외) (객체 , 배열 포함)
    
    예전에는 unknown 타입이 없어서 {} 타입을 썼었으나 지금은 unknown이 있음.
    
    > 만약, 정말로 null과 undefined가 아니라면 {} 타입을 사용할 수 있음
    > 
    

### 아이템 43 : 몽캐 패치보다는 안전한 타입을 사용하기

---

> 몽키패치 : 원래 소스코드를 변경하지 않고 런타임 환경에서 코드의 기본동작을 변경 하는 기술
ex) window.__DEBUG__ = {} 할당해서 사용 등 (원래 없던 프로퍼티들)
> 

사실 몽키패치는 좋은 설계가 아님 (당연한거 아닌가?…)

타입스크립트 환경에서는 타입체커가 동작하지 못함.

```tsx
document.monkey = 'Patch'; //~~~ Document 유형에 monkey 속성이 없습니다.
```

1. as any 사용해서 해결하기 - 비추
2. 설계를 바꿔서 DOM 객체 등에 디커플링 시키기 
3. interface의 보강 활용하기 (any보다 나은 방법, 그래도 다음 방법 추천)
    
    ```tsx
    interface Document {
    	monkey : string;
    }
    
    document.monkey = 'Patch';
    ```
    
    전역으로 적용되기 때문에, 다른 곳에 sideEffect 줄 수 있음
    
4. 더 구체적인 타입 단언문 사용하기
    
    ```tsx
    interface MonkeyDocument extends Document {
    	monkey : string;
    }
    
    (document as MonkeyDocument).monkey = 'Patch';
    ```
    
    위 처럼 하면 필요한 곳에서만 사용 가능
    

> 하지만 몽키패치는 좋은 설계가 아니다. 되도록 설계를 제대로 하자.
> 

### 아이템 50 : 오버로딩 타입보다는 조건부 타입 사용하기

---

특정 함수가 여러 타입에 동작해야한다면 아래와 같이 설정 할 수 있다.

```tsx
function double(x : number | string) : number|string;
function double(x: any) { return x + x; }
```

위 처럼 하면 

```tsx
double(3); //string|number;
double('x'); //string|number;
```

위와 같은 문제가 발생한다.

제네릭을 활용하면

```tsx
function double<T extends number|string>(x: T) : T;

const num = double(12); //12
const str = double('x'); //'x' 너무 타입이 좁혀서 설정된다.
```

너무 타입이 좁혀진다.

함수 오버로딩을 사용하면

```tsx
function double(x: number):number;
function double(x: string):string;

double(12); //number
double('x');//string

const temp : number| string;

double(temp); //error 'string | number'형식의 인수는 'string'형식의 매개변수에 할당 할 수 없습니다
```

위와 같이 유니온타입에서 에러가 발생한다.

여기서는 조건부 타입으로 해결해보자

```tsx
function double<T extends number | string>
(x: T) : T extends string ? string : number;
```

삼항 연사자를 통해 특정 조건에 해당되는지 체크하고 리턴 타입을 결정한다.

### 아이템 51 : 의존성 분리를 위해 미러 타입 사용하기

---

특정 타입을 위해 의존성을 추가해야하는 경우에
해당 타입의 일부분만 사용한다면 미러타입을 고려해보는 것도 좋다. (구조적 타이핑 활용)

만약 대부분 추출해야한다면 차라리 명시적으로 @types를 추가하는게 좋다 

미러 타입