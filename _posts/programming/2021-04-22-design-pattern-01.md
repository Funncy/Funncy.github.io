---
layout: post
title: '[Programming] - 디자인 패턴 01 : 싱글톤 '
subtitle: 'design pattern singleton'
categories: programming
tags: designpattern singleton
comments: true
---

### 싱글톤 이란?

- 전역 변수 없이 객체를 항상 하나만 생성하도록 한다.
- 생성된 객체를 어디서든 참조할 수 있또록 하는 패턴.

```dart
class SingleTonB {
  static SingleTonB _singleton;

  SingleTonB._internal() {
    //초기화 코드
  }

  static SingleTonB getInstance() {
    if(_singleton == null){
      _singleton = SingleTonB._internal();
    }
    return _singleton;
  }
}
```

일반적인 프로그래밍 언어들은 위와 같은 방식으로 싱글톤을 구현한다.

```dart
class SingleTonC {
   static SingleTonC _singleton = SingleTonC._internal();

  SingleTonC._internal() {
    //초기화 코드
  }

  factory SingleTonC() {
    return _singleton;
  }
}
```

Dart는 factory 키워드를 지원해주어 아주 간단하게 구현이 가능하다.

> Dart는 Static키워드 자체가 Lazy initialization 해주기 때문에 위와 같이
> 작성해도 괜찮다
