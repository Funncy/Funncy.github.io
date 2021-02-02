---
layout: post
title: '[Django] -  Firebase Authentication 연동 '
subtitle: 'Django - firebaase authentication '
categories: django
tags: django android ios restframework jwt firebaase
comments: true
---

이번 프로젝트에서는

- Flutter : 화면
- Firebase : Auth + 실시간채팅
- Django : Rest API

이렇게 구성하게 되었다.

그러다보니 Firebase에서 User를 만들고

Django에서 Rest API 호출을 할 때 Authentication 관련 처리에 대해 스터디 하였다.

기존에 Django restframework와 JWT token처리를 정리해놔서 쉽게 처리 할 수 있었다.

## 1. Firebase설정

---

먼저 Firebase프로젝트로 가보자

> 나는 카카오톡 클론 Flutter 앱을 만들어놓았는데
> 여기에 test로 Django를 연결해보았다.

![_2021-02-02__3.36.47.png](/assets/img/notion/_2021-02-02__3.36.47.png)

이렇게 비공개 키를 생성해서 Django 프로젝트에 넣어주자.

전체 Flow는 다음과 같다.

![_2021-02-02__4.09.14.png](/assets/img/notion/_2021-02-02__4.09.14.png)

1. Flutter에서 FirebaseAuth로 유저를 생성한다.
2. 생성된 유저로부터 id Token 값을 받아온다.
3. 받아온 토큰으로 Django Rest api에 Header : Authorization에 넣어서 보낸다.
4. Django 쪽에서 FirebaseAuthentication이 Token을 확인 후 유저 생성 및 반환을 해준다.
5. 반환된 내용으로 Rest API 요청사항이 진행된다.

## Flutter

---

일단 만들어놓은 유저를 로그인했을때 토큰을 반환하게 해봤다.

![_2021-02-02__4.12.27.png](/assets/img/notion/_2021-02-02__4.12.27.png)

이제 이 토큰을 Rest api로 넣어보자.

> 당장은 귀찮아서 Postman으로 보내보았다.

## Django

---

Django와 Django restframework를 설치하고

추라고 firebase-admin을 설치하자.

그리고 authentication을 설정해주자.

### authentication.py

```python
import os
import firebase_admin
from firebase_admin import auth as firebase_auth
from firebase_admin import credentials
from rest_framework import authentication, exceptions
from django.contrib.auth.models import User
from django.utils import timezone

if not firebase_admin._apps:
    cred = credentials.Certificate("./firebase.json")
    default_app = firebase_admin.initialize_app(cred)

class FirebaseAuthentication(authentication.BaseAuthentication):
    def authenticate(self, request):
        auth_header = request.META.get("HTTP_AUTHORIZATION")
        if not auth_header:
            raise NoAuthToken("No auth token provided")

        id_token = auth_header.split(" ").pop()
        decoded_token = None
        try:
            decoded_token = firebase_auth.verify_id_token(id_token)
        except Exception:
            raise InvalidAuthToken("Invalid auth token")

        if not id_token or not decoded_token:
            return None

        try:
            uid = decoded_token.get("uid")
        except Exception:
            raise FirebaseError()

        user, created = User.objects.get_or_create(username=uid)
        return (user, None)
```

위 내용은 jwt token이 넘어왔을 때 처리하는 process이다.

이제 위 파일을 settings.py에 연결해주자.

```python
REST_FRAMEWORK = {
    "DEFAULT_AUTHENTICATION_CLASSES": (
        "rest_framework.authentication.SessionAuthentication",
        "config.authentication.FirebaseAuthentication",
    ),
}
```

이제 테스트로 유저 리스트를 반환하는 class를 만들어보자

```python
class ListUsers(APIView):

    permission_classes = [permissions.IsAuthenticated]

    def get(self, request, format=None):
        usernames = [user.username for user in User.objects.all()]
        return Response(usernames)
```

이제 Postman으로 테스트 해보겠다.

![_2021-02-02__4.19.05.png](/assets/img/notion/_2021-02-02__4.19.05.png)

아주 성공적으로 잘 나오며

새로운 유저 생성 및 로드도 잘 처리 된다.

이제 Token으로 나머지 rest api들을 처리하면 되겠다.
