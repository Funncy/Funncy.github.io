---
layout: post
title: '[Flutter] - BuildContext '
subtitle: 'Flutter buildContext'
categories: flutter
tags: flutter buildcontext
comments: true
---

Flutter를 개발하다보면 context와 자주 만나게 된다.

Theme.of(context)

provider.of(context)

MediaQuery.of(context) ...

이 BuildContext는 무엇을 하는 놈일까?

### 공식 문서

---

공식 문서에 따르면 위젯 트리에서 위젯의 위치를 관리해준다고 나온다.

flutter공식 영상을 보면

BuildContext를 상속받아 구현한것이 Element이고

Widget이 만들어지면서 Element가 생성된다.

Flutter tree 구조를 보면

Widget Tree → Element Tree → Render Object Tree 가 있는데

여기서 Element Tree에 각각 Node들이 BuildContext인것이다. (추상클래스를 상속했다)

```tsx
class BuildContextExample extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    print("BuildContextExample Context : ${context.hashCode}");
    return Builder(
      builder: (context) {
        print("Builder Context :${context.hashCode}");
        return FloatingActionButton(onPressed: () {
        Scaffold.of(context).showSnackBar(
          new SnackBar(
            backgroundColor: Colors.blue,
            content: Builder(builder: (context) {
              print("SnackBar Context : ${context.hashCode}");
              return Text('SnackBar}');
            }),
          ),
        );
      });
      },
    );
  }
}
```

```tsx
# result

flutter: BuildContextExample Context : 143
flutter: Builder Context :421
flutter: SnackBar Context : 722
```

각각의 위젯별로 BuildContext가 고유하게 생성된다.

builder 위젯을 사용하지 않으면 하나의 context를 공유한다.

> 바로 하나의 context를 공유하기에 발생하는 문제들이 있다... 마지막에 알아보자.

### .of(context)

---

.of함수는 무엇을 하는 함수일까?

```tsx
static MediaQueryData of(BuildContext context) {
    assert(context != null);
    assert(debugCheckHasMediaQuery(context));
    return context.dependOnInheritedWidgetOfExactType<MediaQuery>()!.data;
  }
```

MediaQuery의 of함수를 타고 들어가보면 위와 같다.

여기서 .of()의 공통점은

`context.dependOnInheritedWidgetOfExactType<T>()` 이다.

BuildContext를 통해서 Widget을 탐색해 찾아주는 것이다.

즉, Theme, MediaQuery, Provider등 위젯 트리에 속해져 있다면

그것을 찾아서 반환해주는 구조이다.

> 그래서 항상 Provider를 상위 위젯에 넣어주고 시작한다.

### 주의 할 포인트

---

여기서 많이들 만나는 문제를 살펴보자.

```tsx
@override
Widget build(BuildContext context) {
  return Scaffold(
    appBar: AppBar(title: Text('BuildContext')),
    body:FlatButton(
          child: Text('button'),
          onPressed: () {
            // 여기서 에러가 발생한다.
            Scaffold.of(context).showSnackBar(SnackBar(
              content: Text('Error??')
            ));
          }
        )
  );
}
```

다음과 같은 에러가 발생한다.

`Scaffold.of() called with a context that does not contain a Scaffold.`

해석하면 Scaffold.of()에서 쓰인 context에는 Scaffold가 담겨있지 않아~

분명 Scaffold가 있는데 왜 그럴까?

그 이유는 다음과 같다.

build method가 호출되어서 들어온 context는 return되는 Scaffold의 부모 Widget의 BuildContext이다.
(build 함수가 실행되는 시점에 아직 Scaffold는 만들어져 있지 않다)

즉, Scaffold가 만들어져서 BuildContext화 되기전에 Scaffold.of(context)로 찾으려고 하니

찾을수 없다는 에러가 나온다.

그렇다면 Scaffold의 BuildContext가 붙은다음 자식의 BuildContext에서 찾아보면 찾을 수 있다.

이 부분은 builder method로 해결이 가능하다.

```tsx
@override
Widget build(BuildContext context) {
  return Scaffold(
    appBar: AppBar(title: Text('BuildContext')),
    body: Builder(
      builder: (BuildContext context) {
				//여기서 context는 Scaffold의 Buildcontext를 찾아올 수 있다.
        return FlatButton(
          child: Text('button'),
          onPressed: () {
            // Scaffold 만들어진 이후에 BuildContext라서 찾아갈 수 있다.
            Scaffold.of(context).showSnackBar(SnackBar(
              content: Text('Error??')
            ));
          }
        );
      }
    )
  );
}
```
