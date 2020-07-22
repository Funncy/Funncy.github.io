---
layout: post
title: "[flutter] - google map Marker 생성 및 이동하기 "
subtitle:   "flutter google map marker"
categories: flutter
tags: flutter googlemap marker
comments: true
---

Flutter 공식 [라이브러리](https://pub.dev/packages/google_maps_flutter)를 기준으로 진행하였다.

회사에서 LBS기반 POI 서비스를 기획 및 개발하느라 Marker의 이동이 필수 였다.

카카오택시 화면처럼 이동이 가능한 Marker를 만들려고 계획하였다.

![이미지](https://Funncy.github.io/assets/img/google_map/2020-07-21-google-map-00.jpeg "카카오택시")

이렇게 지도 중앙에 Custom Marker를 띄우고 지도를 이동하여 나의 위치 정보를 전송해줘보자

## 1. Flutter Google Map

먼저 위의 라이브러리를 사용해 구글 지도부터 생성해보자.

pubspec.yaml에 google map을 등록해주고

```java
dependencies:
  flutter:
    sdk: flutter
  cupertino_icons: ^0.1.3
	google_maps_flutter: ^0.5.28+1
```

그리고  `StatefulWidget`으로 다음처럼 기본 생성해주자.

```java
class _GoogleMapBodyState extends State<GoogleMapBody> {
  static final CameraPosition _kGooglePlex = CameraPosition(
    target: LatLng(37.4537251, 126.7960716),
    zoom: 14.4746,
  );

  @override
  Widget build(BuildContext context) {
    return Container(
      child: GoogleMap(
        mapType: MapType.normal,
        initialCameraPosition: _kGooglePlex,
        onCameraMove: (_) {},
        myLocationButtonEnabled: false,
      ),
    );
  }
}
```

![이미지](https://Funncy.github.io/assets/img/google_map/2020-07-21-google-map-01.png "기본 구글 맵")

> `myLocationButtonEnabled` 은 오른쪽 하단의 나의 위치로 가기 버튼을 On/Off 하는 변수이다.

이제 기본적인 지도는 생성되었으니 `Marker` 를 연결해보자.

## 2. Marker 생성

```java
List<Marker> _markers = [];

 @override
 void initState() {
   super.initState();
   _markers.add(Marker(
       markerId: MarkerId("1"),
       draggable: true,
       onTap: () => print("Marker!"),
       position: LatLng(37.4537251, 126.7960716)));
 }

@override
  Widget build(BuildContext context) {
    return Container(
      child: GoogleMap(
        mapType: MapType.normal,
        markers: Set.from(_markers),
        initialCameraPosition: _kGooglePlex,
        onCameraMove: (_) {},
        myLocationButtonEnabled: false,
      ),
    );
  }
```

이렇게 내부에 Marker를 생성해주었다. 

![이미지](https://Funncy.github.io/assets/img/google_map/2020-07-21-google-map-02.png "기본 구글 맵 마커 생성")

기본적인 마커 생성은 쉽게 끝났다. 이제 드래그 이벤트를 진행해보자!

## 3. Marker 이벤트

`Marker`에는 기본적으로 `onTap` 말고도 `Drag` 이벤트가 등록되어있다.

```java
void _updatePosition(CameraPosition _position) {
  var m = _markers.firstWhere((p) => p.markerId == MarkerId('1'),
      orElse: () => null);
  _markers.remove(m);
  _markers.add(
    Marker(
      markerId: MarkerId('1'),
      position: LatLng(_position.target.latitude, _position.target.longitude),
      draggable: true,
    ),
  );
  setState(() {});
}

@override
Widget build(BuildContext context) {
  return Container(
    child: GoogleMap(
      mapType: MapType.normal,
      markers: Set.from(_markers),
      initialCameraPosition: _kGooglePlex,
      myLocationButtonEnabled: false,
      onCameraMove: ((_position) => _updatePosition(_position)),
    ),
  );
}
```

위에서 생성한 Marker를 카메라 움직임에 따라 지웠다 생성하며 이동하게 연결하였다.

잘 동작한다!! 하지만 문제는 애니메이션이 적용되어 내가 원하던 방식이 아니다.

## 4. 꼼수

인터넷에 검색 결과 이렇게 Marker 이동을 만들 수 있지만 Marker를 Fake로 화면에 보여주는 방법을 추천해서 그렇게 개발하기로 하였다.

Stack을 활용하자.

```java
Stack(children: <Widget>[
  GoogleMapBody(),
  Align(
    alignment: Alignment.center,
    child: Container(
    width: 50,
    height: 50,
    child: Image.asset("assets/img/marker.png"),
    ),
  ),
])
```

![이미지](https://Funncy.github.io/assets/img/google_map/2020-07-21-google-map-03.png "기본 구글 마커 이동")

이제 [Location](https://pub.dev/packages/location) 라이브러리를 활용하여 현재 위치를 받아오면 끝이다.

다음에는 Custom Marke를 공부해보고 위치 등록까지 진행해보자.