---
layout: post
title: "[Python] - 리스트 인덱스 슬라이싱 "
subtitle:   "Python - list index slicing "
categories: python
tags: python list index slicing
comments: true
---


python에서 list를 슬라이싱(slicing)하는 방법을 살펴보자

```python
a[start:end:step]

a = [a, b, c, d, e]
  # | a| b| c| d| e|
  # | 0| 1| 2| 3| 4|
  # |-5|-4|-3|-2|-1|
```

기본적으로 start, end, step으로 구성되며 양수와 음수로 나눠진다

- 양수 : 제일 앞부터 0으로 시작하여 번호를 센다
- 음수 : 제일 뒤부터 -1로 시작하여 번호를 센다
- start : 시작 위치, 0부터 시작한다

```python
>>> a = ['a', 'b', 'c']
>>> a[ 1 :  ]
['b', 'c']
```

- end : 마지막 위치 , end는 포함하지 않는다

```python
>>> a = ['a', 'b', 'c']
>>> a[  : 2 ]
['a', 'b']
```

- step : 몇 개씩 가져올지 단계 (옵션)

```python
>>> a = ['a', 'b', 'c', 'd']
# 2칸씩 이동하면서 가져온다.
>>> a[ : : 2 ]
['a', 'c']
# 전체를 거꾸로 가져온다.
>>> a[ : : -1]
['d', 'c', 'b', 'a']
# 시작점이 -3인곳에서부터 끝점 모두 2칸씩 띄어서 가져온다.
>>> a[-3: : 2]
['b', 'd']
```

주의할 점은 a[0:3] 하면 [a,b,c]만 가져온다. 즉 end인 인덱스 3번은 가져오지 않는다.