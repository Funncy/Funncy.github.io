---
layout: post
title: '[Flutter] - Firebase  FireStore 총정리 '
subtitle: 'Flutter Firebase FireStore'
categories: flutter
tags: flutter firebase firestore
comments: true
---

Flutter에서 FireStore 사용했던 내용을 기록하였다.

기본적으로 FireStore는 Collection-Document 기반 Nosql 데이터베이스이다.

# 1. init

---

```javascript
FirebaseFirestore firestore = FirebaseFirestore.instance;
```

위와 같은 코드로 firestore를 가져와서 사용할 수 있다.

하지만 이거 이전에 `Firebase.initializeApp()`을 실행해주어야 한다.

```javascript
import 'package:flutter/material.dart';

// Import the firebase_core plugin
import 'package:firebase_core/firebase_core.dart';

void main() {
  WidgetsFlutterBinding.ensureInitialized();
  runApp(App());
}

class App extends StatelessWidget {
  // Create the initialization Future outside of `build`:
  final Future<FirebaseApp> _initialization = Firebase.initializeApp();

  @override
  Widget build(BuildContext context) {
    return FutureBuilder(
      // Initialize FlutterFire:
      future: _initialization,
      builder: (context, snapshot) {
        // Check for errors
        if (snapshot.hasError) {
          return SomethingWentWrong();
        }

        // Once complete, show your application
        if (snapshot.connectionState == ConnectionState.done) {
          return MyAwesomeApp();
        }

        // Otherwise, show something whilst waiting for initialization to complete
        return Loading();
      },
    );
  }
}
```

# 2. Read

---

FireStore에서는 2가지 Read방식이 존재한다.

### 2-1. One-time Read

한번 읽는 방식으로 우리에게 친숙하다.

```javascriptscript
var documentSnapshot = await users.doc(documentId).get();
print(documentSnapshot.data());
```

### 2-2. Real-time Read

stream을 이용해 변경되는 사항을 Stream으로 넘겨준다. 실시간 반영이 이루어진다.

Collection의 Stream을 받아서 전체 Documents의 변경 사항을 실시간으로 받을 수도 있고

Document의 Stream을 받아서 하나의 Document의 변경 사항을 실시간으로 받을 수 있다.

```javascript
Stream collectionStream = FirebaseFirestore.instance.collection('users').snapshots();
Stream documentStream = FirebaseFirestore.instance.collection('users').doc('ABC123').snapshots();
```

> 받아온 Stream으로 StreamBuilder를 이용해 화면을 구성해보자.

# 3. Write

---

이제 FireStore에 Write로 저장해보자.

먼저 내용을 적을 collection을 생성 혹은 불러와보자.

```javascript
CollectionReference users = FirebaseFirestore.instance.collection('users');
```

이제 이렇게 가져온 collection에 document를 적는 크게 2가지 방법이 있다.

### 3-1. Document 아이디 자동 생성

```javascript
users.add({
	full_name: fullName, // John Doe
	company: company, // Stokes and Sons
	age: age, // 42
});
```

나중에 Model을 사용한다면 다음과 같이 편하게 작성이 가능하다.

```javascript
class User {
  final String fullName;
  final String company;
  final int age;

  User({
    @required this.fullName,
    @required this.company,
    @required this.age,
  });

  User.fromJson(Map<String, dynamic> json)
      : cid = json['full_name'],
        title = json['company'],
        number = json['age'];

  Map<String, dynamic> toJson() => {
        'full_name': fullName,
        'company': company,
        'age': age,
      };
}

...

User userModel = User(fulleName: 'John Doe', comapny: 'Stokes and Sons', age: 42);
users.add(userModel.toJson());

```

### 3-2. Document 아이디 지정 생성

```javascript
//위와 거의 동일 하나 id를 지정해 먼저 빈 document를 생성한다.
users.doc('docId').set(userModel.toJson());
```

