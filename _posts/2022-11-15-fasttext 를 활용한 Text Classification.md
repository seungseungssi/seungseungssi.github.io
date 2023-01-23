---
title: fasttext 를 활용한 Text Classification
tags: text classification 
---

# 서론
Facebook 이 개발한 word embedding을 간단히??? 실행하는 방법을 정리해두고자 한다. 참고로 fasttext 는 WordNet + subword segmentaion 이라고 기억해두면 간단하다. 

**활용 데이터**는  [문장 유형 분류 AI 경진대회 - DACON](https://dacon.io/competitions/official/236037/overview/description) 에서 다운로드 받은 데이터를 활용하기로 한다. Multilabel 로 되어있는 데이터이다. 

- label1 : 유형 (사실형/추론형/대화형/예측형)
- label2 : 극성 (긍정/부정/미정)
- label3 : 문장시제(과거/현재/미래)
- label4 : 확실성(확실/불확실)

train data set : 16541 rows / test data set : 7090 rows 
(데이터 사이즈는 좀 작아서… deep learning은 택도 없을듯 하다…)

문장을 몇 개 읽어보니 뉴스기사 같은 느낌이 나는 텍스트였는데, 사실 label 이 이게 맞나 싶은거도 많았다. 가령 긍정인지 부정인지 당최 알수 없는 문장들, 그리고 사실인지 추론인지 모르겠는 것들 등등…. (나중에 다른 더 좋은 명확한 데이터를 활용하도록 해야겠다… 크롤링을 하든지…) 

Label3 (문장시제) 가 그나마 명확한 label이라 생각하고(개인적 느낌) 그나마 unbalance 도 심하지 않아서 label 3를 중심으로 하기로 했다. 
(과거 : 8032 / 현재 : 6966/ 미래 : 1643)

또한 파이썬으로 하는 방법도 있지만, Linux 랑 친숙해지고 싶어서 주로 Linux를 사용하려 한다. 


# fasttext 를 통한 분류
## fasttext 설치
Facebook 에서 만든 프로그램을 그대로 이용하는게 편하다. 
fasttext라고 구글링 하면 제일 먼저 나오는 홈페이지의 우상단 github를 클릭하거나 다음의 페이지에 들어가도록 하자

[GitHub - facebookresearch/fastText: Library for fast text representation and classification.](https://github.com/facebookresearch/fastText/#text-classification)

building fastText 항목에 여러 설치 방법이 나오는데 그 중에서도 **make** 방법으로 설치하는것을 추천한다고 해서 그대로 따라했다 . 

Directory는 원하는 아무데나 해도 되는 것 같아서 그냥 프로젝트 폴더 안에다가 설치를 해 주었다.

```
# cd "설치하기 원하는 프로젝트 경로"
$ wget https://github.com/facebookresearch/fastText/archive/v0.9.2.zip
$ unzip v0.9.2.zip
$ cd fastText-0.9.2
$ make
```

(여기서 나는 MAC북에서 wget 이 없다는 오류가 떠서  다음과 같이 wget 부터 설치해 주었다.)

```
$ brew install wget 
```

## fasttext classification - 전처리

Github README 페이지 목차에 “Text Classification” 이란 항목이 있다.  클릭하여 들어가보면, 텍스트 분류를 위해 필요한 여러가지 설명을 볼 수 있다.  가장 중요한 점 두 가지는 다음과 같다.

- 텍스트 파일에는 각 row 마다 label 과 문장이 align 되어 들어가 있어야 한다는 것  
- label 앞에 __label__ 과 같은 prefix 를 달아줘야 한다는것  (즉 “긍정” label 을 “__label__긍정” 과 같이 바꿔줘야 함)

fasttext Classification 을 위해서는 위 상황에 알맞게  형태만 고쳐주면 된다! 
Process 는 다음과 같다.

#### 1. csv 파일이라면 tsv 파일로  변환해주기  **python**

텍스트 자체에 “,”가 있기 때문에 리눅스 상에서 작업시 문제가 생길수 있다.

#### 2. Label, text  컬럼만으로 이뤄지도록 데이터 편집 **python**

위의 1, 2 번은 다음과 같이 파이썬으로 해결했다. 
(어차피 파이썬 쓸꺼면 왜 리눅스로 꾸역꾸역 쓰는지에 대해 현타가 오지만…..)

참고로, header = None 옵션을 주어 , 컬럼명을 날리는것이 중요하다. (리눅스에서 컬럼명을  첫번째  row 데이터로 인식해버리기 때문)

```
train = train[['극성','문장']]
train.to_csv("파일경로/train.tsv", sep = "\t", index = False, encoding = "utf-8")
test.to_csv("파일경로/test.tsv", sep = "\t", index = False, encoding = "utf-8")
```

#### 3. 형태소 분석기 돌리기 (mecab ) **Linux**

한국어의 경우 띄어쓰기가 개판이기 때문에 정규화를 꼭 시켜줘야 한다. 

```
$ cd "파일 있는 상위폴더 경로"
### train 데이터 
$ cut -f1 ./train.tsv > train.label.tsv # 일단 라벨 떼어준다.
$ cut -f2 ./train.tsv | mecab -O wakati > ./train.text.tok.tsv # text 에서만 mecab 먹이기
$ paste ./train.label.tsv ./train.text.tok.tsv  > ./train.tok.tsv
```

#### 4. Shuffling 하기  **Linux**

Training 전에 data 섞는 것은 기본 중 기본

```
$ shuf ./train.tok.tsv > train.tok.shuf.tsv
```


#### 5. “Label”  에 __label__ prefix 달아주기  **sublimetext**

이것은 sublime text 라는 프로그램을 통해 해보도록 하겠다. (현재는 공짜 프로그램)

- 프로그램을 연 후에 Find 탭 > replace 를 누름 
- **find** 항목에 ^(긍정|부정|미정)\t   **replace** 항목에 __label__$1  입력하고 저장 ($1 는 첫번째 단어로 대체하란 뜻) 

## fasttext classification - 모델 활용

#### 6. 다음과 같은 코드로 순식간에 학습이 된다. **Linux**

1초도 안걸리는 듯

```
$ ./fasttext supervised -input train.tok.shuf.tsv -output model.fasttext
```

#### 7. 분류 성능 보기  **Linux**

원래는 test data 로 분류성능을 확인해야하는데, 대회 데이터라 label을 알수가 없어서 ^^;; 편의상 training 데이터으로 성능을 보기로 하자. 다음과 같이 입력하면 된다. 

```
$ ./fasttext test model.fasttext.bin ./train.tok.shuf.tsv
```

그 결과는 다음과 같이 나온다. 

```
N	16541
P@1	0.869
R@1	0.869
```

#### 8.  Predict 하기 **Linux**

Test 데이터로 해보기로 하자 

```
### test 데이터
$ cut -f2 ./test.tsv | mecab -O wakati > test.text.tsv   
$ ./fasttext predict model.fasttext.bin ./test.text.tsv  > test.text.predict.tsv
## 모델 적용
$ cut -f1 ./test.tsv > test.index.tsv #인덱스 저장
$ paste ./test.index.tsv ./test.text.predict.tsv ./test.text.tsv > test.result.tsv   # 인덱스 + 라벨 + mecab 먹인 형태소 test.result.tsv 란 파일로 저장
```

***

**현재로 분류된 예시**

- 손가락 을 까닥 해 강아지 를 멋대로 제어 하 는 맛 이 묘한 쾌감 을 준다는 것 이 다 . 
- 제주 흑 돼지 등 요리 는 1 만 5000 원 에서 2 만 원 이하 , 홍어 세트 는 3 만 원 이 다 .
- 다만 배달 앱 시장 의 경쟁 체제 가 구축 될 때 까지 수수료 인상 이나 수수료 할인 축소 등 소 상공 인 피해 로 이어지 는 영업 을 한시 적 으로 제한 할 필요 는 있 다 . 
- 튜브 에 들 어 있 는 내용물 을 치약 처럼 짜 잇솔 질 을 하 는 방법 으로 잇 몸약 복용 에 부담 이 있 는 사람 들 도 손쉽 게 잇몸 및 치아 관리 가 가능 하 다 . 
- 몇몇 철학자 들 은 중국 사회 에 만연 한 ＇ 아 Q ＇ 의 모습 이 장자 ( 莊子 ) 에서 비롯 된 것 이 라는 주장 을 펼쳤 다 . 


**과거로 분류된 예시**

- 지난해 1 분기 128 억 원 이 었 던 영업 이익 이 올해 1 분기 505 억 원 으로 급증 했 다 .
수상 작가 와 맺 으려던 계약서 내용 가운데 일부 가 ＇ 독소 조항 ＇ 으로 해석 돼 수정 을 요청 받 았 으나 문학사상사 가 수용 하 지 않 자 우수 상 으로 뽑힌 후보 3 인 이 올해 수상 작품집 의 작품 수록 을 거부 해서 다 .
- 결국 최근 KDB 산업은행 은 대 규모 손실 위기 에 닥친 에어부산 에 140 억 원 금융 지원 을 결정 했 는데 에어부산 부채 비율 은 2018 년 98 . 7 % 에서 지난해 811 . 8 % 로 급등 했 다 .
- 유성구 어은동 은 본인 에게 딱 맞 는 캐릭터 를 찾아가 는 자아 탐색 과 역량 강화 를 통한 메이커 및 크리에이터 등 비즈니스 자립 발판 을 마련 해 지역 사회 와 함께 성장 하 는 ＇ 슬기 로운 부캐 마을 ＇ 을 조성 해 나간다는 구상 이 다 . 
- 제이다 유안 워싱턴 포스트 기자 는 ＂＇ 기생충 ＇ 의 작품상 수상 에 는 오스카 가 어느 한쪽 으로 편향 돼서 는 안 된다는 공감대 도 작용 한 것 으로 보인다 ＂ 며 ＂ 오스카 는 2 ~ 3 년 전 백인 이외 의 인종 구성 을 2 ~ 3 배 늘리 면서 다양 성 을 반영 하 려는 노력 을 했 다 ＂ 고 평했 다 .


**미래로 분류된 예시**

- 단기 조정 시 최대 1 , 000 만 원 구간 까지 는 하락 할 수 있 으니 단기 투자자 분 들 은 1 , 100 만 원 을 돌파 하 거나 1 , 000 만 원 부근 까지 하 락할 때 접근 해 보 면 좋 을 것 같 습니다 .
- 근로자 들 은 이날 오전 8 시 부터 연말 정산 간소 화 서비스 에서 소득 · 세액 공제 자료 를 조회 할 수 있 다 .
- 오 는 3 월 께 에 는 재심 공판 기일 을 열 어 사건 을 재 심리 할 계획 이 다 .
- 그만큼 가격 인하 로 이어질 전망 이 다 . 
- 통화 중 에 새로운 참여 자 를 추가 할 수 도 있 다 . 

***

# 마무리
- 예시를 보면 분류가 이게 맞나??? 싶은게 좀 있다. 데이터가 애초에 애매한 데이터인게 큰 것이겠다. Garbage in garbage out 이란 명언이 있다.
- 다른 명확한 데이터에서는 빠르게 할 수 있는 방벙인거 같아서 효용이 높을것이다. 
- 위에 예시는 진짜 간단하게 한 수준이고 이외에 [Text classification · fastText](https://fasttext.cc/docs/en/supervised-tutorial.html) 에서 보면, 더 advanced 된 모델을 구성할 수 있으므로,  나중에 사용할일 있으면 정독할 것 .
- 참고로 단어 간에 관계를 보고 싶으면 fasttext 의 skipgram 모듈을 사용하면 된다. 나중에 기회되면 해봐야지
코드는 그냥 간단. 자세한 내용은  [Word representations · fastText](https://fasttext.cc/docs/en/unsupervised-tutorial.html) 참고 

```
$fasttext skipgram -input "파일이름"  -output "파일이름" -dim 256 -epoch 100 -minCount 5

# bin에는 뉴럴네트워크 모델 vec에는 단어벡터가 저장됨

$ echo "히히" | fasttext nn "모델경로" 10 
# 히히랑 가장 가까운 단어 10개 골라서 print 해줌
```

- 근데 사실 쓸일은 없을듯. 딥러닝으로 넘어가자.





