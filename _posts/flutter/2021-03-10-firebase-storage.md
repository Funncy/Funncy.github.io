---
layout: post
title: '[Flutter] - Firebase Storage 정리 '
subtitle: 'Flutter Firebase storage'
categories: flutter
tags: flutter firebase cloudstorage
comments: true
---

Firebase에서도 미디어 파일을 관리하는 Colud Storage를 지원해준다.

# Reference

---

reference는 file의 포인터이다.

```java
firebase_storage.Reference ref =
  firebase_storage.FirebaseStorage.instance.ref('/notes.txt');
```

2방식으로 사용가능하다.

```java
firebase_storage.Reference ref = firebase_storage.FirebaseStorage.instance
    .ref('images/defaultProfile.png');
// or
firebase_storage.Reference ref = firebase_storage.FirebaseStorage.instance
    .ref()
    .child('images')
    .child('defaultProfile.png');
```

# List files & directories

---

list와 listAll 함수를 통해 ListResult를 받아 올 수 있다.

```java
Future<void> listExample() async {
  firebase_storage.ListResult result =
      await firebase_storage.FirebaseStorage.instance.ref().listAll();

  result.items.forEach((firebase_storage.Reference ref) {
    print('Found file: $ref');
  });

  result.prefixes.forEach((firebase_storage.Reference ref) {
    print('Found directory: $ref');
  });
}
```

> listAll 함수는 모두 가져오기에 오랜 시간이 걸릴 수 있다.
> 그래서 list함수를 써보자

```java
Future<void> listExample() async {
  firebase_storage.ListResult result = await firebase_storage
      .FirebaseStorage.instance
      .ref()
      .list(firebase_storage.ListOptions(maxResults: 10));
  // ...
}
```

갯수 제한을 걸어 갯수 만큼만 받아오게 할 수 있다.

또한 reuslt에 있는 pageToken을 활용해 다음 페이지를 이어서 받아올 수 있다.

```java
Future<void> listExample() async {
  firebase_storage.ListResult result = await firebase_storage
      .FirebaseStorage.instance
      .ref()
      .list(firebase_storage.ListOptions(maxResults: 10));

  if (result.nextPageToken != null) {
    firebase_storage.ListResult additionalResults = await firebase_storage
        .FirebaseStorage.instance
        .ref()
        .list(firebase_storage.ListOptions(
          maxResults: 10,
          pageToken: result.nextPageToken,
        ));
  }
}
```

# Download URLs

---

```java
Future<void> downloadURLExample() async {
  String downloadURL = await firebase_storage.FirebaseStorage.instance
      .ref('users/123/avatar.jpg')
      .getDownloadURL();

  // Within your widgets:
  // Image.network(downloadURL);
}
```

Firebase CDN을 통해 확장가능하고 효율적이게 제공해준다.

# Uploading Files

---

### 1. File uploads

파일업로드 이전에 가장 먼저 디바이스의 절대 경로를 생성해주어야 한다.

```java
import 'package:path_provider/path_provider.dart';

Future<void> uploadExample() async {
  Directory appDocDir = await getApplicationDocumentsDirectory();
  String filePath = '${appDocDir.absolute}/file-to-upload.png';
  // ...
  // e.g. await uploadFile(filePath);
}
```

그리고 File 객체를 putFile 함수를 통해 보내주자.

```java
Future<void> uploadFile(String filePath) async {
  File file = File(filePath);

  try {
    await firebase_storage.FirebaseStorage.instance
        .ref('uploads/file-to-upload.png')
        .putFile(file);
  } on firebase_core.FirebaseException catch (e) {
    // e.g, e.code == 'canceled'
  }
}
```

### 2. Upload from a String

---

putString 메서드를 사용하여 raw, base64, base64url 또는 data_url 인코딩 문자열을 업로드할 수 있다.

