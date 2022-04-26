---
layout: post
title: '[Programming] - XCode 회전 분기 설정 '
subtitle: 'programming -  XCode orientation branch setting'
categories: programming
tags: programming xcode orientation
comments: true
---

# [Issue Solved] XCode 회전 분기 설정

이번에 모바일버전을 개발하면서 아래와 같은 요구사항이 생겼다.

- 태블릿은 가로 모드 고정
- 모바일은 세로 모드 고정

그런데 Xcode에는 아래와 같은 옵션뿐이였다...


![이미지](https://Funncy.github.io/assets/img/0426/스크린샷_2022-04-26_오전_11.06.23 "xcode orientation 설정")

찾아보니 info.plist에서 아래와 같이 분기로 설정이 가능했다.

![이미지](https://Funncy.github.io/assets/img/0426/스크린샷_2022-04-26_오전_11.07.10"xcode orientation info.plist")
