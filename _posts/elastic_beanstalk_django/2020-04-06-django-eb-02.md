---
layout: post
title: "[Django] -  Elastic Beanstalk 배포 02 : 기본 프로젝트 배포 "
subtitle:   "Django - Elastic Beanstlak Deploy"
categories: django
tags: django ealsticbeanstalk eb deploy
comments: true
---

[이전시간](https://funncy.github.io/django/2020/04/06/django-eb-01/)에 이어서 `Django`기본 프로젝트를 `Elastic Beanstalk`에 배포해보자.

* 목차

   * [Instance 생성](#1-instance-생성)
        * [requirements.txt](#1-1-requirementstxt)
        * [.ebextensions](#1-2-ebextensions)
        * [django.config](#1-3-djangoconfig)
        * [git](#1-4-git)
        * [eb create](#1-5-eb-create)
   * [서버 구동](#2-서버-구동)
        * [Allow Host](#2-1-allow-host)
        * [Default DB](#2-2-default-db)
        * [eb deploy](#2-3-eb-deploy)

---

## 1. Instance 생성

#### 1-1. requirements.txt
먼저 `Django`프로젝트의 `requirements.txt`파일을 만들어주자.

> `pipenv`를 사용한다면 `pipenv run pip freeze > requirements.txt`를 사용하자.
> `pip`는 `pip freeze > requirements.txt`를 사용하자

#### 1-2. .ebextensions

가장 바깥쪽에 `.ebextensions` 폴더를 생성하자 
폴더 안에는 `django.config`파일을 생성해주자

> 폴더 구조에 주의하자. 반드시 `.ebextensions`을 폴더 내에 생성해야한다. 

![이미지](https://Funncy.github.io/assets/img/django-eb/2020-04-06-django-eb-11.png "폴더 구조")

다음과 같은 폴더구조를 가진다. (`.elasticbeanstlak`폴더는 무시한다)
* `requirements.txt`와 `.ebextensions`는 같은 라인에 있어야한다.

#### 1-3. django.config

![이미지](https://Funncy.github.io/assets/img/django-eb/2020-04-06-django-eb-10.png ".ebextensions django.config")

`django.config`내용을 다음과 같이 작성한다.

> `WSGIPath`에는 내가 생성한 프로젝트 구조에 맞춰준다. 일반적인 구조라면 `ProjectName`/`config`/`wsgi.py`이다.

#### 1-4. git

이제 `.gitignore`에 `django`내용을 넣어주고 

`git add . && git commit -m `을 해주자 

> `eb cli`는 `local git`에 있는 내용을 `.zip`파일로 만들어 서버로 전송한다. 그러니 항상 `deploy` 하기 전에 `git add && commit`을 해주자.

#### 1-5. eb create

이제 `eb create instance-name`을 해주자

![이미지](https://Funncy.github.io/assets/img/django-eb/2020-04-06-django-eb-12.png "eb create instance")

이렇게 성공적으로 생성되었다는 메시지를 볼 수 있다.
> Error 발생시 Log를 잘 읽어보면 문제를 해결할 수 있다. 보통 `requirements.txt`가 올바른 경로에 없거나 `.ebextensions`폴더 경로가 잘못된 경우이다.

AWS에 가보면 이렇게 정상적으로 생성되었다.

![이미지](https://Funncy.github.io/assets/img/django-eb/2020-04-06-django-eb-13.png "eb instance")

---

## 2. 서버 구동

자 이렇게 진행하고 `AWS`콘솔에서 알려주는 `url`을 타고 들어가보면 500번 에러가 발생한다.

#### 2-1. Allow Host

먼저 해야할건 `django` `settings.py`에 `ALLOWED_HOSTS`에 `url`을 추가해주자.

`url`은 `eb status`를 통해 나오는 `CNAME`에서 확인 가능하다.

> `django`는 `Web Application Server`라서 `Apache`나 `Nginx`와 같은 `Web Server`를 통해서 운영된다. 그렇기에 `Web Server`가 접근 가능하게 `allow`해주어야 한다.

#### 2-2. Default DB

그런데 나는 여기서 실행하니 문제가 발생했다. => `Log` 확인
`eb logs`에서 확인이 가능하고 `aws`에서 `log`를 다운 받을 수도 있다.

읽어보니 `settings.py`에 설정되있는 기본 `sqlite`버전과 관련해서 문제가 생기는것 같다. 그래서 일단 `settings.py`에서 `db`부분을 주석처리해서 임시로 해결했다. (`DB설정`은 다음 챕터에서 알아보자)

#### 2-3. eb deploy

수정된 사항을 `git`에 올리고 `eb deploy`해주면 변경사항이 반영된다.

이제 `eb open`으로 사이트를 열어보자.

![이미지](https://Funncy.github.io/assets/img/django-eb/2020-04-06-django-eb-14.png "eb open")

정상적으로 페이지가 로드 되었다.

---

## 다음 내용

이제 알아볼 내용은

 * DB
 * Static/Media
 * Custom Command 

순서대로 알아보자.

---

### `elastic beanstalk`을 활용한 `django` 배포
#### [1. 프로젝트 준비](https://funncy.github.io/django/2020/04/06/django-eb-01/)
#### [2. 기본 프로젝트 배포](https://funncy.github.io/django/2020/04/06/django-eb-02/)
#### [3. DB 설정 (mysql)](https://funncy.github.io/django/2020/04/06/django-eb-03/)
#### [4. static / media 설정](https://funncy.github.io/django/2020/04/06/django-eb-04/)
#### [5. custom command](https://funncy.github.io/django/2020/04/08/django-eb-05/)