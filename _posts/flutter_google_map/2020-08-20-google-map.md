---
layout: post
title: "[flutter] - google map 사용하기 "
subtitle:   "flutter google map "
categories: flutter
tags: flutter googlemap 
comments: true
---

Flutter에서 Google Map을 사용해보자

# 1. API Key 발급

구글 관련 API를 사용하기 위해서는 Key를 발급 해야한다.

그러면 [구글 클라우드 플랫폼](https://console.cloud.google.com/)에 접속해야 합니다.

그리고 새로운 프로젝트를 생성합니다.

![이미지](https://Funncy.github.io/assets/img/google_map/2020-08-20-google-map-01.png "new project")

그리고 API 라이브러리에 접속하여 2가지 라이브러리를 Enable 시켜줍니다.

- Maps SDK for iOS
- Maps SDK for Android

![이미지](https://Funncy.github.io/assets/img/google_map/2020-08-20-google-map-02.png "API")

그 다음에 프로젝트 내부 `사용자 인증 정보` 화면으로 이동합니다.

![이미지](https://Funncy.github.io/assets/img/google_map/2020-08-20-google-map-03.png "API key")

이제 Android와 iOS의 Key 생성을 진행합니다.

### 1) Android Key

![이미지](https://Funncy.github.io/assets/img/google_map/2020-08-20-google-map-04.png "Android key")

위 내용 처럼 Android 앱으로 설정하고 SHA1 키를 얻어서 등록 해주자

SHA1 키는 위 화면 가이드를 따라 진행했다

```java
keytool -list -v -keystore ~/.android/debug.keystore -alias androiddebugkey -storepass android -keypass android
```

그렇게 SHA1 과 Package name을 입력해주고 저장하면 된다

그리고 받아온 Key를 이제 설정해주자

### 2) Android 설정

받아온 Key를 안드로이드 폴더 안에 있는  `AndroidManifest.xml` 에 넣어주자

```java
<application
        android:name="io.flutter.app.FlutterApplication"
        android:label="google_map_test"
        android:icon="@mipmap/ic_launcher">

...

<meta-data android:name="com.google.android.geo.API_KEY"
               android:value="API Key"/>
</application>
```

API Key부분에 받아온 Key를 입력해주면 끝이다

### 3) iOS Key

![이미지](https://Funncy.github.io/assets/img/google_map/2020-08-20-google-map-05.png "iOS key")

위 사진 처럼 package name 만 설정해주고 Key를 받아오자

### 4) iOS 설정

그리고 iOS 폴더 안에 있는 AppDelegate.swift 파일을 열어서 Key를 넣어주자

```swift
import UIKit
import Flutter
import GoogleMaps

@UIApplicationMain
@objc class AppDelegate: FlutterAppDelegate {
  override func application(
    _ application: UIApplication,
    didFinishLaunchingWithOptions launchOptions: [UIApplication.LaunchOptionsKey: Any]?
  ) -> Bool {
    GMSServices.provideAPIKey("API KEY")
    GeneratedPluginRegistrant.register(with: self)
    return super.application(application, didFinishLaunchingWithOptions: launchOptions)
  }
}
```

API Key부분에 받아온 Key를 입력해주면 끝이다

> Android 와 iOS의 API KEY는 서로 다르니 주의하자

# 2. Google map 테스트

`main.dart`를 `google_map` 예제 내용으로 돌려보자

```java
import 'dart:async';

import 'package:flutter/material.dart';
import 'package:google_maps_flutter/google_maps_flutter.dart';

void main() => runApp(MyApp());

class MyApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Flutter Google Maps Demo',
      home: MapSample(),
    );
  }
}

class MapSample extends StatefulWidget {
  @override
  State<MapSample> createState() => MapSampleState();
}

class MapSampleState extends State<MapSample> {
  Completer<GoogleMapController> _controller = Completer();

  static final CameraPosition _kGooglePlex = CameraPosition(
    target: LatLng(37.42796133580664, -122.085749655962),
    zoom: 14.4746,
  );

  static final CameraPosition _kLake = CameraPosition(
      bearing: 192.8334901395799,
      target: LatLng(37.43296265331129, -122.08832357078792),
      tilt: 59.440717697143555,
      zoom: 19.151926040649414);

  @override
  Widget build(BuildContext context) {
    return new Scaffold(
      body: GoogleMap(
        mapType: MapType.normal,
        initialCameraPosition: _kGooglePlex,
        onMapCreated: (GoogleMapController controller) {
          _controller.complete(controller);
        },
      ),
      floatingActionButton: FloatingActionButton.extended(
        onPressed: _goToTheLake,
        label: Text('To the lake!'),
        icon: Icon(Icons.directions_boat),
      ),
    );
  }

  Future<void> _goToTheLake() async {
    final GoogleMapController controller = await _controller.future;
    controller.animateCamera(CameraUpdate.newCameraPosition(_kLake));
  }
}
```

# 3. 결과

![이미지](https://Funncy.github.io/assets/img/google_map/2020-08-20-google-map-06.png "result")

안드로이드 iOS 모두 다 잘 나온다