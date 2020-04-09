---
layout: post
title: "[flutter] -  social login : google login "
subtitle:   "Flutter - google social login"
categories: flutter
tags: flutter sociallogin google 
comments: true
---

- 소셜로그인
    * [카카오톡](https://funncy.github.io/flutter/2020/03/24/kakao-01/)
    * [페이스북](https://funncy.github.io/flutter/2020/03/25/facebook/)

이번에는 구글로그인을 공부해보자

참고로 나는 `Firebase`를 사용하지 않고 진행할 예정이다.

## 준비 

1. Plugin
    [google_sign_in](https://pub.dev/packages/google_sign_in) 이라는 외부 라이브러리를 사용할 예정이다. (flutter팀 공식라이브러리다.)

2. Google APIs
    [Google APIs](https://console.developers.google.com/) 에 접속하여 앱을 생성하고 `OAuth 동의 화면`을 통해 동의하고 `사용자 인증 정보`로 이동한다. `OAuth 클라이언트 ID 만들기`는 프로젝트 생성한 후 진행한다.

---

## 1. 프로젝트 생성

이전 소셜 로그인을 만들었던 앱에 추가해서 진행하겠다.

`pubspec.yaml`에 플러그인을 등록한다.

```yaml
dependencies:
  flutter:
    sdk: flutter

  cupertino_icons: ^0.1.2
  google_sign_in: ^4.3.0
  ...
```

---

## 2. 화면 구성

이전 앱에 `google`로그인 버튼을 추가하자

<img src="https://Funncy.github.io/assets/img/social/2020-04-08-google-01.png" height="500"> 

---

## 3. 로그인 기능 작성

`button`에 로그인 기능을 작성하자.

```java
  Future<Null> _login() async {
    GoogleSignIn _googleSignIn = GoogleSignIn(
      scopes: <String>[
        'email',
      ],
    );

    try {
      var data = await _googleSignIn.signIn();
      Navigator.push(
        context,
        MaterialPageRoute(builder: (context) => Login(data.displayName)),
      );
    } catch (error) {
      print(error);
    }
  }
```

`Future` `async`를 통해 구글로그인 기능을 작성했다. 여기서 `scopes`는 가져올 데이터 범위인데 나는 `email`만 필요하므로 `email`만 작성했다.
`data`에는 `GoogleSignInAccount`객체가 들어있다. 그중 `displayName`을 `Login`페이지로 넘겨줬다.

---

## 4. Android 설정

코드는 간단하게 끝났지만 OAuth2를 위한 설정을 진행해줘야한다.

[`Google APIs`](https://console.developers.google.com/apis/dashboard)로 이동해서 `OAuth 동의 화면`에서 동의를 진행해주자.

그리고 `사용자 인증 정보`로 이동하면 다음 화면이 나온다.
여기서 `OAuth 클라이언트 ID`를 생성하자.

![이미지](https://Funncy.github.io/assets/img/social/2020-04-08-google-02.png "google Apis")

`android`를 선택하면 아래와 같은 화면이 나타난다.
이름은 나의 `App`이름으로 설정하고 아래 `서명 인증서 지문`을 채워주자.

![이미지](https://Funncy.github.io/assets/img/social/2020-04-08-google-03.png "create OAuth clientID")

> 이 파트를 진행하기 위해서는 `android keystore`파일이 있어야한다.

이렇게 `SHA-1`의 내용을 `keytool`로 뽑아 채워주고 패키지 이름도 채워주자.

![이미지](https://Funncy.github.io/assets/img/social/2020-04-08-google-04.png "keytool")

생성하면 다음과 같이 `Android Client ID` 가 생성된다.

![이미지](https://Funncy.github.io/assets/img/social/2020-04-08-google-05.png "create OAuth clientID")

`Android` 설정은 간단하게 끝났다.

---

## 5. IOS 설정

현재 `google_sign_in`에서 `ios`를 `firebase`를 거쳐서 로그인되도록 설명되어있다.
여기서는 `firebase`를 사용하지 않는 방법을 설명하겠다.

먼저 `google apis`에서 `OAuth 클라이언트 ID`를 `ios`로 생성하자.
번들이름을 넣어주자

![이미지](https://Funncy.github.io/assets/img/social/2020-04-08-google-06.png "ios clientID create")

그리고 생성된 `클라이언트 ID`를 클릭하면 아래와 같이 `클라이언트 ID`와 `ios URL 스키마`가 나온다. 이걸 가지고 `xcode`로 가자.

![이미지](https://Funncy.github.io/assets/img/social/2020-04-08-google-07.png "ios clientID create")

`xcode`로 `ios`프로젝트를 열면 다음과 같은 `Runner`폴더가 보인다.
`Runner`폴더 아래에 새로운 파일을 생성하자.
파일은 `Property List`로 하고 이름은 `GoogleService-Info`로 하자.
위 이름은 `google_sign_in`설정이니깐 맞춰주자.

![이미지](https://Funncy.github.io/assets/img/social/2020-04-08-google-08.png "xcode runner")

그리고 내부 속성에 `클라이언트 ID`와 `ios URL 스키마`를 각각 작성해주자.

![이미지](https://Funncy.github.io/assets/img/social/2020-04-08-google-09.png "clientID reverseID")

> `CLIENT_ID`, `REVERSED_CLIENT_ID` 이름을 잘 지켜주자.

그러고 나면 `Runner > Info > URL Types`에 `REVERSED_CLIENT_ID`를 추가해주자.

![이미지](https://Funncy.github.io/assets/img/social/2020-04-08-google-10.png "ios url Types")

> `CLIENT_ID`가 아니라 `REVERSED_CLIENT_ID`이다. 주의하자.

자 이렇게 다 설정하고 실행하면 다음과 같이 정상 작동한다.

![이미지](https://Funncy.github.io/assets/img/social/2020-04-08-google-11.png "ios google sign in")

