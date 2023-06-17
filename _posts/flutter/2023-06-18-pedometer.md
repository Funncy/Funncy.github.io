---
layout: post
title: '[Flutter] - 우당탕탕 만보기 개발기 '
subtitle: 'Flutter pedometer '
categories: flutter
tags: flutter googleFitness pedometer
comments: true
---

이번에 회사에서 신규 기능으로 만보기 기능을 출시하려고 했다.

요구사항은 캐시워크 같은 서비스와 비슷했다.

1. 매일 오늘 당일의 걸음수가 보여야함
2. 자정이 지나면 걸음수 0으로 초기화
3. 안드로이드의 경우 앱을 설치한 시점 부터 0으로 시작

일단 이걸 어떻게 구현할지 계획을 세우며 조사를 진행해보았다.

# 1. 사전 조사

---

다행히 아이폰은 Apple HealthKit을 통해서 쉽게 걸음수를 가져올 수 있었다.

> 추가적인 운동 데이터도 쉽게 가져올 수 있었다. (ex, 계단 올라간 수)
> 

문제는 안드로이드였다.

일단 안드로이드는 애플처럼 통일된 운동 데이터셋이 없었다.
안드로이드는 자체 걸음수 카운터 센서가 있는데 좀 특이하게 동작했다.

1. 부팅 이후부터 누적된 걸음수 반환 (즉, 앱에서 걸음수를 읽었을때 이게 몇일 동안 누적된 걸음인지 알수가없음)
2. 핸드폰 재부팅시 0부터 시작 (즉, 재부팅시 기존 걸음수를 유지하는 로직을 짜줘야함)

그나마 찾아보니 Google Fitness API가 있지만 테스트해본 결과 걸음수도 내가 직접 센서 연동 코드를 작성해줘야 그때부터 STEP_COUNTER 센서로부터 걸음수가 누적되기 시작했다. 
(그래도 다행히 걸음수 연산은 안작성해도 됬다.)

> 하지만 더 조사해보지 않고 Google  Fitness API로 가서 더 고생했다.. 이유는 아래에서 설명하겠다.
> 

먼저 가장 쉽게 해결된 애플부터 보자.

# 2. IOS 만보기

---

## 2.1 기능 구현

---

Apple HealthKit을 연동해서 걸음수를 가져오려고 찾아보니 Flutter에 health라이브러리가 있었다.

