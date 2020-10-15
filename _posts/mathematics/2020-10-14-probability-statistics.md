---
layout: post
title: "[확률과통계] - 조건부 확률과 Bayes 정리 "
subtitle: "probability and statistics - conditional probability and Bayesian Theroem "
categories: mathematics
tags: mathematics
comments: true
---

## 1) Sample Space ⇒ S (set) : 전체 집합

---

아웃풋이 랜덤 할 때, 결과의 모든 집합

## 2) Event(A) ⇒ $A ⊂ S$ : 부분 집합

---

$$P(A) = prob( outcome  ∈ A )$$

- A가 발생할 확률

## 3) Conditional Probability : 조건부 확률

---

$$P(B|A) = \frac{P(B∩A)}{P(A)} : A라는 조건에서 B가 발생 할 확률$$

> A라는 전제 조건, A가 발생한 경우에서 B가 발생 할 확률

$P(B|A) = \frac{P(B∩A | S)}{P(A|S)}$ 와 같다.

전체집합 안에서 일어난 사건의 경우

⇒ $P(B|A)$는 A라는 새로운 Sample Space에서 일어난 사건

## 4) Total Probability

---

$$
$$

- A1, A2 ... An ⇒ Partition of S
- exclusive한 배반 사건 조각들의 합

$$P(A1) = P(A_1∩A)=P(A|A_1)P(A_1)[*Conditional Prob]$$

$$\therefore P(A) = \Sigma^{n}_{i=1}P(A|A_i)P(A_i)$$

## 5) Bayesian Theorem

---

$$P(B|A) = \frac{P(B∩A)}{P(A)} = \frac{P(A|B)P(B)}{P(A)}$$

$$\therefore P(A_i|A)=\frac{P(A|A_i)P(A_i)}{P(A)}$$

- $A_i$는 Original, Input, 아직 정확히 모르는 것. 알아내야 할 것
- $A$는 Observation, 이미 관측한 Data

> 즉, 수식을 뒤집어서 이미 관측한 Data로 수식 계산

### Ex. Binray Symmetric Channel

---

- input symbols : {$x_1, x_2$} ⇒ Transmitter (0,1)
- output symbols : {$y_1,y_2$} ⇒ receiver (0,1)

$$x_1\quad\rightarrow\quad y_1 = P_{11} \\x_2\quad\rightarrow\quad y_2 = P_{22}\\x_1\quad error\quad\rightarrow\quad y_2 = P_{12}\\x_2\quad error\quad\rightarrow\quad y_1 = P_{21}$$

> $P_{11}=P_{22}, P_{12}=P_{21}$ ⇒ Symmetric, 대칭적이다.

$P_{11}=P(y_1|x_1)$ : $x_1$을 보냈을 때(전제 조건), $y_1$을 받을 확률

$$P_{11}=P(y_1|x_1)\\P_{12}=P(y_1|x_2)\\P_{11}+P_{12}=1$$

$$P_{22}=P(y_2|x_2)\\P_{21}=P(y_2|x_1)\\P_{22}+P_{21}=1$$

> Priori : 사전에 실험등을 통해서 알 수 있다.

### 1) $P_{error}$

---

$P_{error}= Prob(x_1\quad trans,\quad y_2\quad receive) + Prob(x_2\quad trans, \quad y_2\quad recive)\\\quad\quad\quad=P(y_2|x_1)P(x_1) +P(y_1|x_2)P(x_2)$

> $P(y_2|x_1)P(x_1)$ : x1을 전송하고 그다음 y2가 오는 확률

### 2) when $y_2$ received, What prob of $x_1$ transmission?

---

$$P(x_1|y_2) = (Baysion) \rightarrow \frac{P(y_2|x_1)P(x_1)}{P(y_2)}$$

$$(total\quad probability)\rightarrow\frac{P(y_2|x_1)P(x_1)}{P(y_2|x_1)P(x_1)+P(y_2|x_2)P(x_2)}$$

### 3) $P(x_1|error)$

---

$$\rightarrow \frac{P(y_2|x_1)P(x_1)}{P(y_2|x_1)P(x_1)+P(y_1|x_2)p(x_2)}$$

## 1.8 Independent Events

---

## If A and B are (mutually) independent.

$$P(B|A)=P(B), \rightarrow P(A|B)=P(A)\\\rightarrow \frac{P(A∩B)}{P(A)}=P(B)$$

$$\therefore P(A∩B)=P(A)P(B)$$

이렇게 나오면 Independent 증명

> Independent ≠ exclusive
> 서로 독립적 ≠ 공통된 요소 없음

> A랑 B가 서로 Independent하면 A와 B의 여사건들도 Independent하다

## 1.9 Combined Experiments

---

- for tow experiments with $S1 \quad S2$

$$S = S_1 *S_2 =\{(x_i,y_j) | x_i ∈S_1,y_j ∈S_2\}\\* : Cartesion \quad Product$$

ex) 3 coins tossing

⇒ 3 experiments of 1 coin tossing

$$S_1=\{H,T\}\quad\quad\quad\quad\quad\quad\quad\quad\\S_2=\{H,T\}  \rightarrow S=S_1*S_2*S_3 \\S_3=\{H,T\}\quad\quad\quad\quad\quad\quad\quad\quad \\ =\{HHH,HHT,....,TTT\}$$
