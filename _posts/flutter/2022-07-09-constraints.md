---
layout: post
title: '[Flutter] - Constraints Layout'
subtitle: 'Flutter Constraints layout'
categories: flutter
tags: flutter constraints layout renderObject
comments: true
---

플러터로 개발하면서 항상 궁금했던 Constraints를 정리해보고자 한다.

[Understanding constraints](https://docs.flutter.dev/development/ui/layout/constraints)

공식문서를 기준으로 정리해보자.

아래 공식을 이해해야 Constraints를 이해한다고 한다.

<aside>
⚠️ **Constraints go down. Sizes go up. Parent sets position.**

</aside>

먼저 핵심 포인트들을 정리해보자.

- Widget은 부모로부터 제약사항(Constraints)를 받는다. 제약사항이란 최대 너비, 최소 너비, 최대 높이, 최소 높이이다. **(Constraints go down)**
- 그런다음 Widget은 자식들에게 한명씩 Constraints를 내려준다. 각각 자식마다 다르게 내려갈 수 있다. (해당 자식이 차지 하는 만큼 빼고 줘야한다) 그리고 Widget이 **자식에게 어떤 사이즈가 될거냐고 묻는다. (Sizes go up)**
- 그런 다음 Widget이 자식들의 위치를 잡는다. (x,y 좌표) **(Parent sets position)**
- 그리고 최종적으로 Widget이 부모에게 자신의 사이즈를 전달한다. (재귀적으로 돈다는 의미)

위의 제약사항을 공식문서에서는 아주 재미있게 대화형으로 표현하고 있다.

한번 살펴보자.

### Constraints 진행 flow

![layout](https://user-images.githubusercontent.com/22514912/178112401-3a65a8ce-0f8c-43de-acf1-c39048575fa4.png)

1. **Constraints go down**
    - Widget : “부모님 제 제약사항은 무엇인가요?”
    - Parent : “너는 80~300 너비이고, 30~85 높이야. 꼭 지켜야해!”
    - Widget : “흠… 내가 패딩이 5가 있으니…(all) 너비는 그럼 최대 290이고, 높이는 75구나!”
    1. **Constraints go down**
        - Widget : “첫번째 자식아! 너는 무조건 0~290 너비이어야 하고 높이는 0~75여야 한단다”
    2. **Sizes go up**
        - First Child : “좋아요, 저는 너비는 290으로 하구요 높이는 20할래요”
    3. **Constraints go down**
        - Widget : “흠… 두번째 자식을 첫번째 자식 아래 넣어야 하니… 높이가 이제 55 안에 들어와야겠군!”
        - Widget : “두번째 자식아! 너는 무조건 0~290 너비이어야 하고 높이는 0~55 란다”
    4. **Sizes go up**
        - Second Child : “좋아요, 저는 너비는 140으로 하구요 높이는 30 할래요”
    5. **Parent sets position**
        - Widget : “좋아! 첫번째 자식의 위치는 x: 5, y:5 이고, 두번째 자식은 x: 80, y: 25 란다!”
2. **Sizes go up**
    - Widget : “부모님 이제 저 Size 계산이 끝났어요! 저 너비는 300이구요 높이는 60이에요!”

추가적으로 여기에 **중요한 제한사항**이 있다고 말한다.

- 위젯은 부모의 제약사항을 받아야만 사이즈를 결정할 수 있다. 즉, 일반적으로 내가 원하는 사이즈를 가져갈 수 없다.
- 위젯의 위치는 부모 위젯이 결정합니다. 스스로 결정할 수 없습니다.
- 부모 위젯또한 상위 위젯으로부터 크기가 결정되기 때문에 전체 트리 없이 위젯의 위치를 정할 수 없습니다.
- 만약 자녀위젯이 부모위젯과 다른 사이즈를 원한다면 align정보가 필요합니다. 없다면 사이즈가 무시됩니다.

## 예제들

---

전체 예제를 정리하기에는 너무 많아 몇개만 정리해보고자 한다.

(자세한건 공식문서에서…)

```jsx
Container(color: red)
```

- 최상위에 선언된다면 최상위인 Screen의 제약사항을 따라 전체 화면 사이즈 만큼 Container가 그려지게 됩니다.

```jsx
Container(width: 100, height: 100, color: red)
```

- 여기서도 최상위 위젯이 무엇이냐에 따라 달라집니다. 만약 부모위젯이 Screen이라면 강제적으로 전체 사이즈를 따라가게 만듭니다. 즉, Container의 사이즈가 무시됩니다.

```jsx
Center(
  child: Container(width: 100, height: 100, color: red),
)
```

- 이제서야 Container가 Center의 제약사항을 받아와서 자신이 그리고 싶은 사이즈를 그릴 수 있습니다.

```jsx
Align(
  alignment: Alignment.bottomRight,
  child: Container(width: 100, height: 100, color: red),
)
```

- 이번에도 원하는대로 그려집니다. alignment를 준대로 그려집니다.

```jsx
Center(
  child: Container(
      width: double.infinity, height: double.infinity, color: red),
)
```

- Center는 Screen의 제약사항을 받아 화면 전체를 잡고 있습니다. Container는 Center의 제약사항 안에서 자유롭게 그릴 수 있지만 너비와 높이를 최대한으로 가진다고 하여 전체 화면을 다 채우게 됩니다.

```jsx
Center(
  child: Container(color: red),
)
```

- Center는 전체 스크린의 제약사항을 가집니다. 만약 Container가 자식도 없고 자체적인 너비 높이가 없다면 Container는 부모의 너비 높이를 가득 채우게 됩니다.

```jsx
UnconstrainedBox(
  child: Container(color: red, width: 4000, height: 50),
)
```

- UnconstrainedBox가 Screen의 제약사항을 받고 자식에게는 자유롭게 사이즈를 가질 수 있게 해줍니다.
- 하지만 스크린 사이즈보다 width가 커서 화면이 overflow error가 발생하게 됩니다.

```jsx
OverflowBox(
  minWidth: 0.0,
  minHeight: 0.0,
  maxWidth: double.infinity,
  maxHeight: double.infinity,
  child: Container(color: red, width: 4000, height: 50),
)
```

- OverflowBox는 UnconstrainedBox와 유사하지만 다른 점은 overflow가 나도 error가 발생하지 않고 그냥 보여줄 수 있는 부분만 보여줍니다.

```jsx
UnconstrainedBox(
  child: Container(color: Colors.red, width: double.infinity, height: 100),
)
```

- 제약사항을 제거하고 width를 무제한으로 그리게 하니 error가 발생합니다.

```jsx
UnconstrainedBox(
  child: LimitedBox(
    maxWidth: 100,
    child: Container(
      color: Colors.red,
      width: double.infinity,
      height: 100,
    ),
  ),
)
```

- 하지만 위처럼 LimitedBox로 감싸서 제약사항을 추가하면 문제가 사라집니다.
- 그러나 만약 UnconstrainedBox를 Center 위젯으로 변경하면 LimitedBox가 동작하지 않습니다.
LimitedBox는 부모로부터 받은 제약사항이 없어 무제한으로 그리려고 할때 동작합니다.

```jsx
const FittedBox(
  child: Text('Some Example Text.'),
)
```

- FittedBox는 Screen의 제약사항을 받아 화면크기에 무조건 맞춰집니다. Text는 본인의 사이즈가 FontSize, text에 맞춰서 생성됩니다.
- 하지만 FittedBox가 Text 위젯을 자신의 남아있는 너비만큼 강제적으로 키웁니다.

```jsx
const Center(
  child: FittedBox(
    child: Text('Some Example Text.'),
  ),
)
```

- 하지만 만약 최상단이 Center 위젯으로 감싸져있으면 FittedBox는 동작하지 않습니다.
- FittedBox는 Text와 같은 사이즈를 가집니다.

```jsx
const Center(
  child: FittedBox(
    child: Text(
        'This is some very very very large text that is too big to fit a regular screen in a single line.'),
  ),
)
```

- 만약 텍스트가 화면보다 길어지면 어떻게 될까요?
- FittedBox가 알아서 화면 사이즈 맞춰 Text사이즈를 줄여줍니다.

<aside>
💡 사실 내가 정말 원하던 기능인데 여기서 찾았다

</aside>

- 만약 FittedBox가 없다면 2줄로 내려가서 글이 작성됩니다.

```jsx
FittedBox(
  child: Container(
    height: 20.0,
    width: double.infinity,
    color: Colors.red,
  ),
)
```

- FittedBox에 무한 너비와 높이를 넣으면 에러가 발생합니다.

```jsx
Row(
  children: [
    Container(color: red, child: const Text('Hello!', style: big)),
    Container(color: green, child: const Text('Goodbye!', style: big)),
  ],
)
```

- Row는 UnconstrainedBox와 같이 자식들에게 제약사항을 넘기지 않습니다.
- Row는 현재 Screen의 제약사항에 따라 전체 화면을 차지하고 있으며, 남은 공간은 여백으로 가지고 있습니다.

```jsx
Row(
  children: [
    Container(
      color: red,
      child: const Text(
        'This is a very long text that '
        'won\'t fit the line.',
        style: big,
      ),
    ),
    Container(color: green, child: const Text('Goodbye!', style: big)),
  ],
)
```

- 그렇기 때문에 Row는 UnconstrainedBox와 마찬가지로 전체 너비보다 길어지면 overflow 에러를 발생시킵니다.

```jsx
Row(
  children: [
    Expanded(
      child: Center(
        child: Container(
          color: red,
          child: const Text(
            'This is a very long text that won\'t fit the line.',
            style: big,
          ),
        ),
      ),
    ),
    Container(color: green, child: const Text('Goodbye!', style: big)),
  ],
)
```

- 만약 Row 안에 Expanded를 사용하면 기존처럼 자식이 스스로 자신의 너비를 결정하지 못합니다.
- 대신 다른 자식들이 사용하고 남은 너비를 자신의 너비로 가져가며, Expanded 내부 자식의 사이즈 또한 그것에 맞춰집니다.

```jsx
Row(
  children: [
    Flexible(
      child: Container(
        color: red,
        child: const Text(
          'This is a very long text that won\'t fit the line.',
          style: big,
        ),
      ),
    ),
    Flexible(
      child: Container(
        color: green,
        child: const Text(
          'Goodbye!',
          style: big,
        ),
      ),
    ),
  ],
)
```

- Expanded는 자식의 너비를 자식에게 강제한다면, Flexible은 자신의 영역과 같거나 작도록 자식의 너비를 지정합니다. (즉, 너무 큰 자식은 강제로 자기에게 맞추고 작은 자식은 작은 대로 그대로 그립니다. Expanded는 무조건 나에게 맞춥니다.)

> Row는 2가지 옵션이 있습니다.
1. 제약사항 없이 자식이 스스로 사이즈를 지정한다.
2. Expanded, Flexible을 사용할 경우 제약사항에 따라 사이즈를 지정한다.
> 

```jsx
Scaffold(
  body: Container(
    color: blue,
    child: Column(
      children: const [
        Text('Hello!'),
        Text('Goodbye!'),
      ],
    ),
  ),
)
```

- Scaffold가 Screen의 제약사항에 따라 화면 전체에 맞춰집니다.
- Scaffold는 자식에게 화면보다 작게 너가 원하는 사이즈가 되라고 합니다. (느슨한 제약사항)
- Container도 자식에게 너가 원하는 사이즈 되라고 합니다. (제약사항 내려감)
- Column은 높이는 전체를 다 먹습니다. 하지만 너비는 내 자식들에게 물어봅니다. 즉, Text중 가장 긴 녀석의 너비 만큼이 사이즈가 됩니다.
- 사이즈 결정이 위로 올라가며 Container의 너비는 Text만크 높이는 화면 전체가 됩니다.

```jsx
Scaffold(
  body: SizedBox.expand(
    child: Container(
      color: blue,
      child: Column(
        children: const [
          Text('Hello!'),
          Text('Goodbye!'),
        ],
      ),
    ),
  ),
)
```

- 만약 Scaffold의 자식이 Scaffold처럼 전체 화면에 맞춰지려면, SizedBox.expand를 사용하면 됩니다.

> 제약사항이 특정 사이즈보다 작으면 된다고 하면 **느슨한 제약사항**이라고 하고,
특정 사이즈가 되어야한다고 하면 **엄격한 제약사항**이라고 합니다. (looese vs tight)
> 

### 엄격한 제약사항 vs 느슨한 제약사항

---

엄격한 제약사항이란 특정 사이즈를 강제하는 경우를 말합니다. 

다른 말로 maxWidth == minWidth, maxHeight == minHeight 인 경우 입니다.

```jsx
BoxConstraints.tight(Size size)
   : minWidth = size.width,
     maxWidth = size.width,
     minHeight = size.height,
     maxHeight = size.height;
```

대표적으로 위의 예시에서 Screen이 그랬습니다. (화면 사이즈를 강제로 채워!)

느슨한 제약사항이란 특정 사이즈 이하로는 괜찮은 경우를 말합니다.

다른 말로 maxHeight와 maxWidth가 정해져있고 minHeight = minHeight = 0 인 경우 입니다.

```jsx
BoxConstraints.loose(Size size)
   : minWidth = 0.0,
     maxWidth = size.width,
     minHeight = 0.0,
     maxHeight = size.height;
```

여기서 재미있는 점은 **Center** 위젯의 숨겨진 역할은 바로 **tight constraints**(엄격한 제약사항)을 **loose constraints**(느슨한 제약사항)으로 **변경**한다는 것 입니다.

### 특정한 위젯들의 레이아웃 동작 학습 방법

---

사실 결론은 문서 봐라 이지만 만약 소스코드로 공부하려면 코드를 까서 찾아봐라라며 친절하게 예시를 보여준다. 

- Column의 내부를 들어가보면 Flex를 상속받았다.
- Flex를 들어가서 createRenderObject() 를 찾으면 RenderFlex를 따라 들어갈 수 있다.
- 거기서 performLayout()를 살보면 어떻게 레이아웃이 동작하는지 살펴볼수 있다.

```jsx
...
switch (_mainAxisAlignment) {
      case MainAxisAlignment.start:
        leadingSpace = 0.0;
        betweenSpace = 0.0;
        break;
      case MainAxisAlignment.end:
        leadingSpace = remainingSpace;
        betweenSpace = 0.0;
        break;
      case MainAxisAlignment.center:
        leadingSpace = remainingSpace / 2.0;
        betweenSpace = 0.0;
        break;
      case MainAxisAlignment.spaceBetween:
        leadingSpace = 0.0;
        betweenSpace = childCount > 1 ? remainingSpace / (childCount - 1) : 0.0;
        break;
      case MainAxisAlignment.spaceAround:
        betweenSpace = childCount > 0 ? remainingSpace / childCount : 0.0;
        leadingSpace = betweenSpace / 2.0;
        break;
      case MainAxisAlignment.spaceEvenly:
        betweenSpace = childCount > 0 ? remainingSpace / (childCount + 1) : 0.0;
        leadingSpace = betweenSpace;
        break;
    }
...
```

예를 들면 위처럼 Column Row의 AxisAlignment를 살펴볼 수 있다.

## 마무리

---

일단 새로운 사실은 한국어 공식문서에는 없던 내용을 영어 문서에서 발견했다는 거고

(한국어 사이트도 사실 영어로 되있는 자료가 많은데… 왜 안넣어줬을까…)

내용이 너무 수준이 만족스러웠다

그동안 그냥 존재만 알고 지나갔던 Constraints에 대해서 좀 깊게 알아봐서 좋았고

재귀적으로 Flutter가 어떻게 사이즈와 위치를 잡는지 알게되었다.