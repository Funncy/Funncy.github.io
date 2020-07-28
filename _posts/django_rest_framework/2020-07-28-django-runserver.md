---
layout: post
title: "[Django] - Runserver 설정 "
subtitle:   "Django - Runserver setting "
categories: django
tags: django runserver
comments: true
---


기본적으로 Django에서 runserver 명령어를 지원하여 내부 IP 8000 포트로 개발 서버를 띄울 수 잇다.

```python
python manage.py runserver
```

여기서 포트를 바꾸는 법은 뒤에 숫자를 붙여 주는 방법이 있다.

```python
python manage.py runserver 8080
```

추가적으로 내부 IP접속을 허용 하려면 

```python
python manage.py runserver 0:8000
```

이렇게 하고 settings.py에서 아래와 같이 설정해주자.

```python
ALLOWED_HOSTS = ["*"]
```