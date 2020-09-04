---
layout: post
title: "[Dart] -  async(async*) , yield(yield*) "
subtitle: "Django - async(async*) , yield(yield*)"
categories: programming
tags: programming dart async yield async* yield*
comments: true
---

보통 `async`는 비동기 함수에 붙는다.

그런데 코드를 보다 보면 `async*`을 확인 할 수 있다.

둘의 차이는 무엇일까?

## async

---

보통 Future와 같이 비동기 데이터가 들어올때 then 구문 보다 async/await 구문을 많이 쓴다.

```java
void test() async {
	await Future.delayed(Duration(milliseconds: 1000));
	print("async/await");
}
```

⇒ 1초 후에 프린트 문이 동작한다 (await 대기)

## async\*

---

async 와 똑같이 비동기지만 차이점은 Stream을 반환한다는 거다.

그래서 yield로 데이터를 반환할 수 있다. (Generator를 만든다)

```java
Stream<int> foo() async* {
  for (int i = 0; i < 42; i++) {
    await Future.delayed(const Duration(seconds: 1));
    yield i;
  }
}
```

⇒ 1 초마다 데이터를 내뱉는다. return이 아니라 yield로 데이터를 중간 중간 반환한다.

## yield

---

이제 yield에 대해서 알아보기 전에 먼저 Generators에 대해서 알아보자.

Generator는 Lazily Produce가 필요한 경우 사용한다고 공식 문서에 나와있다.

lazily Produce는 말그대로 중간 중간에 계속 데이터를 반환하는걸 말한다.

그리고 2가지 종류가 있다.

하나씩 예시를 봐보자

- Synchronous

  ```java
  Iterable<int> naturalsTo(int n) sync* {
    int k = 0;
    while (k < n) yield k++;
  }
  ```

  Iterable 객체를 반환하며 동기로 돌아간다.

- Asynchronous

  ```java
  Stream<int> asynchronousNaturalsTo(int n) async* {
    int k = 0;
    while (k < n) yield k++;
  }
  ```

  Stream 객체를 반환하며 비동기로 돌아간다.

## yield\*

---

여기서 yield\*의 다른점을 알아보자.

위와 같이 Generator를 만들어 동작 시킬 때 재귀 형식으로 반환하면 효율적일 때가 있다.

그때 사용한다.

```java
Iterable<int> naturalsDownFrom(int n) sync* {
  if (n > 0) {
    yield n;
    yield* naturalsDownFrom(n - 1);
  }
}
```

이러면 yield 이후에 yield\*에서 재귀로 다시 시작한다.
