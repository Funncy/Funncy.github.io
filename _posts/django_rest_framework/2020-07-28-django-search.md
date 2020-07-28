---
layout: post
title: "[Django] - View Search 설정 "
subtitle:   "Django - View search setting "
categories: django
tags: django view search
comments: true
---

`django view filter setting`에서 이어지는 내용이다. (라이브러리 설치 진행 필요)

> django-filter는 django3.0 혹은 2.2로 진행하자. 
> 그러지 않은 경우 ImportError: cannot import name 'ORDER_PATTERN' from 'django.db.models.sql.constants' 와 관련된 에러가 발생한다.

`Filtering`이 아니라 특정 단어들로 `Search`가 필요한 순간이 있다.

그때 `SearchFilter`를 통해서 쉽게 처리해 보자.

`View`를 다음가 같이 설정해주자.

```python
from rest_framework import filters

class LocationViewSet(viewsets.ModelViewSet):

    queryset = Location.objects.all()
    serializer_class = LocationSerializer
    filter_backends = [filters.SearchFilter]
    search_fields = ["status"]
```

만약에 `filtering`과 같이 사용하려면 다음과 같이 작성하자.

```python
from django_filters.rest_framework import DjangoFilterBackend
from rest_framework import filters

class LocationViewSet(viewsets.ModelViewSet):

    queryset = Location.objects.all()
    serializer_class = LocationSerializer
    filter_backends = [DjangoFilterBackend, filters.SearchFilter]
    filterset_fields = ["category"]
    search_fields = ["status"]
```

그리고 url을 다음과 같이 작성하여 검색을 진행해보자

```
http://example.com/api/users?search=russell
```

그러면 검색어가 포함된 status를 검색해준다.

추가적으로 `search option`을 알아보자.

- ^  : 검색어로 시작하는 경우만
- = : 정확히 일치하는 경우만
- @ : 단어/구문으로 검색 (mysql에서만 지원)
- $ : 정규표현식

옵션은 이렇게 사용 가능하다.

```python
search_fields = ["^status"] 
search_fields = ["=status"]
```

추가적으로 직접 SearchFilter를 수정하여 만들 수 있다.

```python
class CustomSearchFilter(filters.SearchFilter):
    def get_search_fields(self, view, request):
        if request.query_params.get("search"):
            return ["category"]
        return super(CustomSearchFilter, self).get_search_fields(view, request)
```

`SearchFilter`를 상속받아 `get_search_fields`를 override해주자

위의 예시는 `search`로 들어온 `parameter`가 있으면 `return['category']` 

즉, `category`에서 그 `parameter`와 값이 같은걸 찾아주도록 변경하였다. (원래는 `status`)