```java
Future<void> uploadString() async {
  String dataUrl = 'data:text/plain;base64,SGVsbG8sIFdvcmxkIQ==';

  try {
    await firebase_storage.FirebaseStorage.instance
        .ref('uploads/hello-world.text')
        .putString(dataUrl, format: firebase_storage.PutStringFormat.dataUrl);
  } on firebase_core.FirebaseException catch (e) {
    // e.g, e.code == 'canceled'
  }
}
```

## 3. Uploading raw data

---

Uint8List 타입의 데이터도 지원해준다.

```java
import 'dart:convert' show utf8;
import 'dart:typed_data' show Uint8List;

Future<void> uploadData() async {
  String text = 'Hello World!';
  List<int> encoded = utf8.encode(text);
  Uint8List data = Uint8List.fromList(encoded);

  firebase_storage.Reference ref =
      firebase_storage.FirebaseStorage.instance.ref('uploads/hello-world.text');

  try {
    // Upload raw data.
    await ref.putData(data);
    // Get raw data.
    Uint8List downloadedData = await ref.getData();
    // prints -> Hello World!
    print(utf8.decode(downloadedData));
  } on firebase_core.FirebaseException catch (e) {
    // e.g, e.code == 'canceled'
  }
}
```

## 4. Adding Metadata

---

메타 데이터 추가 및 꺼내기 기능을 지원해준다.

```java
Future<void> uploadFileWithMetadata(String filePath) async {
  File file = File(filePath);

  // Create your custom metadata.
  firebase_storage.SettableMetadata metadata =
      firebase_storage.SettableMetadata(
    cacheControl: 'max-age=60',
    customMetadata: <String, String>{
      'userId': 'ABC123',
    },
  );

  try {
    // Pass metadata to any file upload method e.g putFile.
    await firebase_storage.FirebaseStorage.instance
        .ref('uploads/file-to-upload.png')
        .putFile(file, metadata);
  } on firebase_core.FirebaseException catch (e) {
    // e.g, e.code == 'canceled'
  }
}
```

```java
Future<void> getMetadataExample() async {
  firebase_storage.FullMetadata metadata = await firebase_storage
      .FirebaseStorage.instance
      .ref('uploads/file-to-upload.png')
      .getMetadata();

  // As set in previous example.
  print(metadata.customMetadata['userId']);
}
```

## Downloading Files

---

로컬 디바이스에 저장하기 위해서는 writeToFile 함수를 사용하면 된다.
먼저 File 객체를 이용해 절대 경로를 설정해주자.

```java
import 'package:path_provider/path_provider.dart';

Future<void> downloadFileExample() async {
  Directory appDocDir = await getApplicationDocumentsDirectory();
  File downloadToFile = File('${appDocDir.path}/download-logo.png');

  try {
    await firebase_storage.FirebaseStorage.instance
        .ref('uploads/logo.png')
        .writeToFile(downloadToFile);
  } on firebase_core.FirebaseException catch (e) {
    // e.g, e.code == 'canceled'
  }
}
```

## Handling Tasks

---

파일 업로드 혹은 다운로드는 UploadTask, DownloadTask를 리턴한다.

Task는 업로드,다운로드에서 상태 및 제어가 가능하다.

- TaskState.running : 현재 러닝 상태
- TaskState.paused : task 멈춰있는 상태
- TaskState.success : 완료된 상태

또한 아래처럼 pause, resume, cancle 함수를 사용 가능하다.

```java
Future<void> handleTaskExample3(String filePath) async {
  File largeFile = File(filePath);

  firebase_storage.UploadTask task = firebase_storage.FirebaseStorage.instance
      .ref('uploads/hello-world.txt')
      .putFile(largeFile);

  // Pause the upload.
  bool paused = await task.pause();
  print('paused, $paused');

  // Resume the upload.
  bool resumed = await task.resume();
  print('resumed, $resumed');

  // Cancel the upload.
  bool cancelled = await task.cancel();
  print('cancelled, $cancelled');

  // ...
}
```

> cancle은 Exception으로 넘어가게 되니 유의하자.
