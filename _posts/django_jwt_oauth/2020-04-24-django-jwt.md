---
layout: post
title: "[Django] -  REST Framework를 사용한 JWT OAuth 로그인 "
subtitle:   "Django - jwt social login"
categories: django
tags: django android ios restframework jwt oauth allauth rest_auth
comments: true
---

`flutter`로 소셜 로그인 앱을 만들었지만 이제 `django` 서버와 `rest api`로 연동해야한다. 로그인 인증을 `rest`로 연결해보자.


---

### 0. JWT 인증

`Django Rest Framework` 에서 기본적으로 지원하는 `Token` 은 `User` 와 1:1 매칭되는 단순한 랜덤문자이다.

`JWT(JSON WEB TOKEN)`은 토큰 자체에 데이터를 가지고 있다.
+ 포맷 : 헤더.내용.서명
+ 로직만으로 인증이 가능

`Token`은 반드시 안전한 장소에 보관해야합니다.
`App`에서는 `SharedPreferences`에 저장하도록 합시다.

### 1. Django 프로젝트 준비

먼저 `Django` 기본 프로젝트에 `RestFramework`와 `JWT`를 설치해줍시다.

```python
pip install djangorestframework
pip install djangorestframework-jwt
```

프로젝트/urls.py에 url과 패키지를 등록합시다.
+ `obtain_jwt_token` : 토큰 획득
+ `refresh_jwt_token` : 토큰 갱신
+ `verify_jwt_token` : 토큰 확인

```python
from rest_framework_jwt.views import obtain_jwt_token, refresh_jwt_token, verify_jwt_token

urlpatterns = [
    ... ,
    path('api-jwt-auth/', obtain_jwt_token),          # JWT 토큰 획득
    path('api-jwt-auth/refresh/', refresh_jwt_token), # JWT 토큰 갱신
    path('api-jwt-auth/verify/$', verify_jwt_token),   # JWT 토큰 확인
]
```

settings.py에서 `REST_FRAMEWORK` 세팅값을 설정해줍시다.
그리고 `JWT` 설정을 해줍니다.

+ `JWT_EXPIRATION_DELTA` : `Token` 만료시간으로 기본값은 5분 입니다. 5분은 너무 짧으니 늘려 줍시다.
+ `JWT_ALLOW_REFRESH` : `Token Refresh`가 가능하도록 해줘야 합니다. 따라서 `True`로 바꾸어 줍니다.
+ `JWT_REFRESH_EXPIRATION_DELTA` : `Refresh` 가능 시간 입니다. 상식적으로 만료 되기전에 `Refresh`를 하는게 맞으므로 위의 `JWT_EXPIRATION_DELTA` 시간보다 짧게 해주면 됩니다.

+ `REST_USE_JWT` : `Token` 발급 시 `JWT`를 기본으로 사용하도록 합니다.

```python
...

REST_FRAMEWORK = {
    ...
    'DEFAULT_AUTHENTICATION_CLASSES' : [
        ...
        'rest_framework_jwt.authentication.JSONWebTokenAuthentication',
    ]
}

...

JWT_AUTH = { 
    'JWT_SECRET_KEY': settings.SECRET_KEY, 
    'JWT_ALGORITHM': 'HS256', 
    'JWT_EXPIRATION_DELTA': datetime.timedelta(seconds=300), 
    'JWT_ALLOW_REFRESH': True, 
    'JWT_REFRESH_EXPIRATION_DELTA': datetime.timedelta(days=7), 
}

...

REST_USE_JWT = True

...
```

상단 url에서 테스트를 진행해봅시다.
## JWT 토큰 테스트

문제가 없으면 다음 단계를 진행합니다.


### 2. allauth rest_auth 설정

이제 app단에서 `Access token`을 서버로 보내서 `jwt`토큰으로 변경합니다.

일단 소셜로그인 진행할 개발자사이트에서 `web`플랫폼으로 현재 `django` 프로젝트를 연결해줍시다. (로컬환경이니 http://127.0.0.1:8000 으로 했습니다.)

#### 2-1. rest-auth, allauth

패키지를 설치합시다.
+ `allauth` : 소셜로그인을 연동하기 위한 패키지입니다.
+ `rest-auth` : `allauth`와 연동하여 `rest`로 연결해줍니다.

```python
pip install django-rest-auth
pip install django-allauth
```

settings.py에서 `INSTALLED_APPS` 을 추가해줍시다.

```python
INSTALLED_APPS = (
    'rest_framework',
    'rest_framework.authtoken',
    ...,
    'rest_auth'
    'django.contrib.sites',
    'allauth',
    'allauth.account',
    'rest_auth.registration',
)
```

그리고 settings.py에 `SITE_ID` 기본값을 설정해줍시다.

```python
SITE_ID = 1
```

`urls.py` 에 다음처럼 등록해줍시다.

```python
urlpatterns = [
    ...,
    path('rest-auth/', include('rest_auth.urls')),
    path('rest-auth/registration/', include('rest_auth.registration.urls'))
]
```

그리고 `migrate`를 진행해줍시다.

```python
python manage.py migrate
```

#### 2-2. Social Authentication 등록

이제 `INSTALLED_APPS`에 `allauth.socialaccount`와 내가 연동할 `providers`를 추가합니다. 
저는 `facebook`과 `kakao`를 연동해보겠습니다.

```python
INSTALLED_APPS = [
    ...,
    'rest_framework',
    'rest_framework.authtoken',
    'rest_auth'
    ...,
    'django.contrib.sites',
    'allauth',
    'allauth.account',
    'rest_auth.registration',
    ...,
    'allauth.socialaccount',
    'allauth.socialaccount.providers.facebook',
    'allauth.socialaccount.providers.kakao',
]
```

#### 2-3. SNS 등록

`accounts` 라는 앱을 만들어서 urls.py에 등록해줍시다.

```python
urlpatterns = [
    ...
    path('accounts/', include('accounts.urls')),
    ...
]
```

이제 연동할 `SNS` 별로 url을 설정해줍시다.

```python
from .views import KakaoLogin, FacebookLogin

app_name = 'accounts'

urlpatterns = [
    path('rest-auth/kakao/$', KakaoLogin.as_view(), name='kakao_login'),
    path('rest-auth/facebook/$', FacebookLogin.as_view(), name='facebook_login'),
]
```

views.py 를 다음과 같이 작성합시다.

```python
from allauth.socialaccount.providers.kakao.views import KakaoOAuth2Adapter
from allauth.socialaccount.providers.facebook.views import FacebookOAuth2Adapter
from rest_auth.registration.views import SocialLoginView

class KakaoLogin(SocialLoginView):
    adapter_class = KakaoOAuth2Adapter
    
class FacebookLogin(SocialLoginView):
    adapter_class = FacebookOAuth2Adapter
```

#### 2-4. django admin 등록

`django admin`에서 `Social Accounts` > `Social applications` > `Add social application`를 클릭해서 등록해줍시다.

`provider`와 `이름(임의로)`, `ClientID`, `SecretKey`를 등록해줍시다.
* `Kakao`는 `SecretKey`가 없습니다.

그리고 `Sites`에서 `example.com`을 연결해줍니다.

## 이미지

### 3. 연동 테스트

앱에서 직접 `access token`을 서버로 보내서 테스트해볼수있습니다.
아니면 개발자 사이트에서 `access token`을 직접 받아서 진행해도 됩니다.

저는 먼저 개발자 사이트에서 `access token`을 받아서 `post man` 으로 테스트해보겠습니다.

