---
layout: post
title: "[데이터] - GPS 분석 및 시각화"
subtitle: "data - gps analysis & visualization "
categories: python
tags: python gpsdata analysis visualization
comments: true
---

이번에 연남도에서 tracker를 통해

실험군들의 GPS 데이터를 수집했다.

이걸 시각적으로 만들어 보자.

## 0. 사전 준비

---

먼저 지도는 folium을 사용하자.

```python
pip install folium
```

그리고 gps 데이터는 excel로 받을 거니 pandas를 설치하자.

```python
pip install pandas
```

gps 거리 계산을 위해 다음 라이브러리를 설치해주자.

```python
pip install haversine
```

## 1. 데이터 준비

---

위치 정보는 개인별 엑셀파일로 넘어왔다.

- 위도
- 경도
- 시간

이 3가지 데이터를 각 파일별로 읽어보자.

```python
import pandas as pd

df = pd.read_excel('~~파일 경로')

lines = df[['위도', '경도']].values[:].tolist()
```

이렇게 1명의 gps데이터를 list형식으로 만들었다.

시간도 가져오려면 위와 같이 작성해보자

```python
times = df['시간'].tolist()
```

## 2. 라인 데이터 시각화

---

folium에 gps데이터들을 시각화 해보자.

### 1) 시작 위치 설정

먼저 카메라 시작 위치를 설정하자.

```python
# 연남동 중앙점 기반 지도 로드
center = [37.561020, 126.923472]
m = folium.Map(location=center, zoom_start=17)
```

### 2) 라인 그리기

```python
# 라인 추가
folium.PolyLine(locations = lines).add_to(m)
```

### 3) 결과

![이미지](https://Funncy.github.io/assets/img/python/2020-10-23-gps-analysis-01.png "gps line")

## 3. 데이터 분석

---

어떤 데이터를 분석할지 고민을 했다.

먼저 전체 이동 시간을 계산해보자.

```python
times = df['시간'].tolist()
total_time = (times[0]-times[-1])
```

excel이 정렬이 잘 되어있다면 위와 같은 코드로 쉽게 전체 걸린 시간을 계산할 수 있다.

여기서 30초 이상 머물런던 장소를 유추해보자.

### 1-1) 30초 이상 머무름

```python
over_times = [] #30초 이상 머무른 시간
over_gps = [] # 장소

for i in range(count-1):
    temp = (times[i]-times[i+1]).seconds
    # 몇초 멈춘걸 의미 있게 볼 것 인가?
    if temp > 30:
        over_gps.append(lines[i])
        over_times.append(temp)

for i in range(len(over_gps)):
    folium.CircleMarkerlocation = over_gps[i],
		tooltip = str(over_times[i])+"초", radius = 10 ,  color='#FF2D00',
    fill_color='#F38168',).add_to(m)
```

### 1-2) 결과

![이미지](https://Funncy.github.io/assets/img/python/2020-10-23-gps-analysis-02.png "gps line & stop")

### 2-1) 근처 POI불러오기

사용자가 이동하면서 체크한 POI 정도를 엑셀로 가져와 화면에 올려보자.

```python
poi = pd.read_excel('~~~파일 경로')

# POI 포인트들
points = poi[['위도', '경도']].values[:].tolist()
name = poi[['장소이름', '성공']].values[:].tolist() # 방문에 성공한 여부도 들어있음

for i in range(len(points)):
    if name[i][1] == 'O':
        color = 'blue'
    elif name[i][1] == 'X':
        color = 'red'
    else:
        color = 'blue'
    folium.Marker(points[i], tooltip=str(name[i][0]),
    popup=f'<i>{name[i][0]}</i>', icon=folium.Icon(color=color,icon='info-sign')
    ).add_to(m)
```

![이미지](https://Funncy.github.io/assets/img/python/2020-10-23-gps-analysis-03.png "gps line & marker")

### 2-2) 의미없는 경로 확인

근처에 POI가 없는 길로 간 곳을 다른 색으로 체크해보자.

```python
# 의미 없는 이동 polyline
meaningless_polyline = []
temp_meaningless = []
for i in range(len(lines)):

    # True면 사용자 이동 경로에서 point를 찾음
    find_polyline = False

    for j in range(len(points)):
        # 현재 gps와 가까운 point(성공)이 20m이내에 있는가?
        # 없다면 meaningless_polyline에 추가하자
        # 연달아 나온것들을 하나의 선으로 간주해서 list로 추가하자
        if name[j][1] == 'O' and haversine(lines[i], points[j], unit='m') <= 50:
            find_polyline = True
            break

    # 현재 사용자 이동 경로에서 point 찾음 => 의미없는 사용자 경로가 아님 or
    # 현재 의미없는 사용자 경로가 끝남
    if find_polyline:
        if len(temp_meaningless) > 0:
            meaningless_polyline.append(temp_meaningless)
            temp_meaningless = []
    else:
        # 사용자 이동 경로에서 point 못찾음 => 의미없는 사용자 이동 경로
        temp_meaningless.append(lines[i])

if len(temp_meaningless) > 0:
    meaningless_polyline.append(temp_meaningless)

# 의미없는 사용자 경로 추가
for i in range(len(meaningless_polyline)):
    # gps 5개 이상이 의미없는 곳으로 가고 있으면 => 의미 없는 경로
    if len(meaningless_polyline[i]) > 10:
        folium.PolyLine(locations = meaningless_polyline[i], color='#ba1818').add_to(m)
```


![이미지](https://Funncy.github.io/assets/img/python/2020-10-23-gps-analysis-04.png "gps meaningless lines")

> 잘 보이게 하려고 line color를 검은색으로 변경하였다.
