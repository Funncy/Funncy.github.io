---
layout: post
title: "[flutter] - RxDart : Extension Methods Part2"
subtitle:   "Let's study a RxDart"
categories: flutter
tags: flutter rxdart 
comments: true
---
## Extension method

`Extension method`는 `stream`에 추가 기능을 더해서 새로운 `stream`을 만들어주는 `stream`내부 함수이다.

[Part1](https://funncy.github.io/flutter/2020/03/19/rxdart-04/)을 이어서 알아보자

* 순서
    - [part 1](https://funncy.github.io/flutter/2020/03/19/rxdart-04/)
    - [mapTo](#mapto-method)
    - [max](#max-method)
    - [min](#min-method)
    - [mergeWith](#mergewith-method)
    - [onErrorResume](#onerrorresume-method)
    - [onErrorResumeNext](#onerrorresumenext-method)
    - [onErrorReturn](#onerrorreturn-method)
    - [onErrorReturnWith](#onerrorreturnwith-method)
    - [pairwise](#pairwise-method)
    - [sample](#sample-method)
    - [sampleTime](#sampletime-method)
    - [scan](#scan-method)
    - [skipUntil](#skipuntil-method)
    - [startWith](#startwith-method)
    - [startWithMany](#startwithmany-method)
    - [switchIfEmpty](#switchifempty-method)
    - [switchMap](#switchmap-method)
    - [takeUntil](#takeuntil-method)
    - [debounce](#debounce-method)
    - [debounceTime](#debouncetime-method)
    - [throttle](#throttle-method)
    - [throttleTime](#throttletime-method)
    - [timeInterval](#timeinterval-method)
    - [timestamp](#timestamp-method)
    - [whereType](#wheretype-method)
    - [window](#window-method)
    - [windowCount](#windowcount-method)
    - [windowTest](#windowtest-method)
    - [withLatestFrom](#withlatestfrom-method)
    - [zipWith](#zipwith-method)
---

## mapTo method

```java
Stream<S> mapTo <S>(
    S value
)
```


+ Example : 
    ```java
    Stream.fromIterable([1, 2, 3, 4])
        .mapTo(true)
        .listen(print); // prints true, true, true, true
    ```

`mapTo`는 주어진 스트림 데이터를 지정된 `상수`로 출력한다.

---

## max method

```java
Future<T> max (
    [Comparator<T> comparator]
)
```

+ Example : 
    ```java
    final max = await Stream.fromIterable([1, 2, 3]).max();

    print(max); // prints 3
    ```

+ Example With `Comparator`:
    ```java
    final stream = Stream.fromIterable(['short', 'looooooong']);
    final max = await stream.max((a, b) => a.length - b.length);

    print(max); // prints 'looooooong'
    ```

`max`는 `list`형의 데이터들에서 가장 큰 값을 찾는 함수이다. 나만의 `Comparator`를 지정해 사용할 수 있다.

---

## min method

```java
Future<T> min (
    [Comparator<T> comparator]
)
```

+ Example : 
    ```java
    final min = await Stream.fromIterable([1, 2, 3]).min();

    print(min); // prints 1
    ```

+ Example With `Comparator`:
    ```java
    final stream = Stream.fromIterable(['short', 'looooooong']);
    final min = await stream.min((a, b) => a.length - b.length);

    print(min); // prints 'short'
    ```

`min`는 `list`형의 데이터들에서 가장 작은 값을 찾는 함수이다. 나만의 `Comparator`를 지정해 사용할 수 있다.

---

## mergeWith method

```java
Stream<T> mergeWith (
    Iterable<Stream<T>> streams
)
```

+ Example : 
    ```java
    TimerStream(1, Duration(seconds: 10))
        .mergeWith([Stream.fromIterable([2])])
        .listen(print); // prints 2, 1
    ```

`mergeWith`는 다른 `stream`들과 조합하여 출력해준다. 출력 순서는 들어온 데이터 순서이다.

---

## onErrorResume method

```java
Stream<T> onErrorResume (
    Stream<T> recoveryFn(
        dynamic error
    )
)
```

+ Example : 
    ```java
    ErrorStream(Exception())
        .onErrorResume((dynamic e) =>
            Stream.fromIterable([e is StateError ? 1 : 0])
        .listen(print); // prints 0
    ```

`onErrorResume`는 `error` 이벤트를 차단하고 `recoveryFn`함수를 통해 새로운 `stream`을 반환해줍니다.
`onError`가 발생하지 않습니다.

`error`에 따른 `logic`처리가 가능합니다.

---

## onErrorResumeNext method

```java
Stream<T> onErrorResumeNext (
    Stream<T> recoveryStream
)
```

+ Example : 
    ```java
    ErrorStream(Exception())
        .onErrorResumeNext(Stream.fromIterable([1, 2, 3]))
        .listen(print); // prints 1, 2, 3
    ```

`onErrorResumeNext`는 위에 `onErrorResume`과 같이 `error`를 차단하고 `stream`을 반환해주지만 `logic`처리는 필요 없을 때 사용합니다.

---

## onErrorReturn method

```java
Stream<T> onErrorReturn (
    T returnValue
)
```

+ Example : 
    ```java
    ErrorStream(Exception())
        .onErrorReturn(1)
        .listen(print); // prints 1
    ```

`onErrorReturn`은 `error`상황에서 `onError`이벤트를 막고 고정된 데이터를 출력시켜줍니다. 정상종료합니다.

---

## onErrorReturnWith method

```java
Stream<T> onErrorReturnWith (
    T returnFn(
        dynamic error
    )
)
```

+ Example : 
    ```java
    ErrorStream(Exception())
        .onErrorReturnWith((e) => e is Exception ? 1 : 0)
        .listen(print); // prints 1
    ```

`onErrorReturnWith`는 `onErrorReturn`과 같지만 `error`데이터를 받아서 `logic`처리를 해줄 수 있다.

---

## pairwise method

```java
Stream<Iterable<T>> pairwise ()
```

+ Example : 
    ```java
    RangeStream(1, 4)
        .pairwise()
        .listen(print); // prints [1, 2], [2, 3], [3, 4]
    ```

`pairwise`는 `pair`로 쌍으로 데이터를 보내준다.
>`특이점`은 그냥 쌍으로 묶는게 아니라 `다음 데이터`가 `중복`되서 나온다.
`[1,2,3,4] => [1,2] , [2,3], [3,4]`

---

## sample method

```java
Stream<T> sample (
    Stream sampleStream
)
```

+ Example : 
    ```java
    Stream.fromIterable([1, 2, 3])
        .sample(TimerStream(1, Duration(seconds: 1)))
        .listen(print); // prints 3
    ```

`sample`은 `sample`내부 `sampleStream`이 데이터를 출력하면 메인 `stream`에서 가장 최근에 출력된 데이터를 출력해준다. 

---

## sampleTime method

```java
Stream<T> sampleTime (
    Duration duration
)
```

+ Example : 
    ```java
    Stream.fromIterable([1, 2, 3])
        .sampleTime(Duration(seconds: 1))
        .listen(print); // prints 3
    ```

`sampleTime`은 `sample`과 거의 다 같지만 `stream`이 아니라 `Duration`으로 정의해준다.

---

## scan method

```java
Stream<S> scan <S>(
    S accumulator(
        S accumulated,
        T value,
        int index
    ),
    [S seed]
)
```

+ Example : 
    ```java
    Stream.fromIterable([1, 2, 3])
        .scan((acc, curr, i) => acc + curr, 0)
        .listen(print); // prints 1, 3, 6
    ```

`scan`은 데이터의 누적값을 출력해주는 함수이다.
`accumulator`를 지정해 `logic`을 짤 수 있으며 `seed`값은 초기값이다.

---

## skipUntil method

```java
Stream<T> skipUntil <S>(
    Stream<S> otherStream
)
```

+ Example : 
    ```java
    MergeStream([
        Stream.fromIterable([1]),
        TimerStream(2, Duration(minutes: 2))
    ])
        .skipUntil(TimerStream(true, Duration(minutes: 1)))
        .listen(print); // prints 2;
    ```

`skipUntil`은 `otherStream`에서 데이터가 출력될 때 까지 메인 `stream`에서의 데이터는 무시한다. 

--- 

## startWith method

```java
Stream<T> startWith (
    T startValue
)
```

+ Example : 
    ```java
    Stream.fromIterable([2]).startWith(1).listen(print); // prints 1, 2
    ```

`startWith`는 `startValue`를 `stream`의 데이터 앞에 붙여준다.

---

## startWithMany method

```java
Stream<T> startWithMany (
    List<T> startValues
)
```

+ Example : 
    ```java
    Stream.fromIterable([3]).startWithMany([1, 2])
        .listen(print); // prints 1, 2, 3
    ```

`startWithMany`는 `startValues`를 `stream`의 데이터 앞에 붙여준다.

> `startWith`랑 거의 같지만 단일이 아닌 `list`형태의 데이터를 앞에 붙임.

---

## switchIfEmpty method

```java
Stream<T> switchIfEmpty (
    Stream<T> fallbackStream
)
```

+ Example : 
    ```java
    // Let's pretend we have some Data sources that complete without
    // emitting any items if they don't contain the data we're looking for
    Stream<Data> memory;
    Stream<Data> disk;
    Stream<Data> network;

    // Start with memory, fallback to disk, then fallback to network.
    // Simple as that!
    Stream<Data> getThatData =
        memory.switchIfEmpty(disk).switchIfEmpty(network);
    ```

`switchIfEmpty`는 `stream`에 데이터가 없을 경우 `fallbackStream`으로 교체한다.

> 예시에서는 `memory`->`disk`->`network`로 데이터가 없을 경우이다. `Repository Pattern`에서 유용하다.

---

## switchMap method

```java
Stream<S> switchMap <S>(
    Stream<S> mapper(
        T value
    )
)
```

+ Example : 
    ```java
    RangeStream(4, 1)
        .switchMap((i) =>
            TimerStream(i, Duration(minutes: i))
        .listen(print); // prints 1
    ```

`switchMap`은 `stream`으로 부터 들어온 데이터를 `mapper`를 활용해서 새로운 `stream`을 만든다. 
특이점은 가장 최신 데이터만 받고 나머지 `stream`은 정지시킨다. 

> `flatMap`과 유사하지만 특이점은 최신 데이터만 받는다는점이다. `비동기 API`에서 가장 최신 데이터를 원할때 유용하다.

---

## takeUntil method

```java
Stream<T> takeUntil <S>(
    Stream<S> otherStream
)
```

+ Example : 
    ```java
    MergeStream([
        Stream.fromIterable([1]),
        TimerStream(2, Duration(minutes: 1))
    ])
        .takeUntil(TimerStream(3, Duration(seconds: 10)))
        .listen(print); // prints 1
    ```

`takeUntil`은 `skipUntil`과 반대로 `otherStream`이 데이터를 출력할 때까지만 데이터를 출력한다.

---

## debounce method

```java
Stream<T> debounce (
    Stream window(
        T event
    )
)
```

+ Example : 
    ```java
    Stream.fromIterable([1, 2, 3, 4,5,6,7,8,9]).interval(Duration(milliseconds: 500))
        .debounce((_) => TimerStream(true, Duration(seconds: 1)))
        .listen(print); // prints 4
    ```

`debounce`는 여러 데이터 요청이 겹쳐오는 경우 `debounce`의 값이 `True`이면서 `stream`에서 새로운 데이터가 들어오지 않을 때만 데이터를 출력한다.

예를 들어 클릭 이벤트의 경우 수많은 이벤트가 발생하지만 `debounce`에서 `Duration`을 `5ms`로 하면
첫 입력이 들어오고 `5ms`동안 새로운 입력이 없어야 이벤트를 발생시킨다.

> `debounce`에서 `(x) => TimerStream` 을 사용 하면 `x`에는 메인 `stream`의 값들이 들어온다

> `throttle`과 비교해서 공부하자

--- 

## debounceTime method

```java
Stream<T> debounceTime (
    Duration duration
)
```

+ Example : 
    ```java
    Stream.fromIterable([1, 2, 3, 4])
        .debounceTime(Duration(seconds: 1))
        .listen(print); // prints 4
    ```

`debounceTime`은 `debounce`와 똑같이 여러 요청을 한번만 처리하게 해준다. 여기서는 `Duration` 시간만큼만 대기한다.

---

## throttle method

```java
Stream<T> throttle (
    Stream window(
        T event
    ),
    {bool trailing: false}
)
```

+ Example : 
    ```java
    Stream.fromIterable([1, 2, 3])
        .throttle((_) => TimerStream(true, Duration(seconds: 1)))
    ```

`throttle`은 `window stream`이 `onDone`일때 까지 데이터를 `queue`에 모아서 첫 데이터를 출력해준다. 즉 특정 시간동안 이벤트가 한번만 발생한다.

`trailing`을 `true`로 하면 `queue`에서 가장 마지막 데이터를 꺼내준다.


>`throttle`은 `debounce`와 비교해보면 좋다.
 `debounce` : 첫 입력이 들어오면 주어진 시간동안 새로운 입력이 들어오지 않아야 이벤트를 처리한다.
 `throttle` : 주어진 시간동안 입력을 한번만 처리한다. 


```Dart  
Stream.fromIterable([1, 2, 3, 4, 5, 6, 7, 8, 9, 10])
    .interval(Duration(milliseconds: 500))
    .throttle((data) {
        print('throttle data =$data');
        return TimerStream(true, Duration(seconds: 1));
    }, trailing: false).listen(print);
```

> 내가 혼자 예습중 이상동작을 발견했다... 
설명대로라면 `1,3,5,7,9`만 출력해야하지만 가장 마지막 데이터인 `10`도 출력된다... `backpressure.dart`를 읽어봤더니 `window`의 `onDone`이 출력되어야 `queue`가 정리가 되는데 마지막 데이터에서는 `main Stream`의 `onDone`이 출력되어서 문제가 생긴다. 
현재 `github issue`에 등록해놓았다.

---

## throttleTime method

```java
Stream<T> throttleTime (
    Duration duration,
    {bool trailing: false}
)
```

+ Example : 
    ```java
    Stream.fromIterable([1, 2, 3])
        .throttleTime(Duration(seconds: 1))
    ```

`throttleTime`은 `trottle`에 `window`대신에 `Duration`으로 설정한다. 동작 내용은 똑같다.

---

## timeInterval method

```java
Stream<TimeInterval<T>> timeInterval ()
```

+ Example : 
    ```java
    Stream.fromIterable([1])
        .interval(Duration(seconds: 1))
        .timeInterval()
        .listen(print); // prints TimeInterval{interval: 0:00:01, value: 1}
    ```

`timeInterval`은 `interval`처럼 데이터가 시간을 두고 연속적으로 오는 경우 시간을 기록해준다. (일반 `stream`도 사용가능)

> 데이터의 형태가 `TimerInterval`로 묶인다. 
`TimeInterval.interval` : `interval` 시간 정보
`TimerInterval.value` : 데이터 정보

---

## timestamp method

```java
Stream<Timestamped<T>> timestamp ()
```

+ Example : 
    ```java
    Stream.fromIterable([1])
        .timestamp()
        .listen((i) => print(i)); // prints 'TimeStamp{timestamp: XXX, value: 1}';
    ```

`timestamp`는 데이터 출력의 시간을 기록해준다. 

> `TimeStamp{timestamp: 2020-03-20 14:16:10.224771, value: 1}`와 같은 형태로 출력된다.

---

## whereType method

```java
Stream<S> whereType <S>()
```

+ Example : 
    ```java
    Stream.fromIterable([1, 'two', 3, 'four'])
        .whereType<int>()
        .listen(print); // prints 1, 3

    //as opposed to:
    Stream.fromIterable([1, 'two', 3, 'four'])
        .where((event) => event is int)
        .cast<int>()
        .listen(print); // prints 1, 3
    ```

`whereType`은 주어진 `<S> Type`을 보고 맞지 않는 것은 버리고 출력해준다.

> `where`과 `cast`를 활용해 똑같이 작성할 수 있다.

---

## window method

```java
Stream<Stream<T>> window (
    Stream window
)
```

+ Example : 
    ```java
    Stream.periodic(Duration(milliseconds: 100), (i) => i)
        .window(Stream.periodic(Duration(milliseconds: 160), (i) => i))
        .asyncMap((stream) => stream.toList())
        .listen(print); // prints [0, 1] [2, 3] [4, 5] ...
    ```

`window`는 [buffer](https://funncy.github.io/flutter/2020/03/19/rxdart-04/#buffer-method)와 매우 유사하지만 `buffer`는 데이터를 모아서 출력해준다면 여기는 데이터를 모아서 `stream`으로 출력해준다. 

---

## windowCount method

```java
Stream<Stream<T>> windowCount (
    int count,
    [int startBufferEvery = 0]
)
```

+ Example : 
    ```java
    RangeStream(1, 4)
        .windowCount(2)
        .asyncMap((stream) => stream.toList())
        .listen(print); // prints [1, 2], [3, 4] done!


    RangeStream(1, 5)
        .bufferCount(3, 2)
        .listen(print); // prints [1, 2, 3], [3, 4, 5], [5] done!
    ```

`windowCount`는 [bufferCount](https://funncy.github.io/flutter/2020/03/19/rxdart-04/#buffercount-method)와 비슷하게 `count`를 지정해 데이터를 모아서 출력한다. 단 `stream`형태로 출력한다. 

`startBufferEvery`는 다음 `buffer`의 시작점을 지정해준다. 기본값은 `0`이며 `2`로 지정할 경우 위의 예제처럼 동작한다. (`buffer`출력 후 메인 데이터의 `index`처리 후 다시 시작)

---

## windowTest method

```java
Stream<Stream<T>> windowTest (
    bool onTestHandler(
        T event
    )
)
```

+ Example : 
    ```java
    RangeStream(1, 10)
        .windowTest((i) => i % 2 == 0)
        .asyncMap((stream) => stream.toList())
        .listen(print); // prints [1, 2], [3, 4] [5, 6] ...
    ```

`windowTest`는 주어진 `onTestHandler`함수가 `true`를 반환할때 까지 값을 `buffer`에 모아서 `stream`형태로 반환해준다.

---

## withLatestFrom method


```java
Stream<R> withLatestFrom <S, R>(
    Stream<S> latestFromStream,
    R fn(
        T t,
        S s
    )
)
```

+ Example : 
    ```java
    Stream.fromIterable([1, 2]).withLatestFrom(
    Stream.fromIterable([2, 3]), (a, b) => a + b)
        .listen(print); // prints 4 (due to the async nature of streams)
    ```

`withLatestFrom`은 여러개의 `stream`을 조합해서 출력한다. `latestFromStream`의 값이 출력되면 가장 마지막에 들어온 메인 `stream`의 값과 주어진 `fn`에 따라 조합되서 출력된다.

> 여기서는 메인 `stream`이 아니라 `latestFromStream`이 메인이다. `latestFromStream`에서 데이터가 출력되지 않으면 아무것도 출력되지 않는다.

![이미지](https://Funncy.github.io/assets/img/rxdart/2020-03-20-rx-dart-05-01.png "withLatestFrom")

> `withLatestForm`은 종류가 여러개이다. 
`withLatestFrom`, `withLatestFrom2` ... `withLatestFrom9` 까지 있으며 List형태를 받는
`withLatestFromList`가 있다.
참고로 `withLatestFromList`는 `fn`을 지정할 수 없다.

---

## zipWith method

```java
Stream<R> zipWith <S, R>(
    Stream<S> other,
    R zipper(
        T t,
        S s
    )
)
```

+ Example : 
    ```java
    Stream.fromIterable([1, 3, 4, 5, 6])
        .zipWith(Stream.fromIterable([2, 10, 11, 12]), (one, two) => one + two)
        .listen(print); // prints 3 , 13, 15, 17
    ```

`zipWith`은 `withLatestFrom`과 비슷하게 데이터를 모아서 `fn`를 통해 출력한다.
다른점은 `withLatestFrom`은 메인 `stream`의 마지막 데이터와 조합해서 출력했다면 `zipWith`는 현재 데이터와 조합해서 출력한다.

---

#### 드디어 extension method를 다 정리했다..

기나고 긴 여정이었다 ㅠㅠ 이제 정리된거를 공통 분모로 나눠보고 해야겠다..... (소셜 로그인도 해야하는데...)