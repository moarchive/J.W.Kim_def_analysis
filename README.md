![](https://velog.velcdn.com/images/baseballdev/post/547869f1-21d4-4cbd-8474-4f9e185a8753/image.jpg)


안녕하세요. 야구를 볼 때 마다 드는 생각 중 하나는 야구에선 수비의 중요도가 매우 크다는 것입니다. 성공하면 실점을 막기도하지만, 실패하면 실점을 내주는, 어찌보면 ‘성공하는 것’보다 ‘실패하지 않는 것’이 더 중요한 분야 중 하나죠. 그래서 오늘은 ‘어떻게 하면 수비를 더 잘할 수 있을까’, ‘어떤 부분을 고치면 더 높은 수비 성공률을 기록할 수 있을까?’란 물음에서 시작한 ‘수비 성공률을 높이는 솔루션’을 만들어보았습니다.

분석 대상은 수비의 꽃인 유격수, 그 중 현재 우리나라에서 가장 주목받는 유격수 중 한 명인 김주원 선수로 정했습니다. 분석을 위해 직접 김주원 선수의 2023시즌 모든 수비를 데이터화 했으며, 어떤 것들이 실책과 연관있는지 알아본 결과를 토대로 간단한 훈련 솔루션을 만들어봤습니다. 이런 것들이 실패와 이렇게 연관되어있으니, 이 부분을 더 집중적으로 훈련하면 될 것 같다는 의미를 담았어요.

### 사용한 기술

- Python
- Pandas
- Seaborn
- matplotlib

### 데이터 소개

데이터 갯수는 총 588개로, 날짜와 상대타자 등 기본적인 정보부터, 공을 잡은 후 던질 때까지 걸린 시간, 포구를 할 때 공의 위치 등 세부적인 정보들을 포함하고 있습니다. 

### 데이터에 포함된 컬럼

- Position : 수비 당시 포지션
- Date : 날짜
- Team : 상대팀
- Inning : 수비 당시 이닝
- Batter : 상대타자
- Batter_hand : 상대타자의 좌타/우타여부
- Start_area : 수비 시작 시 위치
- Catch_area : 공을 포구한 위치
- Catch_pos : 포구 당시 공의 높이
- Bounds : 공의 출발부터 포구까지 바운드 수
- Backhand : 백핸드 포구 여부
- Throw_area : 송구한 위치
- Throw_to : 송구를 받는 동료 수비수
- Throw_direction : 송구를 받는 수비수 기준으로 송구가 향한 방향
- Throw_pos : 송구를 받는 수비수 기준으로 송구가 도착한 당시의 공의 높이
- Throw_time : 포구를 한 시점부터 송구가 손을 떠난 시점까지의 시간
- Catch_success : 포구의 성공/실패 여부
- Throw_success : 송구의 성공/실패 여부
- Situation : 수비의 결과(공식기록)
- Note_1 : 비고 1, 수비 이벤트 관련 비고
- Note_2 : 비고 2, 수비 이벤트 관련 비고

## 포구 실패 분석

### 상대팀별

포구 수비의 성공,실패 여부는 기록상 실책이 아닌, ‘한 번에 공을 잘 포구했는지’의 여부입니다. 가장 먼저 상대팀별 기록을 비교했는데요. 그래프에서 가장 눈에 띄는 팀은 키움,LG,기아였습니다.

```python
sns.countplot(x=df['Team'], hue=df['Catch_success'])
```

![](https://velog.velcdn.com/images/baseballdev/post/f756526b-7ed1-42cb-8f4f-e9cc045498ee/image.png)


총 실패 갯수는 키움전이 10개, LG전이 6개, 기아전은 4개지만, 전체 수비 대비 실패 비율은 키움전이 25.6%, 기아전 20%, LG전 13%로 이 세 팀을 상대로 꽤나 고전한 모습을 알 수 있습니다.

세 팀을 상대로 한 포구 실패 전체 사례를 보면 눈에 띄는 한 단어 ‘숏바운드 처리미스’가 있습니다. ‘Note_1’ 데이터에는 수비 이벤트 관련 원인이나 결과를 기록해뒀는데요. 이 세 팀 상대로 한 포구 실패 20개 중 총 8개가 숏바운드 처리미스였습니다. 가장 고전한 세 팀을 상대로 가장 고전했던 이유가 바로 숏바운드 처리미스인 것이죠.

### 숏바운드 처리미스

그래서 숏바운드 처리미스에 대해 더 자세히 알아보고 싶어졌는데요. 포구 실패 총 갯수는 34개인데, 이 중 숏바운드 처리미스는 13개입니다. 3분의 1이 넘을 정도로 많은 비중을 차지하고 있죠.

또 이 중 8개는 홈인 창원에서, 5개는 원정에서 기록했습니다. 2023시즌은 NC가 홈보다 원정에서 2경기 더 치렀는데도 더 많은 포구 실패를 기록했다는 점은 홈 숏바운드 처리에 더 애를 먹고 있다는 것을 알 수 있습니다.

숏바운드 처리미스의 Catch_pos는 무릎이 8개, 발목이 5개였는데요. Catch_pos는 포구 당시 공의 높이를 의미합니다. 확실히 숏바운드여서 그런지 다리 아래, 특히 무릎 쪽에 가장 많은 데이터가 있는 것을 알 수 있습니다.

숏바운드 처리미스에서 알 수 있었던 점은 세 가지입니다. 첫번째는 전체 실패 중 3분의 1 이상이 넘을 정도로 숏바운드 처리미스의 비중이 높은 점, 두번째로 홈에서 더 많이 일어났으며, 세번째로 무릎 쪽 숏바운드는 특히 더 많았다는 점이죠.

### Catch_pos

다음은 앞에서 알아본 Catch_pos에 대해 더 자세히 보겠습니다. 숏바운드 처리미스에서 보여준 흐름은 전체 데이터에서도 비슷한 흐름입니다. 

```python
sns.histplot(x=df['Catch_pos'], hue=df['Catch_success'], multiple='stack')
```

![](https://velog.velcdn.com/images/baseballdev/post/7dc64ae7-e746-43e0-835b-d2e941e2fafe/image.png)

위 그래프는 전체 수비데이터인데요. 가장 많은 공이 발목 쪽으로 왔고, 그 다음으로 가슴, 무릎, 머리위 등이 있습니다. 

```python
sns.histplot(x=df_cf['Catch_pos'], multiple='stack')
```

![](https://velog.velcdn.com/images/baseballdev/post/47c6e192-c243-48e2-b849-43bc00630ee8/image.png)

그런데 실패 사례는 전체 데이터와 아예 다른 흐름을 가져갑니다. 발목에 더해 무릎에서 가장 많은 실패가 나오죠. 무릎과 발목의 개수 차이는 1개지만, 이를 비율로 보면 꽤나 의미있는 데이터가 나옵니다. 발목은 전체 수비 사례 대비 실패율이 4.9%인데 반해 무릎은 전체 사례가 적어 실패 비율이 18.5%인 것이죠. 

```python
cf_bycatchpos = df_cf['Catch_pos'].value_counts()

df_catchposcount = df['Catch_pos'].value_counts()
catchpos = cf_bycatchpos / df_catchposcount * 100

plt.plot(catchpos)
```

![](https://velog.velcdn.com/images/baseballdev/post/1cdedd95-f448-4fee-9ce8-b43e5bd494b6/image.png)

결국 앞서 살펴본 숏바운드 처리 시 무릎이 중요한 점에 더해 일반적인 수비에서도 무릎쪽으로 오는 공을 더 신경쓸 필요가 있습니다. 다른 부위에 비해 가장 두드러지니까요.

### 수비 위치별

마지막으로 알아볼 부분은 수비 위치별 기록입니다. 수비 위치는 수비를 시작할 때 위치인 ‘Start_area’, 공을 포구한 당시 위치인 ‘Catch_area’가 있는데요. 수비 위치를 나누는 기준은 3루수와 2루수 사이 흙 부분인 ‘3유간흙’, 유격수의 기본 위치인 ‘유격수정면’ 등 직관적으로 나타냈습니다.

```python
fig, axes = plt.subplots(nrows=1, ncols=2, figsize=(30,10))

sns.histplot(x=df['Start_area'], hue=df['Catch_success'], multiple='stack', ax=axes[0])
sns.histplot(x=df['Catch_area'], hue=df['Catch_success'], multiple='stack', ax=axes[1])
plt.show()
```

![](https://velog.velcdn.com/images/baseballdev/post/d23ea2fa-40ae-4aa7-b3f2-6015c6a09d23/image.png)


위 그래프는 전체 수비 데이터의 시작 위차와 포구 위치입니다. 시작과 포구 위치에서 가장 많았던 데이터는 역시 유격수정면이었는데요. 포구 위치에선 2루베이스 앞, 3유간흙 등 다른 위치도 높은 비율을 차지했지만, 시작 위치는 유격수정면이 78퍼센트라는 높은 비중을 가져갔습니다.

```python
plt.figure(figsize=(15,10))

sns.histplot(x=df_cf['Start_area'], hue=df_cf['Catch_area'], multiple='stack')
plt.show()
```

![](https://velog.velcdn.com/images/baseballdev/post/1439cfce-b79c-4742-9fa9-65d2984551a8/image.png)

그렇다면 실패 데이터에선 어떨까요. 위 그래프는 실패 데이터의 시작 위치 대비 포구 위치 데이터를 담고 있습니다. 실패 데이터에서도 시작위치가 유격수정면인 경우가 가장 많았지만, 눈여겨봐야할 점은 그것이 아닌 위치 간의 관계입니다.

유격수정면에서 시작한 경우 같은 위치에서 포구한 경우(빨간색)가 상대적으로 매우 큰 비중을 차지하고 있다 할 수 없습니다. 오히려 다른 위치에서 포구한 경우가 더 많죠. 이는 다른 위치에서 시작했을 때도 비슷한 흐름입니다. 실제로 시작한 곳과 포구한 곳이 같은 경우는 실패 데이터 전체 34개 중 14개입니다. 반대로 시작한 곳과 포구한 곳이 달라 실패한 적은 20번이죠. 이는 곧 포구를 위한 위치 이동이 포구의 성공과 실패에 영향을 주고 있는 것을 알 수 있습니다.

## 송구 실패 분석

다음은 송구입니다. 송구 성공과 실패는 포구한 수비수가 공을 잡지 못하거나(잡는 수비수 실책 제외), 공을 잡더라도 베이스에서 발을 떼면서 잡았다면 실패로 기록했으며 나머지 모두 성공으로 기록했습니다.

### 먼송구

송구 실패는 좌우상하로 멀거나 높게 던져 수비가 잡을 수 없는 경우나, 이른 타이밍에 바운드가 일어나도록 짧게 던져 수비가 잡을 수 없는 경우 등이 있었습니다.

이 중 가장 많았던 건 좌우로 멀게 던지는 ‘먼송구’인데요. 송구 실패는 전체가 29개인데, 먼송구는 14개로 거의 절반에 해당했습니다. 수비 성공률을 높이기 위해선 이 먼송구들을 잡아낼 필요가 있어보이죠.

```python
sns.countplot(x=(df_tf['Note_1']), hue=df_tf['Throw_success'])
```

![](https://velog.velcdn.com/images/baseballdev/post/d1cd408d-943a-4d26-820b-f3cd228056b7/image.png)

이 먼송구를 다른 데이터들과 연관지어 살펴보겠습니다.

```python
sns.countplot(x=(df_tf['Note_1']=='먼송구'), hue=df_tf['Backhand'])
```

![](https://velog.velcdn.com/images/baseballdev/post/74e0e286-78b9-49d8-94f4-4cd117edab33/image.png)

먼저 백핸드 포구의 여부인데요. 여기서 False는 먼송구가 아닐 때, True는 먼송구일 때를 의미합니다. 다른 실패사례에선 백핸드가 아닐 때 실패한 경우가 월등히 높았지만, 먼송구에선 백핸드일 경우가 더 많은 먼송구를 불러왔습니다. 

여기에 더해 전체 수비 대비 백핸드의 비율은 19%지만, 송구 실패 데이터에선 백핸드의 비율이 37퍼센트로 급증합니다. 이 결과는 곧, 백핸드 포구는 송구 안정성에 영향을 준다는 것을 의미합니다.

다음은 던진 위치입니다. 

```python
sns.countplot(x=(df_tf['Note_1']=='먼송구'), hue=df_tf['Throw_area'])
```

![](https://velog.velcdn.com/images/baseballdev/post/1ce8cac1-6c59-45cb-88b9-7a853126ce08/image.png)

먼송구(그래프상 True)가 가장 많이 일어난 위치는 3유간인데요. 3유간흙과 3유간잔디(내야 안쪽 잔디)를 합해 총 6회를 기록했습니다. 3유간에서 송구할 경우는 백핸드와도 연결될 수 있습니다. 대부분 수비 시작을 유격수정면에서 했기에 3유간에서 공을 잡는다면 자연스럽게 백핸드일 경우도 많겠죠.

다음은 포구 시 공 위치입니다.

```python
sns.countplot(x=(df_tf['Note_1']=='먼송구'), hue=df_tf['Catch_pos'])
```

![](https://velog.velcdn.com/images/baseballdev/post/4c9bfac2-e8ca-480f-ad25-e6b7c2e36ea0/image.png)

앞서 본 포구 실패와 다르게 송구 실패는 무릎이 아닌 발목에서 더 많이 나온걸 알 수 있습니다. 포구할 때는 무릎 쪽 공이 제일 잡기 어려웠지만, 송구할 때는 무릎보다 더 낮은 발목쪽으로 오는 공이 더 처리하기 까다로웠다는 뜻이겠죠.

## 결론

이렇게 살펴본 데이터를 총정리해보면

포구 실패의 주 원인은 숏바운드 처리미스이며, 이 실수는 무릎 쪽으로 오는 공과 홈 경기장에서 많이 일어났습니다. 또, 무릎 쪽으로 오는 공은 기본 수비에서도 매우 신경써야할 부분이죠. 그리고 이동하면서 포구를 할 때 실수가 많이 일어났으니 이 부분을 신경써야 합니다.

결론적으로, 숏바운드 처리와 **무릎 쪽으로 오는 공에 대한 훈련, 그리고 이동하면서 포구하는 훈련을 늘려 이 부분에 더 적응하고 집중력을 키우면 포구 성공률이 더 올라갈 수 있을 것**이라 생각합니다.

또, 송구 실패의 주된 원인인 좌우로 먼 송구는 백핸드 포구를 할 때 훨씬 더 잘 나오며, 3유간에서 송구할 때, 그리고 발목 위치의 공을 포구할 때 가장 많이 일어났다는 점을 알 수 있었습니다.

이걸 앞으로 훈련에 적용해 **무릎과 더불어 발목으로 오는 공, 여기에 더해 백핸드 수비, 3유간으로 이동해서 던지는 송구까지, 이러한 것들에 더 적응하고 대처하는 능력을 키운다면 송구 성공률 역시 더 올라갈 것**이라고 생각합니다.

무릎으로 오는 공은 당연히 가슴으로 오는 공보다 잡기 어렵고, 이동하면서 포구하는 건 서있는 채로 포구하는 것보다 어렵기에 위 해결책들을 진행해야 한다는 것은 어찌보면 너무나 당연한 얘기입니다. 하지만, 이번 분석의 목적은 ‘당연한 부분을 강화하자’라는 의미에 더해 그동안 약했던 부분은 무엇인지 수치와 그래프로 확인하고 앞으로 어떤 부분을 더 보강해야할지 더 와닿게 체감할 수 있는 솔루션을 만드는 것이었습니다. 여러분께도 좋은 인사이트를 남긴 분석이었으면 좋겠습니다.



사진출처 : ncdinos
