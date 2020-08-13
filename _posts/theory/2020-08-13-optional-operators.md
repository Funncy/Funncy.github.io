---
layout: post
title: " Optional Operators : Dart "
subtitle:   "optional operators "
categories: programming
tags: programming
comments: true
---

# 1. 삼항 연산자 ( ? : )

```java
int height = 185;
String heightCategory = "";

if(height > 170) {
	heightCategory = "키 큼";
} else {
	heightCategory = "키 작음";
}
```

위의 코드를 삼항 연산자를 대입해서 사용해보자

```java
int height = 185;
String heightCategory = height > 170 ? "키 큼" : "키 작음";
```

삼항 연산자 구조는 다음과 같다.

```java
condition ? (statement if true) : (statement if false);
```

# 2. ??

??는 Null 체크 연산자이다. 
예를들어 person.name이 null이라면 이름을 "효준"으로 설정해준다고 해보자.

보통 아래와 같이 작성을 할 것이다.

```java
String name = "";

if(person.name == null) {
	name = "효준";
} else {
	name = person.name;
}

//혹은 삼항연산자

String name = person.name != null ? person.name : "효준";
```

위의 코드를 ?? 널 체크 연산자를 사용해보자.

```java
String name = person.name ?? "효준";
```

# 3. ?.

다음과 같은 클래스가 있다고 해보자.

```java
class Point {
	int x;
	int y;
	
	Point(this.x, this.y);
}
```

만약 여기서 Point 객체 멤버 변수를 초기화 하지 않고 접근하면 에러가 발생할 거다.

예를 들어

```java
Point point;
print(point.x); // NoSuchMethodError: The getter 'x' was called on null.
```

이걸 또 해결하려면 다음과 같은 방법을 사용할 것이다.

```java
Point point;

if(point != null) {
	print(point.x);
} else {
	print("값 없음!");
}
```

위의 방법은 좋은 방법이 아니다. 만약 JSON Parsing을 한다고 하면 아래와 같은 경우가 많을 것이다.

```java
users[1].datas[0].projectDetail.info.name
```

위와 같은 경우 모든 곳에 Null Error가 발생할 수 있다. 여기서 if-else로 해결하려면 쉽지 않다.
여기서 ?. 연산자를 사용해보자. 

?. 연산자는 접근하려는 데이터가 null일 경우 error가 아니라 null을 반환한다. 
그리고 null check operator인 ?? 연산자로 0을 초기값으로 넣어주었다.

```java
Point point;

int x = point?.x ?? 0;
```

```java
users[1]?.datas[0]?.projectDetail?.info?.name ?? "N/A";
```

위의 경우 어느 파트에서던 Null이 감지되면 N/A가 리턴된다.

# 4. ??=

??= 연산자의 의미는 만약 왼쪽 항이 Null이면 오른쪽 항으로 초기화 시켜라는 내용이다.

```java
Point point;

point ??= Point(x:1, y:2); //현재 point는 null이라서 오른쪽 항으로 초기화 된다.

point ??= Point(x:3, y:4); // 현재 point는 null이 아니라서 그대로 유지된다.
```

# 5. ⇒

⇒ 는 2가지 사용법이 있다.

⇒x 는 return x;를 의미한다.

```java
String testFunction(String s){
	return s.toLowerCase();
}

String testFunction(String s) => s.toLowerCase();
```

다른 방법은 return이 없는 한 줄 짜리 코드 실행이다.

```java
void testFunction(String s) => print(s);
```

# 6.  .. (Cascade)

Cascade 표기법은 개체의 참조를 가져와서 하나씩 변경하는것이 아니라 단계별로 변경하는 방법입니다.

```java
data1.setA(2);
data1.setB(10);
data1.showVal();

///Cascade
data2..setA(2)
		 ..setB(10)
		 ..showVal();
```

# 7. ~/

수학 연산 속도를 높이기 위한 연산자 입니다.

~/ 연산자는 내림용 나눔 연산자 입니다.

```java
int a = 3;
int b = 7;
int c = b ~/ a;
print(c); //2
```

# 8. ... (Spread operator)

...연산자는 List를 Unpack하여 List에 추가할 때 사용됩니다.

```java
var testList = [1, 2, 3, 4, 5];

var otherList = [
	6,
	7,
	8,
	9,
	...testList,
];
```

출처 : [https://medium.com/flutter-community/simple-and-bug-free-code-with-dart-operators-2e81211cecfe](https://medium.com/flutter-community/simple-and-bug-free-code-with-dart-operators-2e81211cecfe)