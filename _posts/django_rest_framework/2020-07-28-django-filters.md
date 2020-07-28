---
layout: post
title: "[Django] - View filter 설정 "
subtitle:   "Django - View filter setting "
categories: django
tags: django view filter
comments: true
---

`Django` `rest-framework`를 적용하여 빠르게 Test 데이터를 `CRUD`하게 만들었다.

그런데 문제가 `category`별 검색이 필요하였다.

현재의 `ModelViewset`은 가장 기본 설정이다.

```python
class LocationViewSet(viewsets.ModelViewSet):

    queryset = Location.objects.all()
    serializer_class = LocationSerializer
```

여기서 category값을 pk처럼 받아와서 그걸 가지고 queryset을 filtering 해줘야 한다.

하지만 여기서 아주 쉬운 방법을 소개하겠다.

`DjangoFilterBackend` 은 이와 같은 경우에 사용하는 라이브러리다.

먼저 `pip`를 사용해 설치해주자. (내 환경은 `pipenv`이다.)

```python
pipenv install django-filter
```

>django-filter는 django3.0 혹은 2.2로 진행하자. 
>그러지 않은 경우 ImportError: cannot import name 'ORDER_PATTERN' from 'django.db.models.sql.constants' 와 관련된 에러가 발생한다.

그리고 `settings.py`에 설정을 추가해주자.

```python
INSTALLED_APPS = [
...
    "django_filters",
...
]

...

REST_FRAMEWORK = {
...
    "DEFAULT_FILTER_BACKENDS": ["django_filters.rest_framework.DjangoFilterBackend"],
}
```

그리고 View파일에서 다음과 같이 수정해주자.

```python
from django_filters.rest_framework import DjangoFilterBackend

class LocationViewSet(viewsets.ModelViewSet):

    queryset = Location.objects.all()
    serializer_class = LocationSerializer
    filter_backends = [DjangoFilterBackend]
    filterset_fields = ["category"]
```

이렇게 되면 url에서 다음과 같이 작성하면 정상 동작한다.

```
http://example.com/api/products?category=clothing
```

category가 비어있다면 모든 데이터를 가져온다.