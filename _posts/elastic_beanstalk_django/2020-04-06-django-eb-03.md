---
layout: post
title: "[Django] -  Elastic Beanstalk 배포 03 : DB 설정 - mysql "
subtitle:   "Django - Elastic Beanstlak Deploy"
categories: django
tags: django ealsticbeanstalk eb deploy
comments: true
---

[이전시간](https://funncy.github.io/django/2020/04/06/django-eb-02/)에 이어서 `DB`설정을 해보자

---

### DB 설정

`Django`를 `Elastick Beanstalk`에서 배포할 때 `mysql`을 사용하는 법을 알아보자.

* 목차
    * [RDS 생성](#1-rds-생성)
    * [파라미터 그룹](#2-파라미터-그룹)
    * [보안그룹 설정](#3-보안그룹-설정)
    * [mysql 설정](#4-mysql-설정)


---

#### 1. RDS 생성

`AWS`에서 `RDS`를 생성하자
> Elastic Beanstalk 내부 구성에서 DB를 생성하지 않는 이유는 나중에 instance가 많아지고 잠깐 instance를 죽여아하거나 업데이트 해야할 때 DB서버는 따로 분리하기 위함이다.

<img src="https://Funncy.github.io/assets/img/django-eb/2020-04-06-django-eb-15.png" height="500">
<img src="https://Funncy.github.io/assets/img/django-eb/2020-04-06-django-eb-16.png" height="500">
<img src="https://Funncy.github.io/assets/img/django-eb/2020-04-06-django-eb-30.png" height="500">

꼭 네트워크 보안에서 `퍼블릭 액세스` 가능성을 예로 설정해주자
* 설정해줘야 터미널에서 접속이 가능하다

---

#### 2. 파라미터 그룹

`한글`과 `Timezone`을 사용하려면 `파라미터 그룹`을 새로 생성해야한다.

![이미지](https://Funncy.github.io/assets/img/django-eb/2020-04-06-django-eb-17.png "parameter group")

생성 후 `character_set` 을 검색해 전부 `utf-8`로 수정한다. 
그리고 `time_zone`을 검색해 `Asia/Seoul`로 수정한다.

<img src="https://Funncy.github.io/assets/img/django-eb/2020-04-06-django-eb-18.png" height="500">
<img src="https://Funncy.github.io/assets/img/django-eb/2020-04-06-django-eb-20.png" height="500">

> 이미 `DB 테이블`이 생성된 경우 `character_set`과 `time_zone`설정 변경 후 `DATABASE`를 지우고 새로 생성해야한다.

그리고 생성했던 `RDS`에서 `수정`버튼을 누르고 `파라미터 그룹`을 변경한다.

![이미지](https://Funncy.github.io/assets/img/django-eb/2020-04-06-django-eb-19.png "parameter group change")

저장하고 재부팅한다.

---

#### 3. 보안그룹 설정

`RDS`의 보안그룹에 `Elastic Beanstalk`을 연결한다.
생성한 `RDS`의 보안그룹의 인바운드 규칙을 누른다.

![이미지](https://Funncy.github.io/assets/img/django-eb/2020-04-06-django-eb-21.png "security group")

<br>

![이미지](https://Funncy.github.io/assets/img/django-eb/2020-04-06-django-eb-23.png "security group")

인바운드 규칙 편집에서 `django-instance`의 보안그룹을 모든 트래픽으로 추가해준다.
> `django-instnace`의 `보안그룹`은 `aws elastic beanstalk` `구성화면`에서 확인 가능하다.

![이미지](https://Funncy.github.io/assets/img/django-eb/2020-04-06-django-eb-22.png "security group")

---

#### 4. mysql 설정

아래와 같이 `django-dotenv`를 통해 (선택사항) `DEBUG`용과 `RELEASE`용 DB를 구분하여 설정해도 된다.
> `mysqlclient`, `django-dotenv`(선택)을 설치해주자

![이미지](https://Funncy.github.io/assets/img/django-eb/2020-04-06-django-eb-24.png "settings.py mysql")

하는김에 `django-dotenv`를 통해 `SECRET_KEY`와 `DEBUG`도 환경변수에서 읽게 만들자.

![이미지](https://Funncy.github.io/assets/img/django-eb/2020-04-06-django-eb-25.png "settings.py secret key")

로컬환경에서는 `.env`파일을 만들어서 관리하고 꼭 `gitignore`에 등록해주자.

> 패키지가 변경될 경우 `pipenv run pip freeze > requirements.txt`를 꼭 실행하자.

이제 `Elastic Beanstalk` 화면에서 환경변수를 등록해주자.
`RDS`에서 설정했던 내용을 그대로 적어준다.

![이미지](https://Funncy.github.io/assets/img/django-eb/2020-04-06-django-eb-27.png "eb enviroment setting")

그리고 실행하면 아마 `DB`가 없다는 에러가 나올것이다. 아직 `DATABASE`가 생성되지 않아서 그렇다.
직접 `터미널`을 통해 연결해보자.
그리고 `DATABASE`를 `DB_NAME`과 동일하게 생성한다. (`DB_NAME`에는 `-`가 못들어가니 `_`로 수정하자.)

![이미지](https://Funncy.github.io/assets/img/django-eb/2020-04-06-django-eb-28.png "mysql connect")

`.ebextensions`에 `django.config`파일을 아래와 같이 수정하자.
`deploy`할 때 `makemigrations`와 `migrate`동작을 위함이다.

![이미지](https://Funncy.github.io/assets/img/django-eb/2020-04-06-django-eb-29.png "django.config - migration")

`deploy`후 확인해보자
정상적으로 `DB`가 구성되었다.

![이미지](https://Funncy.github.io/assets/img/django-eb/2020-04-06-django-eb-31.png "created table")

> 주의 사항 1.파라미터 그룹(한글, 타임존) 2.보안그룹 3.DB_NAME 4.퍼블릭 액세스 허용


---

다음은 `STATIC`과 `MEDIA` 설정법을 알아보자.

---

### `elastic beanstalk`을 활용한 `django` 배포
#### [1. 프로젝트 준비](https://funncy.github.io/django/2020/04/06/django-eb-01/)
#### [2. 기본 프로젝트 배포](https://funncy.github.io/django/2020/04/06/django-eb-02/)
#### [3. DB 설정 (mysql)](https://funncy.github.io/django/2020/04/06/django-eb-03/)
#### [4. static / media 설정](https://funncy.github.io/django/2020/04/06/django-eb-04/)
#### [5. custom command](https://funncy.github.io/django/2020/04/08/django-eb-05/)