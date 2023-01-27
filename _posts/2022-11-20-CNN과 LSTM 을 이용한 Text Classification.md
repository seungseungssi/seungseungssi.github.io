---
title: CNN과 LSTM 을 이용한 Text Classification
tags: text classification 전처리 CNN RNN
---


**김기현의 자연어 클래스 참고해서 그대로 복습 해본것**


CNN과 LSTM을 이용한 Text Classification 을 실습해보도록 하겠다.
이를 위해  DACON 에서의 **기후기술 분류 데이터** 를 활용하도록 하였다. 

[자연어 기반 기후기술분류 AI 경진대회 - DACON](https://dacon.io/competitions/official/235744/overview/description)

이 데이터 역시 퀄러티가 그리 좋지는 못하다.  
Train 데이터에 총 174304의 row가 있는데 그 중 142571 건이 NaN이다…. 

또한, class 는 43개 였다…
(그리고 각 클래스당 관측치는 많아봐야 4900개 정도이고 대부분은 100개 미만이다.)

어차피 잘 안될게 뻔해서, 가장 많은 클래스 4개만 선정하여 분류모델을 만들어 보기로 하였다. (어차피 복습이 목적이니깐 ^^ ;;) 

클래스는 다음과 같다. 

- 19 : 산업효율화  (4938개)
- 24 : 작물재배-생산 (3520개)
- 23 : 유전자원/유전개량 (1840개)
- 5 : 태양광 (1698개)
- 14 : 전력저장 (1672개)

또한, 텍스트로 활용할 수 있는 컬럼은 **과제명**, **요약문_연구목표**, **요약문_연구내용**, **요약문_기대효과**가 있었는데, 그냥 전처리하기 간단한  **과제명**으로 분류를 실시해보기로 했다.

# 데이터 전처리  

- 특별한 표시가 없으면 Linux 로 돌린 것
- (참고로 .py 코드 돌릴때 source activate … 해서  환경변수 지정하는거 잊지 말것.


## 1. tsv파일로 바꿔주기 

(Python)

```
$ import pandas as pd
$ train = pd.read_csv(".../train.csv")
$ train["text"] = train["과제명"]

$ train = train[(train.label == 19) | (train.label == 24) |(train.label == 23) |(train.label == 5) |(train.label == 14) ]

$ train[["label", "text"]].to_csv(".../data.tsv", sep = "\t", header = None, index= False, encoding = "utf-8")
```

data.tsv 란 이름의 파일로 저장해주었다.

## 2. 전각문자 등 기타 문자 바꿔주기

```
$ python ./regex/refine.py ./regex/refine.regex.txt 1 < ./data.tsv > ./data.refined.tsv
```

프로그램 에러가 날 수도 있는데, 에러난 행을 확인해 보면 잘못되어있을 확률이 높다. 손으로 수동으로 해주는것을 추천(case가 많지 않은 경우). 

또는, 이 전단계에 sublime text 를 통한 전처리를 미리 해봐도 좋을 것 같다. 

관련된 refine.py 는 전각문자(주로 중국어나 일본어 문서에서 보임) 을 일반 문자로 바꿔주는 기능을 하는 함수인데,  refine.regex.txt 란 단어리스트를 참조하여 돌아간다.
  
강의자료로 얻은 코드인데, 그렇게 어려운 로직은 아니라 나중에 직접 짜봐도 될듯 하다.

## 3. Shuffling 해주기

```
$ shuf data.refined.tsv > data.refined.shuf.tsv
```

## 4. Interactive 한 전처리 (Sublimetext )

모든 문자 전처리를 파이썬으로 해결하려 하면, 끝이없다. 웬만큼 깨끗한 corpus라면 interacive 하게 전처리하는게 시간을 훨씬 아낄수 있다.

다음의 단계로 전처리를 진행했다. 

* [0-9][0-9][0-9][0-9]년도 >> 공백
* ^([0-9]+)\t\s  >>  ^([0-9]+)\t
* [^0-9가-힣A-Za-z\n\t ] >>  \s
* \)  >> \s
* /) >> \s

더 할수도 있는데 시간 cost를 고려해 이 정도에서 마무리했다.

## 5. Tokenization 해주기 

Mecab을 이용했다. 

