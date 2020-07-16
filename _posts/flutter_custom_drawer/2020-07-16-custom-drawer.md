---
layout: post
title: "[flutter] -  Custom Slide Panel "
subtitle:   "flutter slide panel"
categories: flutter
tags: flutter
comments: true
---

![이미지](https://Funncy.github.io/assets/img/custom_drawer/2020-07-15-SLIDE_PANEL.gif "SLIDE PANEL")

`TodoCare` 앱을 만들려고 하다보니 좌우에서 Slide Panel이 나올 필요가 있었다.

하지만 내가 찾아보기로 Plugin이 없어 내가 직접 만들어 배포하기로 하였다.

`Transform`과 `AnimationController`를 활용하여 개발하였다.

### 1. 화면 설계

먼저 Panel의 Width와 Height 그리고 손잡이 Handler Width를 직접 지정해서 사용하게 해주고 싶었다.

```java
SlidePanel(
	slideHandlerWidth : 20,
	slidePanelWidth: 300,
  slidePanelHeight: 300,
)
```

그리고 Body와 Left, Right를 구분하여 내용물을 집어넣게 하였다.

```java
SlidePanel(
	slideHandlerWidth : 20,
	slidePanelWidth: 300,
  slidePanelHeight: 300,
	body: body,
	leftSlide: ...,
	rightSlide: ...,
)
```

Left와 Right는 동일하니 Left를 기준으로 설명하겠다.

`LeftSlidePanel`에는 내용물은 main에서 받아온 `body`를 넘겨주고 `slidePanel`의 사이즈를 넣어준다.

```java
LeftSlidePanel(
	body: Container(
		width: widget.slidePanelWidth + widget.slideHandlerWidth,
		height: widget.slidePanelHeight,
		color: Colors.red, //Test용 색 
		child: widget.leftSlide),
	slideHandlerWidth: widget.slideHandlerWidth,
	slidePanelHeight: widget.slidePanelHeight,
	slidePanelWidth: widget.slidePanelWidth,
)
```

여기서 `body`에 `Container`로 감싸준 이유는 `SlidePanel`의 사이즈만큼의 공간에 내용물을 넣기 위함이다.

그리고 `Positioned`을 활용해 `SlidePanel`을 화면에서 숨겨주자.

```java
Positioned(
	top: (size.height - widget.slidePanelHeight) / 2, // 화면 정중앙에 놓기
	left: -widget.slidePanelWidth, // Handler를 제외한 너비만큼 숨기기
	child: widget.body, // 내용물
)
```

### 2. 애니메이션

자 이제 `Transform`과 `AnimationController`를 활용하여 슬라이드 효과를 줘보자.

```java
AnimatedBuilder(
      animation: _animationController,
      builder: (context, _) {
        final leftSlide = widget.slidePanelWidth * _animationController.value;
				// 애니메이션 값(0.0 ~ 1.0) 만큼 Width값 이동  
        return Positioned(
          top: (size.height - widget.slidePanelHeight) / 2,
          left: -widget.slidePanelWidth,
          child: Transform(
              alignment: Alignment.centerLeft,
              transform: Matrix4.identity()..translate(leftSlide), // 화면 이동
              child: widget.body ),
        );
      },
    )
```

드래그에 따라서 화면이 이동해야하니 `GestureDetector`를 활용하여 Drag를 해보자.

```java
class _LeftSlidePanelState extends State<LeftSlidePanel>
    with SingleTickerProviderStateMixin {
  static const Duration toggleDuration = Duration(milliseconds: 250); // 애니메이션 시간
  AnimationController _animationController; // 애니메이션 컨트롤러
  bool _canBeDragged = false; // Drag 가능 여부
  @override
  void initState() {
    super.initState();
    _animationController =
        AnimationController(vsync: this, duration: toggleDuration);
  }

  void close() {
    _animationController.reverse();
  }

  void open() {
    _animationController.forward();
  }

  // Drag가능 범위 (Handler)인지 확인 후 Drag 가능 여부 판단
  void _onDragStart(DragStartDetails details) {
    bool isDragOpenFromLeft = _animationController.isDismissed &&
        details.globalPosition.dx < widget.slideHandlerWidth;
    bool isDragCloseFromRight = _animationController.isCompleted &&
        details.globalPosition.dx >
            (widget.slidePanelWidth - widget.slideHandlerWidth);
    _canBeDragged = isDragOpenFromLeft || isDragCloseFromRight;
  }

  // Drag 가능 하면 애니메이션 컨트롤러 값 증가
  void _onDragUpdate(DragUpdateDetails details) {
    if (_canBeDragged) {
      double delta = details.primaryDelta / widget.slidePanelWidth;
      _animationController.value += delta;
    }
  }

	// Drag 끝나면 분기에 따라 처리
  void _onDragEnd(DragEndDetails details) {
    
		// 이미 애니메이션이 끝나거나 취소 됬으면 바로 종료
		if (_animationController.isDismissed || _animationController.isCompleted) {
			return ;
    }

		// 특정 속도 이상으로 드래그 할 경우
    if (details.velocity.pixelsPerSecond.dx.abs() >= 365.0) {
      double visualVelocity = details.velocity.pixelsPerSecond.dx /
          MediaQuery.of(context).size.width;

			//  애니메이션 Fling
      _animationController.fling(velocity: visualVelocity);
    } else if (_animationController.value < 0.5) {
			// 절반 이상 못오면 애니메이션 reverse
      close();
    } else {
			// 절반 이상 진행 했으면 애니메이션 forward
      open();
    }
  }

  @override
  Widget build(BuildContext context) {
    var size = MediaQuery.of(context).size;

    return AnimatedBuilder(
      animation: _animationController,
      builder: (context, _) {
        final leftSlide = widget.slidePanelWidth * _animationController.value;

        return Positioned(
          top: (size.height - widget.slidePanelHeight) / 2,
          left: -widget.slidePanelWidth,
          child: Transform(
              alignment: Alignment.centerLeft,
              transform: Matrix4.identity()..translate(leftSlide),
              child: GestureDetector(
                      onHorizontalDragStart: _onDragStart,
                      onHorizontalDragUpdate: _onDragUpdate,
                      onHorizontalDragEnd: _onDragEnd,
                      child: widget.body,
                    )
							),
        );
      },
    );
  }
}
```

이제 `Drag`가 정상적으로 동작한다.

그런데 문제가 있다. 

오른쪽과 왼쪽 `Slide Panel`이 한쪽이 나와있으면 다른쪽이 못나오거나 들어가줘야한다.

이부분을 `Provider`를 활용해 해결해봤다.

```java
class IsOpenedProvider with ChangeNotifier {
  bool isLeftOpened = false;  // 왼쪽 슬라이드 패널 오픈 여부
  bool isRightOpened = false; // 오른쪽 슬라이드 패널 오픈 여부

  bool getIsLeftOpened() => isLeftOpened;
  bool getIsRightOpened() => isRightOpened;

  void setOpenLeftState(bool state) {
    isLeftOpened = state;
    notifyListeners();
  }

  void setOpenRightState(bool state) {
    isRightOpened = state;
    notifyListeners();
  }
}
```

그리고 패널별 동작을 추가해주자

먼저 왼쪽 패널에서는

```java
void _onDragEnd(DragEndDetails details, IsOpenedProvider isOpenedProvider) {
    if (_animationController.isDismissed) {
			// 애니메이션 취소된 상태이면 왼쪽 패널 상태를 열리지 않음으로 변경
      isOpenedProvider.setOpenLeftState(false);
      return;
    }

    if (_animationController.isCompleted) {
			// 애니메이션 성공이면 왼쪽 패널 다 열린 상태로 변경
      isOpenedProvider.setOpenLeftState(true);
      return;
    }
    if (details.velocity.pixelsPerSecond.dx.abs() >= 365.0) {
      double visualVelocity = details.velocity.pixelsPerSecond.dx /
          MediaQuery.of(context).size.width;

			//여는 속도가 +면 오른쪽 이동중(열리는 중) 열린거로 오픈
      if (visualVelocity > 0) {
        isOpenedProvider.setOpenLeftState(true);
      } else {
				// 여는 속도가 -면 왼쪽으로 이동중(닫히는 중) 닫힌거로 설정 
        isOpenedProvider.setOpenLeftState(false);
      }

      _animationController.fling(velocity: visualVelocity);
    } else if (_animationController.value < 0.5) {
			//열거나 닫을때 맞춰서 상태 설정
      isOpenedProvider.setOpenLeftState(false);
      close();
    } else {
      isOpenedProvider.setOpenLeftState(true);
      open();
    }
  }

@override
  Widget build(BuildContext context) {
		// 프로바이더 가져오기
    IsOpenedProvider isOpenedProvider = Provider.of<IsOpenedProvider>(context);
    var size = MediaQuery.of(context).size;

		// 현재 패널의 상태가 False(안열림)이지만 열려있는 상태 (애니메이션 완료)면 애니메이션 역재생
    if (isOpenedProvider.getIsLeftOpened() == false &&
        _animationController.isCompleted) {
      _animationController.reverse();
    }

    return AnimatedBuilder(
      animation: _animationController,
      builder: (context, _) {
        final leftSlide = widget.slidePanelWidth * _animationController.value;
        return Positioned(
          top: (size.height - widget.slidePanelHeight) / 2,
          left: -widget.slidePanelWidth,
          child: Transform(
              alignment: Alignment.centerLeft,
              transform: Matrix4.identity()..translate(leftSlide),
              child: isOpenedProvider.getIsRightOpened()
                  ? GestureDetector(
                      onTap: () {
                        isOpenedProvider.setOpenRightState(false);
                      },
                      child: widget.body)
                  : GestureDetector(
                      onHorizontalDragStart: _onDragStart,
                      onHorizontalDragUpdate: _onDragUpdate,
                      onHorizontalDragEnd: (details) =>
                          _onDragEnd(details, isOpenedProvider),
                      child: widget.body,
                    )),
        );
      },
    );
  }
```

오른쪽 패널도 다음과 같이 설정하자.

```java
void _onDragEnd(DragEndDetails details, IsOpenedProvider isOpenedProvider) {
    if (_animationController.isDismissed) {
      isOpenedProvider.setOpenRightState(false);
      return;
    }

    if (_animationController.isCompleted) {
      isOpenedProvider.setOpenRightState(true);
      return;
    }

    if (details.velocity.pixelsPerSecond.dx.abs() >= 365.0) {
      double visualVelocity = details.velocity.pixelsPerSecond.dx /
          MediaQuery.of(context).size.width;

			//여는 속도가 +면 오른쪽 이동중(닫히는 중) 닫힌거로 설정
      if (visualVelocity > 0) {
        isOpenedProvider.setOpenRightState(false);
      } else {
				//여는 속도가 -면 왼쪽 이동중(열리는 중) 열린거로 설정
        isOpenedProvider.setOpenRightState(true);
      }

      _animationController.fling(velocity: -visualVelocity);
    } else if (_animationController.value < 0.5) {
      isOpenedProvider.setOpenRightState(false);
      close();
    } else {
      isOpenedProvider.setOpenRightState(true);
      open();
    }
  }

  @override
  Widget build(BuildContext context) {
    var size = MediaQuery.of(context).size;
    IsOpenedProvider isOpenedProvider = Provider.of<IsOpenedProvider>(context);

    if (isOpenedProvider.getIsRightOpened() == false &&
        _animationController.isCompleted) {
      _animationController.reverse();
    }

    return AnimatedBuilder(
      animation: _animationController,
      builder: (context, _) {
        final rightSlide = -widget.slidePanelWidth * _animationController.value;
        return Positioned(
          top: (size.height - widget.slidePanelHeight) / 2,
          left: size.width - widget.slideHandlerWidth,
          child: Transform(
              alignment: Alignment.centerLeft,
              transform: Matrix4.identity()..translate(rightSlide),
              child: isOpenedProvider.getIsLeftOpened() == true
                  ? GestureDetector(
                      onTap: () {
                        isOpenedProvider.setOpenLeftState(false);
                      },
                      child: widget.body)
                  : GestureDetector(
                      onHorizontalDragStart: (details) =>
                          _onDragStart(details, size),
                      onHorizontalDragUpdate: _onDragUpdate,
                      onHorizontalDragEnd: (details) =>
                          _onDragEnd(details, isOpenedProvider),
                      child: widget.body,
                    )),
        );
      },
    );
  }
```

이제 정상적으로 Drag할 경우 동작한다.

3. 추가 옵션

여기서 추가로 옵션으로 설정한게

1. 메인 화면 터치 시 슬라이드 닫기
2. 좌우 슬라이드 on/off 

먼저 메인 화면 터치 시 슬라이드 닫기부터 변수를 내려서 설정해주었다.

```java
SlidePanel(
	...
	slideOffBodyTap: true,
	...
)
```

`BodyPanel`에서 설정해주자.

`slideOffBodyTap`가 `true`인 경우에는 `GestureDector`를 통해 Tap에 slide 닫기 기능을 연결해주자.

`Provider`의 값을 변경하면 화면이 rebuild되면서 그 값에 따라 슬라이드가 동작한다.

```java
class _BodyPanelState extends State<BodyPanel> {
  @override
  Widget build(BuildContext context) {
    IsOpenedProvider isOpenedProvider = Provider.of<IsOpenedProvider>(context);
    if (widget.slideOffBodyTap) {
      return GestureDetector(
        onTap: () {
          isOpenedProvider.setOpenLeftState(false);
          isOpenedProvider.setOpenRightState(false);
        },
        child: widget.body,
      );
    }
    return widget.body;
  }
}
```

두번째로 좌우 슬라이드를 옵션으로 설정하게 해주자.

```java
SlidePanel(
	...
	leftPanelVisible: true,
  rightPanelVisible: true,
	...
)
```

이렇게 변수를 전달하여 다음과 같이 코드를 작성하였다.

```java
class _SlidePanelState extends State<SlidePanel> {
  bool isOpened = false;
  @override
  Widget build(BuildContext context) {
    return ChangeNotifierProvider<IsOpenedProvider>(
      create: (_) => IsOpenedProvider(),
      child: Stack(
        children: <Widget>[
          BodyPanel(
            body: widget.body,
            slideOffBodyTap: widget.slideOffBodyTap,
          ),
					// 왼쪽 패널 Visible이 true면 그리기 아니면 빈 Container
          widget.leftPanelVisible
              ? LeftSlidePanel(
                  body: Container(
                      width: widget.slidePanelWidth + widget.slideHandlerWidth,
                      height: widget.slidePanelHeight,
                      color: Colors.red,
                      child: widget.leftSlide),
                  slideHandlerWidth: widget.slideHandlerWidth,
                  slidePanelHeight: widget.slidePanelHeight,
                  slidePanelWidth: widget.slidePanelWidth,
                )
              : Container(),
					// 오른쪽 패널 Visible이 true면 그리기 아니면 빈 Container
          widget.rightPanelVisible
              ? RightSlidePanel(
                  body: Container(
                      width: widget.slidePanelWidth + widget.slideHandlerWidth,
                      height: widget.slidePanelHeight,
                      color: Colors.green,
                      child: widget.rightSlide),
                  slideHandlerWidth: widget.slideHandlerWidth,
                  slidePanelHeight: widget.slidePanelHeight,
                  slidePanelWidth: widget.slidePanelWidth,
                )
              : Container()
        ],
      ),
    );
  }
}
```

이제 모두 정상 작동한다.

나중에 수정할 부분은 Handler와 Panel body를 구분하여 Handler를 따로 만들어서 연결하게 수정할 예정이다.

일단 다 작성한 내용을 Plugin으로 만들계획이다.