> dot operator : FireStore에서 . 접근자로 접근하면 Map형식으로 데이터가 들어간다.
> ex) add({'users.name' : 'teddy'});
> ⇒ user['name'] = 'teddy;

# 4. Update

---

크게 2가지 방법이있다.

### 4-1. 다 지우고 대체 하는 방법

위에서 사용한 `set` 함수를 사용하면 된다.

```javascript
var test = await firebaseFirestore.collection('test').add({ data: 'test' });
test.set({ haha: 'haha' });
```

> set은 모두 대체 해버린다.

### 4-2. 기존 값 유지하며 특정 필드만 대체 하는 방법

`update` 함수를 사용하면 된다.

```javascript
var test = await firebaseFirestore.collection('test').add({ data: 'test' });
test.set({ data: 'test update', data2: 'new data' });
```

> 기존 필드는 업데이트 되고,
> 새로운 필드는 생성 된다.

그리고 특정 데이터 필드 영역 (ex. GeoPoint, Blob...) 은 클래스를 지원해준다.

```javascript
CollectionReference users = FirebaseFirestore.instance.collection('users');

Future<void> updateUser() {
  return users
    .doc('ABC123')
    .update({'info.address.location': GeoPoint(53.483959, -2.244644)})
    .then((value) => print("User Updated"))
    .catchError((error) => print("Failed to update user: $error"));
}
```

```javascript
CollectionReference users = FirebaseFirestore.instance.collection('users');

Future<void> updateUser() {
  return rootBundle
    .load('assets/images/sample.jpg')
    .then((bytes) => bytes.buffer.asUint8List())
    .then((avatar) {
      return users
        .doc('ABC123')
        .update({'info.avatar': Blob(avatar)});
      })
    .then((value) => print("User Updated"))
    .catchError((error) => print("Failed to update user: $error"));
}
```

# 5. Remove

---

삭제에는 크게 2가지 방법이 있다.

### 5-1. 전체 Document 삭제

```javascript
//document있다는 가정
doc.delete();
```

### 5-2. 특정 Field 삭제

```javascript
doc.update({
	data: FieldValue.delete(),
});
```

# 6. Transactions

---

만약 대량 트래픽이 발생하는 앱이라고 가정해보자.
그곳에서 `좋아요` 버튼을 누르면 갯수를 증가시켜주는 기능을 만들었다.

```javascript
//특정 post를 찾았다는 가정 post => DocumentReference;
var postSnapshot = await post.get();
var currentLikes = postSnapshot.data()['likes'];

//현재 값을 기준으로 증가
post.update({ likes: currentLikes + 1 });
```

위와 같은 코드는 정상 작동 한다. 하지만 ! 트래픽이 몰렸을 경우

현재 읽어들인 값이 변경되는 상황이 발생한다. `데이터 일관성`이 깨지게 된다.

이 문제를 해결하는 방법이 `Transaction`이다.

```javascript
DocumentReference doc =
        firebaseFirestore.collection('test').doc('6gDFEgWt6TE5mPVZLz8X');

return firebaseFirestore.runTransaction((transaction) async {
      DocumentSnapshot snapshot = await transaction.get(doc);
      if (!snapshot.exists) {
        throw Exception('Does not exists');
      }

			//기존 갑을 가져와 1을 더해준다.
      int currentLikes = snapshot.data()['likes'] + 1;

			//직접 값을 더하지 말고 transaction을 통해서 더하자!
      transaction.update(doc, {'likes': currentLikes});
      return currentLikes;
 });
```

> transaction 내부에서 직접 write해서 값을 수정 하게 되면 문제가 발생 할 수 있다.
> 왜냐하면 transaction 내부는 여러번 반복 될 수 있기 때문이다 (5번까지)

transaction은 실행 도중 snapshot데이터가 변경되면 다시 처음부터 실행한다.

- 항상 읽기 뒤에 쓰기를 진행하자
- 오프라인에서는 동작 하지 않는다

# 7. Batch write

---

여러개의 write 기능을 동작해야할 때 사용한다.

set, update, delete등을 조합해서 사용 가능하다.

