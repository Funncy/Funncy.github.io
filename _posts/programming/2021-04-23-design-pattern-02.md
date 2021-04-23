---
layout: post
title: '[Programming] - 디자인 패턴 02 : 전략 패턴 '
subtitle: 'design pattern stratgey'
categories: programming
tags: designpattern stratgey
comments: true
---

## 전략패턴이란?

전략패턴이란 어떤 행위를 캡슐화해서 교체하기 쉽도록 만든 패턴이다.

특히 게임같은 곳에서 장비, 공격방식, 행동방식을 바꾸고 싶을때 유용하다.

만약 철권같은 게임을 만든다고 해보자.

![이미지](https://Funncy.github.io/assets/img/programming/2021-04-23-pattern-01.png 'Stratgey pattern 01')

위와 같이 Fighter class내부에서 Jump, Kick등이 정의되어있다.

그런데 **캐릭터마다 다른 kick과 jump를 설정해야한다면??** 어떻게 해야할까?

오바라이딩을 통해 함수를 변경해주어야 할 것이다.

캐릭터가 늘어날 수록 **유지보수가 어려워 질 것**이다.

![이미지](https://Funncy.github.io/assets/img/programming/2021-04-23-pattern-02.png 'Stratgey pattern 02')

위와 같이 **Interface를 활용**하여 Jump와 Kick을 빼줄 수 있다.

하지만 이것또한 **캐릭터별로 Jump와 Kick을 설정**해주야 하므로

**중복된 코드**가 넘쳐나게 된다.

![이미지](https://Funncy.github.io/assets/img/programming/2021-04-23-pattern-03.png 'Stratgey pattern 03')

전략패턴은 위와 같이 Behavior interface를 만들어서 Fighter 클래스가 **교체할 수 있도록 만든 패턴**이다.

코드로 봐야 명확할것 같다.

```dart
abstract class Fighter {
  KickBehavior kickBehavior;
  JumpBehavior jumpBehavior;

  Fighter(this.kickBehavior, this.jumpBehavior);

  setKickStrategy(KickBehavior kickBehavior) {
    this.kickBehavior = kickBehavior;
  }

  setJumpStrategy(JumpBehavior jumpBehavior) {
    this.jumpBehavior = jumpBehavior;
  }

  void kick() {
    kickBehavior.kick();
  }

  void jump() {
    jumpBehavior.jump();
  }

  void display();
}
```

위와 같이 Fighter 클래스는 이제 Kick과 Jump의 행동클래스를 변수로 가진다.

캡슐화로 감춰져있기 때문에 내부 구현상태는 모르게 함수를 호출해서 사용만 한다.

```dart
abstract class KickBehavior {
  void kick();
}

class LightningKick implements KickBehavior {
  @override
  void kick(){
    print('라이트~~~~닝~~~킥~!!!!');
  }
}

class TornadoKick implements KickBehavior {
  @override
  void kick() {
    print('토~~네이도~!!!! 킥!!');
  }
}
```

```dart
abstract class JumpBehavior {
  void jump();
}

class LongJump implements JumpBehavior {
  @override
  void jump() {
    print("쇼오오오오오옹~~~");
  }
}

class ShortJump implements JumpBehavior {
  @override
  void jump() {
    print("쇽!");
  }
}
```

Kick과 Jump의 행동 interface를 통해서 구현체들은 해당 함수 내용을 구현해준다.

이제 각 캐릭터들 클래스를 만들어보자.

```dart
class Paul extends Fighter {
  Paul(KickBehavior kickBehavior,JumpBehavior jumpBehavior ) :
super(kickBehavior, jumpBehavior);

  @override
  void display() {
    print('안녕 난 폴이야');
  }
}

class King extends Fighter {
    King(KickBehavior kickBehavior,JumpBehavior jumpBehavior ) :
super(kickBehavior, jumpBehavior);

  @override
  void display() {
    print("안녕 난 킹이야");
  }
}
```

이렇게 캐릭터마다 해당 Kick과 Jump 를 넣어주면 그에 맞춰 동작하게 된다.

```dart
void main() {
  JumpBehavior shortJump = ShortJump();
  JumpBehavior longJump = LongJump();
  KickBehavior lightningKick = LightningKick();
  KickBehavior tornadoKick = TornadoKick();

  Fighter paul = Paul(tornadoKick, shortJump);
  paul.display(); //  '안녕 난 폴이야'
  paul.kick(); //  '토~~네이도~!!!! 킥!!'
  paul.jump(); // "쇽!"

  paul.setJumpStrategy(longJump);
  paul.setKickStrategy(lightningKick);
  paul.jump(); // "쇼오오오오오옹~~~"
  paul.kick(); // '라이트~~~~닝~~~킥~!!!!'
}
```

## 정리

위의 해당 개념은 Flutter 개발을 하면서 의존성 역전에서 많이 쓰이는 개념 같다.

캡슐화를 통해 행동을 숨기고 Context는 구현 내용을 모르고 사용하게 만든다.

장점:

- 구조를 바꾸지 않고 여러 기능을 교체하기 쉽다.
- if, switch문 없이 필요에 따라 기능을 넣고 뺄 수 있다.
- Context에 영향을 주지 않고 변화가 가능하다.
- Testable해진다.

단점:

- 전략 패턴을 사용하기 위해서는 interface를 만들어야하기 때문에 코드스트레스가 증가한다.
- 특정 구체적인 상황에서는 구현이 어려울 수 있다.
- 모든 전략을 알고 있어야 올바른 사용이 가능하다. (즉, 사용법, 다양한 파생 클래스를 알고 있어야 한다.)
