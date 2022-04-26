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


![이미지](https://Funncy.github.io/assets/img/0426/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA_2022-04-26_%E1%84%8B%E1%85%A9%E1%84%8C%E1%85%A5%E1%86%AB_11.06.23.png "xcode orientation 설정")

찾아보니 info.plist에서 아래와 같이 분기로 설정이 가능했다.

![이미지](https://Funncy.github.io/assets/img/0426/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA_2022-04-26_%E1%84%8B%E1%85%A9%E1%84%8C%E1%85%A5%E1%86%AB_11.07.10.png "xcode orientation info.plist")