```javascript
WriteBatch batch = firebaseFirestore.batch();
CollectionReference collection = firebaseFirestore.collection('test');

return collection.get().then((querySnapshot) {
  querySnapshot.docs.forEach((document) {
    batch.delete(document.reference);
  });
	//batch를 돌리며 한번에 처리해준다.
  batch.commit();
});
```

> batch를 이용하면 여러개의 write, update, set, delete를 한 operation으로 동작시켜준다.

# 8. Reference ? Query ? Snapshot?

---

다음과 같은 도표를 만들어보았다.

![이미지](https://Funncy.github.io/assets/img/flutter/2021-03-06-firestore.png 'Firestore diagram')

위 도표는 FireStore에서 데이터를 접근 하는 과정을 도식화 해보았다.

## 8-1. Collection

Collection에서는 크게 3가지 방향으로 갈린다.

### 8-1-1. Query

.orderBy(), where()... 을 통해 Query를 만들고

.get()을 통해 서버통신으로 데이터를 가져온다. (바로 .get()을 쓰면 모두 가져오라는 Query이다)

⇒ Return 값은 `QuerySnapshot`이다.

### 8-1-2. snapshot()

이 내용은 real-time Read를 위한 Stream을 받아오는 함수이다.

⇒ Return 값은 `Stream<QuerySnapshot>`이다.

### 8-1-3. doc

특정 documentId를 통해 단일의 document에 접근한다.

아직 서버 통신을 하는 상태는 아니며 접근할 document의 reference를 반환해준다.

⇒ Return 값은 `DocumentReference`이다.

## 8-2. QuerySnapshot

Collection으로 부터 Query, snapshot을 통해 받아온 데이터 타입이다.

사실상 Snapshot은 비동기로 실제 서버 데이터를 가져온 내용물이다.

Collection으로 부터 특정 Doucment들을 가져왔기에 하나씩 까봐야한다.

### 8-2-1. docs

QuerySnapshot의 내부 데이터 리스트에 접근한다.

⇒ Return 값은 List<`QueryDocumentSnapshot`>이다.

## 8-3. QueryDocumentSnapshot

사실상 DocumentSnapshot과 같은 내용이다. DocumentSnapshot을 상속받았다.

> 왜 굳이 따로 만들었는지는 Class에 주석으로 친절하게 나와있다.
>
> 1. DocumentSnapshot과 다르게 항상 exists 가 true이다.
> 2. data()가 절대 null을 반환하지 않는다.

### 8-3-1. data()

`실제 데이터`가 들어있으며 `Map<String, dynamic>`형태로 넘어온다.

> 만약 넘어온 field의 데이터를 List<String>으로 받고 싶다면 다음과 같이 형변환을 해주어야 한다.
> json['string_array']?.cast<String>()

### 8-3-2. reference

Snapshot을 받아 올 수 있는 `DocumentRefernce`를 반환해준다.

QueryDocumentSnapshot은 기본적으로 DocumentSnapshot을 상속했기 때문에
똑같이 동작한다.

## 8-4. DocumentSnapshot

실제 Document의 데이터가 들어있는 객체이다.

### 8-4-1. data()

위와 동일하게 실제 데이터를 받아 올 수 있다.

### 8-4-2. reference

위와 동일하게 `DocumentReference`를 반환해준다.

## 8-5. DocumentReference

DocumentReference에는 기본적인 동작들이 들어있다.

### 8-5-1. set(data)

document의 내용을 넘어온 data로 대체해버린다.

### 8-5-2. update(data)

data로 넘어온 특정 field의 값을 update한다.

### 8-5-3. delete()

document를 삭제한다.

### 8-5-4. collection('collection_name')

내부 collection을 호출한다 혹은 생성한다.

### 8-5-5. get()

Future이다. 즉 서버통신을 통해 `DocumentSnapshot()` 데이터를 가져온다.

도표와 아래 내용들은 하나하나 뜯어본 내용이니 도움이 되길 바란다.
