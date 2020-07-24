---
layout: post
title: "[flutter] - google map 커스텀 마커 "
subtitle:   "flutter google map custom marker"
categories: flutter
tags: flutter googlemap marker
comments: true
---

Google Map에서 마커 생성과 이동은 이전에 해결하였다.

이제는 구글 기본 마커가 아닌 이미지를 대체하여 커스텀 마커를 만들어보자. 

아주 간단하다.

### 1. 이미지 준비

먼저 마커로 쓸 이미지를 준비하고 pubsepc.yaml에 준비해주자

```yaml
flutter:

  # The following line ensures that the Material Icons font is
  # included with your application, so that you can use the icons in
  # the material Icons class.
  uses-material-design: true

  # To add assets to your application, add an assets section, like this:
  assets:
    - assets/img/
```

나는 `assets/img/marker.png` 에 등록해놓았다.

### 2. 커스텀 마커 등록

구글맵 코드에 아래 코드를 추가하자.

```java
class _GoogleMapBodyState extends State<GoogleMapBody> {
	List<Marker> _markers = [];
	Uint8List markerIcon;

  @override
  void initState() {
    super.initState();
    setCustomMapPin();
  }

  void setCustomMapPin() async {
    markerIcon = await getBytesFromAsset('assets/img/marker.png', 130);
  }

	Future<Uint8List> getBytesFromAsset(String path, int width) async {
    ByteData data = await rootBundle.load(path);
    ui.Codec codec = await ui.instantiateImageCodec(data.buffer.asUint8List(),
        targetWidth: width);
    ui.FrameInfo fi = await codec.getNextFrame();
    return (await fi.image.toByteData(format: ui.ImageByteFormat.png))
        .buffer
        .asUint8List();
  }

...

}
```

getBytesFromAsset의 width로 사이즈를 조절할 수 있다.

이제 마커를 등록해주는 코드에 icon을 연결해주자.

```java
Marker(markerId: MarkerId(element.id),
			draggable: false,
			icon: BitmapDescriptor.fromBytes(markerIcon),
			position: element.latLng)
```

그리고 실행해보면 내가 수정한 이미지로 구글맵에 마커가 등록되는걸 확인 할 수 있다.

<img src="https://Funncy.github.io/assets/img/google_map/2020-07-24-google-map-custom-marker.png" height="700"> 