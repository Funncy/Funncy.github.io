---
layout: post
title: '[Flutter] - Widget Tree, Element Tree, RenderObject Tree '
subtitle: 'Flutter widget tree & element tree & renderobject tree'
categories: flutter
tags: flutter widgettree elementtree renderobjecttree
comments: true
---

# Widget

---

Flutter를 사용하면 가장 많이 쓰게 되는 Widget이다. Flutter는 모두 Widget으로 되어있다.

EX)

```java
@override
  Widget build(BuildContext context) {
    return Container(
      child: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            RaisedButton(
              onPressed: emailSignUp,
              child: Text("EmailSignUp"),
            ),
            SizedBox(
              height: 10,
            ),
          ],
        ),
      ),
    );
  }
```

setState()를 통해서 Widget을 변경하며 화면을 다시 그린다.

> 그런데 Widget은 immutable이다.

즉, 변경된 화면을 위해서는 Widget을 새로 만들어줘야 한다.
그렇다면 화면을 다시 그릴 때 어떻게 동작할까?

# Element

---

Widget이 생성되면 그에 따라 Element도 생성이 된다.
Element는 변경 가능하며 변경내용에 따라 새로 생성하지 않아도 된다.

> BuildContext는 Element Tree에 담겨있다.

# Render Object

---

실제 UI를 그리기 위한 요소이다. Flutter는 화면을 그릴때 Widget이나 Element를 보는게 아니라 Render Object를 바라보고 화면을 그린다.
그리기 위한 정보가 담겨있다. ex) constaints, size 등등...

# All Tree

---

```java
String _text = "Hello Flutter";

@override
Widget build(BuildContext context) => Center(
	child: FlatButton(
		onPressed: () => setState(()=> _text = "Hey"),
		child: Text(_text),
	),
);
```

![이미지](https://Funncy.github.io/assets/img/flutter/2021-03-09-render-tree-01.png 'flutter render tree')

- Flutter는 build 함수를 통해 Widget Tree를 만든다.
- 매 Widget마다 createElement() 함수를 통해 Element Tree또한 생성된다.
- 각각 Element들도 createRenderObject() 함수를 통해 Render Object tree를 생성한다.

여기서 버튼을 누르면 Text는 변하게된다.

변하는 흐름을 살펴보자.

1. Element object가 Text Widget과 Render Object의 runtime type을 비교한다.
2. 서로 다른 경우
   - 새로운 Render Object를 생성한다.
3. 서로 같은 경우
   - 새로 생성하지 않고 updateRenderObject() 함수를 통해 업데이트를 해준다.

# State

---

![이미지](https://Funncy.github.io/assets/img/flutter/2021-03-09-render-tree-02.png 'flutter state tree')

stateless Widget 말고 StateFul widget에서는 어떻게 동작할까?

```java
class HomeScreen extends StatefulWidget {
  @override
  _HomeScreenState createState() => _HomeScreenState();
}

class _HomeScreenState extends State<HomeScreen> {

	String data = "State!";

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text("test app"),
      ),
      body: Container(
        child: Center(
					child : Text("data : ${data}"),
				),
      ),
    );
  }
}
```

위 내용에서 보듯이 Widget에서는 내용이 비어있고 바로 createState()함수를 호출해서 State를 생성한다.
위 이미지 처럼 Widget에 State즉 데이터가 있는게 아니라 Element에 State, 데이터가 존재한다.

> 그래서 Widge을 Swap하더라도 Type이 같으면 Element가 변경되지 않는 특성때문에
> State값이 변경되지 않을 수 있다.
> 이때 필요한 게 Key이다.
