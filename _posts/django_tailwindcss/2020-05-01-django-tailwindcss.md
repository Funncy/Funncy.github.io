---
layout: post
title: "[Django] -  tailwindcss 설치하기 "
subtitle:   "Django - tailwindcss "
categories: django
tags: django tailwindcss css sass gulp
comments: true
---

`Django`프로젝트를 진행하며 `front-end`에 대해서 고민하게 되면서 `tailwindcss`를 알게되었다.
설치법을 여기 공부겸 정리해보겠다.

+ 1. npm init
+ 2. gulp 설치
+ 3. tailwindcss 설치
+ 4. gulpfile.js 설정
+ 5. npm run css

---

### 1. npm init

`npm`을 설치하고 `django`디렉토리에서 `npm init`을 실행해주자.
그러면 `package name`부터 여러가지를 물어보겠지만 일단 다 넘겨준다.

그러면 디렉토리에 `package.json`파일이 생긴다. 파일을 열어서 필요없는건 다 지워주자.

```json
{
  "name": "wheels",
  "version": "1.0.0",
  "description": ""
}

```

### 2. gulp 설치

다음과 같이 터미널에서 설치해주자. `-D`는 개발환경에서 설치이다.

```python
npm i gulp gulp-postcss gulp-sass gulp-csso node-sass -D
```

---

### 3. tailwindcss 설치

`npm`으로 `tailwindcss`를 설치해주자.

```python
npm install tailwindcss -D
npm install autoprefixer -D
```

설치가 끝나면 디렉토리에 `node_modules`라는 폴더가 생긴다.

`gitignore`에 꼭 넣어주자. 

그리고 터미널에서 아래 명령어를 쳐주자.

```python
npx tailwind init
```

그러면 디렉토리에 `tailwind.config.js`가 생긴다.

---

### 4. gulpfile.js 설정

먼저 메인 디렉토리에 `assets > scss > styles.scss`를 만들고 아래 내용을 넣어주자.

```js
@tailwind base;
@tailwind components;
@tailwind utilities;
```

그리고 메인 디렉토리에 `gulpfile.js` 파일을 만들어주자.

```js
const gulp = require("gulp");

const css = () => {
    const postCSS = require("gulp-postcss");
    const sass = require("gulp-sass");
    const minify = require("gulp-csso");
    sass.compiler = require("node-sass");

    return gulp
        .src("assets/scss/styles.scss")
        .pipe(sass().on("error", sass.logError))
        .pipe(postCSS([require("tailwindcss"), require("autoprefixer")]))
        .pipe(minify())
        .pipe(gulp.dest("static/csss"));
}

exports.default = css;
```

---

### 5. npm run css

이제 `css`파일로 생성해보자.

먼저 `package.json`에 `scripts`를 추가해주자.

```json
{
  "name": "wheels",
  "version": "1.0.0",
  "description": "",
  "devDependencies": {
    "autoprefixer": "^9.7.6",
    "gulp": "^4.0.2",
    "gulp-csso": "^4.0.1",
    "gulp-postcss": "^8.0.0",
    "gulp-sass": "^4.1.0",
    "node-sass": "^4.14.0"
  },
  "scripts": {
    "css": "gulp"
  },
  "dependencies": {
    "tailwindcss": "^1.4.1"
  }
}
```

그리고 터미널에서 다음 명령어를 실행한다.

```python
npm run css
```

그러면 `static/css`에 `styles.css`가 생성된다.
이제 `django-template`에서 불러와서 사용하면된다.



