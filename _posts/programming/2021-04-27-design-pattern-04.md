---
layout: post
title: '[Programming] - 디자인 패턴 04 : Visitor 패턴 '
subtitle: 'design pattern visitor'
categories: programming
tags: designpattern visitor
comments: true
---

[참고] : [https://thecodinglog.github.io/design/2019/10/29/visitor-pattern.html](https://thecodinglog.github.io/design/2019/10/29/visitor-pattern.html)

## 방문자 패턴이란?

- 실제 로직을 가지고 있는 객체(Visitor)가 로직을 적용 할 객체(Element)를 방문하면서 실행하는 패턴

> 말이 너무 어렵다...

정리해보면 보통 OOP에서는 객체가 그 객체가 하는 행동을 함수로 가지고 있다.

그리고 그 행동의 대상이 되는 객체가 있을 경우, 파라미터로 입력 받아서 실행한다.

그런데, 방문자 패턴은 행동의 대상이 되는 객체가 행동을 일으키는 객체를 입력으로 받는다.

**예시**

"나는 상점에 방문한다. 나는 '구매'를 한다."

'나'라는 객체가 '상점'이라는 객체를 입력받은 후, 이 상점에서 '구매'를 한다. → 일반적인 OOP

"상점에 내가 방문했다. 내가 '구매'를 하게 한다."

**'상점'이라는 객체가 '나'라는 객체에 입력받은 후, '나'라는 객체의 행동을 호출하는 것이다.**

이 때, 상점에 대한 정보를 파라미터로 넘겨준다.

즉, 사용자는 방문자의 입장이 아니라, **방문 공간의 입장에서 먼저 생각해보게 된다.**

[출처] : [https://dailyheumsi.tistory.com/216](https://dailyheumsi.tistory.com/216)

## 예시 - 쇼핑몰 회원 등급제

등급별로 고객이 받을 혜택이 다르다. 확장 가능한 해결책을 찾아보자.

### 직접 구현

```dart
abstract class Member {
  void point();
  void discount();
}

class GoldMember implements Member {
  void point() {
    print("Gold Point 적립되었습니다.");
  }

  void discount() {
    print("Gold Discount 적용되었습니다.");
  }
}

class VipMember implements Member {
  void point() {
    print("Vip Point 적립되었습니다.");
  }

  void discount() {
    print("Vip Discount 적용되었습니다.");
  }
}

void main() {
  Member goldMember = GoldMember();
  Member vipMember = VipMember();

  goldMember.point();
  vipMember.point();
  goldMember.discount();
  vipMember.discount();
}
```

### 문제점

- 고객들을 순회하며 혜택을 주고자 할 때 명시적으로 혜택을 주기 위해 메소드를 호출해야 한다.
  순회 처리가 안된다.
- 혜택이 늘어났을 때 모든 멤버에 대해서 그 혜택을 **구현했다는 보장**이 없다.

### 방문자 패턴 구현

1. 혜택별 Benefit 인터페이스 및 구현

```dart
abstract class Benefit {
  void getBenefit(Member member);
}

class DiscountBenefit implements Benefit {
  void getBenefit(Member member) {
    if(member is GoldMember) {print("Discount for Gold Member");}
    else if(member is VipMember) {print("Discount for Vip Member");}
  }
}

class PointBenefit implements Benefit {
  void getBenefit(Member member) {
    if(member is GoldMember) {print("Point for Gold Member");}
    else if(member is VipMember) {print("Point for Vip Member");}
  }
}
```

> Dart는 메소드 오버로딩이 되지 않으므로 위와 같이 처리

2. Member에 혜택을 받을 수 있는 메소드 추가

```dart
abstract class Member {
  void getBenefit(Benefit benefit);
}

class GoldMember implements Member {
  void getBenefit(Benefit benefit) {
    benefit.getBenefit(this);
  }
}
class VipMember implements Member {
  void getBenefit(Benefit benefit) {
    benefit.getBenefit(this);
  }
}
```

3. 호출

```dart
void main() {
  Member goldMember = GoldMember();
  Member vipMember = VipMember();
  Benefit pointBenefit = PointBenefit();
  Benefit discountBenefit = DiscountBenefit();

  goldMember.getBenefit(pointBenefit);
  vipMember.getBenefit(pointBenefit);
  goldMember.getBenefit(discountBenefit);
  vipMember.getBenefit(discountBenefit);
}
```

> 새로운 GreenMember를 추가한다고 가정했을 때
> 구현이 완료되지 않으면 benefit.getBenefit(this);에서 컴파일 오류가 발생한다.
> 구현 누락을 방지할 수 있다.

## 언제 써야 할 까?

1. 대상 객체가 잘 바뀌지 않고 (갯수) : 예시) Gold, Vip Member 등
2. 알고름이 추가될 가능성이 많은 : 예시) 적립 알고리즘 추가, 프로모션 알고리즘 추가 등

![이미지](https://Funncy.github.io/assets/img/programming/2021-04-27-pattern-05.png 'Visitor Pattern')
