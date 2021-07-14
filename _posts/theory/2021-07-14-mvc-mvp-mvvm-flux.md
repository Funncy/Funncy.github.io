---
layout: post
title: ' MVC vs MVP vs MVVM vs Flux'
subtitle: 'MVC vs MVP vs MVVM vs flux'
categories: programming
tags: programming mvc mvp mvvm flux
comments: true
---

# MVC vs MVP vs MVVM

## 탄생 배경

---

초기 웹 개발에서는 하나의 jsp만으로도 충분했으나 점점 복잡해지며 하나의 jsp로만 개발하기 어려워졌다.

빈번한 수정에 유연하게 대응하는 구조를 고민하다가 탄생한 것이 MVC 패턴

## 들어가기 전에

---

이런 패턴들은 정답이 있는 것이 아니고 개념이라는것에 유의하자.

MVC도 MVP도 MVVM도 무엇하나 완벽한 정답이 있는것이 아니다.

> 심지어 인터넷마다 설명이 조금씩 다르다...
> [https://medium.com/@esung/mvc-정말제대로알고계신가요-875f1323f6c0](https://medium.com/@esung/mvc-%EC%A0%95%EB%A7%90%EC%A0%9C%EB%8C%80%EB%A1%9C%EC%95%8C%EA%B3%A0%EA%B3%84%EC%8B%A0%EA%B0%80%EC%9A%94-875f1323f6c0)

내가 개발해야하는 프레임워크에서 최적인 패턴과 best practice로 공부 방향을 잡아 가자

React.js : Flux 패턴
Nest.js : MVC 패턴

## 이런 패턴들은 결국 "어떻게 나눌 것인가" 의 문제이다.

## 1. MVC

가장 오래된 패턴이다. (1979년 ~ )

Model - View - Controller

- Model : 데이터를 저장하고, 읽어오고, 삭제한다.
- View : 화면을 나타내며 사용자와 인터렉션 한다.
- Controller : 입력 로직을 처리하고 Model을 가져와 비즈니스 로직 실행 후 View에 던져준다.

![이미지](https://Funncy.github.io/assets/img/MVC%20vs%20MVP%20vs%20MVVM%20928d419e6ec24654b7376938498ec88e/_2021-07-13__3.02.48.png, 'MVC')

- Controller는 Model도 알고 있고 View도 알고 있다.

### Front-end에서의 MVC

- View는 User의 Interaction을 받아와 Model을 갱신한다.
  - 즉, View는 어떤 Model을 갱신 시켜야 하는지 Model에 대한 의존성을 알고 있다.
- Model은 Controller에서도 View에서도 변경이 가능하며 그 원인도 다르다.

### Back-end에서의 MVC

![이미지](https://Funncy.github.io/assets/img/MVC%20vs%20MVP%20vs%20MVVM%20928d419e6ec24654b7376938498ec88e/_2021-07-13__3.14.37.png, 'MVC2')

- 대표적으로 스프링에서 사용된다. (Django는 MTV인데 거의 비슷하다)
- 백엔드는 화면에서 인터렉션을 받지 않고 데이터만 뿌려준다.
- 그렇기 때문에 MVC에서 의존성 문제가 발생하지 않는다.

## 2. 제왕적 MVC

![이미지](https://Funncy.github.io/assets/img/MVC%20vs%20MVP%20vs%20MVVM%20928d419e6ec24654b7376938498ec88e/_2021-07-13__3.11.51.png, 'MVC3')

- Model과 View의 의존구조는 없어졌지만 Controller가 훨씬 비대해진다.
- 그래서 점차 사용하지 않는 추세이다.

## 3. MVC의 발전 역사

---

초기에는 Controller가 비즈니스 로직 담당했다.

⇒ "데이터를 다루는 것은 모델인데 왜 비즈니스 로직은 컨트롤러에 있는가? 에 대한 문제"가 제기

> 게다가 Controller는 대부분 EndPoint이므로 재사용이 어려움

그렇다면 새로운 Layer를 추가하자!

View - Controller - `Service` - Model

하지만... 이또한 Service 코드만 늘어가고 나머지 Layer들은 점점 의미 없는 보일러 플레이트 코드(Boilerplate code)가 되어갔다.

"그렇다면 각자의 데이터는 각자 모델이 처리하고, 서비스는 각자 모델을 엮어주는 역할만 하자." 로 논의가 발전되어 갔다.

> ORM으로 대부분 서비스가 개발 되고 있는 부분도 고려하자.

또 새로운 Layer 추가!

View - Controller - Service - `Domain` - Model 의 5개 계층

도메인 객체는 순수 클래스로 비즈니스 논리를 제어하고 서비스는 도메인 객체를 컨트롤합니다

그렇다면 이것이 가장 정답일까??

정답은 없다.

아주 간단한 서비스 짜려고 기본 소스 파일만 5개로 시작해야한다...

## 4. Test관점에서 MVC

Controller가 점점 비대해지며 의존성이 올라가 테스트가 어려워진다.

![이미지](https://Funncy.github.io/assets/img/MVC%20vs%20MVP%20vs%20MVVM%20928d419e6ec24654b7376938498ec88e/2020-08-13-mvc.png, 'MVC4')

## 5. MVP

이제 그 다음 패턴인 MVP 패턴을 알아보자. (1990년~)

(2006년 MS가 .NET Framework에 적용)

MVC에서의 문제는 View와 Model의 의존관계가 문제였다.

이걸 완전히 끊을 수 있다면??

![이미지](https://Funncy.github.io/assets/img/MVC%20vs%20MVP%20vs%20MVVM%20928d419e6ec24654b7376938498ec88e/_2021-07-13__9.20.02.png, 'MVP')

- 이제는 View와 Presenter가 1:1 관계이다.
- 입력을 받아서 Model과의 처리를 Presenter가 중앙에서 다 관리해준다.

> 그렇다면 제왕적 MVC와는 무엇이 다른가??
> MVC에서는 보통 화면 rebuild를 View에서 처리했다면
> MVP에서는 Presenter가 rebuild를 호출한다.

### MVC보다 무조건 좋은가??

⇒ View와 Presenter가 1:1관계가 되어서 화면의 숫자만큼 Presenter가 필요해졌다...

자신의 프레임워크 상황에 맞춰서 진행하자.

안드로이드 같은 경우 MVP 패턴을 많이 활용한다.
Activity자체가 View와 Controller의 역할을 이미 하고 있기에 역할 분리와 로직 단순화를 위해 MVP 패턴을 많이 채용한다.
이건 해당 프레임워크의 특성에 따라 달라진다.

## 6. Test에서의 MVP

![이미지](https://Funncy.github.io/assets/img/MVC%20vs%20MVP%20vs%20MVVM%20928d419e6ec24654b7376938498ec88e/2020-08-13-mvp.png, 'MVP2')

## 7. MVVM

대망의 mvvm이다. 마틴 파울러 아저씨가 만드셨고 WPF, 실버라이트에 사용되었다.

MVP에서 개선한 아이디어이며 데이터 바인딩을 통해 업데이트를 해결한다.

- ViewModel : View를 위한 Model이란 의미로 View는 ViewModel만 바라보며 Model의 의존성이 끊기게 된다.
  그러면서 MVP의 문제점인 View와 Presenter의 의존성 문제도 `데이터 바인딩`으로 해결하였다.

M:1 관계로 Presenter처럼 화면마다 한개씩 만들어 줄 필요는 사라졌다.
여기서 특징은 View는 이제 입력을 받고 데이터 변경을 Subscription하여서 ViewModel을 바라본다.
ViewModel은 View를 신경쓰지 않고 데이터 처리를 하면 데이터 변경에 따라 View가 변경된다.

![이미지](https://Funncy.github.io/assets/img/MVC%20vs%20MVP%20vs%20MVVM%20928d419e6ec24654b7376938498ec88e/_2021-07-13__9.29.06.png, 'MVVM')

## 8. Test에서의 MVVM

![이미지](https://Funncy.github.io/assets/img/MVC%20vs%20MVP%20vs%20MVVM%20928d419e6ec24654b7376938498ec88e/2020-08-13-mvvm.png, 'MVVM2')

## 9. Flux

페이스북에서 만든 패턴이며 기존 MVC의 단점에서 출발했다.

![이미지](https://Funncy.github.io/assets/img/MVC%20vs%20MVP%20vs%20MVVM%20928d419e6ec24654b7376938498ec88e/_2021-07-13__9.32.28.png, 'FLUX')

양방향 데이터 흐름때문에 여러 사이드 이팩트와 의존 문제가 발생한다.

![이미지](https://Funncy.github.io/assets/img/MVC%20vs%20MVP%20vs%20MVVM%20928d419e6ec24654b7376938498ec88e/_2021-07-13__9.33.16.png, 'FLUX2')

- 대표적으로 메시지 숫자는 이곳 저곳에서 업데이트가 동시에 되야하므로 버그가 많이 발생했다고 한다.

이 문제를 단방향 데이터 흐름으로 해결하고자 하였고 그것을 Flux 패턴이라고 칭했습니다.

![이미지](https://Funncy.github.io/assets/img/MVC%20vs%20MVP%20vs%20MVVM%20928d419e6ec24654b7376938498ec88e/_2021-07-13__9.33.49.png, 'FLUX3')

- Dispatcher : Flux의 모든 데이터 흐름을 관리하며 분배해주는 역할을 한다.
  Store 사이에 의존성이 있다면 waitFor()를 사용하여 동기적 처리도 가능하다.
- Store : 모든 상태와 로직을 담고 있다. Dispatcher에 콜백함수로 등록하여 액션이 호출되면 실행된다. 상태가 변경되면 View에 알려준다.
- View : Flux에서의 View는 컨트롤러의 성격또한 가지고 있다.
  최상위 View는 자식들에게 Store를 내려보내주는 역할을 한다.
- Action : Store 업데이트는 Dispatcher에서 콜백함수로 이뤄지는데
  이때 파라미터로 들어가는 객체를 Action이다.

- 부록

  ![이미지](https://Funncy.github.io/assets/img/MVC%20vs%20MVP%20vs%20MVVM%20928d419e6ec24654b7376938498ec88e/_2021-07-13__3.21.13.png, 'performance')

  ![이미지](https://Funncy.github.io/assets/img/MVC%20vs%20MVP%20vs%20MVVM%20928d419e6ec24654b7376938498ec88e/_2021-07-13__3.21.19.png, 'performance 2')
