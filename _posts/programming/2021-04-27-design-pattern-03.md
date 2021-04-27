---
layout: post
title: '[Programming] - 디자인 패턴 03 : Command 패턴 '
subtitle: 'design pattern stratgey'
categories: programming
tags: designpattern stratgey
comments: true
---

[참고] : [https://gmlwjd9405.github.io/2018/07/07/command-pattern.html](https://gmlwjd9405.github.io/2018/07/07/command-pattern.html)

## 커맨드 패턴이란?

- 실행될 기능을 캡슐화하여 재사용성을 높인다.
- 기능이 변경되어도 호출하는 클래슨느 수정 없이 그대로 사용 가능

## 예시

- 버튼이 눌리면 Lamp가 켜지는 프로그램

```jsx
class Lamp {
  void turnOn() {
    print("Lamp On");
  }
}

class Button {
  Lamp _theLamp;

  Button(Lamp theLamp) : _theLamp = theLamp;

  void pressed() {
    _theLamp.turnOn();
  }
}

void main() {
  Lamp lamp = Lamp();
  Button lampButton = Button(lamp);
  lampButton.pressed();
}
```

### 문제점

1. 버튼을 눌렀을때 다른 동작을 실행시키고 싶다면? (ex. Alarm)

- 새로운 기능으로 변경될때 기존 코드 수정해야하므로 OCP(Open Closed Principle)에 위배
- Button 클래스의 pressed 전체를 변경해야한다.

2. 버튼을 누르는 상태에 따라 다른 기능을 실행하는 경우

- 버튼을 처음 눌렀으면 램프를 켜고, 두번째는 알람 동작
- 필요기능 추가 될 때마다 Button 클래스를 수정 해야함.

### 해결법

- 실행될 기능을 캡슐화하자.

```dart
abstract class Command {
  void execute();
}

class Button {
  Command _theCommand;

  Button(Command theCommand) : _theCommand = theCommand;

  void setCommand(Command newCommand) {
    _theCommand = newCommand;
  }

  void pressed() {
    _theCommand.execute();
  }
}
```

```dart
//Lamp
class Lamp {
  void turnOn() {
    print("Lamp On");
  }
}

class LampOnCommand implements Command {
  Lamp _lamp;
  LampOnCommand(Lamp lamp) : _lamp = lamp;

  void execute() {
    _lamp.turnOn();
  }
}

//Alarm
class Alarm {
  void start() {
    print("Arlaming~~!!!");
  }
}

class AlarmStartCommand implements Command {
  Alarm _alarm;
  AlarmStartCommand(Alarm alarm) : _alarm = alarm;

  void execute() {
    _alarm.start();
  }
}

//..계속 추가 가능
```

```dart
void main() {
  Lamp lamp = Lamp();
  Command lampOnCommand = LampOnCommand(lamp);
  Alarm alarm = Alarm();
  Command alarmStartCommand = AlarmStartCommand(alarm);

  Button turnOnLamp = Button(lampOnCommand);
  turnOnLamp.pressed();

  Button startAlarmAndTurnOnLamp = Button(alarmStartCommand);
  startAlarmAndTurnOnLamp.pressed();
  startAlarmAndTurnOnLamp.setCommand(lampOnCommand);
  startAlarmAndTurnOnLamp.pressed();
}
```

![이미지](https://Funncy.github.io/assets/img/programming/2021-04-27-pattern-04.png 'Command Pattern')

Lamp와 Alarm의 기능을

Command 추상클래스를 통해 캡슐화 하였다.

그리하여 언제든 Button Class의 변경없이 교체 및 사용이 가능하다.

단, Command 인터페이스에 정해놓은 규칙대로 사용이 가능하다.
