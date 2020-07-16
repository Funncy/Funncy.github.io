---
layout: post
title: "[flutter] - Package 제작 및 배포 "
subtitle:   "flutter package"
categories: flutter
tags: flutter package
comments: true
---

### 1. Package 프로젝트 생성

[pub.dev](http://pub.dev)에 배포를 진행해보자.

먼저 package 프로젝트를 생성해주자.

```dart
flutter create --template=package MY_PACKAGE_NAME
```

나는 slide_panel로 만들어주었다.

그리고 lib 폴더에 있는 패키지 이름과 같은 dart 파일을 열어서 수정해준다.

```dart
library slide_panel;

export 'views/slide_panel.dart';
```

나는 views라는 폴더를 만들어 ui관련된 내용을 넣을거라서 그 내부에 있는 slide_panel을 export 시켜줬다.

나머지는 내가 만든 Package를 옮겨서 넣어주자.

### 2. Example 생성

코드를 잘 옮겼으면 이제 정상 작동하는지 여부를 보여줘야할 Example을 넣어주자.

현재 디렉토리에서 example 이름으로 flutter 프로젝트를 생성해주자.

```bash
flutter create example
```

그리고 Example에 있는 pubspec.yaml 을 열어서 내가 만든 package를 연결해주자.

```yaml
dependencies:
  flutter:
    sdk: flutter

  cupertino_icons: ^0.1.3
  slide_panel:
    path: ../
```

이제 Example을 시뮬레이터로 실행해 정상 작동하는지 확인한다.

정상 작동을 한다면 이제 배포를 준비해보자.

### 3. 배포

배포 전 

- README.md
- pubspec.yaml
- CHNGELOG.md

를 작성해주자.

README.md는 내 플러그인의 설명을 담아주자. Gif가 필요하다면 GIPHY 를 활용해 촬영해주자.

그리고 pubspec.yaml에는 다음과 같은 정보를 넣어주자.

```yaml
name: 패키지 이름
description: 내 패키지 설명
version: 0.0.1
publisher: 나의 이메일
homepage: 나의 홈페이지 
repository: 나의 깃주소
```

그리고 CHANGELOG.md에 처음이니 0.0.1에 대한 기록만 적어주자.

```markdown
## [0.0.1] - Initial pre-release

* slide panel functionality
```

이제 기본적인 배포 준비를 끝냈으니 배포 최종 테스트를 해보자.

```java
flutter packages pub publish --dry-run
```

정상적으로 종료되었다면 이제 진짜 배포를 해보자.

```java
flutter packages pub publish
```

배포를 시작하면 구글 로그인을 통해 인증을 요청한다. 터미널에 나온 url로 들어가 인증해주자.

업로드가 완료되면 1~2일 후 승인 되었다고 메일이 도착한다.