---
layout: post
title: '[Javascript] -  실행컨텍스트 '
subtitle: 'javascript execution context'
categories: web
tags: javascript
comments: true
---

여러 영상들을  통해 자바스크립트의 실행컨텍스트를 정리해보고자 한다.

### 실행 컨텍스트의 구성

---

![스크린샷 2022-05-05 오후 11.12.44.png](https://classic-sandal-da4.notion.site/image/https%3A%2F%2Fs3-us-west-2.amazonaws.com%2Fsecure.notion-static.com%2F9e02eb5d-9394-486d-bae5-7ec68acd1bde%2F%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA_2022-05-05_%E1%84%8B%E1%85%A9%E1%84%92%E1%85%AE_11.12.44.png?table=block&id=74e06713-4ee2-4fd0-ba77-f5d515e666f7&spaceId=56f598aa-e89d-4146-acaa-3fed6afbd13b&width=2000&userId=&cache=v2)

여기서 `EnvironmentRecord`와 `Outer Enviroment Reference`를 살펴보자.

![스크린샷 2022-05-05 오후 11.13.46.png](https://classic-sandal-da4.notion.site/image/https%3A%2F%2Fs3-us-west-2.amazonaws.com%2Fsecure.notion-static.com%2Fe70be90a-1fc1-4bee-bef9-6dffd781bb30%2F%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA_2022-05-05_%E1%84%8B%E1%85%A9%E1%84%92%E1%85%AE_11.13.46.png?table=block&id=1b6f1c42-0049-42a6-8e8a-c00ad6c80068&spaceId=56f598aa-e89d-4146-acaa-3fed6afbd13b&width=2000&userId=&cache=v2)

## 콜스택과 실행컨텍스트

---

자바 스크립트 엔진은 `콜스택`이라는 곳에서 함수의 `실행컨텍스를 담아서 관리`한다.

stack이라는 이름처럼 실제 stack자료 구조로 실행된다.

최초에는 Global, 전역 실행컨텍스트가 담기고 함수가 실행되면 함수의 실행컨텍스트가 담기게 된다.

stack 답게 가장 `마지막에 넣은 함수`의 `실행컨텍스트가 활성화` 된다고 보면 된다.

![스크린샷 2022-05-05 오후 11.15.18.png](https://classic-sandal-da4.notion.site/image/https%3A%2F%2Fs3-us-west-2.amazonaws.com%2Fsecure.notion-static.com%2Fee08b5a4-97cc-47b3-8bf2-96f16c587603%2F%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA_2022-05-05_%E1%84%8B%E1%85%A9%E1%84%92%E1%85%AE_11.15.18.png?table=block&id=a983030d-fffc-48c5-8b63-bbed6868578b&spaceId=56f598aa-e89d-4146-acaa-3fed6afbd13b&width=2000&userId=&cache=v2)

함수가 종료되면 실행콘텍스트는 pop되고 한칸씩 줄어들어간다.

모든 내용이 실행되고 다면 전역 실행컨텍스트도 콜스택에서 사라지게된다.

## 호이스팅

---

![스크린샷 2022-05-05 오후 11.16.54.png](https://classic-sandal-da4.notion.site/image/https%3A%2F%2Fs3-us-west-2.amazonaws.com%2Fsecure.notion-static.com%2F49003f8c-3cbe-4096-9b76-6a8e3a8e6c86%2F%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA_2022-05-05_%E1%84%8B%E1%85%A9%E1%84%92%E1%85%AE_11.16.54.png?table=block&id=69dd8404-2013-4b23-874a-0e70d4821633&spaceId=56f598aa-e89d-4146-acaa-3fed6afbd13b&width=2000&userId=&cache=v2)

위 처럼 선언문 전에 접근할때 undefined로 출력되는 현상을 호이스팅이라고 한다.

자바스크립트 엔진이 먼저 전체 코드를 스캔하면서 변수를 미리 선언해놓는다.

![스크린샷 2022-05-05 오후 11.23.16.png](https://classic-sandal-da4.notion.site/image/https%3A%2F%2Fs3-us-west-2.amazonaws.com%2Fsecure.notion-static.com%2F2715f95f-f87d-476c-b170-35c3ea1fc4f3%2F%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA_2022-05-05_%E1%84%8B%E1%85%A9%E1%84%92%E1%85%AE_11.23.16.png?table=block&id=08af32bb-2146-4c5f-9dec-26a13fda4da6&spaceId=56f598aa-e89d-4146-acaa-3fed6afbd13b&width=2000&userId=&cache=v2)

그때 기록되는 공간이 환경 레코드이다.

정식명칭은 `Environment Record`로 식별자와 식별자에 바인딩 된 값을 기록한다.

### var 변수 호이스팅

---

![스크린샷 2022-05-05 오후 11.25.11.png](https://classic-sandal-da4.notion.site/image/https%3A%2F%2Fs3-us-west-2.amazonaws.com%2Fsecure.notion-static.com%2Fdefba497-a9a4-4c43-ae24-18bd7433173d%2F%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA_2022-05-05_%E1%84%8B%E1%85%A9%E1%84%92%E1%85%AE_11.25.11.png?table=block&id=b701d6bb-d384-4a9f-a036-c116bb5ab719&spaceId=56f598aa-e89d-4146-acaa-3fed6afbd13b&width=2000&userId=&cache=v2)

자바스크립트 엔진은 생성 단계에서 선언문만 먼저 실행해서

TVChannel의 변수를 undefined로 생성해놓는다.

그리고 실행단계를 통해 변수의 값을 변화시키고 함수를 실행하는 등의 동작이 실행된다.

> 그래서 TVChannel은 생성단계 이후 undefined로 이미 생성되있기 때문에
console.log(TVChannel);이 에러가 아니라 undefined가 뜨게 된다.
> 

### const, let 변수 호이스팅

---

![스크린샷 2022-05-05 오후 11.26.17.png](https://classic-sandal-da4.notion.site/image/https%3A%2F%2Fs3-us-west-2.amazonaws.com%2Fsecure.notion-static.com%2F905fb4c8-687f-43dc-b510-699f8185881c%2F%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA_2022-05-05_%E1%84%8B%E1%85%A9%E1%84%92%E1%85%AE_11.26.17.png?table=block&id=c933a8fb-c090-4e35-a9f7-623f40ba8377&spaceId=56f598aa-e89d-4146-acaa-3fed6afbd13b&width=2000&userId=&cache=v2)

const와 let은 똑같이 생성단계에 환경레코드에 식별자를 등록하는 과정을 거친다.

하지만 var와 다른점은 var는 undfined로값을 초기값을 넣어준것과 다르게 초기값을 넣지 않는다.

> var는 선언과 초기화가 동시에 이뤄진다.
const, let은 선언만 실행한다. 초기화가 되어있지 않아 접근이 되지않는다.
> 

그렇기에 접근하면 Reference Error가 발생한다.

이미 생성되었지만 접근하지 못하는 이 구간을 `일시적 사각지대`라고 부른다.

### 함수 호이스팅

---

![스크린샷 2022-05-05 오후 11.36.19.png](https://classic-sandal-da4.notion.site/image/https%3A%2F%2Fs3-us-west-2.amazonaws.com%2Fsecure.notion-static.com%2F98850879-4c5e-464f-b62e-653390acfec5%2F%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA_2022-05-05_%E1%84%8B%E1%85%A9%E1%84%92%E1%85%AE_11.36.19.png?table=block&id=b5a04fde-38df-4fd0-b6b1-4b359a9c1e3c&spaceId=56f598aa-e89d-4146-acaa-3fed6afbd13b&width=2000&userId=&cache=v2)

변수에 함수를 담아서 사용하는 것을 `함수 표현식`이라고 한다.

함수 표현식은 결국 변수이기 때문에 상단에서 접근할때 var로 선언하면 undefined라서  Type Error가 발생하고,  const,let은 접근이 안돼서 Reference Error가 발생한다.

> var ⇒ undfined ⇒ Type Error
const, let ⇒ Reference Error
> 

![스크린샷 2022-05-05 오후 11.37.46.png](https://classic-sandal-da4.notion.site/image/https%3A%2F%2Fs3-us-west-2.amazonaws.com%2Fsecure.notion-static.com%2F0d37b0c2-875c-4b00-a013-1bc916dd0c3d%2F%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA_2022-05-05_%E1%84%8B%E1%85%A9%E1%84%92%E1%85%AE_11.37.46.png?table=block&id=a3e14c19-e225-4fb4-9872-bf492497fac2&spaceId=56f598aa-e89d-4146-acaa-3fed6afbd13b&width=2000&userId=&cache=v2)

function 키워드로 생성하는 것은 `함수 선언문`이라고 한다.

함수 선언문은 선언과 동시에 함수 생성을 하게 된다.

## 외부 환경 참조

---

![스크린샷 2022-05-05 오후 11.39.50.png](https://classic-sandal-da4.notion.site/image/https%3A%2F%2Fs3-us-west-2.amazonaws.com%2Fsecure.notion-static.com%2F9686d2fd-2a2d-46ac-9b43-9d3a05c3f51a%2F%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA_2022-05-05_%E1%84%8B%E1%85%A9%E1%84%92%E1%85%AE_11.39.50.png?table=block&id=c46a489c-8d0e-4de0-bb12-ca02fc887e89&spaceId=56f598aa-e89d-4146-acaa-3fed6afbd13b&width=2000&userId=&cache=v2)

실행 컨텍스트에 있는 내용으로 바깥 Lexical Environment를 가르킨다.

즉 나의 상위 실행 컨텍스트로 이동할 수 있다.

### 식별자 결정

---

![스크린샷 2022-05-05 오후 11.40.45.png](https://classic-sandal-da4.notion.site/image/https%3A%2F%2Fs3-us-west-2.amazonaws.com%2Fsecure.notion-static.com%2Fc79dd468-78f0-468c-84fa-20acc5ae691d%2F%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA_2022-05-05_%E1%84%8B%E1%85%A9%E1%84%92%E1%85%AE_11.40.45.png?table=block&id=0e5fb245-587d-4ac4-8cab-c14e502c37fb&spaceId=56f598aa-e89d-4146-acaa-3fed6afbd13b&width=2000&userId=&cache=v2)

위와 같이 lamp라는 변수가 함수 바깥에도 내부에도 선언된 경우 어떤 변수를 선택할 것인가의 문제를

식별자 결정이라고 한다. 이 과정에서 outer 가 사용된다.

![스크린샷 2022-05-05 오후 11.41.43.png](https://classic-sandal-da4.notion.site/image/https%3A%2F%2Fs3-us-west-2.amazonaws.com%2Fsecure.notion-static.com%2F98cc3674-171a-497d-ba6d-c948376588e2%2F%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA_2022-05-05_%E1%84%8B%E1%85%A9%E1%84%92%E1%85%AE_11.41.43.png?table=block&id=a0dadf8c-0cae-48a9-bd44-1bd204364478&spaceId=56f598aa-e89d-4146-acaa-3fed6afbd13b&width=2000&userId=&cache=v2)

기본적으로 자바스크립트 엔진은 현재 활성화된 실행 컨텍스트를 기준으로 식별자를 확인하며

만약 식별자가 현재 실행 컨텍스트에 없다면 outer를 통해 이전 실행컨텍스트를 찾아가서 식별자를 결정한다.

![스크린샷 2022-05-05 오후 11.43.31.png](https://classic-sandal-da4.notion.site/image/https%3A%2F%2Fs3-us-west-2.amazonaws.com%2Fsecure.notion-static.com%2F128d6d1b-7c09-455f-96b0-56f55a18694d%2F%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA_2022-05-05_%E1%84%8B%E1%85%A9%E1%84%92%E1%85%AE_11.43.31.png?table=block&id=afa73a14-a632-4194-983c-f0ba8407c00e&spaceId=56f598aa-e89d-4146-acaa-3fed6afbd13b&width=2000&userId=&cache=v2)

그러다보니 동일한 식별자가 선언될 경우 상위 식별자가 가려지게 되는데 이것을 `변수 섀도잉`이라고 한다.

![스크린샷 2022-05-05 오후 11.42.26.png](https://classic-sandal-da4.notion.site/image/https%3A%2F%2Fs3-us-west-2.amazonaws.com%2Fsecure.notion-static.com%2F5f47c0e6-7c85-4833-823f-35128d9415b0%2F%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA_2022-05-05_%E1%84%8B%E1%85%A9%E1%84%92%E1%85%AE_11.42.26.png?table=block&id=38a867a8-8b41-4f9d-bd9c-0245172fc407&spaceId=56f598aa-e89d-4146-acaa-3fed6afbd13b&width=2000&userId=&cache=v2)

이렇게 식별자를 찾을때 사용하는 스코프들의 연결을 `스코프 체인`이라고 하고 

찾는 과정을 `스코프 체이닝`이라고 한다.

## 결론

---

![스크린샷 2022-05-05 오후 11.44.23.png](https://classic-sandal-da4.notion.site/image/https%3A%2F%2Fs3-us-west-2.amazonaws.com%2Fsecure.notion-static.com%2F90d98c6a-62d3-4f10-9576-6e5f8a80f504%2F%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA_2022-05-05_%E1%84%8B%E1%85%A9%E1%84%92%E1%85%AE_11.44.23.png?table=block&id=0294f98e-3385-40f9-851c-5e216474b40b&spaceId=56f598aa-e89d-4146-acaa-3fed6afbd13b&width=2000&userId=&cache=v2)

쉽게 말해 코드 실행을 위한 환경을 관리해주는게 실행 컨텍스트이다.

식별자 결정을 더욱 효율적으로 하기위한 수단이다.

실행 컨텍스트도 처음부터 완성된것이 아니라 점차적으로 발전해 왔었다.

![스크린샷 2022-05-05 오후 11.45.12.png](https://classic-sandal-da4.notion.site/image/https%3A%2F%2Fs3-us-west-2.amazonaws.com%2Fsecure.notion-static.com%2F3d332480-e6c5-4a3b-a0c4-7c5d40b01a2a%2F%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA_2022-05-05_%E1%84%8B%E1%85%A9%E1%84%92%E1%85%AE_11.45.12.png?table=block&id=2bbbf8ed-e0fc-46b9-bff4-6ce339e8dd48&spaceId=56f598aa-e89d-4146-acaa-3fed6afbd13b&width=2000&userId=&cache=v2)

ES3에는 어디서 실행되느냐에 따라서 동적으로 스코프를 관리했지만

ES5이후부터는 선언된 위치를 기준으로 정적으로 스코프를 관리한다.

### 계속 추가 예정 중...

### 참고 자료

---

[[10분 테코톡] 💙 하루의 실행 컨텍스트](https://www.youtube.com/watch?v=EWfujNzSUmw)