# 지하철 2호선 데이터(지리데이터) 시각화 및 분석

**강남역, 잠실역, 구로 디지털 단지 등 회사가 많은 역을 포함한 대표적인 지옥철 “2호선”의 승차인원 현황(시간별, 역별)을 알아보기 위해 분석을 진행했다.**

**tmi지만 나는 실제로 강남역에서 영어학원을 다닐때 학원이 매일 5시 45분에 끝나 항상 지옥철때문에 집에 도착하면 몸이 녹초가 된 경험이 있다. 심지어 인파에  당시 아끼는 모자도 잃어버렸다. 나를 힘들게 한 2호선에 대해 자세히 알아보고자(미래 회사가 2호선역 중 하나일 수도 있으니까!! ) 이번 분석을 진행하게 되었다.**

**2호선 역에 위치한 회사에 다니게 된다면 이번 분석을 반영하여 사람이 몰리는 시간대를 피하여 출퇴근하고 싶다.(유연출퇴근이 가능하다면,,)**

# 개요

**군집분석을 통해 아래와 같은 결과를 얻었다.**

- **주거와 상업 시설, 회사가 비슷하게 분포한 지역→ 출퇴근 시간 모두 탑승객 수가 일정 수준이상인 지역들**

['건대입구', '구로디지털단지', '당산', '도림천', '문래', '방배', '사당', '신당', '신도림', '신설동', '신촌', '영등포구청', '왕십리', '이대', '잠실', '종합운동장', '충정로', '합정', '홍대입구']

- **회사가 많이 분포한 지역→ 저녁에 탑승객이 몰리는 지역(6시 ~7시)**

['강남', '교대', '동대문역사문화공원', '뚝섬', '삼성', '서초', '선릉', '성수', '시청', '역삼', '을지로3가', '을지로4가', '을지로입구', '한양대']

- **주거 지역이 많이 분포한 지역 → 아침에 탑승객이 몰리는 지역(8시 ~9시)**

['강변', '구의', '낙성대', '대림', '봉천', '상왕십리', '서울대입구', '신답', '신대방', '신림', '신정네거리', '아현', '양천구청', '용답', '용두', '잠실나루', '잠실새내']

# 데이터

**서울시 지하철 승하차 데이터**

![image.png](image.png)

**서울시역사마스터 데이터**

![image.png](image%201.png)

**필드설명은 따로 필요없을 듯 하다.** 

**주의해야 할것은 데이터를 훑어보니 서울시 지하철 승하차 데이터의 경우 동대문역사문화공원역, 동대문역사문화공원역(DDP)케이스처럼 같은 역이 다르게 표시된 경우가 있어서 전처리 때 반영해줘야 한다.**

**출처: 서울시 열린 데이터 광장**

# 분석 질문

- **승차인원이 가장 많은 역?**
- **연도별, 월별 승차 인원 차이가 있는지?**
- **시간대별로 승차인원이 많은역 → 만약 유연출퇴근을 한다면 피해야 할 출퇴근 시간을 알 수 있다!**

# 분석

```python
data=pd.read_csv("C:\\Users\\user\\Downloads\\서울시지하철데이터.csv",encoding='cp949')
```

## **사람들이 항상 붐비는 2호선의 승차인원이 가장 많은 역을 알아보자!**

```python
data["month"]=pd.to_datetime(data["사용월"],format='%Y%m').dt.month
data["year"]=pd.to_datetime(data["사용월"],format='%Y%m').dt.year
```

```python
data=data[(data["호선명"]=="2호선") & (data["year"]>2017)]
```

```python
data['지하철역'].nunique()# 실제 역 개수보다 많음 # 중복이 있음을 확인
```

```python
sorted(data['지하철역'].unique())
```

![image.png](image%202.png)

```python
data["지하철역"]= [i[0] for i in data["지하철역"].str.split('(')]
#위도, 경도 데이터와의 통일을 위해 괄호가 있는 역명은 괄호를 없앤다.
```

```python
take_list=[i for i in data.columns if '승차' in i]
```

**여기서 take_list를 통해 승차인원 필드만 추출했다**

```python
set_data=data[['사용월','호선명', '지하철역','작업일자', 'month', 'year']+take_list]
```

```python
set_data.head()
```

![image.png](image%203.png)

```python
set_data["합 계"]=set_data[take_list].sum(axis=1)
```

```python
passengers_data=pd.DataFrame(set_data.groupby(["지하철역"])["합 계"].mean()).reset_index().rename({'합 계':'월평균'},axis=1).sort_values('월평균',ascending=False)
passengers_data.head(10)
```

![image.png](image%204.png)

```python
px.bar(data_frame=passengers_data,x="지하철역",y="월평균",title= "역별 월 평균 승차인원")
```

