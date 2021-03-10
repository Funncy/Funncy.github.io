---
layout: post
title: '[Flutter] - Firebase Auth 정리 '
subtitle: 'Flutter Firebase authentication'
categories: flutter
tags: flutter firebase firebaseauth
comments: true
---

# AuthState

---

FirebaseAuth에서는 User에 대한 변화를 Stream으로 전송해준다.

```java
FirebaseAuth.instance
  .authStateChanges()
  .listen((User user) {
    if (user == null) {
      print('User is currently signed out!');
    } else {
      print('User is signed in!');
    }
  });
```

User가 없는 경우 초기에 null이 들어온다.

# 로그인 종류

---

## 1. 익명 로그인

앱을 로그인 없이 사용가능할 경우 아예 로그인없이 사용하게 하는 방법도 있다.

하지만 Firebase에서는 Anonymous 로그인을 지원한다.

이것을 사용하는 이유는 다음과 같다.

- FireStore에 대한 권한 및 보안
- 앱 분석을 위한 유저 고유값 부여 (로그인을 하지 않았어도)

```java
UserCredential userCredential = await FirebaseAuth.instance.signInAnonymously();
```

## 2. 이메일 로그인

회원가입과 로그인으로 크게 2개 함수로 나눠진다.

### 2-1. 회원가입

```java
try {
  UserCredential userCredential = await FirebaseAuth.instance.createUserWithEmailAndPassword(
    email: "barry.allen@example.com",
    password: "SuperSecretPassword!"
  );
} on FirebaseAuthException catch (e) {
  if (e.code == 'weak-password') {
    print('The password provided is too weak.');
  } else if (e.code == 'email-already-in-use') {
    print('The account already exists for that email.');
  }
} catch (e) {
  print(e);
}
```

### 2-2. 로그인

```java
try {
  UserCredential userCredential = await FirebaseAuth.instance.signInWithEmailAndPassword(
    email: "barry.allen@example.com",
    password: "SuperSecretPassword!"
  );
} on FirebaseAuthException catch (e) {
  if (e.code == 'user-not-found') {
    print('No user found for that email.');
  } else if (e.code == 'wrong-password') {
    print('Wrong password provided for that user.');
  }
}
```

### 2-3. 이메일 인증

이메일 인증이 필요할 경우 간단하게 이메일 인증이 가능하다.

전송되는 이메일 내용은 Firebase Console에서 설정할 수 있다.

```java
User user = FirebaseAuth.instance.currentUser;

if (!user.emailVerified) {
  await user.sendEmailVerification();
}
```

## 3. 소셜 로그인

기본적으로

- Apple
- Facebook
- Twitter
- Google

을 지원해준다. 그외 카카오, 네이버 로그인을 사용하려면 자체 token서버를 운영해야 한다.

> 추후 Colud Function을 통해 만들어보자.

구글 로그인은 별다른 설정 없이 라이브러리 추가 후 사용 가능하다.

```java
import 'package:google_sign_in/google_sign_in.dart';

Future<UserCredential> signInWithGoogle() async {
  // Trigger the authentication flow
  final GoogleSignInAccount googleUser = await GoogleSignIn().signIn();

  // Obtain the auth details from the request
  final GoogleSignInAuthentication googleAuth = await googleUser.authentication;

  // Create a new credential
  final GoogleAuthCredential credential = GoogleAuthProvider.credential(
    accessToken: googleAuth.accessToken,
    idToken: googleAuth.idToken,
  );

  // Once signed in, return the UserCredential
  return await FirebaseAuth.instance.signInWithCredential(credential);
}
```

facebook이나 kakao등은 자체 개발자 사이트에서 설정을 해줘야하고 document를 보고 진행해주자.

# 유저 관리

---

## 1. 현재 유저

```java
FirebaseAuth auth = FirebaseAuth.instance;

if (auth.currentUser != null) {
  print(auth.currentUser.uid);
}
```

위와 같이 유저를 받아 올 수 있다. (로그인 완료 상태)

## 2. 유저 삭제

```java
try {
  await FirebaseAuth.instance.currentUser.delete();
} catch on FirebaseAuthException (e) {
  if (e.code == 'requires-recent-login') {
    print('The user must reauthenticate before this operation can be executed.');
  }
}
```

## 3. 재인증

```java
// Prompt the user to enter their email and password
String email = 'barry.allen@example.com';
String password = 'SuperSecretPassword!';

// Create a credential
EmailAuthCredential credential = EmailAuthProvider.credential(email: email, password: password);

// Reauthenticate
await FirebaseAuth.instance.currentUser.reauthenticateWithCredential(credential);
```
