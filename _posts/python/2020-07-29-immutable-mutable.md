---
layout: post
title: "[Python] - immutable vs mutable "
subtitle:   "Python - immutable vs mutable "
categories: python
tags: python immutable mutable datastructure
comments: true
---

## immutable

---

`immutable`은 변경 불가능한 객체입니다.

`일반적인 자료형(int, string, float 등)`과 `튜플(tuple)`이 있습니다.

```python
#Int
>>> a = 1
>>> id(a)
4323290896
>>> a += 1
>>> id(a)
4323290928 # 참조값이 변경된다. 즉 새로 생성된다.

#String
>>> a = 'str'
>>> id(a)
4324224752
>>> a += 'ing'
>>> id(a)
4326203504 # 참조값이 변경된다. 즉 새로 생성된다.

#Float
>>> a = 1.5
>>> id(a)
4326076944
>>> a += 0.5
>>> id(a)
4326076976 # 참조값이 변경된다. 즉 새로 생성된다.

#Tuple
>>> a = (1, 2)
>>> id(a)
4326200016
>>> a += (3, )
>>> id(a)
4326090864 # 참조값이 변경된다. 즉 새로 생성된다.
```

`immutable`은 `call by value`의 속성을 가지고 있습니다.

객체의 값이 변경되면 새로 생성되기 때문에 기존의 객체는 변경되지 않습니다.

```python
#Int 
>>> a = 1
>>> b = a
>>> a == b  # 객체가 새로 생성된다.
True
>>> a += 1
>>> a == b
False  

#String
>>> a = 'str'
>>> b = a
>>> a == b # 객체가 새로 생성된다.
True
>>> a += 'ing'
>>> a

#Float
>>> a = 1.5
>>> b = a
>>> a == b # 객체가 새로 생성된다.
True
>>> a += 0.5
>>> a == b
False

#Tuple
>>> a = (1, 2)
>>> b = a
>>> a == b # 객체가 새로 생성된다.
True
>>> a += (3, )
>>> a == b
False
```

## mutable

---

`mutable`은 변경 가능한 객체이다.

`리스트(List)`와 `딕셔너리(dict)`등이 포함됩니다.

```python
#List
>>> a = [1, 2]
>>> id(a)
4326139936
>>> a.append(3) # a += [3, ]
>>> id(a)
4326139936 #참조값 동일, 객체 유지

#Dict
>>> a = {1: '1' , 2: '2'}
>>> id(a)
4325320896
>>> a[3] = '3'
>>> id(a)
4325320896 #참조값 동일, 객체 유지
```

`mutable`은 `call by reference` 속성을 가지고 있다.

객체의 값이 변경되면 주소가 참조하는 값이 모두 변경되는 것을 확인 할 수 있다.

```python
#List
>>> a = [1, 2, 3]
>>> b = a
>>> a == b
True
>>> a.append(4)
>>> a == b   # 주소를 참조하기 때문에 주소는 변하지 않는다.
True

#Dict
>>> a = {1: '1', 2: '2'}
>>> b = a
>>> a == b
True
>>> a[3] = '3'
>>> a == b   # 주소를 참조하기 때문에 주소는 변하지 않는다.
True
```