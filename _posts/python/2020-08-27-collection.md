---
layout: post
title: "[Python] - Collection [Set, Dictionary] "
subtitle:   "Python - collection [set, dictionary] "
categories: python
tags: python collection set dictionary
comments: true
---

Collection 타입을 알아보자

List는 Sequence 타입이다.

## Set

---

셋은 반복 가능하고, 가변적이며, 중복 요소가 없고, 정렬되지 않은 컬렉션 데이터 타입이다.

- 삽입 시간복잡도 O(1)
- 합집합 O(m+n)
- 교집합 O(n) (더 작은 집합)

> frozenset() : 불변 객체

> Set, List, Dict 는 서로 사용 가능하다.
> ex) list(set(data)) , dict(list)

## Dict

---

딕셔너리는 해시 테이블로 구현되어 있다. 컬렉션 매핑 타입인 딕셔너리는 반복 가능하다. 

### setdefault()

---

키의 존재 여부를 모른 채 접근 할 때 사용된다. 

```python
def usual_dict(dict_data):
	newdata = {}
	for k, v in dict_data:
		if k in newdata:
			newdata[k].append(v)
		else:
			newdata[k] = v
	return newdata

# setdefault 사용
def setdefault_dict(dict_data):
	newdata = {}
	for k, v in dict_data:
		newdata.setdefault(k,[]).append(v) #key값이 없으면 자동으로 default값으로 생성
	return newdata
```

### 딕셔너리 분기

---

두 함수를 조건에 따라 분기할때 보통 if문을 사용한다 

다음은 dict를 활용한 예제이다.

```python
#기존 방식
action = "h"

if action == "h":
	hello()
else action =="w":
	world()

#dict 사용
functions = dict(h=hello, w=world)
functions[action]()
```

## Dict 종류 ( collections 모듈)

---

### 1. defaultdict()

key가 없어도 입력 가능

기존 dict는 key가 없으면 Error

### 2. OrderedDict()

정렬된 딕셔너리 타입이다.

### 3. Counter()

해시 가능한 객체를 카운팅하는 특화된 서브클래스다.

단어 횟수 세기

```python
from collections import Counter

def find_top_N_recurring_words(seq, N):
	dcounter = Counter()
	for word in seq.split():
		dcounter[word] += 1
	return dcounter.most_common(N)
```

