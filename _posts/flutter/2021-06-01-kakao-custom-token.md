---
layout: post
title: '[Flutter] - Kakao Custom Token by Firebase Cloud Function '
subtitle: 'Flutter kakao Custom Token Firebase CloudFunction'
categories: flutter
tags: flutter firebase cloudfunction kakako jwt
comments: true
---

이번에 앱을 개발하다가 kakao login과 firebase auth를 연동해야했다.

그러다보니 access_token을 jwt 토큰으로 변환시키는 서버 로직이 필요했지만

이것만을 위해서 서버를 구성하기에는 배보다 배꼽이 크다고 판단하여 serverless인 Cloud Function을 이참에 공부해서 사용해보기로 하였다.

## Cloud Function

---

firebase init을 통해서 프로젝트를 생성하는 부분은 넘어가도록 하겠다.

### 환경 변수 설정

---

추가적으로 firebase admin SDK를 위한 json 파일을 git에 올릴 수 없어 환경변수로 설정하였다.

해당 파일을 프로젝트에 넣은 뒤 아래 명령어로 환경 변수를 설정해주자.

```jsx
firebase functions:config:set service_account=$(cat service_account.json)
```

### 전체 로직

---

이미 accessToken이 발급이 된 상태에서 진행됨을 유의하자.

1. requestMe 함수에서는 accessToken을 사용하여 유저 데이터를 가져온다.
2. 전체 유저 profile 데이터를 설정한다. (updateParams)
3. emil을 기준으로 Firebase Auth에 해당 유저가 있는지 확인한다.
   1. 없다면 유저를 생성해준다.
   2. 있다면 유저 정보를 넘겨준다.
4. 유저 아이디와 Provider(Kakao)를 넘겨서 커스텀토큰을 발급한다.

### 전체 코드

---

```jsx
const functions = require('firebase-functions');
const axios = require('axios').default;
const async = require('async');
var admin = require('firebase-admin');

const serviceAccount = functions.config().service_account;

admin.initializeApp({ credential: admin.credential.cert(serviceAccount) });

// // Create and Deploy Your First Cloud Functions
// // https://firebase.google.com/docs/functions/write-firebase-functions
//
const kakaoRequestMeUrl = 'https://kapi.kakao.com/v2/user/me';

exports.kakaoToken = functions.region('asia-northeast3').https.onCall((data, context) => {
	var access_token = data['access_token'];
	var token = createFirebaseToken(access_token);
	return token;
});

/**
 * requestMe - Returns user profile from Kakao API
 *
 * @param  {String} kakaoAccessToken Access token retrieved by Kakao Login API
 * @return {Promiise<Response>}      User profile response in a promise
 */
async function requestMe(kakaoAccessToken) {
	console.log('Requesting user profile from Kakao API server. ' + kakaoAccessToken);
	var result = await axios.get(kakaoRequestMeUrl, {
		method: 'GET',
		headers: { Authorization: 'Bearer ' + kakaoAccessToken },
	});
	return result;
}

async function updateOrCreateUser(updateParams) {
	console.log('updating or creating a firebase user');
	console.log(updateParams);
	try {
		var userRecord = await admin.auth().getUserByEmail(updateParams['email']);
	} catch (error) {
		if (error.code === 'auth/user-not-found') {
			return admin.auth().createUser(updateParams);
		}
		throw error;
	}
	return userRecord;
}

async function createFirebaseToken(kakaoAccessToken) {
	var requestMeResult = await requestMe(kakaoAccessToken);
	const userData = requestMeResult.data; // JSON.parse(response)
	console.log(userData);

	const userId = `kakao:${userData.id}`;
	if (!userId) {
		return response.status(404).send({ message: 'There was no user with the given access token.' });
	}

	let nickname = null;
	let profileImage = null;
	if (userData.properties) {
		nickname = userData.properties.nickname;
		profileImage = userData.properties.profile_image;
	}
	//! Firebase 특성상 email 필드는 필수이다.
	//! 사업자등록 이후 email을 필수옵션으로 설정할 수 있으니 (카카오 개발자 사이트) 꼭 설정하자.
	//! 테스트 단계에서는 email을 동의하지 않고 로그인 할 경우 에러가 발생한다.
	const updateParams = {
		uid: userId,
		provider: 'KAKAO',
		displayName: nickname,
		email: userData.kakao_account.email,
	};

	if (nickname) {
		updateParams['displayName'] = nickname;
	} else {
		updateParams['displayName'] = userData.kakao_account.email;
	}
	if (profileImage) {
		updateParams['photoURL'] = profileImage;
	}

	console.log(updateParams);

	var userRecord = await updateOrCreateUser(updateParams);

	return admin.auth().createCustomToken(userId, { provider: 'KAKAO' });
}
```

### 유의 사항

---

> region 설정을 까먹지 말자. 기본값은 us-central1이다.

> Firebase 특성상 Email 필드는 필수다.
> 개발자 계정에 사업자등록증을 추가하면 Email 필드를 필수로 바꿀 수 있다.
> 테스트 단계에서는 동의를 꼭 해주도록 하자.
