---
layout: post
title: "[flutter] -  social login : facebook login "
subtitle:   "Flutter - facebook social login"
categories: flutter
tags: flutter sociallogin facebook 
comments: true
---

[카카오톡 로그인](https://funncy.github.io/flutter/2020/03/24/kakao-01/)을 진행하고 다음으로 `Facebook`로그인을 개발해보자


## 준비 

1. [flutter_facebook_login](https://pub.dev/packages/flutter_facebook_login) - `facebook login`
2. [dio](https://pub.dev/packages/dio) - `token`을 활용해 `user`정보를 가져올때 필요하다. 

3. [facebook developer](https://developers.facebook.com/) - 여기에 개발자 등록을 하고 `App`을 추가해야한다. 

---

## 1. 프로젝트 생성

`카카오 로그인`을 만들었던 앱에 추가해서 진행하겠다.

`pubspec.yaml`에 플러그인을 등록한다.

```yaml
dependencies:
  flutter:
    sdk: flutter

  cupertino_icons: ^0.1.2
  flutter_facebook_login: ^3.0.0
  dio: ^3.0.9
  ...
```

---

## 2. 화면 구성

`test 앱`이니 빠르게 facebook 버튼을 만들어보자.

<img src="https://Funncy.github.io/assets/img/social/2020-03-25-facebook-01.png" height="500"> 

---

## 3. 로그인 기능 작성

로그인 `button`의 기능을 작성하자 

```java
  static final FacebookLogin facebookSignIn = new FacebookLogin();

  Future<Null> _login() async {
    final FacebookLoginResult result =
        await facebookSignIn.logIn(['email', 'public_profile']);

    switch (result.status) {
      case FacebookLoginStatus.loggedIn:
        final FacebookAccessToken accessToken = result.accessToken;
        print('token = ${accessToken.toMap()}');
        final token = result.accessToken.token;
        final graphResponse = await Dio().get(
            'https://graph.facebook.com/v2.12/me?fields=name,first_name,last_name,email&access_token=${token}');

        final data = jsonDecode(graphResponse.data);

        Navigator.push(
          context,
          MaterialPageRoute(builder: (context) => Login(data["name"])),
        );

        break;
      case FacebookLoginStatus.cancelledByUser:
        print('error cancelledByUser');
        break;
      case FacebookLoginStatus.error:
        print('error errorerror');
        break;
    }
  }
```

> 위에서는 `dio(http)`를 통해 유저 이름을 `Login Page`로 전송하게 구현했다.

---

## 4. Android 설정

먼저 [facebook developer](https://developers.facebook.com/docs/facebook-login/android)에서 `Step`별로 진행한다.

- 모든 `Step`이 아니라 아래 순서만 진행한다.
  * `1단계` `App` 선택
  * `4단계` `Manifest` 설정
  * `5단계` `Package` 설정
  * `6단계` `hash key` 설정

---

<br>

![이미지](https://Funncy.github.io/assets/img/social/2020-03-25-facebook-02.png "android step 1")

`facebook developer`에서 생선한 `app`을 선택해준다.
그러면 자동으로 `App ID`와 `Key`값을 찾아준다.

---
<br>

![이미지](https://Funncy.github.io/assets/img/social/2020-03-25-facebook-03.png "android step 2")

`step2,3`을 건너뛰고 `step4`에 있는 `manifest`설정을 진행한다.
그대로 복사해서 붙여넣자.

---
<br>

![이미지](https://Funncy.github.io/assets/img/social/2020-03-25-facebook-04.png "android step 3")

내가 생성한 `package`이름을 적어준다. 
> 이대로 진행하면 경고가 뜰 것이다. `com.example`을 그대로 사용하면 안된다는 경고이다. 나는 `test 앱`이기때문에 그대로 진행하지만 출시를 목적으로 한다면 수정해야한다.

---
<br>

![이미지](https://Funncy.github.io/assets/img/social/2020-03-25-facebook-05.png "android step 4")

그리고 `android hash key`를 등록해준다.
`hash key` 가져오는 법은 아래를 참고한다.

> `keytool`로 가져올 수 있지만 정확하지 않는 경우가 많다. `native`가 가장 정확하다.

---

#### hash key 가져오기 

카카오톡 로그인과 동일하다.

다음은 `kotlin`기준으로 작성하겠다.
먼저 `android/app/src/main/kotlin` 아래 있는 `MainActivity.kt`파일을 연다.

그리고 아래와 같이 작성한다.

```kotlin
class MainActivity: FlutterActivity() {
    override fun configureFlutterEngine(@NonNull flutterEngine: FlutterEngine) {
        GeneratedPluginRegistrant.registerWith(flutterEngine);
    }

    @Override
    override fun onCreate(savedInstanceState: Bundle?) {
    super.onCreate(savedInstanceState)

        try {
            val info: PackageInfo = packageManager.getPackageInfo(packageName, PackageManager.GET_SIGNATURES)
            for (signature in info.signatures) {
                var md: MessageDigest
                md = MessageDigest.getInstance("SHA")
                md.update(signature.toByteArray())
                val something = String(Base64.encode(md.digest(), 0))
                Log.d("hash", something.toString())
            }
        } catch(e: Exception) {
            Log.e("name not found", e.toString())
        }
    }

}
```

> `android studio`에서 진행하는걸 추천하며 `vscode`에 `plugin`이 없을 시 `import`를 직접해줘야한다.
그리고 꼭 `import io.flutter.Log`를 넣어주자

이렇게 작성하고 `android studio`로 실행해서 `logcat`을 확인하자.

![이미지](https://Funncy.github.io/assets/img/social/2020-03-24-kakao-02.png "hash key")

---

## 5. IOS 설정

IOS도 [facebook developer](https://developers.facebook.com/docs/facebook-login/ios)에서 `Step`별로 진행한다.

> 모든 `Step`이 아니라
`1단계` `App` 선택 
`3단계` `Bundle Identifier` 설정
`4단계` `Info.plist` 설정
이 순서만 진행한다.

![이미지](https://Funncy.github.io/assets/img/social/2020-03-25-facebook-02.png "ios step 1")

`android`설정과 똑같이 `facebook developer`에서 생선한 `app`을 선택해준다.
그러면 자동으로 `App ID`와 `Key`값을 찾아준다.

![이미지](https://Funncy.github.io/assets/img/social/2020-03-25-facebook-08.png "ios step 2-1")

`xcode`를 실행해서 `Bundle Identifier`를 수정하고

![이미지](https://Funncy.github.io/assets/img/social/2020-03-25-facebook-09.png "ios step 2-2")

똑같이 입력해준다.

![이미지](https://Funncy.github.io/assets/img/social/2020-03-25-facebook-10.png "ios step 3")

`Info.plist`에 다음 내용을 복사 붙여넣기를 진행한다.

---

## 6. 결과 화면

<img src="https://Funncy.github.io/assets/img/social/2020-03-25-facebook-06.png" height="500"> 

<img src="https://Funncy.github.io/assets/img/social/2020-03-25-facebook-07.png" height="500"> 



생가보다 쉽게 `facebook login`을 완성했다. `카카오 로그인`에 비하면 많이 쉬웠다.

다음은 `google login`을 진행해보겠다.