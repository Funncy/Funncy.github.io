---
layout: post
title: '[Programming] - 디자인 패턴 05 : Delegate 패턴 '
subtitle: 'design pattern delegate'
categories: programming
tags: designpattern delegate
comments: true
---

## Delegate 패턴이란?

---

delegate 단어의 뜻대로 위임, 즉 대행자이다.

하나의 특정 객체가 모든 일을 처리하는 것이 아니라

처리 해야 할 일 중 일부를 다른 객체에 넘기는 것을 뜻한다.

> FirebaseFirestore도 delegate패턴으로 만들어져 있다.

이것이 사용되는 곳은 보통 아래와 같은 곳에 쓰인다.

1. 데이터 전달
2. 상호 통신
3. 이벤트 리스너

[참조] : [https://sangminem.tistory.com/289](https://sangminem.tistory.com/289)

```dart
mixin GameTimerDelegate {
  void gameOver();
}
```

```dart
class GameBoard extends StatefulWidget with GameTimerDelegate {
  // ... 생략

  void gameOver() {
    print("game over");
  }
}
```

```dart
class GameTimer extends StatefulWidget {
  GameTimerDelegate delegate; // 델리게이트 선언

  @override
  _GameTimerState createState() => _GameTimerState();
}

class _GameTimerState extends State<GameTimer> {
  // ... 생략

  void startTimer() {
    // ... 타이머 로직 생략

    if (remainingTime < 1) {
      timer.cancel();
      widget.delegate.gameOver(); // 델리게이트를 통해 함수 호출
    } else {
      remainingTime = remainingTime - 1;
    }
  }
}
```

```dart
import 'gameBoard.dart';
import 'gameTimer.dart';

class Game extends StatefulWidget {
  @override
  _GameState createState() => _GameState();
}

class _GameState extends State<Game> {
  @override
  Widget build(BuildContext context) {
    final gameBoard = GameBoard();
    final gameTimer = GameTimer();
    gameTimer.delegate = gameBoard; // 객체 연결하기
    return Scaffold(body: gameBoard, floatingActionButton: gameTimer);
  }
}
```

Firebase도 위와 같이 delegate 패턴으로 구성되어있다.

### Firestore.dart

```dart
class FirebaseFirestore extends FirebasePluginPlatform {
  FirebaseFirestorePlatform _delegatePackingProperty;

  FirebaseFirestorePlatform /*!*/ get _delegate {
    if (_delegatePackingProperty == null) {
      _delegatePackingProperty =
          FirebaseFirestorePlatform.instanceFor(app: app);
    }
    return _delegatePackingProperty;
  }
...

}
```

위에 있는 Delegate인 FirebaseFirestorePlatform을 살펴보자.

### FirebaseFirestorePlatform.dart

```dart
/// Defines an interface to work with Cloud Firestore on web and mobile
abstract class FirebaseFirestorePlatform extends PlatformInterface {
  ...

  /// The current default [FirebaseFirestorePlatform] instance.
  ///
  /// It will always default to [MethodChannelFirebaseFirestore]
  /// if no other implementation was provided.
  static FirebaseFirestorePlatform get instance {
    if (_instance == null) {
      _instance = MethodChannelFirebaseFirestore(app: Firebase.app());
    }
    return _instance;
  }
	...
  /// Creates a write batch, used for performing multiple writes as a single
  /// atomic operation.
  ///
  /// Unlike transactions, write batches are persisted offline and therefore are
  /// preferable when you don’t need to condition your writes on read data.
  WriteBatchPlatform batch() {
    throw UnimplementedError('batch() is not implemented');
  }

  /// Clears any persisted data for the current instance.
  Future<void> clearPersistence() {
    throw UnimplementedError('clearPersistence() is not implemented');
  }

  /// Enable persistence of Firestore data. Web only.
  Future<void> enablePersistence(
      [PersistenceSettings /*!*/ persistenceSettings]) async {
    throw UnimplementedError('enablePersistence() is not implemented');
  }

  /// Gets a [CollectionReferencePlatform] for the specified Firestore path.
  CollectionReferencePlatform collection(String /*!*/ collectionPath) {
    throw UnimplementedError('collection() is not implemented');
  }
	...
}
```

보면 내부 내용은 비어있고 MethodChannelFirebaseFirestore로 instance를 넘겨준다.

### MethodChannelFirebaseFirestore.dart

```dart
/// The entry point for accessing a Firestore.
///
/// You can get an instance by calling [FirebaseFirestore.instance].
class MethodChannelFirebaseFirestore extends FirebaseFirestorePlatform {

  ...

  @override
  CollectionReferencePlatform collection(String path) {
    return MethodChannelCollectionReference(this, path);
  }

  @override
  QueryPlatform collectionGroup(String path) {
    return MethodChannelQuery(this, path, isCollectionGroupQuery: true);
  }

...

}
```
