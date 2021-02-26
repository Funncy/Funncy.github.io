---
layout: post
title: '[Dart] -  async/await 가 Array 함수(map, forEach, filter..) 에서 동작 안할 때 '
subtitle: 'dart - aasync/await doesn't work in Array method(map, forEach, filter..) cycle'
categories: programming
tags: programming dart async await array list map forEach filter 
comments: true
---

이번에 Firestore를 테스트 하다가 List형의 map 혹은 forEach 함수를 사용할 경우가 생겼다.

```jsx
var result = await firebaseFirestore.collection('topics').get();

result.docs.forEach((doc) {
....
});
```

그런데 여기서 forEach내부 함수를 async로 처리를 진행하고 싶었다.

```jsx
var result = await firebaseFirestore.collection('topics').get();

result.docs.forEach((doc) async {
	var nestedResult = await doc.reference.collection('data').get();
	print(nestedResult.data());
});
```

그러나 예상 외로 동작 순서가 문제가 있었다.
다음 예시로 설명하겠다.

```jsx

result.forEach((element) async {
	await Future.delayed(Duration(seconds : 1);
  print("Hello I'm async forEach");
});

print("Why ?");

// 출력 순서
// Why ?
// Hello I'm async forEach
// ...
```

나는 분명 await를 걸었지만 뒤늦게 실행되었다.
그래서 forEach의 구조를 살펴보았다.

```jsx
//iterable.dart

void forEach(void f(E element)) {
    for (E element in this) f(element);
}
```

여기서 Array&List에서 지원해주는 map, filter, forEach와 같은 함수에 async 콜백 함수를 넣어주면 안되는 문제를 알게 되었다.

Array & List 에서 지원해주는 함수는 기본적으로 async를 고려하지 않고 callback 함수를 등록해주는 구조이기 때문에 나의 함수 스텍 단계에서 아무리 await를 걸어둔 함수를 콜백으로 넣어주어도 순서가 원하는대로 실행되지 않는 것이다.

이걸 이제 풀어서 해결해보자.

```jsx
for (var data in result) {
	await Future.delayed(Duration((seconds: 1)));
	print("Hello I'm async for");
}

print('Why ?');

//실행순서
// Hello I'm async for
// ...
// Why ?
```

당연한 소리겠지만 순서대로 동작이 잘된다.

### 성능

---

이제 해결법은 알았으나 다음과 같은 문제가 또 발생한다.

```jsx
for (var data in result) {
	await Future.delayed(Duration((seconds: 1)));
	print("Hello I'm async for");
}

//Data 가 10개일 때
// 1초씩 10번 즉 10초의 실행시간이 걸린다.
```

> 10개라서 10초인데 만약 1만개면?...

이렇게 실행시간이 늘어지는 문제를 해결하는 방법을 알아보자.

`Future.wait()` 를 이용해 javascript의 promise.all 처럼 구현해보자.

```jsx
Future<void> getData(QueryDocumentSnapshot data) async {
    await Future.delayed(Duration(seconds: 1));
    print("Hello Async");
}

List<Future<dynamic>> listFuture = [];

result.docs.forEach((element) {
  listFuture.add(getData(element));
});

await Future.wait(listFuture);
print('why ?');
```

이렇게 forEach에서는 들어온 데이터를 Future List에 넣어주자.

기존에 하던 동작을 async 함수로 뽑아주고 넣어주자.

그리고 `await Future.wait(listFure)` 에서 모든 Future가 완료될 때 까지 기다리게 된다.

여기서는 10개의 데이터가 1초씩 총 10초를 대기하는게 아니라 병렬적으로

1초만에 10개의 데이터의 결과값이 들어온다. (정확히 1초는 아니겠지만)

앞으로 async함수를 forEach에서 사용하고 싶다면 다음과 같은 방법을 사용하자.

> javascript에서는 promise.all() 을 사용하자