[health | Flutter Package](https://pub.dev/packages/health)

걸음수를 가져오기 위해서는 권한 허용부터 진행해야했다.

```java
if (Platform.isIOS) {
      HealthFactory health = HealthFactory();

      // define the types to get
      var types = [
        HealthDataType.STEPS,
      ];

      return await health.requestAuthorization(types);
 }
```

그런데 여기서 함수 명세를 잘 읽어보니 주의사항이 있었다.

As Apple HealthKit will not disclose if READ access has been granted for a data type due to privacy concern, this method will return **true if the window asking for permission was showed to the user without errors** if it is called on iOS with a READ or READ_WRITE access.

번역하면

Apple HealthKit은 개인정보 보호 문제로 인해 데이터 유형에 대한 READ 액세스 권한이 부여되었는지 여부를 공개하지 않으므로, 이 메서드는 iOS에서 READ 또는 READ_WRITE 액세스 권한으로 호출된 경우 사용자에게 권한을 묻는 창이 오류 없이 표시되면 true를 반환합니다.

즉, 애플은 유저가 권한을 허용했는지 알 방법이 없다…. (어이없네…)

이걸 좀 우회해서 해결한 방식은

```java
Future<bool> _hasPermissionsOniOS() async {
    try {
      HealthFactory health = HealthFactory();

      final now = DateTime.now();
      final lastMonth = DateTime(now.year, now.month - 1, now.day);
      final stepsOfLastMonth =
          await health.getTotalStepsInInterval(lastMonth, now);

      var midnight = DateTime(now.year, now.month, now.day);
      final stepsOfToday = await health.getTotalStepsInInterval(midnight, now);

      return stepsOfLastMonth != null || stepsOfToday != null;
    } catch (err) {
      FirebaseCrashlytics.instance.recordError(err, null);
      return false;
    }
 }
```

저번달부터 지금까지의 걸음수 , 오늘 걸음수 모두 비어있으면 권한이 없다고 판단하게 했다.

> 설마 핸드폰 사고 미동도 없이 거기서 우리 앱을 설치해서 사용하진 않겠지…
> 

이렇게 개발하고 연동해서 테스트를 진행해보았다.

아주 잘 동작한다.

![IMG_1657.PNG](https://github.com/Funncy/Funncy.github.io/assets/22514912/6a5c7e09-3ce2-420b-9d0f-b17ad8aa74ba)

![IMG_1658.PNG](https://github.com/Funncy/Funncy.github.io/assets/22514912/f2630a09-4bd4-464d-839a-aaddca9efd7a)

![IMG_1660.PNG](https://github.com/Funncy/Funncy.github.io/assets/22514912/fa91b359-77c1-4985-b8e6-2bbbf42017b1)

<aside>
⚠️ 만약 권한 허용을 거절하면 건강앱에서 데이터 카테고리를 직접 들어가서 권한을 허용해줘야한다.
설정창 처럼 앱에서 자동으로 켜주는건 어려운것 같다…

</aside>

## 2-2. 대망의 스토어 배포

---

나는 솔직히 애플이 개인정보에 민감해서 이 부분에서 꽤 난관을 예상했는데 생각보다 너무 쉽게 끝났다.

Flutter로 실제 앱을 배포해본 사람이라면 당연히 Info.plist에 권한 추가는 진행했을거다.

```java
<key>NSHealthShareUsageDescription</key>
<string>히로인스에서 사용자의 움직임을 측정합니다</string>
<key>NSHealthUpdateUsageDescription</key>
<string>히로인스에서 사용자의 움직임을 측정합니다</string>
```

그리고 한번 심사에 넣어봤으나 역시 리젝을 당했다.

해당 사유를 확인해보니 Apple HealtKit과 통합을 해야한다고 왔다. (통합이 뭔데….)

그래서 계속 추가로 대화하다보니 정리된 사항은

1. 앱이 수집하는 개인정보에 건강 및 피트니스 추가하기
2. 카테고리 건강 및 피트니스 (이건 정확하지 않음)
3. 앱내 설명에 Apple Healthkit을 언급하며 해당 데이터로 뭘 하는지 설명

그리고 바로 심사에 통과했다.

자 이제 개발 기간을 2배로 늘려준 안드로이드를 알아봐보자.

# 2. 안드로이드

---

## 2-1. 망할 Google Fitness API

---

일단 결론을 먼저 말하면 개발을 다 해놓고 쓸 수 없었다. 그 이유는 Google 에서 민감 정보, 제한 정보라고해서 개인 정보에서도 등급을 나누는데 피트니스 데이터는 제한 정보이다. 그래서 심사를 받아야한다. 
(제한된 정보의 OAuth 심사는 진짜 개빡세다)

단순히 앱 심사처럼 간단한 심사가 아니라 우리 앱이 해킹 기법이나 데이터 유출 사고가 없을만 한지 ISO 기준을 가지고 심사한다.
심사 방법으로는 3가지를 제안한다. 

1. 무료 툴 사용해라
2. 유료 툴 사용해라
3. 제3자 인증기관 통해서 해라.

> 인증 작업은 최대 6주 (아마 우리가 자료정리하는 시간은 제외한 시간일거다)
너희가 자료를 준비만 해주면 무료로 해줄게~
근데 너네 혼자 못하겠으면 `Tier 2 Authorized Lab Scan` (제 3자 인증기관) 를 진행해
> 

제 3자 인증 기관은 당연히 무료가 아니고 비용이 발생한다. 인터넷 검색으로 좀 찾아보고 Google OAuth 심사 담당자랑 메일을 준 결과 진행 불가로 판단했다.

[OAuth2 심사 변경 사항](https://app-developement-sharing-forum.tistory.com/entry/OAuth2-심사-변경-사항)

> 구글 OAuth 담당자는 비용은 말해주지 않았지만 위 블로그 기준으로는 매년 $4500 ~ $75000까지 나온다고 한다. (스타트업이라 빠르게 테스트를 해보고 해당 기능이 사라질수도 있는 입장에서 너무 큰 비용이였다.)
위 블로그에서는 걸음수는 제외라고 했지만 내가 메일로 문의해본 결과 해당된다고 한다.
저 블로그 글 이후에서 구글에서는 정책이 변경되고 있으니, 이걸 보고 Google fitness API를 적용하실 분들은 구글 사이트에서 직접 확인하는걸 추천한다.
> 

[OAuth API verification FAQs - Google Cloud Platform Console Help](https://support.google.com/cloud/answer/9110914#how-long-sec-assess-valid&zippy=,how-long-is-the-security-assessment-valid-for)

그래도 누군가는 필요할수 있으니 내가 알게된 Google Fitness API에 대한 정보를 공유해보겠다.

## 2-2. Google Fitness API 구조

---

먼저 Google Fitness API는 내가 이해하기로 그냥 구글에서 관리해주는 유저 건강 정보 Database라고 보면 된다. 그래서 구글 계정에서 해당 정보를 지울수도 있다.

![스크린샷 2023-06-17 오후 4.16.41.png](https://github.com/Funncy/Funncy.github.io/assets/22514912/9c067998-7235-4b18-b7e6-17199b318e2e)

나는 모바일에서 가져오게 구현했지만, REST API로 웹에서도 가져올 수 있다.

나는 처음에 단순히 가져오는 기능만 구현하면 끝인줄 알았다…

> 말 그대로 Database니깐 읽는 부분을 만들면 쓰는 부분이 있어야한다.
그래서 타사 만보기(Google FitnessAPI 활용하는)앱을 깔면 내가 쓰는 부분을 작성하지 않아도 데이터가 읽어진다.
이거 때문에 왜 이 폰에서는 잘 읽어지는데 저 폰에서는 데이터가 안가져와지지?? 하면서 엄청 당황해했었다.
> 

안타깝게도 flutter health 라이브러리에서 Google Fitness 에서 데이터를 읽는 부분은 있었지만 데이터를 update 해주는 부분은 구현되어있지 않았다.

> 시간이 여유가 있었다면 PR을 날려볼텐데… 요즘은 Flutter보다 Nextjs로 웹앱 개발이 더 바쁜 핑계로… 나중에 해보자.
> 

## 2-3. Google Fitness API Native 구현

---

타사 앱을 설치하게 해서 걸음수를 업데이트 하는건 아닌것 같고 결국 Android Native코드를 작성해서 걸음수 update 부분을 채워줘야 했다.

생각보다 쉽게 구현할 수 있었다.

[원시 센서 데이터 액세스      |  Google Fit  |  Google for Developers](https://developers.google.com/fit/android/sensors?hl=ko)

```java
private fun subscribe() {
         try {
             val stepsDataType = DataType.TYPE_STEP_COUNT_DELTA

             val fitnessOptions = FitnessOptions.builder()
                     .addDataType(stepsDataType)
                     .build()

             val account = GoogleSignIn.getAccountForExtension(this, fitnessOptions)

             Fitness.getRecordingClient(this, account)
                     .subscribe(DataType.TYPE_STEP_COUNT_DELTA)
                     .addOnSuccessListener {
                         println("Successfully subscribed!")
                     }
                     .addOnFailureListener { e ->
                         println( "There was a problem subscribing.")
                     }
        } catch (e: Exception) {
            false
        }
}
```

위 코드를 참고해서 구현하면 된다.

걸음수 가져오는 로직도 추가로 구현했다.

그 이유는 flutter health에서 가져오는 데이터는 조작이 가능했다.

> Google Fitness 앱을 설치해서 오늘 걸음수를 추가하면 해당 걸음이 유저가 입력한 걸음인지 검증을 할 방법이 없었다.
이걸 해결하기 위해 찾아보니 datasource라는 필드가 있었고, 유저가 입력하면 해당 필드가 유저 입력으로 표기된다. 
flutter health 라이브러리는 해당 필드를 거를 수 있는 기능을 제공하지 않았고, 나는 순수하게 sensor 에서 누적된 데이터만 가져오고 싶었다. 그러면? 결국 자체 개발
> 

```java
private fun getTotalStepsInInterval(call: MethodCall, result: MethodChannel.Result) {
        val start = call.argument<Long>("startTime")!!
        val end = call.argument<Long>("endTime")!!

        val stepsDataType = DataType.TYPE_STEP_COUNT_DELTA
        val aggregatedDataType = DataType.AGGREGATE_STEP_COUNT_DELTA

        val fitnessOptions = FitnessOptions.builder()
                .addDataType(stepsDataType)
                .addDataType(aggregatedDataType)
                .build()

        val account = GoogleSignIn.getAccountForExtension(this, fitnessOptions)

        val ds = DataSource.Builder()
                .setAppPackageName("com.google.android.gms")
                .setDataType(stepsDataType)
                .setType(DataSource.TYPE_DERIVED)
                .setStreamName("estimated_steps")
                .build()

        val duration = (end - start).toInt()

        val dataReadRequest = DataReadRequest.Builder()
                .aggregate(ds)
                .bucketByTime(duration, TimeUnit.MILLISECONDS)
                .setTimeRange(start, end, TimeUnit.MILLISECONDS)
                .build()

        Fitness.getHistoryClient(this, account)
            .readData(dataReadRequest)
            .addOnSuccessListener { response ->

                    for (bucket in response.buckets) {
                        val dp = bucket.dataSets.firstOrNull()?.dataPoints?.firstOrNull()
                        if (dp != null) {
													  //데이터 소스 확인해서 걸러버리자~~
                            val sourceName = dp.originalDataSource.streamName
                            if(sourceName.contains("user_input")) {
                                result.success(-2)
                            } else {
                                val count = dp.getValue(aggregatedDataType.fields[0])

                                val startTime = dp.getStartTime(TimeUnit.MILLISECONDS)
                                val startDate = Date(startTime)
                                val endDate = Date(dp.getEndTime(TimeUnit.MILLISECONDS))
                                Log.i("FLUTTER_HEALTH::SUCCESS", "returning $sourceName $count steps for $startDate - $endDate")
                                val steps = count.asInt()
                                result.success(steps)
                            }

                        } else {
                            val startDay = Date(start)
                            val endDay = Date(end)
                            Log.i("FLUTTER_HEALTH::ERROR", "no steps for $startDay - $endDay")

                            result.success(0)
                        }
                    }

            }
            .addOnFailureListener { e ->
                result.error("ERROR", "Failed to get steps: ${e.message}", null)
            }
    }
```

위 처럼 데이터를 가져오는 부분에서 user_input으로 데이터소스가 되어있으면 그냥 -2를 반환하게 했다.

-2를 받은 flutter에서 나머지 처리를 하도록 구현했다.

> 당연히 위 코드에서 정상 구동하려면 flutter health 라이브러리 설명에도 나와있지만
Google OAuth API 키를 발급 받아야한다. 해당 부분은 라이브러리 설명을 참고하길…
> 

Google OAuth API키를 생성해서 하루 100명까지는 테스트로 가입할 수 있었다.
그래서 테스트 해본 결과 아주 잘 동작했다. 

하지만 위에 언급했듯이 OAuth API 심사를 위해 메일을 서로 주고 받으며… (심지어 메일도 평균 3일이 지나야 답변해준다.) 개발 끝나고 거의 한주가 지나서 Google Fitness 없이 아예 STEP_COUNTER 센서를 통한 자체 개발로 방향을 틀었다.

## 2-4. 자체 개발 설계

---

사실 센서 데이터는 안드로이드에서 주기 때문에 해당 센서 데이터를 어떻게 활용할지 계획을 세워야했다.

일단 STEP_COUNTER 센서 데이터의 주의사항을 다시 살펴보면

1. 핸드폰이 부팅된 이후 누적된 걸음수이다. (몇날 몇일인지 알 수없음)
2. 재부팅시 센서 데이터 초기화 된다.

그리고 우리가 만들려는 만보기 앱은 앱이 죽어있어도 정상적으로 걸음수를 측정해야한다.

하지만 위 2가지 제약사항과 우리 앱의 생명주기랑 만나면 고려해야할 사항들이 있다.

고려해야하는 상황

1. 앱이 첫 실행되었을때 디바이스에 누적된 걸음수(몇일이 쌓여있는지 모름) 말고 오늘치 걸음수를 가쟈와야함.
    - 이건 그냥 초기값을 저장해서 0부터 시작하게 처리하면 된다. (다른 만보기 앱들도 그렇게 함)
    - 데이터 저장은 `SharedPreference`로 하기로 했다.
2. 핸드폰을 재부팅해도 걸음수를 기억해서 누적해야함.
    - 걸음수가 update 될때마다 현재의 걸음수를 저장해서 재부팅시에 해당 값을 셋팅해주면 된다.
    - 핸드폰이 강제 종료되면 onDestory 같은 생명주기 함수가 안호출될 수 있다.
    - 그러므로 생명주기 함수가 아닌 재부팅 된 신호를 확인해서 걸음수 값을 계산 해야한다. 즉, `Broadcast Receiver` 가 필요하다.
3. 자정이 넘어가면 바로 걸음수가 0이 되어야 함.
    - 단순히 자정에 특정 이벤트를 실행하려면 `alarmManager` 를 활용하면 된다.
    - 다음날이 되면 걸음수를 0부터 새로 카운팅 해야하는데, 문제는 센서값이 몇일인지 모르게 계속 누적된다. 예를들어, 우리 앱을 점심에 키고 이때 까지 걸음수가 1,000보 였다가 다음날 점심에 켰을때 5,000보이면 다음날 몇보인지 계산이 불가능하다. (전날 몇보까지 걸었는지 모른다. 앱이 죽어있을때 카운팅하지 않으면….)
    - 앱이 죽어있어도 계속해서 카운팅을 해야한다. 즉 `Foreground Service`가 필요하다.

## 2-5. 안드로이드 구조와 만보기 설계

---

참고로 나는 안드로이드를 찍먹만 해봤기 때문에 안드로이드 전문가가 아니므로 틀린 정보가 있을 수 있다.

안드로이드에는 4대 컴포넌트가 있다.

- Activty : 화면을 담당
    
    > 플러터에서는 화면을 플러터가 그리기 떄문에 `FlutterActivity` 를 상속한 `MainActivity` 에서 `method channel`만 잘 연결해주자.
    > 
- Service : 백그라운드 동작을 담당
    
    > 안드로이드 Oreo 버전부터는 백그라운드에서 작업을 지속적으로 하려면 유저에게 인지 시켜주기 위해서 Notification으로 알림영역에 표시해야한다. (음악 재생앱들 처럼)
    이런 service를 `foreground service`라고 한다.
    > 
- Broadcast Receiver : 이벤트 수신 담당
    
    > 핸드폰 재부팅시에 `BOOT_COMPLETED` 라는 이벤트가 발생하고 해당 이벤트를 수신해서 로직을 처리할 수 있다.
    > 
    
    > alaramManager도 특정 시간에 Broadcast 이벤트를 발생시켜 해당 이벤트를 수신할 Broadcast Receiver를 등록해야한다.
    > 
- Contents Provider : 리소스 관리 담당
    
    > 이건 여기서 쓸 일이 없었다.
    > 
    

위 내용을 기반으로 설계를 해보면

1. 앱이 최초 실행됬을때 걸음수를 0으로 설정한다.
2. 앱이 실행되면 foreground service를 실행해 sensor event를 listen한 다음 걸음수를 update 해준다.
3. foregorund service가 실행될때 alarm을 등록해서 reset receiver를 호출하게 한다.
4. `BOOT_COMPLETED` 를 수신받는 BootCompletedReceiver를 만들어 걸음수를 누적할 수 있게 한다.
5. ResetStepCountReceiver를 만들어 alarm 즉, 자정이 넘었을때 걸음수를 초기화하는 로직을 돌린다.
    
    <aside>
    ⚠️ 주의 사항으로는 알람매니저는 핸드폰이 꺼져있을 때 실행되지 않는다. 
    즉, 자정이 넘어가는 시간에 폰이 죽어있다면 실행되지 않는다.
    해당 부분은 추후에 걸음수 계산 로직에서 처리하도록 하자.
    
    </aside>
    

그리고 위 내용을 기반으로 걸음수 계산 로직을 설계해보자.

1. 앱이 첫 실행시 현재까지의 stepCount는 initialValue로 잡아서 차이값으로 계산하게 하자.
    - stepCount = sensorData - initialValue (0부터 시작하기 위함, 그러지 않으면 오늘 데이터를 알 수 없음)
2. 핸드폰이 재시작 되면 기존까지의 걸음수 데이터를 누적해주자.
    - stepCount = prevStepCount + sensorData - initialValue
    - 위 식에서 sensorData는 이제 다시 0부터 시작이므로 initialValue는 0이 된다.
    - 위 식을 조금 정리하면 
    stepCount = sensorData - (-prevStepCount) 로 변경 할 수 있으니 initialValue에 이전까지의 걸음수를 음수로 넣어주면 된다.
3. 자정니 넘어가면 걸음수를 0으로 초기화한다.
    - stepCount = sensorData - (initialValue = prevStepCount)
4. 추가적으로 해당 데이터가 당일 데이터인지 검증하기 위해 센서 데이터를 가져온 날짜도 저장해서 보내주자.

자 이제 코드를 살펴보자.

## 2-6. 만보기 자체 개발

---

### 2-6-1. Foreground Service 띄우기

---

먼저 foregorund Service를 띄위기 위한 설정부터 하자.

```java
<uses-permission android:name="android.permission.FOREGROUND_SERVICE" />
```

`AndroidManifest.xml` 에 위 권한을 등록해주자. 

그 후에 Service 코드를 작성해주자. 

```java

class StepCountService : Service( {

	companion object {
        val ALARM_CHANNEL_NAME = "com.example.pedometer"
    }

	override fun onCreate() {
        super.onCreate()
  }

	override fun onDestroy() {
        super.onDestroy()
    }

	
	override fun onStartCommand(intent: Intent?, flags: Int, startId: Int): Int {

        val notificationManager = getSystemService(Context.NOTIFICATION_SERVICE) as NotificationManager

        if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.O) {
            val notificationChannel = NotificationChannel(
                ALARM_CHANNEL_NAME,
                "만보기 걸음수",
                NotificationManager.IMPORTANCE_HIGH
            )
            notificationManager.createNotificationChannel(notificationChannel)
        }

        var builder: NotificationCompat.Builder
        if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.O) {
            builder = NotificationCompat.Builder(this, ALARM_CHANNEL_NAME)
        } else {
            builder = NotificationCompat.Builder(this)
        }

        val notificationIntent = Intent(this, MainActivity::class.java)
        val pendingIntent = PendingIntent.getActivity(this, 0, notificationIntent, PendingIntent.FLAG_IMMUTABLE)

				//추후에 걸음수 데이터 가져올 예정
        val stepCount = 0;

        val notification = builder
            .setContentTitle("$stepCount 걸음")
            .setContentText("운동일기 쓰고 포인트 적립 >")
            .setSmallIcon(R.drawable.ic_noti)
            .setSound(null) // 알림 소리 재생하지 않음
            .setOnlyAlertOnce(true) // 알림 소리를 한 번만 재생
            .setContentIntent(pendingIntent)
            .setOngoing(true)  // 알림을 종료할 수 없도록 설정
            .setPriority(NotificationCompat.PRIORITY_HIGH)
            .build()

        startForeground(1, notification)

				
        return Service.START_STICKY
    }
}

```

위와 같이 Service를 만들면 foreground Service로 Notification이 동작하게 된다.

### 2-6-2. 걸음수 센서 연동하기

---

만들어놓은 foreground service에 `SensorEventListener` 를 연결하여 걸음수 데이터를 가져오자.

```java
class StepCountService : Service(), SensorEventListener {
	...
	private var sensorManager: SensorManager? = null
  private var stepCountSensor: Sensor? = null

	override fun onCreate() {
        super.onCreate()

        // SensorManager 초기화
        sensorManager = getSystemService(Context.SENSOR_SERVICE) as SensorManager
        stepCountSensor = sensorManager?.getDefaultSensor(Sensor.TYPE_STEP_COUNTER)

        // Sensor 등록
        stepCountSensor?.let {
            sensorManager?.registerListener(this, it, SensorManager.SENSOR_DELAY_NORMAL)
        }
    }

	override fun onDestroy() {
        super.onDestroy()
        // Sensor 등록 해제
        sensorManager?.unregisterListener(this)
    }

	override fun onSensorChanged(event: SensorEvent) {
        if (event.sensor.type == Sensor.TYPE_STEP_COUNTER) {
            val stepCount = event.values[0].toInt()

            // 걸음수 업데이트 및 Notification 업데이트 로직 작성 예정
        }
    }

	override fun onAccuracyChanged(sensor: Sensor?, accuracy: Int) {
        // 센서 정확도 변경 시 처리 
    }
```

이제 핸드폰을 들고 걸어다니면 `onSensorChanged` 함수에 stepCount 데이터가 들어온다. 

물론 내가 걸은숫자가 아니라 엄청 큰 값이 들어온다. 

### 2-6-3. SharedPreference로 걸음수 계산하기

---

`SharedPreference` 를 활용하여 현재 걸음수를 계산해보자.

먼저 앱 첫 실행시 0 부터 계산되도록 하자.

```java
class StepCountService : Service(), SensorEventListener {
	...
	private lateinit var sharedPreferences: SharedPreferences;
	//Key값들
  private val STEP_COUNT_STORAGE_KEY : String = "StepCounterStorage";
  private val STEP_COUNT_INITIAL_VALUE_KEY : String = "StepInitialValue";
  private val STEP_COUNT_KEY : String = "StepCount";

	override fun onCreate() {
        super.onCreate()
				...
				
				//저장소 가져오기
        sharedPreferences = getSharedPreferences(STEP_COUNT_STORAGE_KEY, Context.MODE_PRIVATE)
  
    }	

	private fun getCalculatedStepCount(sensorCount : Int) : Int {
       
        val editor = sharedPreferences.edit()
			  //1. 초기값 가져오기 (값이 없으면 -1로 설정)
        var initialValue = sharedPreferences.getInt(STEP_COUNT_INITIAL_VALUE_KEY, -1);

        //앱 첫실행의 경우 초기화 진행
        if(initialValue == -1) {
            Log.d("StepCount", "onReceive initialValue 초기화 진행 initialValue=$sensorCount")
            editor.putInt(STEP_COUNT_INITIAL_VALUE_KEY, sensorCount)
            initialValue = sensorCount
        }

        val currentStepCount =  sensorCount - initialValue;
        Log.d("StepCount" , "onReceive initialValue= ${initialValue}  sensorValue=${sensorCount} currentStepCount=${currentStepCount}" );
	      //현재 걸음수 저장
				editor.putInt(STEP_COUNT_KEY, currentStepCount)
        editor.apply()
        return currentStepCount;
    }

	override fun onSensorChanged(event: SensorEvent) {
        if (event.sensor.type == Sensor.TYPE_STEP_COUNTER) {
            val stepCount = event.values[0].toInt()

            val calculatedStep = getCalculatedStepCount(stepCount)
            // 걸음수 업데이트 및 Notification 업데이트 로직
						updateStepCount(calculatedStep)
        }
    }

private fun updateStepCount(stepCount: Int) {
        val notificationManager = getSystemService(Context.NOTIFICATION_SERVICE) as NotificationManager
        val builder: NotificationCompat.Builder
        if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.O) {
            val notificationChannel = NotificationChannel(
                ALARM_CHANNEL_NAME,
                "만보기 걸음수",
                NotificationManager.IMPORTANCE_HIGH
            )
            notificationManager.createNotificationChannel(notificationChannel)
            builder = NotificationCompat.Builder(this, ALARM_CHANNEL_NAME)
        } else {
            builder = NotificationCompat.Builder(this)
        }

        val notificationIntent = Intent(this, MainActivity::class.java)
        val pendingIntent = PendingIntent.getActivity(this, 0, notificationIntent, PendingIntent.FLAG_IMMUTABLE)

        val notification = builder
            .setContentTitle("$stepCount 걸음")
            .setContentText("운동일기 쓰고 포인트 적립 >")
            .setSmallIcon(R.drawable.ic_noti)
            .setSound(null) // 알림 소리 재생하지 않음
            .setOnlyAlertOnce(true) // 알림 소리를 한 번만 재생
            .setContentIntent(pendingIntent)
            .setOngoing(true)  // 알림을 종료할 수 없도록 설정
            .setPriority(NotificationCompat.PRIORITY_HIGH)
            .build()

        notificationManager.notify(1, notification)
    }

...
}
```

이제 앱을 재설치 해보면 0부터 시작하는 것을 확인 할 수 있다.

그리고 먼저 AlarmManager를 등록해 자정에 0부터 시작하는것을 개발하기 전에

AlarmManager는 앱이 죽어있을때 실행할 수 없다고 위에서 이야기했었다. 그럼 자정에 폰을 끄고 다음날 키면? 그래도 0으로 초기화 되어있어야 한다. 

그걸 위해 위 로직에 날짜 검증 로직을 추가해보자.

```java
class StepCountService : Service(), SensorEventListener {
	...
  private val STEP_COUNT_DATE_KEY : String = "StepDate"
	...

	private fun getCalculatedStepCount(sensorCount : Int) : Int {
        //1. 초기값 가져오기

        resetStepsIfNeeded(sensorCount)
	      
				...    
	}

	@SuppressLint("SimpleDateFormat")
    private fun getTodayString(): String {
        return SimpleDateFormat("yyyy-MM-dd").format(Date())
   }

   private fun resetStepsIfNeeded(sensorCount: Int) {
        val editor = sharedPreferences.edit()
        var lastSavedDate = sharedPreferences.getString(STEP_COUNT_DATE_KEY, "")
        val today = getTodayString()
        Log.d("StepCount", "onReceive date has changed. resetting steps initialValue=$sensorCount $lastSavedDate $today")

        if(lastSavedDate == "") {
            editor.putString(STEP_COUNT_DATE_KEY, today);
        }else if (lastSavedDate != today) {
            // 날짜가 변경된 경우 초기화 작업
            Log.d("StepCount", "onReceive 초기화!!!! . resetting steps initialValue=$sensorCount $lastSavedDate $today")
            editor.putString(STEP_COUNT_DATE_KEY, today)
            editor.putInt(STEP_COUNT_INITIAL_VALUE_KEY, sensorCount)
            editor.putInt(STEP_COUNT_KEY, 0)
        }

        editor.apply()
   }

```

오늘 날짜를 저장해 해당 날짜가 아니면 값을 초기화 하도록 했다.

이제 기본적인 걸음수를 보여주는 로직은 구현이 완료되었다.

예외 상황들을 위한 로직 처리를 해보자.

### 2-6-4. AlarmManager - 자정 처리

---

위 로직 처리를 위해서는 BroadCast Receiver의 도움도 필요하다. 

먼저 알람 등록을 진행하자.

```java
class StepCountService : Service(), SensorEventListener {

	...

	override fun onCreate() {
        super.onCreate()
				...
		     // 알람 등록
        val alarmManager = getSystemService(Context.ALARM_SERVICE) as AlarmManager
				//ResetStepCountReceiver 호출하도록 설정
        val alarmIntent = PendingIntent.getBroadcast(this, 0, Intent(this, ResetStepCountReceiver::class.java), PendingIntent.FLAG_IMMUTABLE)
	      //시간은 자정에!
				val calendar: Calendar = Calendar.getInstance().apply {
            timeInMillis = System.currentTimeMillis()
            set(Calendar.HOUR_OF_DAY, 24)
            set(Calendar.MINUTE, 0)
            set(Calendar.SECOND, 0)
        }
				//반복 설정
        alarmManager.setRepeating(AlarmManager.RTC, calendar.timeInMillis, AlarmManager.INTERVAL_DAY, alarmIntent)
    }

	...
}
```

이제 위에서 호출하는 `ResetStepCountReceiver` 를 살펴보면

```java
class ResetStepCountReceiver : BroadcastReceiver() {
    private val STEP_COUNT_STORAGE_KEY : String = "StepCounterStorage";
    private val STEP_COUNT_INITIAL_VALUE_KEY : String = "StepInitialValue";
    private val STEP_COUNT_KEY : String = "StepCount";
    private val STEP_COUNT_DATE_KEY : String = "StepDate";

    override fun onReceive(context: Context, intent: Intent) {
        val sharedPreferences = context.getSharedPreferences(STEP_COUNT_STORAGE_KEY, Context.MODE_PRIVATE)

        var stepCount = sharedPreferences.getInt(STEP_COUNT_KEY, 0);
        var stepInitialValue = sharedPreferences.getInt(STEP_COUNT_INITIAL_VALUE_KEY, 0);
        Log.d("StepCount" , "RESET!!! onReceive stepCount = " + stepCount + " stepInitialValue=" + stepInitialValue);

        //현재까지의 걸음수를 initialValue에 stepCount 누적
        val editor = sharedPreferences.edit()
				//stepCount = sensorData - intialValue의 수식이니. 0부터 시작하려면
				//sensorData = stepCount(이전까지의 걸음수) + initialValue가 된다. 
        editor.putInt(STEP_COUNT_INITIAL_VALUE_KEY, stepCount + stepInitialValue)
        editor.putInt(STEP_COUNT_KEY, 0)
				//날짜도 바꼈으니 StepDate도 바꿔주자.
        editor.putString(STEP_COUNT_DATE_KEY, SimpleDateFormat("yyyy-MM-dd").format(Date()))
        Log.d("StepCount" , "RESET !!  update  prevStepCount=" + stepCount);
        editor.apply()

				//Notification 걸음수 0으로 업데이트
	      updateNotification();
    }
}
```

위 과정을 테스트할때는 알람 호출시간을 변경해서 테스트하는걸 추천한다.

### 2-6-5. BootComplete Receiver

---

핸드폰 재부팅의 상황을 대응하기 위해 `BootCompletedReceiver` 를 만들어보자.

`AndroidManifest.xml` 에 다음과 같이 등록하자.

```java
<receiver
        android:name=".BootCompletedReceiver"
        android:enabled="true"
        android:exported="false" >
        <intent-filter >
            <action android:name="android.intent.action.BOOT_COMPLETED"/>
        </intent-filter>
</receiver>
```

이제 intent-filter를 통한 설정으로 핸드폰 재실행시 이벤트를 수신하게 되고, BootCompletedReceiver의 onReceive 함수가 실행된다.

```java
package com.heroines.heroines

import android.app.AlarmManager
import android.app.PendingIntent
import android.content.BroadcastReceiver
import android.content.Context
import android.content.Intent
import android.os.Build
import android.util.Log
import java.util.*

class BootCompletedReceiver : BroadcastReceiver() {
    private val STEP_COUNT_STORAGE_KEY : String = "StepCounterStorage";
    private val STEP_COUNT_INITIAL_VALUE_KEY : String = "StepInitialValue";
    private val STEP_COUNT_KEY : String = "StepCount";
    private val STEP_COUNT_DATE_KEY : String = "StepDate";

    override fun onReceive(context: Context, intent: Intent) {
        if (intent.action == Intent.ACTION_BOOT_COMPLETED) {
            // 재부팅되면 서비스도 다시 시작해줘야 한다.
            restartForegroundService();

            // 재부팅 되면 알람매니저가 죽어버린다. 즉, 다시 설저해줘야 한다.
            resetAlarmManager();

            val sharedPreferences = context.getSharedPreferences(STEP_COUNT_STORAGE_KEY, Context.MODE_PRIVATE)
            val editor = sharedPreferences.edit()

            val prevStepCount = sharedPreferences.getInt(STEP_COUNT_KEY, 0)
            //이전까지의 걸음수 음수로 저장
						//stepCount = sensorData(0부터 시작) - (-prevStepCount)
            editor.putInt(STEP_COUNT_INITIAL_VALUE_KEY, -prevStepCount)
            editor.apply()
            Log.d("BootCompletedReceiver" , "onReceive prevStepCount= ${prevStepCount}" );
        }
    }
}
```

자 이제 마지막 Method Channel만 잘 연결하자.

### 2-6-6. Method Channel 연결

---

걸음수를 반환해줄 때는 날짜도 같이 반환해주자.

만약에 유저가 만보를 달성해놓은 상태에서 앱의 설정 페이지를 들어가 Service를 임의로 강제로 종료할 경우를 대비해야한다.

위와 같은 케이스는 다음날에도 다다음날에도 만보 데이터가 계속 반환되니, 이게 오늘 생성된 데이터인지 확인하기 위해 위에서 걸음수 생성할때 같이 저장한 날짜 데이터도 반환하게 해주자.

```java
class MainActivity: FlutterActivity() {
    private val CHANNEL = "com.example.pedometer/pedometer"

    private var stepCount : Int = 0;
    private lateinit var sharedPreferences: SharedPreferences;
    private val STEP_COUNT_STORAGE_KEY : String = "StepCounterStorage";
    private val STEP_COUNT_INITIAL_VALUE_KEY : String = "StepInitialValue";
    private val STEP_COUNT_PREV_VALUE_KEY : String = "StepCountPrevValue";
    private val STEP_COUNT_KEY : String = "StepCount";
    private val STEP_COUNT_DATE_KEY : String = "StepDate";

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)

        sharedPreferences = getSharedPreferences(STEP_COUNT_STORAGE_KEY, Context.MODE_PRIVATE)
    }

     override fun configureFlutterEngine(flutterEngine: FlutterEngine) {
        super.configureFlutterEngine(flutterEngine)

        MethodChannel(flutterEngine.dartExecutor.binaryMessenger, CHANNEL).setMethodCallHandler { call, result ->
            try {
                if (call.method == "startForegroundService") {
                    startForegroundService()
                } else if(call.method == "getStep") {
                    result.success(getStepCountWithTime());
                } else {
                    result.notImplemented()
                }
            } catch (err : Exception) {
                Log.d("MainActivity","error. 발생!!!! : " + err);
            }

        }
    }

    private fun startForegroundService() {
        try {
            val i = Intent(this, StepCountService::class.java)
            startForegroundService(i)
        }catch (err: Exception) {
            println(err);
        }

    }

		//걸음수와 해당 데이터 발생 날짜 반환
    private fun getStepCountWithTime(): Map<String, Any> {
        val stepCount = sharedPreferences.getInt(STEP_COUNT_KEY, 0)
        val currentDate = sharedPreferences.getString(STEP_COUNT_DATE_KEY, "") ?: ""

        val stepCountMap = mapOf(
            "stepCount" to stepCount,
            "currentDate" to currentDate
        )

        return stepCountMap
    }

}
```

위와 같이 마무리하면 다음과 같이 정상적으로 동작한다.

> 생각보다 생략한 부분이 많이 있다. Service 생성 및 AndroidManifest.xml 등록, ACTIVITY_RECOGNITION 권한 등록 등… 
위와 같은 부분들은 검색을 통해 하나씩 해결하길 바란다.
> 

![Screenshot_20230618-022830_One UI Home.jpg](https://github.com/Funncy/Funncy.github.io/assets/22514912/58378d8c-341e-4f04-84ff-bb63063542ac)

# 3. 마무리

---

이번에 만보기 피쳐 개발이 생각보다 많이 지연되었다. 그 이유는 위에 나와있듯이 처음부터 자체 개발 했으면 더 빨랐을 수 있지만 한번 돌아서 왔기 때문이다. (심지어 개발 다 끝나고 OAuth 인증 팀과 메일을 주고 받은 시간까지 더해지니 엄청 길어졌다…)

위 과정을 통해 회고를 해보면, 어떤 기능을 개발할때 로직적인 부부만 생각하고 무턱대고 개발하기 보다 여러 부분을 고려하며 (예를들면 심사, 인증 이라던지… 정책적인 부분들도) 조사하고 설계해서 개발을 시작해야겠다.

미리 OAuth API 인증에 대해서 알아봤더라면 이런 일이 발생하지 않았을것이다.

뭐 돌고 돌아 그래도 잘 마무리했고, 현재 유저들이 잘 사용하고있다. (정말 잘 사용하고 있는지는 한주 뒤에 log를 통해 검증할 예정이다. 그 이후 후속 만보기 기능 팔로우업 테스크 진행 예정)

이번 과정을 통해 Android와 java을 조금이나마 공부하고 맛 볼 수 있었다. 
개인적인 소견으로는 너무 얕게 맛만봐서 잘 모르겠지만, 나는 그냥 Flutter가 더 편한것 같다. (특히 화면 만드는건) 

그리고 flutter의 핫리로드가 너무나 소중했다.