![image.png](image%205.png)

**역시 강남역이 승차인원이 가장 많다.**

## **연도별로 혹은 월별로 승차인원 추이에 차이가 있는가?**

**2023년은 7월까지 밖에 없어 2022년까지만 반영했다. (2023년을 포함해버리면 잘못된 해석을 할 수 있다.)**

```python
d1=set_data[set_data["year"]<2023].groupby("year")[["합 계"]].sum().reset_index()
d1["year"]=d1["year"].astype("string")
d1
```

![image.png](image%206.png)

```python
px.line(data_frame=d1,x="year",y="합 계")#코로나때 이용률이 확 줄었다.
```

![image.png](image%207.png)

```python
d2=set_data[set_data["year"]<2023].groupby("month")[["합 계"]].sum().reset_index()
d2["month"]=d2["month"].astype("string")
d2
```

![image.png](image%208.png)

```python
px.line(data_frame=d2,x="month",y="합 계")
```

![image.png](image%209.png)

**2월과 9월에 왜 승차인원이 적을까 생각해봤는데** 

1. **2월은 다른달보다 일수가 적다.**
2. **2월과 9월에는 명절이 있을 수 있다.**

**위 두 이유로 지하철 승차인원 합계가 적은 것 같다.**

## **시간대별로 가장 승차인원이 많은 역은?**

```python
top10 = passengers_data.sort_values('월평균', ascending=False).head(10)['지하철역']
Top_passenger_10=set_data[set_data["지하철역"].isin(top10.tolist())].groupby("지하철역")[take_list].mean()
Top_passenger_10.columns=[i[:3] for i in Top_passenger_10.columns ]
Top_passenger_10.head()
```

![image.png](image%2010.png)

```python
plt.rc("font", family = "Malgun Gothic")
sns.set(font="Malgun Gothic", 
rc={"axes.unicode_minus":False}, style='white')
plt.figure(figsize=(40,20))
sns.heatmap(data=Top_passenger_10,annot=True,cmap="YlGn", fmt='n',linewidth=3)
```

![image.png](image%2011.png)

**신림의 출근시간과 강남의 퇴근 시간은 정말 승차인원이 많음을 알 수 있다.**

**히트맵을 보고 지하철역 군집화가 가능하지 않을까? 라는 아이디어를 얻고 군집분석을 진행했다.**

## 지하철역 군집화

```python
from sklearn.cluster import KMeans
from yellowbrick.cluster import KElbowVisualizer

hour_passenger = set_data.groupby('지하철역')[take_list].mean()
hour_passenger.columns = [i[:3] for i in hour_passenger.columns]
hour_passenger_percent = hour_passenger.div(hour_passenger.sum(axis=1), axis=0)#행별 합으로 나눠준다.
```

```python
model = KMeans()
visualizer = KElbowVisualizer(model, k=(1,10))
visualizer.fit(hour_passenger_percent) #군집 3개까지
```

![image.png](image%2012.png)

**군집개수 설정은 3개가 적절함을 알 수 있다.**

```python
model=KMeans(n_clusters=3, random_state=1)
model.fit(hour_passenger_percent)
```

```python
hour_passenger_percent["cluster"]=model.predict(hour_passenger_percent).astype(str)
hour_passenger_percent
```

```python
px.scatter(data_frame=hour_passenger_percent,x="08시", y="18시",color="cluster", width=700, height=600, title=" 시간대별 승차인원 비중 군집화")

#회사가 많은 지역, 주거가 많은 지역으로 나뉜다.
```

![image.png](image%2013.png)

**cluster 0:
['건대입구', '구로디지털단지', '당산', '도림천', '문래', '방배', '사당', '신당', '신도림', '신설동', '신촌', '영등포구청', '왕십리', '이대', '잠실', '종합운동장', '충정로', '합정', '홍대입구']
cluster 1:
['강남', '교대', '동대문역사문화공원', '뚝섬', '삼성', '서초', '선릉', '성수', '시청', '역삼', '을지로3가', '을지로4가', '을지로입구', '한양대']
cluster 2:
['강변', '구의', '낙성대', '대림', '봉천', '상왕십리', '서울대입구', '신답', '신대방', '신림', '신정네거리', '아현', '양천구청', '용답', '용두', '잠실나루', '잠실새내']**

**군집화 결과를 보니 회사가 많은 지역과 주거지역으로 나뉜 느낌이다.** 

```python
dataformap=pd.read_csv("C:\\Users\\user\\Downloads\\서울시역사마스터.csv",encoding='cp949')
dataformap.head(10)
dataformap= dataformap.query('호선 == "2호선"')
dataformap['역사명'] = [i[0] for i in dataformap['역사명'].str.split('(')]
dataformap.rename({'역사명':'지하철역'}, axis=1, inplace=True)
dataformap
dataformap["지하철역"].nunique()# 다행히 개수가 똑같다.
```

