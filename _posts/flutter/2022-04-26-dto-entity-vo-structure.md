---
layout: post
title: '[Flutter] - DTO, VO, Entity (프로젝트 구조 고민)'
subtitle: 'Flutter DTO, VO, Entity with structure'
categories: flutter
tags: flutter dto vo entity structure
comments: true
---


먼저 백엔드와 프론트엔드에 따라 내용이 좀 달라지는것 같다.

flutter기준에서 한번 고민해보자.

먼저, DTO, VO, Model, Entity에 대한 내용을 확인해보자.

### ****DTO (Data Transfer Object)****

---

****각 계층간 데이터 교환을 위한 객체****

(ex. View <-(DTO)-> Controller <-(DTO)-> Service )

- 로직을 가지지 않는 순수한 데이터 객체

> setter도 만들지 않고 contructor에서 값을 할당한다.
> 

### VO (****Value Object)****

---

- 불변 객체이고 Equals 메소드를 지원해 비교 연산이 가능하다.

### ****Entity****

---

****실제 DB 테이블과 매칭될 클래스****

- Domain Logic만 가지고 있어야 하고 Presentation Logic을 가지고 있어선 안된다.
- 주로 Service Layer에서 사용

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/cb8150d2-678d-4d96-b0c4-cddced161720/Untitled.png)

아키텍쳐를 생각해보면

데이터베이스에서 DAO를 통해서 데이터가 들어와 

비즈니스 로직을 통과해서

화면에 가공된 데이터가 뿌려져야 한다.

## 추가 정리

---

[https://stackoverflow.com/questions/39397147/difference-between-entity-and-dto](https://stackoverflow.com/questions/39397147/difference-between-entity-and-dto)

위에서 좋은 정리글을 발견하였다.

> "데이터 전송 개체"(DTO)라는 용어는 매우 명확하게 정의되지만 "엔티티"라는 용어는 다양한 컨텍스트에서 다르게 해석됩니다.
> 

그래서 Entity는 아래와 같이 크게 3가지로 해석된다.

1. *엔터프라이즈 Java 및 jpa의 컨텍스트에서:*"데이터베이스에서 유지 관리되는 영구 데이터를 나타내는 개체입니다." ⇒ Spring에서의 Entity (DB 테이블 매핑된 객체)
    
    > DB 테이블 매핑된 객체는 DTO와 유사하여 여기서 혼란이 많이 발생
    > 
2. *"domain-driven-design"의 맥락에서(Eric Evans 작성):*"주로 속성보다는 ID에 의해 정의된 개체입니다."
3. *"클린 아키텍처"(Robert C. Martin 작성)의 맥락에서:*"전사적인 중요한 비즈니스 규칙을 캡슐화하는 개체."

<aside>
💡 이걸 보니 이제 조금 정리되었다.
어디서는 Entity가 비즈니스 모델의 객체로 쓰이고 어디서는 DB와 연관된 객체로 설명을 해서 많이 햇갈렸었다...
당장 Flutter에서는 비즈니스 로직에서의 Entity로 해석하면 될것 같다.

</aside>

## 적용

---

### 현재 상황

---

이제 그럼 우리 프로젝트에서 어떻게 적용하면 좋을지 고민해보자.

현재 프론트 개발자 3명 백엔드 1명인 소규모 프로젝트에서 (게다가 쥬니어 개발자들)

빠른 프로토타입으로 시장 반응을 보기 위한 앱을 개발한다는 조건에서

무거운 프로젝트 구조는 아니라고 판단했다.

entity, dto, vo 다 만들어서 견고하게 만드는 선택도 있겠지만

너무 코드가 많아지고, 이 구조를 이해시키고 편하게 사용하기까지도 러닝커브가 있으니

최소한의 구조로 프로젝트를 운영하고자 한다.

### Entity? Model? VO?

---

일단 이 개념전에 기존에 진행했던 프로젝트에서의 불편하고 어려웠던점을 정리해보자.

1. 하나의 모델에서 fromJson, toJson으로 모든걸 처리하는게 매우 불편했다.
당장 user만 봐도 생성할때 보내는 데이터와 생성된 유저를 받을때 데이터, 업데이트할때 데이터가 다르다.
    
    > toCreateJson(), toUpdateJson() 이런 방식으로 할까 하다가 그냥 map으로 데이터를 보내줬다.
    flutter에서는 map의 key가 string이라 오타나기 쉽다
    > 
2. model 데이터를 formating해서 보여줘야 하는 경우가 많았다. 운동 앱이다 보니 속도, 거리, 평균속도, 평균 페이스 등의 double 데이터가 화면에서는 string으로 포맷팅 되야 했는데 (연산 포함) 
이걸 화면에서 매번 하니 중복 코드가 많았고 이걸 model로 옮기자니 model이 너무 뚱뚱해져서 관리하기 어려워졌다.

일단 2번은 freezed 라이브러리를 쓰고 custom method를 쓰면 어느정도 해결될것 같다.

freezed 라이브러리가 여러 장점이 있지만 내가 느끼기에 최고의 장점은 생성된 함수를 다 숨겨버린다는 것이다. 

> 내가 커스텀한 내용만 보이기에 가독성이 좋다!
> 

여기서 고민이 1번이다. nset.js 에서 했던것 처럼 login.dto, user.dto, password_change.dto ... (이벤트 별로) 그리고 Entity 사이에서 mapper 클래스를 만들어서 변경하는 방법도 있을것 같다

결론은 model로 Entity, vo 다 퉁쳐서 사용하고 

action별로 (도메인별) 정리하여 개발해보자.  

> freezed를 사용할거라 불변객체로 생성되니 vo적인 특성도 가질거라 판단되었다.
> 

## 폴더 구조

---

지금 생각하는 폴더 구조는

- data
    - auth
        - login
            - LoginRemoteRepository (with Retrofit)
            - LoginCacheRepository (option)
- Domain
    - auth
        - login
            - LoginModel (with Freezed)
            - LoginUseCaseImpl
- UI
    - page
        - LoginPage
    - ViewModel
        - auth
            - LoginViewModel
            - LoginUseCase

위 폴더 구조를 사용해보고 피드백을 달아보자.