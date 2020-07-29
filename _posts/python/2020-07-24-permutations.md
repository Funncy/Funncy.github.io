---
layout: post
title: "[Algorithm] - 알고리즘 : 문자열 순열 알고리즘 "
subtitle:   "Algorithm - algorithm : permutations "
categories: algorithm
tags: algorithm
comments: true
---

문자열 입력에 대한 모든 경우의 수를 출력한다.

```python
import itertools

def perm(s):
	if len(s) < 2:
		return s
	res = []
	
	for i, c in enumerate(s):
		for cc in perm(s[:i] + s[i+1:]):
			res.append(c + cc)
	return res

def perm2(s):
	res = itertools.permutations(s)
	return ["".join(i) for i in res]

if __name__ == "__main__":
	val = "ABC"
	print(perm(val))

# ['ABC', 'ACB', 'BAC', 'BCA', 'CAB', 'CBA']
```

pers2는 라이브러리를 통한 해결법이다.

perm 을 살펴보면 문자열 입력이 들어오면 이제 재귀 형태로 돌아간다.

1. 먼저 재귀의 탈출 조건은 문자열 갯수가 2개 미만 즉 문자 한개만 들어오면 그걸 반환해준다.
2. enumerate로 문자의 문자와 인덱스를 뽑아서 반복한다.
3. 그리고 2번째 반복문에서는 뽑아낸 문자를 제외한 (s[:i] + s[i+1:]) 문자열을 다시 재귀로 돌린다.
4. 그렇게 반복하다보면 2개의 문자로 구성된 문자열이 남게된다.
5. 2개로 구성된 문자열 ex. AB는 또다시 A와 인덱스 0번으로 뽑히게 된다.
6. A를 제외한 문자열을 재귀를 하면 B혼자 남았기에 return이 되고 
7. 그렇게 처음으로 res.append에 문자열로 합쳐진다. c = A , cc = B  (AB)가 들어간다.
8. 그리고 이번에는 AB에서 A가 아니라 B와 인덱스 1번을 뽑아서 진행한다.
9. B를 제외한 문자열은 다시 문자를 반환하며 A가 나오고 다시 합쳐진다.
10. c = B , cc = A (BA)가 들어간다.
11. 이렇게 모든 순환을 마치면 모든 경우의 수가 나타난다.

위의 코드를 응용하면 쉽게 조합(Combination)도 작성 할 수 있다.

```python
def perm(s):
	if len(s) < 2:
		return s
	res = []
	
	for i, c in enumerate(s):
		res.append(c) # 새로 추가 된 부분
		for cc in perm(s[:i] + s[i+1:]):
			res.append(c + cc)
	return res

if __name__ == "__main__":
	val = "ABC"
	print(perm(val))

# ['A', 'AB', 'ABC', 'AC', 'ACB', 'B', 'BA', 'BAC', 
#  'BC', 'BCA', 'C', 'CA', 'CAB', 'CB', 'CBA']
```

다음은 `Python itertools`에 있는 라이브러리 `permutations`를 분석해보았다.

```python
def permutations(iterable, r=None):
    # permutations('ABCD', 2) --> AB AC AD BA BC BD CA CB CD DA DB DC
    # permutations(range(3)) --> 012 021 102 120 201 210
    pool = tuple(iterable)
    n = len(pool)
    r = n if r is None else r
    if r > n:
        return
    indices = list(range(n))
    cycles = list(range(n, n - r, -1))
    yield tuple(pool[i] for i in indices[:r])
    while n:
        for i in reversed(range(r)):
            cycles[i] -= 1
            if cycles[i] == 0:
                indices[i:] = indices[i + 1 :] + indices[i : i + 1]
                cycles[i] = n - i
            else:
                j = cycles[i]
                indices[i], indices[-j] = indices[-j], indices[i]
                yield tuple(pool[i] for i in indices[:r])
                break
        else:
            return
```

`itertools`에 있는 라이브러리는 `indices`와 `cycles`를 사용한다.

1. 전체적인 내용은 pool에 전체 배열 내용을 넣는다.
2. n은 전체 문자열의 길이 r은 거기서 한 덩어리의 갯수(기본값 n)를 넣는다.
3. 그리고 `indices`에 전체 `문자열의 인덱스` 값을 넣고 `cycles`에는 `반복횟수`를 넣는다.
4. `cycles`의 마지막 인덱스부터 접근하기 위해  내부 반복문에서는 reversed로 거꾸로 접근한다. range(r)은 범위 지정.
5. 그리고 한칸씩 이동하면서 버블정렬처럼 현재위치와 하나씩 자리를 바꾸며 출력한다. 

이렇게 설명하면 어려우니 예시를 들어보겠다.

n = 5, r = 3

indices = [0, 1, 2, 3, 4]

cycles = [5, 4, 3]

1. 먼저 for i in reversed(range(r)): 에서 2 1 0 순으로 돌아간다. 
그러면 cycles의 가장 마지막 값 3에서 1을 뺀다. (반복 횟수 의미)

2. 그리고 0이 아니면 (아직 반복 해야 한다는 뜻) j에 cycles[i] 를 넣는다. (3에서 1을 뺀 2)

3. 그리고 indices에서 i와 -j를 서로 위치를 바꾼다. (i는 왼쪽에서부터 기준 j는 오른쪽에서부터 기준)
indices = [0, 1, `(2, i)`, `(3, -j)`, 4]

4. 이렇게 서로 교환하면 indices = [0, 1, 3, 2, 4]가 된다. 그리고 break를 만났으니 for를 끝내고 while을 만나 다시 들어온다.

5. for에서는 다시 들어왔으니 2부터 시작하고 cycles = [5, 4, 2]에서 마지막 자리를 1을 뺀다 [5, 4, 1]

6. 아직 0이 아니니 다시 j=cycles[i] = 1 을 넣고

7. i와 j 자리를 교체한다.
indicies = [0, 1, `(3, i)`, 2, `(4, -j)`]

8. 그러면 indicies = [0, 1, 4, 2, 3]이 된다. 1차 순회가 다 끝났다.

9. 이제 다시 break를 만나 다시 for를 들어와 i는 2부터 시작한다.
하지만 이번에는 cycles[i]에서 1을 빼면 0이 된다. [5, 4, 0]

10. 그러면 indices[i:] = indices[i + 1: ]+ indices[i : i +1] 을 진행하는데 이건 다시 재정렬 시켜주는 내용이다.

11. 인덱스가 i이후인 내용을 다시 전부 정렬시켜준다. indices⇒ [0, 1, 2, 3, 4]
그리고 cycles[i]에 다시 원래 값을 넣어준다. 즉 초기화를 시켜준다.

12. 하지만 여기서는 break가 없기 때문에 for문을 다시 들어와 i가 1으로 변한다.

13. cycles[i] = [5, 4, 3]에서 1번째 인덱스 4를 줄여준다. [5, 3, 3] (반복횟수)

14. 0이 아니기 때문에 이전 처럼 j= cylces[1] = 3이 되고
i=1 ,j =3 서로 자리를 바꿔준다 

15. indicies = [0, `(1, i)`, `(2, -j)`, 3, 4]

16. 그러면 indicies는 [0, 2, 1, 3, 4]가 된다.

17. 그리고 다시 break를 만나 뒷자리들을 위에와 같이 반복 순회해준다.

이 내용이 itertools에 있는 permutations의 내용이다.