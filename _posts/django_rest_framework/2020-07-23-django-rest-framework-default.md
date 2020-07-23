---
layout: post
title: "[Django] - 기본 REST Framework Template "
subtitle:   "Django - restframework default template"
categories: django
tags: django restframework
comments: true
---

자주 만들어야 하는 `REST API`를 `Django`로 빠르게 만들어보자.

기본 틀이기 때문에 이 상태에서 많은 것을 추가하고 수정해야한다.

일단 기본 장고 프로젝트부터 시작하자.

나는 `pipenv`를 사용해서 환경 설정을 하였다.

# 1. 장고 프로젝트 설정

먼저 프로젝트 이름을 `config`로 생성한다.

```java
django-admin startproject config
```

이렇게 생성하면 폴더 구조가 `config`폴더 아래 `config`가 생기게 된다. 

가장 바깥의 폴더를 삭제하고 전부 바깥으로 꺼내자.

> 이건 폴더 구조라서 취향대로 진행해도 된다.

그리고 기본적인 `app`들을 생성해주자.

```java
python manage.py startapp core
python manage.py startapp poi
```

> core : 추상 테이블을 통해 상속을 위한 모델
poi : GPS 데이터 저장을 위한 모델 (임시)

# 2. 기본 App 설정

일단 `Core`와 `POI` 앱에 기본적인 `url`, `models` 를 작성해보자.

> views는 rest_framework에서 작성한다.

### 2-1. core models

```python
from django.db import models

class TimeStampedModel(models.Model):

    """ Time Stamp Model """

    created = models.DateTimeField(auto_now_add=True)
    udpated = models.DateTimeField(auto_now=True)

    # DB에 등록되지 않는 모델
    class Meta:
        abstract = True
```

추상 클래스로 다른 모델에서 상속을 위한 기본적인 시간 정보를 넣어 두었다.

### 2-2. POI models

```python
from django.db import models
from core import models as core_models

class Location(core_models.TimeStampedModel):

    """ Location Model """

    LOCATION_AVAILABLE = "available"  # 사용 가능 상태
    LOCATION_UNAVAILABLE = "unavailable"  # 사용 불가 상태
    LOCATION_WATING = "wating"  # 평가 대기 상태

    STATUS_CHOICES = (
        (LOCATION_AVAILABLE, "Available"),
        (LOCATION_UNAVAILABLE, "Unavailable"),
        (LOCATION_WATING, "Wating"),
    )

    latitude = models.CharField(max_length=80)
    longitude = models.CharField(max_length=80)
    average_rating = models.FloatField(default=0.0)
    status = models.CharField(choices=STATUS_CHOICES, max_length=50)
		# user = models.ForeignKey("user.User", related_name="location_rating", on_delete=models.CASCADE)

```

테스트 버전이기 때문에 User와 GPS정보는 임시로 처리 하였다.

# 3. REST Framework 적용

이제 `rest_framework`를 적용해보자.

```python
pipenv install djangorestframework
```

라이브러리를 설치해주고 `Settings.py`를 수정해주자.

```python
INSTALLED_APPS = [
...
    "rest_framework",
    "core.apps.CoreConfig",
    "poi.apps.PoiConfig",
]

...

REST_FRAMEWORK = {
    "DEFAULT_PAGINATION_CLASS": "rest_framework.pagination.PageNumberPagination",
    "PAGE_SIZE": 10,
}
```

아래 내용은 페이지네이션을 위한 내용이다.

일단 테스트용이니 DB는 기본설정으로 진행하겠다.

> 검새해보니 GPS데이터는 `PostGIS`를 통해 `GeoDjango`를 사용하면 분석도 진행가능하다.

이제 `serializer`와 `viewset` , `url` 을  작성해주자.

### 3-1. serializer

```python
from .models import Location
from rest_framework import serializers

class LocationSerializer(serializers.ModelSerializer):
    class Meta:
        model = Location
        fields = [
            "latitude",
            "longitude",
            "average_rating",
            "status",
        ]
```

`Model`과 연동을 위해 `ModelSerializer`를 사용해 기본적인 `Serializer`를 만들었다.

이제 `viewset`를 작성해보자.

### 3-2. viewset

```python
from .models import Location
from rest_framework import viewsets
from .serializers import LocationSerializer

class LocationViewSet(viewsets.ModelViewSet):

    queryset = Location.objects.all()
    serializer_class = LocationSerializer
```

### 3-3. poi urls

```python
from django.urls import include, path
from rest_framework import routers
from .views import LocationViewSet

router = routers.DefaultRouter()
router.register(r"locations", LocationViewSet)

app_name = "poi"

urlpatterns = [
    path("", include(router.urls)),
]
```

### 3-4. config urls

```python
from django.contrib import admin
from django.urls import path, include

urlpatterns = [
    path('', include("poi.urls", namespace="poi")),
    path('admin/', admin.site.urls),
]
```

아주 간단하게 끝났다. 

이제 DB생성을 해주고 run해보자

```python
python manage.py makemigrations
python manage.py migrate
python manage.py runserver
```

# 4. 결과

이제 `rest_framework`의 강력한 기능을 살펴보자

![이미지](https://Funncy.github.io/assets/img/django/2020-07-23-django-rest-framework-default-01.png "rest api 1")

이렇게 url로 들어오면 입력된 모든 location 정보를 불러온다.

아래 `POST`를 통해서 생성도 가능하다.

여기서 url을 살짝 수정해 `location/ID/` 이렇게 입력 해주면 자동으로 해당 ID값의 데이터를 가져온다.

![이미지](https://Funncy.github.io/assets/img/django/2020-07-23-django-rest-framework-default-02.png "rest api 2")

아래 해당 내용을 수정하고 `PUT` 하면 자동 업데이트를 해준다.

`CRUD` 기능이 자동으로 생성되기 때문에 `http`통신으로 해당 규약만 잘 지켜서 전송해주면 나머지는 `rest_framework`에서 처리해준다.

다음에는 커스터마이징등 내용을 수정해보자.