---
layout: post
title: " MVC vs MVP vs MVVM "
subtitle:   "MVC vs MVP vs MVVM "
categories: programming
tags: programming mvc mvp mvvm
comments: true
---


항상 헷갈려 왔던 내용들을 이번에 정리 해보려고 한다.

## 1. MVC

역할에 따라 구분하자!

MVC는 Controller가 많은 역할을 감당한다. 

View는 화면을 그리기만 하고 Controller가 Model(DB)와 주고 받으며 정리된 데이터를 View에 던져준다.

그러다 보니 Controller가 너무 커지고 힘들어졌다.

![이미지](https://Funncy.github.io/assets/img/programming/2020-08-13-mvc.png "mvc")


## 2. MVP

그래서 이제 입력은 view가 받고 대신 모든 입력과 처리를 Presenter가 중간에서 처리해준다.
View는 정말 그리기만 하고 들어온 정보와 그릴 정보를 모두 Presenter로 부터 보내거나 받는다.
문제는 View와 Presenter가 1:1 이라서 매번 View가 늘어날 때마다 Presenter를 만들어줘야하는 번거로움이 있다. 

![이미지](https://Funncy.github.io/assets/img/programming/2020-08-13-mvp.png "mvp")

## 3. MVVM

MVP에서 입력을 Presenter가 아니라 View가 받고 그리는걸 신경쓰지 않고 로직에만 신경쓰는게 ViewModel이다. 특징은 M:1 관계로 Presenter처럼 화면마다 한개씩 만들어 줄 필요는 사라졌다.
여기서 특징은 View는 이제 입력을 받고 데이터 변경을 Subscription하여서 ViewModel을 바라본다.
ViewModel은 View를 신경쓰지 않고 데이터 처리를 하면 데이터 변경에 따라 View가 변경된다.

![이미지](https://Funncy.github.io/assets/img/programming/2020-08-13-mvvm.png "mvvm")

출처 : [https://www.youtube.com/watch?v=bjVAVm3t5cQ](https://www.youtube.com/watch?v=bjVAVm3t5cQ)