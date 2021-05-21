---
layout: post
title: '[Programming] - FireStore Index '
subtitle: 'firebase firestore index'
categories: programming
tags: firebase firestore index
comments: true
---

## Firestore의 Index

---

Firebase의 Firestore는 기본적으로 모든 document의 field들을 자동적으로 인덱싱한다.

이렇게 인덱싱 된 덕분에 엄청나게 빠른 검색이 가능하다. (바이너리 서치, 인접 노드 다 가져오기)

> 참고로 map field안에 있는 내용물도 인덱싱을 한다고 한다.
> 20레벨까지 중첩된 map은 인덱싱을 지원한다.

### 인덱스 테이블

---

![이미지](https://Funncy.github.io/assets/img/firestore-index/_2021-05-06__4.24.39.png)

위와 같은 이미지처럼 인덱스 테이블을 통해서 데이터를 조회한다.

> 조회된 데이터가 인접해야만 조회가 가능하다.

### 쿼리 조합

---

![이미지](https://Funncy.github.io/assets/img/firestore-index/_2021-05-06__4.26.00.png)

쿼리가 조합된 경우 위와 같은 이미지 처럼 인덱스 테이블들을 순회하며 해당 데이터를 찾아간다.

![](https://Funncy.github.io/assets/img/firestore-index/_2021-05-06__4.33.25.png)

하지만 위와 같이 avg_rating등과 같은 비교 연산이 들어간 index 테이블은 정렬이 되어있지 않는 문제로 순회 조회가 불가능하다.

이런 경우에는 Composite indexes(복합 색인)로 처리가 가능하다.

![이미지](https://Funncy.github.io/assets/img/firestore-index/_2021-05-06__4.34.46.png)

![이미지](https://Funncy.github.io/assets/img/firestore-index/_2021-05-06__4.36.20.png)

위와 같이 해결된다.

## 불가능한 쿼리

---

이런 인덱싱 처리로 인해 지원하지 않는 쿼리도 있다.

하나씩 알아보자.

### 1. Like문

String 연산을 통해 query를 날리는 것은 firestore에서 지원하지 않는다.

> Algolia or ElasticSearch를 통해 구현이 가능하다.

### 2. OR 연산

index가 되있는 곳에서 인접하지 않은 떨어진 내용들 접근이 안된다.

2번의 쿼리를 통해 조합해야 한다.

혹은 OR 처리를 위한 field를 추가해서 처리가 가능하다.

### 3. not equal , ≠

이또한 index 테이블에서 인접하지 않은 데이터들을 조회 하기 때문에 불가능하다.

### 4. null

null값은 index 테이블에 들어가지 않기 때문에 조회가 불가능하다.