```
$ cut -f2 data.refined.shuf.tsv > data.refined.shuf.text.tsv

$ cat ./data.refined.shuf.text.tsv | mecab -O wakati > data.refined.shuf.text.tok.tsv 
```

## 6. Subword segmentation 해주기

```
$ cat data.refined.shuf.text.tok.tsv | python post_tokenize.py data.refined.shuf.text.tsv > data.refined.shuf.text.tok.subword.tsv
```

post_tokenize.py 함수를 적용하면 다음과 같이 바뀌게 된다.

**옥수수 우량계통 이용촉진사업 >> ▁옥수수 ▁우량 계통 ▁이용 촉진 사업**

원래 띄어쓰기가  있던 부분은 ▁ 로 치환되고, mecab 에 의한 띄어쓰기는 그냥 빈 space 로 남게 된다. 이렇게 하면 mecab 에 의해서 생긴 띄어쓰기와 원래 있던 띄어쓰기를 구분할 수 있는 것이다. 

참고로 “▁” 는 일반적인 underscore 가 아닌 특수문자이다.

## 7. BPE 모델링

BPE 는 Classification 수준에서는 해도 그만, 안해도 그만인 느낌이긴하다. (단, NLG에서는 해줘야한다. )

이와 관련된 함수는 저자가 github에 올려두었다.

