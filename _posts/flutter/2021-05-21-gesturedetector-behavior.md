---
layout: post
title: '[Flutter] - GestureDetector behavior '
subtitle: 'Flutter gestureDetector behavior'
categories: flutter
tags: flutter GestureDetector behavior
comments: true
---

**child 가 null 이 아닌 경우 기본 값 = _[HitTestBehavior.deferToChild]_**

**child 가 null 인 경우 기본 값 = _[HitTestBehavior.translucent]_**

```dart
/// How to behave during hit tests.
enum HitTestBehavior {
  /// Targets that defer to their children receive events within their bounds
  /// only if one of their children is hit by the hit test.
  deferToChild,

  /// Opaque targets can be hit by hit tests, causing them to both receive
  /// events within their bounds and prevent targets visually behind them from
  /// also receiving events.
  opaque,

  /// Translucent targets both receive events within their bounds and permit
  /// targets visually behind them to also receive events.
  translucent,
}
```

## **deferToChild (기본 값)**

- 자식을 갖고있는 대상은 자식 중 하나가 hit test에 해당하는 경우에만 해당 범위 내에서 이벤트를 받는다.

## **opaque**

- 불투명 한 대상은 hit test에 의해 hit 될 수 있으므로 둘 다 해당 범위 내에서 이벤트를 수신하고 시각적으로 뒤에 있는 대상도 이벤트를 수신하지 못하도록 한다.

## **translucent**

- 반투명 타겟은 경계 내에서 이벤트를 수신하고 시각적으로 뒤에 있는 타겟도 이벤트를 수신하도록 허용한다.

## 나의 상황

---

현재 나의 앱을 만들던 도중

GestureDetector를 통해 좋아요 버튼을 구현하였다.

그런데 하트와 텍스트 사이 구간을 클릭하면 (빈 공간이다, 정확히는 SizedBox가 있다) 해당 버튼이 아니라
뒤에 배경이 클릭되며 페이지 전환이 이뤄진다. (내가 원하는건 좋아요 버튼 클릭이다)

그래서 위와 같은 Behavior 옵션을 통해 opaque를 통해 빈 공간도 클릭 이벤트를 수신하도록 수정하였다.
