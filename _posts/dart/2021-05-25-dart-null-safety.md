---
layout: post
title: '[Dart] - Null Safety '
subtitle: 'dart nullsafety'
categories: programming
tags: dart nullsafety nullable null nonnullable
comments: true
---

이번에 Flutter가 버전업그레이드가 되면서 Null-Safety가 기본으로 적용되기 시작했다.

그래서 생소했던 Null-Safety를 정리해보자.

기본적으로 우리는 아래와 같이 변수를 선언하고 사용해왔다.

```dart
int i = 42;
int j = null;
```

하지만 이제 일반적인 변수 선언에서는 null을 넣는 순간 Error가 발생한다.

```dart
int? aNullableInt = null;
//int aNullableInt = null; Error!!
```

# 1. Nullable (?)

---

이제 null이 가능한 변수에는 끝에 ?를 붙여주어야 한다.

```dart
int? a = null;
int? b = 4;
```

> 여기서 주의할 점은 Non-nullable데이터에 nullable을 삽입할 수 없다는 것이다.

그렇다면 우리는 어떻게 해야할까?

### ! 연산자

---

!를 통해서 nullable을 벗겨(?)줄 수 있다.

다만, 이건 프로그래머 책임이므로 null인 데이터를 !하면 에러가 발생하니 주의해서 사용하자.

### ?? 연산자

---

Dart에서는 `?.` 연산자와 `??` 연산자를 지원해준다.

`??` 를 통해서 null인 경우 대체값을 넣어주자.

```dart
int? a = null;
print(a ?? 3);
```

# 2. Non-Nullable

---

우리가 일반적으로 쓰던 변수들은 이제 null이 불가능한 변수들이다.

큰 특징을 알아보자.

### initialized

초기화가 안됬다는건 null을 의미하므로 초기화 진행 코드가 있어야한다. (사용하기 전에)

```dart
void main() {
  String text;

  if (DateTime.now().hour < 12) {  //이 내용이 없으면 컴파일 에러가 발생한다.
   text = "It's morning! Let's make aloo paratha!";
  } else {
   text = "It's afternoon! Let's make biryani!";
  }

  print(text);
  print(text.length);
}
```

### null-check

```dart
int getLength(String? str) {
  // Add null check here

  return str.length; //Error가 발생!!!
}

void main() {
  print(getLength('This is a string!'));
}
```

위와 같이 반환값은 non-nullable인데 parameter가 nullable인 경우에는 non-nullable로 맞춰주어야 한다.

```dart
int getLength(String? str) {
	// if(str == null) {
	//   return str!.length;
  // }
	// return 0;   나는 아래 방법을 선호한다.
  return str?.length ?? 0;
}
```

다른 방식으로 Exception을 던져서 error을 지우는 방법이 있다.

```dart
int getLength(String? str) {
  if(str==null) {
    throw Exception();
  }
  return str.length;
}
```

## Late

---

late키워드는 non-nullable 변수를 바로 초기화하지 않고 lazy하게 initialize하는 키워드이다.

```dart
class Meal {
  late String _description; //late 키워드가 있으므로 컴파일 에러가 발생하지 않는다.
  //원래라면 String _description = ""; 식으로 값이 들어가 있어야 한다.

  set description(String desc) {
    _description = 'Meal description: $desc';
  }

  String get description => _description;
}

void main() {
  final myMeal = Meal();
  myMeal.description = 'Feijoada!';
  print(myMeal.description);
}
```
