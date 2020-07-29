---
layout: post
title: "[Python] - 얕은 복사 vs 깊은 복사 "
subtitle:   "Python - shallow copy vs deep copy "
categories: python
tags: python copy deepcopy shallowcopy
comments: true
---

## 1. 단순 객체 복사

---

`immutable` 객체를 복사하면 주소를 참조하기 때문에 값이 변경되면 같이 변경됩니다.

`mutable` 객체 일 경우 새로 생성하기 때문에 값이 변경되면 기존 값은 유지 됩니다.

```python
# Immutable (int, string, float, tuple..)
>>> a = 4
>>> b = a
>>> b
4
>>> a += 1
>>> a
5
>>> b
4        # 값 다름, call by value로 값을 새로 생성함

# Mutable (list, dict..)
>>> a = [1, 2, 3]
>>> b = a
>>> b
[1, 2, 3]
>>> a.append(4)
>>> a
[1, 2, 3, 4]
>>> b
[1, 2, 3, 4]  # 값 동일하게 변경됨
```

## 2. 얕은 복사(shallow copy)

---

얕은 복사는 단순 복사와 다르게 새로 객체를 생성해서 복사하지만 

`내용물은 같은 객체`입니다.

```python
>>> import copy   # 얕은 복사를 위해 copy 라이브러리 사용
>>> a = [1, [1, 2, 3]]
>>> b = copy.copy(a)  # 얕은 복사
>>> b
[1, [1, 2, 3]]
>>> b[0] = 100   # 객체 내부 immutable 데이터 변경
>>> b
[100, [1, 2, 3]]
>>> a
[1, [1, 2, 3]]   # immutable 데이터는 새로 생성되었기 때문에 기존 값 변경 안됨
>>> b[1].append(4) # 객체 내부 mutable 데이터 변경
>>> b
[100, [1, 2, 3, 4]]
>>> a
[1, [1, 2, 3, 4]]  # mutable 데이터는 주소를 참조하기 때문에 기존 값도 같은 주소 참조 중
```

`객체 내부의 immutable 객체`는 `값이 복사`되기 때문에 기존 값이 변경되지 않습니다.

하지만 `객체 내부의 mutable 객체`는 참조하는 `주소 값이 복사`되어서 기존 값이 같이 변경됩니다.

## 3. 깊은 복사(deep copy)

---

깊은 복사는 내부의 mutable한 객체까지 새로 생성하여 복사를 진행합니다.

```python
>>> import copy   # 깊은 복사를 위해 copy 라이브러리 사용
>>> a = [1, [1, 2, 3]]
>>> b = copy.deepcopy(a) # 깊은 복사
>>> b
[1, [1, 2, 3]]
>>> b[0] = 100   # 객체 내부 immutable 데이터 변경
>>> b[1].append(4)  # 객체 내부 mutable 데이터 변경
>>> b
[100, [1, 2, 3, 4]]
>>> a
[1, [1, 2, 3]]      # 모든 값이 제대로 복사 완료 , 기존값 변경 안됨
```

`deepcopy`는 기존 객체와 복사된 객체가 완전히 달라지기 때문에 복사된 객체를 수정하여도 기존 객체는 변경되지 않습니다.