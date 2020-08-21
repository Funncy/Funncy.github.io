---
layout: post
title: "[Flutter] -  MVVM 패턴 적용 [뉴스 앱] "
subtitle:   "Flutter - applied mvvm pattern "
categories: flutter
tags: flutter mvvm newsapp
comments: true
---

MVVM 패턴을 적용해서 토이 프로젝트를 진행해보자

[유튜브](https://www.youtube.com/watch?v=7n2QprcdHMA&t=376s) puzzleleaft 님꺼를 참고하였고 WebService 쪽 부분만 커스텀 하여서 진행하였다

간단한 뉴스 앱을 만들어보자 

# 1. News API

먼저 뉴스 정보를 가져올 API 부터 설정하자

![이미지](https://Funncy.github.io/assets/img/flutter-mvvm/2020-08-21-flutter-mvvm-01.png "news api")

NewsAPI 사이트에서 API Key를 발급받고 

해당 api를 테스트해보자

![이미지](https://Funncy.github.io/assets/img/flutter-mvvm/2020-08-21-flutter-mvvm-02.png "post man")

잘 가져온다. 

이제 앱 개발을 시작해보자

# 2. Model

나는 news_app 이라는 이름으로 Flutter 프로젝트를 생성하였다

이제 API에서 가져온 데이터를 기반으로 `Model`을 만들어보자

```java
class NewsArticle {
  final String title;
  final String author;
  final String description;
  final String url;
  final String urlToImage;
  final String publishedAt;
  final String content;

  NewsArticle(
      {this.author,
      this.title,
      this.description,
      this.url,
      this.urlToImage,
      this.publishedAt,
      this.content});

  factory NewsArticle.fromJson(Map<String, dynamic> json) {
    return NewsArticle(
      title: json['title'],
      author: json['author'],
      description: json['description'],
      url: json['url'],
      urlToImage: json['urlToImage'],
      publishedAt: json['publishedAt'],
      content: json['content'],
    );
  }
}
```

이제 위의 Model과 연결할 ViewModel을 만들어보자

# 3. ViewModel

일단 Model에서 데이터를 읽어와 뿌려주는 내용부터 만들어보자

```java
import 'package:news_app/models/news_article.dart';

class NewsArticleViewModel {
  NewsArticle _newsArticle;

  NewsArticleViewModel({NewsArticle newsArticle}) : _newsArticle = newsArticle;

  String get title {
    return _newsArticle.title;
  }

  String get author {
    return _newsArticle.author;
  }

  String get url {
    return _newsArticle.url;
  }

  String get urlToImage {
    return _newsArticle.urlToImage;
  }

  String get content {
    return _newsArticle.content;
  }

  String get description {
    return _newsArticle.description;
  }

  String get publishedAt {
    return _newsArticle.publishedAt;
  }
}
```

다음은 List로 가져온 뉴스를 보여줄 ViewModel이다.

```java
import 'package:flutter/material.dart';

enum LoadingStatus { completed, searching, empty }

class NewsArticleListViewModel extends ChangeNotifier {
  //초기에 로딩 데이터 없음
  LoadingStatus loadingStatus = LoadingStatus.empty;

  //기사 가져오기
  void topHeadLines() {
    //...

    //LoadingStatus를 로딩중으로 변경

    //데이터 가져오면 업데이트
    notifyListeners();
  }
}
```

이제 기본적인 Model에서 데이터를 가져와 ViewModel로 연결하는 건 끝이 났다

이제 남은 내용은

- Http 통신을 통해 API 호출
- ViewModel Provider 연결
- UI 코딩 및 ViewModel 연결

하나씩 해보자

# 3. API 호출

Dio 라이브러리를 통해 http 호출을 진행해보자 (pubspec.yaml은 스킵하겠다)

Service라는 폴더에 news_api.dart를 만들어보자

```java
import 'package:dio/dio.dart';
import 'package:news_app/models/news_article.dart';

class NewsAPI {
  var dio = Dio();

  Future<List<NewsArticle>> fetchArticle() async {
    String url = "";

    final response = await dio.get(url);

    if (response.statusCode == 200) {
      final result = response.data;
      Iterable list = result['articles'];
      return list.map((item) => NewsArticle.fromJson(item)).toList();
    } else {
      throw Exception("failed to get top news");
    }
  }
}
```

뉴스들을 List로 가져오도록 만들었다.

화면은 뉴스 List와 뉴스 Detail로 나눠진다.

먼저 `NewsArticleListViewModel` 부터 `API`와 연결해주자.

```java
import 'package:flutter/material.dart';
import 'package:news_app/models/news_article.dart';
import 'package:news_app/services/news_api.dart';
import 'package:news_app/viewmodels/news_article_view_model.dart';

enum LoadingStatus { completed, searching, empty }

class NewsArticleListViewModel extends ChangeNotifier {
  //초기에 로딩 데이터 없음
  LoadingStatus loadingStatus = LoadingStatus.empty;

  List<NewsArticleViewModel> articles = List<NewsArticleViewModel>();

  //기사 가져오기
  void topHeadLines() async {
    //기사 가져오고 현재 상태 로딩중으로 변경
    List<NewsArticle> newsArticles = await NewsAPI().fetchArticle();
    loadingStatus = LoadingStatus.searching;
    notifyListeners();

    this.articles = newsArticles
        .map((item) => NewsArticleViewModel(newsArticle: item))
        .toList();

    //가져온 데이터가 비어있으면 빈 상태 아니면 성공 상태
    this.loadingStatus =
        this.articles.isEmpty ? LoadingStatus.empty : LoadingStatus.completed;
    notifyListeners();
  }
}
```

이제 화면을 작성해주자

# 4. View

먼저 main.dart 에 쓸모 없는것들은 지우고 Provider와 새로 생성할 화면을 연결해주자

```java
import 'package:flutter/material.dart';
import 'package:news_app/viewmodels/news_article_list_view_model.dart';
import 'package:news_app/views/news_screen.dart';
import 'package:provider/provider.dart';

void main() {
  runApp(MyApp());
}

class MyApp extends StatelessWidget {
  // This widget is the root of your application.
  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Flutter Demo',
      home: MultiProvider(providers: [
        ChangeNotifierProvider(
          create: (_) => NewsArticleListViewModel(),
        )
      ], child: NewsScreen()),
    );
  }
}
```

그리고 일단 GridView로 title을 뿌려주도록 만들어보았다.

```java
import 'package:flutter/material.dart';
import 'package:news_app/viewmodels/news_article_list_view_model.dart';
import 'package:provider/provider.dart';

class NewsScreen extends StatefulWidget {
  @override
  _NewsScreenState createState() => _NewsScreenState();
}

class _NewsScreenState extends State<NewsScreen> {
  @override
  void initState() {
    super.initState();
    Provider.of<NewsArticleListViewModel>(context, listen: false)
        .topHeadLines();
  }

  @override
  Widget build(BuildContext context) {
    var listViewModel = Provider.of<NewsArticleListViewModel>(context);

    return Scaffold(
        appBar: AppBar(),
        body: GridView.builder(
          gridDelegate:
              SliverGridDelegateWithFixedCrossAxisCount(crossAxisCount: 2),
          itemBuilder: (context, index) {
            return Text(listViewModel.articles[index].title);
          },
          itemCount: listViewModel.articles.length,
        ));
  }
}
```

![이미지](https://Funncy.github.io/assets/img/flutter-mvvm/2020-08-21-flutter-mvvm-03.png "news app 1")

이제 카드들을 꾸며보자

![이미지](https://Funncy.github.io/assets/img/flutter-mvvm/2020-08-21-flutter-mvvm-04.png "news app 2")

![이미지](https://Funncy.github.io/assets/img/flutter-mvvm/2020-08-21-flutter-mvvm-05.png "news app 3")

이렇게 뉴스 앱이 완성되었다.

# 5. 마무리

MVVM 패턴에 대해서 유튜브와 여러 자료를 보면서 공부했지만

아직 너무 간단한 앱들이라 실제로 어느정도 볼륨이 있는 앱을 만들어봐야 확실히 감이 올 것 같다

아직 ViewModel의 역할 선을 어디까지 해줘야할지 잘 모르겠다

[github](https://github.com/Funncy/flutter_news_app)
