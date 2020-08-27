---
layout: post
title: "[Algorithm] - 알고리즘 : 애너그램, 주사위 합계 경로 "
subtitle:   "algorithm - anagram, find_dice "
categories: algorithm
tags: algorithm python anagram dice
comments: true
---

# 1. 애너그램

---

애너그램은 문장 또 단어의 철자 순서를 바꾸는 놀이를 말한다. 

리스트의 창목을 정렬하는 시간복잡도는 최소 O(n log n)이다. 따라서 딕셔너리를 활용하는 것이 가장 좋은 해결책이 될 수 있다.

### 1) Counter 이용

```python
from collections import Counter

def is_anagram(s1, s2):
	counter = Counter()
	for c in s1:
		counter[c] += 1
	for c in s2:
		counter[c] -= 1

for i in counter.values():
	if i:
		return False
return True
```

### 2) 문자열 합 이용

ord() 함수는 인수가 유니코드 객체일 때, 문자의 유니코드를 나타내는 정수를 반환한다.

```python
import string 

def hash_func(astring):
	s = 0
	for one in astring:
		if one in string.whitespace:
			continue
		s = s + ord(one)
	return s

def find_anagram_hash_function(word1, word2):
	return hash_func(word1) == hash_func(word2)
```

# 2. 주사위 합계 경로

---

주사위를 두 번 던져서 합계가 특정 수가 나오는 경우의 수와 경로를 구해보자.

나는 n^2번 순회하면서 내용물의 합을 비교하여 처리하려고 하였다.

책의 내용은 다음과 같다.

⇒ Dict를 2개 만들어 하나는 주사위 2개의 2중 반복문의 합을 Key값으로 사용하여 중복 카운트를 넣는다

⇒ 하나는 그 조합을 넣어서 

⇒ 최종적으로는 해당 Key값만 넣으면 해당 Key에 따른 조합들을 가져온다.

```python
from collections import Counter, defaultdict

def find_dice_probabilities(S, n_faces=6):
    if S > 2 * n_faces or S < 2:
        return None

    cdict = Counter()
    ddict = defaultdict(list)

    for dice1 in range(1, n_faces + 1):
        for dice2 in range(1, n_faces + 1):
            t = [dice1, dice2]
            cdict[dice1 + dice2] += 1
            ddict[dice1 + dice2].append(t)
    return [cdict[S], ddict[S]]
```