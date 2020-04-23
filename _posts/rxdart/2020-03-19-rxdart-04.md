---
layout: post
title: "[flutter] - RxDart : Extension Methods Part1"
subtitle:   "Let's study a RxDart"
categories: flutter
tags: flutter rxdart 
comments: true
---
## Extension method

`Extension method`는 `stream`에 추가 기능을 더해서 새로운 `stream`을 만들어주는 `stream`내부 함수이다.

각 내용들을 하나씩 알아보자

* 순서
    - [buffer](#buffer-method)
    - [bufferCount](#buffercount-method)
    - [bufferTest](#buffertest-method)
    - [bufferTimer](#buffertimer-method)
    - [concatWith](#concatwith-method)
    - [defaultEmpty](#defaultempty-method)
    - [delay](#delay-method)
    - [materialize](#materialize-method)
    - [dematerialize](#dematerialize-method)
    - [distinctUnique](#distinctunique-method)
    - [doOnCancel](#dooncancel-method)
    - [doOnData](#doondata-method)
    - [doOnDone](#doondone-method)
    - [doOnEach](#dooneach-method)
    - [doOnError](#doonerror-method)
    - [doOnListen](#doonlisten-method)
    - [doOnPause](#doonpause-method)
    - [doOnResume](#doonresume-method)
    - [exhaustMap](#exhaustmap-method)
    - [flatMap](#flatmap-method)
    - [flatMapIterable](#flatmapiterable-method)
    - [groupBy](#groupby-method)
    - [interval](#interval-method)
    - [part2](https://funncy.github.io/flutter/2020/03/20/rxdart-05/)

---

## buffer method

```java
Stream<List<T>> buffer (
    Stream window
)
```


+ Example : 
    ```java
    Stream.periodic(Duration(milliseconds: 100), (i) => i)
        .buffer(Stream.periodic(Duration(milliseconds: 160), (i) => i))
        .listen(print); // prints [0, 1] [2, 3] [4, 5] ...
    ```

+ Example : 
    ```java
    Stream.periodic(Duration(milliseconds: 100), (i) => i)
        .buffer(Stream.periodic(Duration(milliseconds: 400), (i) => 'test'))
        .listen(print); // prints [0, 1, 2, 3, 4] [5, 6, 7, 8] [9, 10, 11, 12] ...
    ```


`buffer`는 주어진 `stream`의 데이터를 `list`에 모아서 출력한다.
`buffer`안에 있는 내부 `stream` (`window`)에서 데이터가 출력될 때 모아 놓은 `list`를 출력한다.

---

## bufferCount method 

```java
Stream<List<T>> bufferCount (
    int count,
    [int startBufferEvery = 0]
)
```

+ Example : 
    ```java
    RangeStream(1, 4)
        .bufferCount(2)
        .listen(print); // prints [1, 2], [3, 4] done!
    ```

+ Example : 
    ```java
    RangeStream(1, 5)
        .bufferCount(3, 2)
        .listen(print); // prints [1, 2, 3], [3, 4, 5], [5] done!
    ```
`bufferCount`는 들어온 데이터를 `count`만큼 모아서 `list`로 출력해준다.
매 출력마다 `startBufferEvery`값을 `index`로 다시 시작하고 앞의 값은 버린다.

> `Range`로 `1~5` 데이터가 들어오는데 `bufferCount(3,2)`를 통해 `3`개씩 `list`로 출력하며
매 출력 마다 `2`번째 `index`에서 실행한다.

---

## bufferTest method

```java
Stream<List<T>> bufferTest (
    bool onTestHandler(
        T event
    )
)
```

+ Example : 
    ```java
    Stream.periodic(Duration(milliseconds: 100), (int i) => i)
        .bufferTest((i) => i % 2 == 0)
        .listen(print); // prints [0], [1, 2] [3, 4] [5, 6] ...
    ```

`bufferTest`는 `bufferTest`의 `function`의 값이 `True`가 될 때 까지의 데이터를 `list`로 모아서 출력한다.

---

## bufferTimer method

```java
Stream<List<T>> bufferTime (
    Duration duration
)
```

+ Example : 
    ```java
    Stream.periodic(Duration(milliseconds: 100), (int i) => i)
        .bufferTime(Duration(milliseconds: 200))
        .listen(print); // prints [0, 1, 2] [3, 4] [5, 6] ...
    ```

`bufferTimer`는 `Duration`의 시간만큼 데이터를 모아서 `list`로 출력한다.

---

## concatWith method

```java
Stream<T> concatWith (
    Iterable<Stream<T>> other
)
```

+ Example : 
    ```java
    TimerStream(1, Duration(seconds: 10))
        .concatWith([Stream.fromIterable([2])])
        .listen(print); // prints 1, 2
    ```

`concatWith`는 메인 `stream`이 모든 출력이 끝나고 종료되면 이어서 `concatWith`의 `Stream`들을 순서대로 출력한다.

---

## defaultEmpty method

```java
Stream<T> defaultIfEmpty (
    T defaultValue
)
```

+ Example : 
    ```java
    Stream.empty().defaultIfEmpty(10).listen(print); // prints 10
    ```

`defaultEmpty`는 `stream`값이 `empty`일때 기본값을 설정해준다.

---

## delay method

```java
Stream<T> delay (
    Duration duration
)
```
+ Example : 
    ```java
    Stream.fromIterable([1, 2, 3, 4])
        .delay(Duration(seconds: 1))
        .listen(print); // [after one second delay] prints 1, 2, 3, 4 immediately
    ```

`delay`는 말 그대로 `stream` 출력을 지연해준다.

---

## materialize method

```java
Stream<Notification<T>> materialize ()
```
+ Example : 
    ```java
    Stream<int>.error(Exception())
        .materialize()
        .listen((i) => print(i)); // Prints onError Notification
    ```

## dematerialize method

```java
Stream<T> dematerialize ()
```
+ Example : 
    ```java
    Stream<Notification<int>>
        .fromIterable([Notification.onData(1), Notification.onDone()])
        .dematerialize()
        .listen((i) => print(i)); // Prints 1
    ```

`dematerialize`와 `materialize`는 한쌍이다. (서로 반대)
이것을 이해하느라 `RxJava`를 참고해봤다.

![이미지](https://Funncy.github.io/assets/img/rxdart/2020-03-18-rx-dart-03-03.png "materialize")

![이미지](https://Funncy.github.io/assets/img/rxdart/2020-03-18-rx-dart-03-04.png "dematerialize")

`캡슐화`에 대한 내용들이 나온다. 단순히 `Stream`에서 데이터를 출력하는게 아니라 `noticiation`으로 묶어서 보내거나 아니면 묶여있는 경우 데이터만 출력을 원할 경우 사용한다. 

> 사용 예를 찾아봐야겠지만 여러 `stream`이 겹쳐있고 `event`들이 겹쳐있을때 사용하는 것 같다.

---

## distinctUnique method

```java
Stream<T> distinctUnique (
    {bool equals(
        T e1,
        T e2
    ),
    int hashCode(
        T e
    )}
)
```
+ Example : 
    ```java
    Stream<Notification<int>>
        .fromIterable([Notification.onData(1), Notification.onDone()])
        .dematerialize()
        .listen((i) => print(i)); // Prints 1
    ```

`distinctUnique`는 주어진 데이터들의 중복을 제거하고 보내준다. 
주의점은 `stream`이 `broadcastStream`인 경우 `listen`할 때 마다 데이터의 `equal`을 확인한다.

---

## doOnCancel method

```java
Stream<T> doOnCancel (
    void onCancel()
)
```
+ Example : 
    ```java
    final subscription = TimerStream(1, Duration(minutes: 1))
        .doOnCancel(() => print('hi'));
        .listen(null);

    subscription.cancel(); // prints 'hi'
    ```

`doOnCancel`은 `subscription`을 `cancle`할 때 호출된다.

---

## doOnData method

```java
Stream<T> doOnData (
    void onData(
        T event
    )
)
```
+ Example : 
    ```java
    Stream.fromIterable([1, 2, 3])
        .doOnData(print)
        .listen(null); // prints 1, 2, 3
    ```

`doOnData`는 `stream`에서 데이터가 넘어오며 넘오는 타이밍에 실행하는 함수이다.

---

## doOnDone method

```java
Stream<T> doOnDone (
    void onDone()
)
```
+ Example : 
    ```java
    Stream.fromIterable([1, 2, 3])
        .doOnDone(() => print('all set'))
        .listen(null); // prints 'all set'
    ```

`doOnDone`는 `onDone`이 호출 되는 시점(정상 종료)에 호출되는 함수이다.

---

##  doOnEach method

```java
Stream<T> doOnEach (
    void onEach(
        Notification<T> notification
    )
)
```
+ Example : 
    ```java
    Stream.fromIterable([1])
        .doOnEach(print)
        .listen(null); // prints Notification{kind: OnData, value: 1, errorAndStackTrace: null}, Notification{kind: OnDone, value: null, errorAndStackTrace: null}
    ```

`doOnEach`는 `onData`, `onDone`, `onError` 각 함수 전에 실행되는 함수이다 (`error` 없을 시 `onError` 동작 안함)

> 데이터는 `noticiation` 형태로 캡슐화 되어있음.

---

## doOnError method

```java
Stream<T> doOnError (
    Function onError
)
```
+ Example : 
    ```java
    Stream.error(Exception())
        .doOnError((error, stacktrace) => print('oh no'))
        .listen(null); // prints 'Oh no'
    ```

`doOnError`는 `error`가 발생했을때 실행하는 함수이다.

---

## doOnListen method

```java
Stream<T> doOnListen (
    void onListen()
)
```
+ Example : 
    ```java
    Stream.fromIterable([1])
        .doOnListen(() => print('Is someone there?'))
        .listen(null); // prints 'Is someone there?'
    ```

`doOnListen`는 `stream`에서 데이터를 `처음` `listen`했을 때 실행하는 함수이다.

---

## doOnPause method

```java
Stream<T> doOnPause (
    void onPause(
        Future resumeSignal
    )
)
```
+ Example : 
    ```java
    final subscription = Stream.fromIterable([1])
        .doOnPause(() => print('Gimme a minute please'))
        .listen(null);

    subscription.pause(); // prints 'Gimme a minute please'
    ```

`doOnPause`는 `subscription`이 `pause`될 때 실행되는 함수이다.

---

## doOnResume method

```java
Stream<T> doOnResume (
    void onResume()
)
```
+ Example : 
    ```java
    final subscription = Stream.fromIterable([1])
        .doOnResume(() => print('Lets do this!'))
        .listen(null);

    subscription.pause();
    subscription.resume(); //'Let's do this!'
    ```

`doOnResume`은 `subscription`이 `pause`후 `resume`할 때 실행되는 함수이다.

---

## exhaustMap method

```java
Stream<S> exhaustMap <S>(
    Stream<S> mapper(
        T value
    )
)
```
+ Example : 
    ```java
    RangeStream(0, 2).interval(Duration(milliseconds: 50))
        .exhaustMap((i) =>
            TimerStream(i, Duration(milliseconds: 75)))
        .listen(print); // prints 0, 2
    ```

`exhaustMap`은 `mapper`를 활용하여 새로운 `stream`을 만든다. `mapper`의 `stream`이 성공할때 까지 `source stream`의 모든 데이터는 무시된다.

`stream nosiy`가 심하고 이전 데이터와 `async`를 맞추고 싶을 때 유용하다. 

> 예시에서는 시간차를 이용해 `source stream`에서 데이터중 1을 무시하고 출력한다. 

--- 

## flatMap method

```java
Stream<S> flatMap <S>(
    Stream<S> mapper(
        T value
    )
)
```
+ Example : 
    ```java
    RangeStream(4, 1)
        .flatMap((i) => TimerStream(i, Duration(minutes: i)))
        .listen(print); // prints 1, 2, 3, 4
    ```

`flatMap`은 `map`처럼 데이터를 가공하는 함수이지만 `data`를 출력하는게 아니라 `stream`을 출력한다.
그래서 예제 처럼 `TimerStream`을 활용해 `sequence`를 뒤집는 것도 가능하다.

---

## flatMapIterable method

```java
Stream<S> flatMapIterable <S>(
    Stream<Iterable<S>> mapper(
        T value
    )
)
```
+ Example : 
    ```java
    RangeStream(1, 4)
        .flatMapIterable((i) => Stream.fromIterable([[i]]))
        .listen(print); // prints 1, 2, 3, 4
    ```

`flatMapIterable`는 위에 `flatMap`과 똑같이 `stream`을 보내지만 `Iterable`형태로 보낸다.

> `API`를 사용할 때 데이터가 `list`형태로 오는 경우 유용하게 사용 가능하다. 

```java
  Stream.fromIterable([
    [1, 2, 3],
    [4, 5, 6],
    [7, 8, 9]
  ]).flatMapIterable((i) {
    print("before : $i");
    return Stream.fromIterable([i]);
  }).listen(print);

// before : [1, 2, 3]
// 1
// 2
// 3
// before : [4, 5, 6]
// 4
// 5
// 6
// before : [7, 8, 9]
// 7
// 8
// 9
```

---

## groupBy method

```java
Stream<GroupByStream<T, S>> groupBy <S>(
    S grouper(
        T value
    )
)
```
+ Example : 
    ```java
    Stream.fromIterable([
        {'name': 'hyojun1', 'age': 10},
        {'name': 'hyojun2', 'age': 10},
        {'name': 'hyojun3', 'age': 13},
        {'name': 'hyojun4', 'age': 13},
        {'name': 'hyojun5', 'age': 13},
        {'name': 'hyojun3', 'age': 15},
    ]).groupBy((data) => data['age']).listen((data){
        print("Group Key : ${data.key}");
        data.listen(print);
    });

    //Group Key : 10
    //{name: hyojun1, age: 10}
    //{name: hyojun2, age: 10}
    //Group Key : 13
    //{name: hyojun3, age: 13}
    //{name: hyojun4, age: 13}
    //{name: hyojun5, age: 13}
    //Group Key : 15
    //{name: hyojun3, age: 15}
    ```

`groupBy`는 데이터를 `Key`를 기준으로 `GroupByStream`으로 묶어준다. 그래서 들어온 `GroupByStream`을 다시 `listen`하여 데이터를 얻는다.

---

## interval method

```java
Stream<T> interval (
    Duration duration
)
```
+ Example : 
    ```java
    Stream.fromIterable([1, 2, 3])
        .interval(Duration(seconds: 1))
        .listen((i) => print('$i sec'); // prints 1 sec, 2 sec, 3 sec
    ```

`interval`은 주어진 `Duration`의 간격마다 데이터 출력한다.

---

#### 나머지는 다음장에서...