[GitHub - kh-kim/subword-nmt: Subword Neural Machine Translation based on rsennrich, but include feature to deal with a sentence already tokenized.](https://github.com/kh-kim/subword-nmt.git)

```
$ python ./subword-nmt/learn_bpe.py --input data.refined.shuf.text.tok.subword.tsv --output ./model --symbols 30000

no pair has frequency >= 2. Stopping # corpus 가 작아서 30000개 못채우고 중간에 끊긴다. 
```

생성되는 아웃풋 model 파일은 다음과 같이 생겼다. 

▁개 발</w>  —> 개발을 합침 
▁및 및</w>
▁s s</w>
▁위 한</w> —> 위함을 합침 
▁기 술</w>
기 술</w>
▁연 구</w>
스 템</w>
개 발</w>
▁이 용</w> 
…..

참고로 세부 옵션 보려면 다음 코드를 활용할 것 

```
$ python ./subword-nmt/learn_bpe.py -h
```

## 8. 생성된 BPE 모델을 그대로 코퍼스에 넣어주기

```
$ python ./subword-nmt/apply_bpe.py --codes ./model < data.refined.shuf.text.tok.subword.tsv > data.refined.shuf.text.tok.subword.bpe.tsv 
```

결과는 다음과 같다. 

**▁▁옥수수 ▁▁우량 ▁계통 ▁▁이용 ▁촉진 ▁사업**

* ▁ 두개인 경우 : 원래 처음부터 띄어쓰기 된 것 
* ▁ 한개인 경우 : mecab 에 의해 띄어쓰기가 생긴 경우 

BPE 를 적용할 경우 가령 **다 용 ▁도** 와 같이 **다** 와 **용**이 쪼개진 것을 관찰할 수 있다. 

~모델을 배포할때는 BPE 모델도 저장해야함을 잊어서는 안된다.~

**참고로  tokenize 로 나누고 subword로 또 나눈 단어들을 원상복귀하려면 다음과 같은 코드를 돌리면 된다~  (Detokenization)**

```
$ sed 's/ //g' data.refined.shuf.tok.subword.bpe.test.tsv | sed 's/▁▁/ /g' | sed 's/▁//g'

# sed “s/ //g” | sed “s/▁▁/ /g” | sed “s_//g
```

- 1. 공백제거 
- 2. ▁▁을 white space 로 치환 
- 3. ▁를 제거


## 9. 다시 Label과 합쳐주기

Text 데이터 전처리시 label 을 따로 떼놔서 다시 붙여야한다. 

```
$ cut -f1 data.refined.shuf.tsv > data.refined.shuf.label.tsv

$ paste data.refined.shuf.label.tsv data.refined.shuf.text.tok.subword.bpe.tsv  > data.refined.shuf.tok.subword.bpe.tsv
```

위까지의 과정은(suffling 빼고) 다음의 코드를 통해 그대로 간단히 재현 가능하다고 하는데 내 컴퓨터에서는 뭔가 잘 안됨…  나중에 고쳐보도록 해야지
(결과물 같음) 

```
$ tokenize.sh ./data.refined.shuf.tsv ./data.refined.shuf.tok.subword.bpe.tsv
```

tokenize .sh 도 강의에서 얻은 함수이다

컴공 도메인이 아닌 한계를 느껴버려서 ㅜㅜ,  아예 리눅스를 좀 배운담에 봐야겠다.

### 5.  Training/Validation/Test  Set 나누기

총 row 수가 13668 인데 각각 9225/3075/1368 개로 나눠주도록 한다.

```
$ head -n 9225 data.refined.shuf.tok.subword.bpe.tsv > data.refined.shuf.tok.subword.bpe.train.tsv 
$ head -n 12300 data.refined.shuf.tok.subword.bpe.tsv | tail -n 3075 > data.refined.shuf.tok.subword.bpe.valid.tsv
$ tail -n 1368 data.refined.shuf.tok.subword.bpe.tsv > data.refined.shuf.tok.subword.bpe.test.tsv
```





# Training  

학습은 쉽다.
솔직히 남의 코드 이용하면  개쉽게 함…

(Epoch은 시간관계상 5개만 했는데 더해주면 조음..)

[GitHub - kh-kim/simple-ntc: This repo provides a simple short-text classification code using RNN and CNN.](https://github.com/kh-kim/simple-ntc.git)

```
$ git clone https://github.com/kh-kim/simple-ntc.git
$  python ./simple-ntc/train.py --model_fn ./models_review.pth --train_fn data.refined.shuf.tok.subword.bpe.train.tsv --batch_size 128 --n_epochs 10 --word_vec_size 256 --dropout .3 --rnn --hidden_size 512 --n_layers 4 --cnn --window_sizes 3 4 5 6 7 8 --n_filters 128 128 128 128 128 128

Epoch 1 - |param|=8.80e+02 |g_param|=1.29e+00 loss=1.5052e+00 accuracy=0.4028                                           
Validation - loss=9.4383e-01 accuracy=0.6521 best_loss=inf                                                              
.......

```

알아서 잘 학습 된다. 
 
(RNN의 경우 전후 문맥을 고려하는 반면, CNN은 단어의 조합만을 가지고 classification을 한다고 이해하면 된다.)

**5 Epoch 후 Validation accuracy**

- RNN의 경우 0.8561 (사실 Epoch 더주면 CNN보다 높을거라 예상)
- CNN의 경우 0.8989


옵션을 어떻게 주느냐에 따라 CNN, RNN 중 하나만 학습할 것인지, 
또는 결과에 CNN과 RNN을 종합할 것인지 결정할 수 있다.

이때도 혹시 에러가 나면 코드보다는 text 파일에서 이상한 점이 없는지를 먼저 확인해보는게 좋을것 같다. (가령 \t delimeter가 모든 행에서 제대로 되어있는지 등)

# Inference 

다음과 같은 코드로 쉽게 시행한다. 

```
$ cut -f2 ./data.refined.shuf.tok.subword.bpe.test.tsv | python ./simple-ntc/classify.py --model ./models_review.pth --top_k 1 > 
```

다음과 같이 파이썬에서 성능을 살펴본 결과 생각보다는 괜찮은 모델이 탄생했다. 

(Python)

```
test = pd.read_csv(".../data.refined.shuf.tok.subword.bpe.test.tsv", header = None, delimiter='\t')
result = pd.read_csv(".../inference.result.tsv", header = None, delimiter='\t')

from sklearn.metrics import confusion_matrix
from sklearn.metrics import accuracy_score

y_true = test[0].astype(str).to_list()
y_pred = result[0].astype(str).to_list()

print(accuracy_score(y_true, y_pred))
print(confusion_matrix(y_true, y_pred))
```

Accuracy는 0.86 이고, confusion matrix를 보니 유전자원/유전개량 class에서 예측력이 떨어진다 . 



일단은 이정도로 마무리하고 내가 직접 짠 코드는 다음에 이용해보는 걸로~~~(이게 더 중요할듯. 시간이 오래걸리더라도 꼭 직접 해봐야지)