```python
hour_passenger1 = hour_passenger.reset_index()[['지하철역','08시','18시']]
dataformap1 = dataformap[['지하철역','위도','경도']]
final_data = pd.merge(hour_passenger1, dataformap1, on='지하철역')
```

```python
model=KMeans(n_clusters=3, random_state=1)
model.fit(hour_passenger_percent)
final_data["cluster"]=model.predict(hour_passenger_percent).astype(str)
final_data.head()
```

```python
import folium
from folium import plugins

mapping = folium.Map(location=[37.5, 127], zoom_start=12)
mapping.add_child(plugins.HeatMap(zip(final_data['위도'], final_data['경도'], final_data['08시'])))

mapping
```

![image.png](image%2014.png)

```python
mapping2 = folium.Map(location=[37.5, 127], zoom_start=12)
mapping2.add_child(plugins.HeatMap(zip(final_data['위도'], final_data['경도'], final_data['18시'])))

mapping2
```

![image.png](image%2015.png)

```python
mapping3 = folium.Map(location=[37.5, 127], zoom_start=12)

for idx in final_data.index:
    lat = final_data.loc[idx, '위도']
    long = final_data.loc[idx, '경도']
    title = final_data.loc[idx, '지하철역']

    if final_data.loc[idx, 'cluster'] == "0":
        color = '#000000'
    elif final_data.loc[idx, 'cluster'] == "1":
        color = '#3A01DF'
    else:
        color = '#DF0101'

    folium.CircleMarker([lat, long]
                        , radius=18
                        ,color = color
                        , fill = color
                        , tooltip = title).add_to(mapping3)
    
    
mapping3
```

![image.png](image%2016.png)

**빨강색: 클러스터2, 주거 지역이 많이 분포한 지역**

**보라색: 클러스터1, 회사가 많이 분포한 지역**

**검정색: 클러스터0, 주거와 상업 시설, 회사가 비슷하게 분포한 지역**

# 분석 결과

**[1] 승차 인원이 가장 많은 역은?**

- 강남, 잠실, 홍대입구, 신림, 구로디지털단지가 top5이다.

**[2] 연도별로 혹은 월별로 승차 인원 추이는?**

- 코로나가 시작된 2020년, 2021년에 인원이 많이 줄었고 2022년도부터 다시 회복 중
- 2월과 9월에 눈에 띄게 승차인원이 다른 달에 비해 적음→ **일수와 명절을 그 이유로 추측한다.**

**[3] 시간대별로 가장 승차인원이 많은 역은?**

- 아침에 비교적 승차 인원이 많은 역과 저녁에 비교적 승차 인원이 많은 역이 있음을 히트맵으로 파악

**[4] 지하철역 군집화 결과**

- **cluster 0: 주거와 상업 시설, 회사가 비슷하게 분포한 지역→ 항상 탑승객 수가 일정 수준이상인 지역들**

['건대입구', '구로디지털단지', '당산', '도림천', '문래', '방배', '사당', '신당', '신도림', '신설동', '신촌', '영등포구청', '왕십리', '이대', '잠실', '종합운동장', '충정로', '합정', '홍대입구']

- **cluster 1: 회사가 많이 분포한 지역→ 저녁에 탑승객이 몰리는 지역(6시 ~7시)**

['강남', '교대', '동대문역사문화공원', '뚝섬', '삼성', '서초', '선릉', '성수', '시청', '역삼', '을지로3가', '을지로4가', '을지로입구', '한양대']

- **cluster 2: 주거 지역이 많이 분포한 지역 → 아침에 탑승객이 몰리는 지역(8시 ~9시)**

['강변', '구의', '낙성대', '대림', '봉천', '상왕십리', '서울대입구', '신답', '신대방', '신림', '신정네거리', '아현', '양천구청', '용답', '용두', '잠실나루', '잠실새내']

# 느낀점

강남역, 잠실역, 선릉역 등 우리나라 많은 회사들이 위치한 곳에서 지옥철을 경험하지 않으려면  11시 출근이 가장 좋음을 알 수 있었다.

처음엔 군집분석이 생각이 없었는데 히트맵을 보고 갑자기 생각이 나서 진행을 했는데 덕분에 분석결과가 다채로워져서 이번에 데이터분석에 대한 약간의 재미와 뿌듯함을 느끼게 되었다.

그리고 2월과 9월의 승차인원이 적은 이유를 일별 승차인원 통계로 더 깊이 있게 분석을 해볼까도 했지만,  원래의 분석 목적은 아니였기 때문에 생략했다. (기회와 시간이 된다면 나중에 해보고 싶긴 하다)
