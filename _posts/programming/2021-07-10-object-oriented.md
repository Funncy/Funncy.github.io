---
layout: post
title: '[Programming] - 객체지향 '
subtitle: 'firebase object-oriented-programming'
categories: programming
tags: oop
comments: true
---

### 비용

프로그램이 출시 이후 시간이 지날 수록 비용이 증가

비용 : 코드 한 줄 작성하는데 들어가는 비용 (Cost / Line of Codes)

But 코드 줄 수는 크게 증가하지 않았음

### 원인

- 코드 분석 시간 증가
- 코드 변경 시간 증가

Keep being useful in a changing world!
SW의 가치는 변화

낮은 비용으로 변화 할 수 있어야 함.

- 패러다임 : 객체지향, 함수형, 리액티브
- 설계, 아키텍쳐 : TDD, SOLID, DDD, DRY, 클린아키텍쳐, MSA...
- 업무 프로세스 : 애자일, Devops

### 객체지향

- 캡슐화
- 다형성(추상화)

# 객체

---

### 절차지향 vs 객체지향

- 절차 지향 : 데이터를 여러 프로시저가 접근하여 처리 한다.

  개발은 쉬우나 점점 유지보수 어려워지는 코드가 발생한다.

  값을 직접 접근하다보면 그 값으로 처리하는 로직이 수정되면 해당 로직을 전부 찾아서 수정해줘야한다.

  ```tsx
  Account account = findOne(id);
  if(account.getState() == DELETED) {
   //로직
  }

  //...추후에 또다른 로직이 추가된다.

  if(account.getState() == DELETED || account.getBlockCount() > 0 ) {
  	//로직
  }

  //계속 추가되면서 이런 로직을 사용하는곳이 여러곳이라면?
  //나중에 로직이 수정되면 모든 곳을 찾아 다시 수정해줘야 한다. (놓치면 버그가 발생한다)
  ```

  절차지향은 처음에는 쉬운것 같으나 점점 문제가 발생한다.

- 객체 지향 : 데이터와 프로시저를 객체라는 단위로 묶는다.
  데이터는 객체의 프로시저(기능)만 접근 가능하다. (캡슐화)

      객체들간의 프로시저 통신으로 데이터를 처리한다. (기능으로 제공한다)

# 캡슐화

---

- 데이터 + 관련 기능 묶기
- 외부에 감춤 (정보 은닉)
- 외부에 영향 없이 객체 내부 구현 변경 가능 (유지 보수 향상)

```tsx
//캡슐화 하지 않은 코드
if (acc.getMembership() == REGULAR && acc.getExpDate().isAfter(now())) {
	//정회원 기능
}

//이런 코드가 추가적인 요구사항이 생겨서 다음과 같이 변경되었다.
if (
	acc.getMembership() ==
	REGULAR(
		(acc.getServiceDate().isAfter(fiveYearAgo) && acc.getExpDate().isAfter(now())) ||
			(acc.getServiceDate().isBefore(fiveYearAgo) && addMonth(acc.getExpDate()).isAfter(now()))
	)
) {
	//정회원 기능
}

//이런 요구사항들이 여러곳에서 발생한다면 유지보수 이슈는?...
```

- 추가적인 요구 사항 예시 : Date를 LocalDateTime으로 변경해라
  계정을 차단하면 모든 실행권한을 없애라

```tsx
//이렇게 캡슐화 되어있다면 사용하는 부분에서는 신경쓰지 않아도 된다.
if(acc.hasRegularPermission()) {
//정회원 기능
}

// class
public class Account {
	private Membership membership;
	private Date expDate;

	public boolean hasRegularPermission() {
		return membership == REGULAR &&
						expDate.isAfter(now())
	}
..
}
//추후에 권한 관련 요구사항 변경은 해당 클래스에 권한 함수에서 처리해주면 된다.
```

캡슐화를 사용하니 코드를 사용하는 곳에서는 변경하지 않았지만
캡슐화 된 곳에서 변경하니 모두 반영된다. 변경을 최소화 한다.

### 캡슐화 규칙

---

1. Tell, Don't Ask

   - 데이터를 달라고 하지 말고 해달라고 하기 (캡슐화된 로직 호출)

     ```tsx
     //데이터 달라는 경우
     acc.getMembership() == REGULAR;

     //해달라는 경우
     acc.hasRegularPermission();
     ```

2. Demeter's Law

- 메서드나 파라미터나 필드로 생성된 객체의 메서드만 호출해라 (2Step-Depth 호출 하지 마라)

  ```tsx
  acc.getExpDate().isAfter(now) //getExpDate()호출한 결과에서 바로 추가 메서드 호출!
  //변경한 결과
  acc.isExpired()

  Date date = acc.getExpDate();
  date.isAfter(now);
  //사실상 위와 같은 메서드 2중 호출
  //변경한 결과
  acc.isValid(now)
  ```

> 캡슐화는
> 기능의 구현을 외부에 감춤 (객체 내에서 관리)
> 캡슐화를 통해 해당 함수를 사용하는 곳이 아니라 객체에서 변경하여 일괄 변경 (코드 유연성)

# 다형성 및 추상화

---

### 다형성

- 한 객체가 상속을 통해 여러 타입을 가지는 것. (여러 모습을 갖는 것)

### 추상화

- 데이터나 프로세스등 특정한 성질 및 공통 성질끼리 정의하는 과정

### 타입 추상화

- 인터페이스를 통해 추상화

![이미지](https://Funncy.github.io/assets/img/oop/_2021-07-20__10.28.15.png '추상화')

- 추상타입을 통해 구현을 감춤.

```tsx
public void cancel(String ono) {
//주문 취소 처리

smsSender.sendSms(...);

}

//점점 새로운 기능이 추가

public void cancle(String ono) {
//주문 취소 처리

	if(pushEnabled) {
	kakaoPush.push(...);
	} else {
	smsSender.sendSms(...);
	}
	mailSvc.sendMail(...);
	//만약 더 다양해진다면?
}
```

### 추상화를 통해 해결

```tsx
public cancel(String ono) {
	//주문 취소 처리
	Notifier notifier = getNotifier(...);
	notifier.notify(...);
}

private Notifier getNotifier(...) {
	if(pushEnabled)
		return new KakaoNotifier();
	else
		return new SmsNotifier();
}
//구현을 감추고 새로운 기능 추가를 Notifier에 역할을 넘김
```

### 접근도 추상화

```tsx
public cancel(String ono) {
	//주문 취소 처리
	Notifier notifier = NotifierFactory.instance().getNotifier(...);
	notifier.notify(...);
}

public interface NotifierFactory {
	Notifier getNorifiter(...);

	static NotifierFactory instance() {
		return new DefaultNotifierFacotry();
	}
}

public class DefaultNotifierFactory implements NotifierFactory {
	public Notifer getNotifier(...) {
		if(pushEnabled)
			return new KakaoNotifier();
		else
			return new SmsNotifier();
	}
}
```

> 추상화를 통해 쉽게 변경이 가능해진다.

## 추상화 하는 시기는?

---

- 너무 빨리 추상화 하면 추상화를 잘못 할 수도 있다. (다시 돌아가야 할 수도)
- 실제 변경 및 확장이 발생할 때 추상화 시도를 해보자.
