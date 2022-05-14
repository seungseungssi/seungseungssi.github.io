---
title: Text Data Linux로 전처리하기 예제 정리
tags: TeXt
---


**본 내용은 김기현의 자연어생성반 내용을 정리한 것입니다.**

김기현의 자연어 셍성반을 들으며, 가장 난감했던 부분은 이론적인 내용이 아니었다. 실습에서의 데이터 전처리를 쥬피터 노트북으로 하지않고 리눅스 기반으로 한다는 점이 더 당혹스러웠다. 리눅스를 다뤄본적도 없거니와, 컴알못 친화적이지도 않아서 코드도 잘 안읽히더라.

리눅스에 대한 충분한 설명없이 진행해서 짜증이 났다. 그래도 선생님이 현업 ML엔지니어들은 쥬피터 노트북을 거의 쓰지 않는다고 하셔서 꾸역구역 따라하기로 했다. (선생님은 더 초급 과정에서 자세히 설명하셨다고 했지만, 내돈내산이라서....강의 추가 구입은 싫다.)
 
처음해보는 것이기에, 그리고 기본적인 문법 자체를 아예 모르기에 정리의 필요성을 느끼게 되었다. 이론적인 부분은 김기현님의 블로그의 [Korean/English Machine Translation using AI-Hub Dataset](https://kh-kim.github.io/blog/2020/12/11/KoEn-MT.html) 포스트를 참고할 것. 

나는 코드 위주로만 정리하려한다. 어차피 나만 볼거 같으니.

![TeXt Theme](https://www.dropbox.com/s/xjof9wc0k859epc/4_Hello%20World.PNG?raw=1)

***

## 1. 데이터 얻기

예제에서 사용된 데이터는 [AIhub](https://aihub.or.kr)에서 얻을수 있다. 
개방 데이터 → 음성/자연어 → 한국어-영어 번역(병렬)말뭉치에서 다운받으면 된다.
선생님은 다운로드 허가 얻는데 사유 쓰고 통과까지 2일 정도 걸렸다는데, 내가 할때는 자동화가 되었는지 라이센스 "동의" 누르니까 자동으로 허가가 되었다.

## 2. 데이터 파일 변환하기

다운받을 수 있는 파일은 엑셀 파일인데, 이를 텍스트 파일로 다시 전환해줘야 한다. 더 간단한 방법이 있는거 같지만, 선생님은 복붙 노가다를 추천해주셔서 나도 그렇게 했다. (이유가 있겠지.. 특히나 텍스트에 이상한 문자가 들어있으면 오류도 많이나기 때문에, 확실하게 오류 안나는 방법이 더 맘편하기는 한 것 같다.)

다운받은 9개 파일 각각에 대해서 "원문" 컬럼과 "번역문" 컬럼만 텍스트편집기(윈도우는 메모장)에 복붙해주면 된다. (컬럼이름은 빼고 복붙해주자) 

그리고 변환된 파일은 이런식으로 text_generation > raw 라는 폴더안에 넣어주었다. 

![TeXt Theme](https://www.dropbox.com/s/34l7qmsg558pbo4/1.png?raw=1)

 
## 3. 리눅스를 이용한 텍스트 전처리 

맥북에는 리눅스가 기본으로 깔려있다. 터미널을 열면 된다. 윈도우는 다른 프로그램을 깔아주면 된다는데 bash를 깔아주면 된다고 한다. (cmder 도 괜찮았던듯 한데 잘 모름! 나중에 GPU 달린 데스크탑 사면 시도해볼 예정이다.)
 
### 3-1 데이터 살펴보기

가장 먼저, 텍스트 파일을 저장한 디렉토리를 설정해주는 것이 편하다. 

```
cd "저장(폴더) 위치"
```

맥북의 경우에는 터미널에 cd를 치고 한 칸 띄운 다음에 text_generation 폴더 자체를 터미널로 Drag & Drop 해주면 편하더라. 다음에는 디렉토리에 어떤 텍스트 파일이 들어있는지 기본적인 정보를 살펴보기 위해서 ls 명령어에 s 옵션을 주도록 한다. 

```
~ % ls -s ./raw/*.txt
 50496 ./raw/1_구어체(1).txt
 52736 ./raw/1_구어체(2).txt
 28080 ./raw/2_대화체.txt
146904 ./raw/3_문어체_뉴스(1).txt
113168 ./raw/3_문어체_뉴스(2).txt
154936 ./raw/3_문어체_뉴스(3).txt
126168 ./raw/3_문어체_뉴스(4).txt
 65216 ./raw/4_문어체_한국문화.txt
 80512 ./raw/5_문어체_조례.txt
 76904 ./raw/6_문어체_지자체웹사이트.txt

```
원래는 ll 명령어를 치면 되는것으로 설명해주셨는데, 내 맥북을 업데이트 하고나서는 내 컴퓨터에서 잘 안되서, 그냥 구글링하여 대안을 찾았다.

예시 파일을 보기 위해서는 다음과 같이 head -n "몇줄까지 볼것인지 숫자 n" "파일 이름" 을 통해서 보면 된다. 

```
 ~ % head -n 2 ./raw/1_구 어체\(1\).txt 
'Bible Coloring'은 성경의 아름다운 이야기를 체험 할 수 있는 컬러링 앱입니다.	Bible Coloring' is a coloring application that allows you to experience beautiful stories in the Bible.
씨티은행에서 일하세요?	Do you work at a City bank?
(base) iseunghun@iseunghun-ui-MacBookPro text_generation % 

```
엑셀에서 복붙해오면 자동으로 Tab으로 구분되어있음을 알 수 있다. 따라서 나중에 raw를 합쳐줄때는 tsv파일로 저장해줘야한다. (참고로 괄호를 나타내줄때는 \를  앞에 붙여주는 것을 잊지 말도록 하자).

### 3-2 데이터 합치기 

cat 명령어를 통해 다음과 같이 모든 텍스트 파일을 합칠 수 있다. 

```
~ % cat ./raw/*.txt > corpus.tsv
```
 .txt 의 이름으로 되어있는 모든 텍스트 파일을 합치고 corpus.txt 로 저장하라는 뜻이다. 또한, 다음을 통해 총 raw 수는 1602408  임을 알 수 있다.  (강의에서는 1602409 인데, 몬가 복붙할때 하나 날라간듯?? 다시하면 너무 노가다라 그냥 진행하도록 하겠다.) 
 
 ```
 % wc -l ./corpus.tsv
 1602408 ./corpus.tsv
 ```

### 3-3 데이터 Suffle

데이터를 순서대로 합쳤기 때문에 Train, Valid, Test를 동등한 성질로 나누려면 셔플링을 해줘야한다. 

```
~ % shuf ./corpus.tsv > ./corpus.shuf.tsv
```

### 3-4 Train, Valid, Test 데이터 나누기 

1602408 문장중에서 120000 문장을 train, 200000 문장을 validation , 202408 문장을 test 셋으로 나누도록 한다. 

**train set** 은 앞에 있는 문장중 1200000 문장

``` 
~ % head -n 1200000 ./corpus.shuf.tsv > corpus.shuf.train.tsv
```

**validation set** 은 마지막 402408 문장 중에서 앞의 200000 문장 ("|" 는 순차적으로 진행하란 뜻이라고 함)  head 는 금방 몇초면 되던데 tail 을 쓰면 좀 더 오래걸리는것 같다. 안심하고 기다리도록 하자. 

``` 
% tail -n 402409 ./corpus.shuf.tsv | head -n 200000 > ./corpus.shuf.valid.tsv   
```

**test set** 은 마지막 202408 문장이다.

```
% tail -n 202408 ./corpus.shuf.tsv > ./corpus.shuf.test.tsv
```
다음과 같이 잘 나눠졌음을 확인할 수 있다.

```
~ % wc -l ./corpus.shuf.*
  202408 ./corpus.shuf.test.tsv
  120000 ./corpus.shuf.train.tsv
 1602409 ./corpus.shuf.tsv
  200000 ./corpus.shuf.valid.tsv
 2124817 total
```
### 3-5 한글과 영어 컬럼 나누기

한국어와 영어는 언어체계가 다르기 때문에, 언어처리를 각각 따로 진행해야 한다. 이를 위하여 cut 이라는 명령어로 컬럼(영어, 한글)을 분래해주고, 다시 각각의 corpus를 저장을 해주도록 한다.

```
% cut -f1 ./corpus.shuf.train.tsv > ./corpus.shuf.train.ko ; cut -f2 ./corpus.shuf.train.tsv > ./corpus.shuf.train.en
```

확인 결과 하기와 같이 잘 나눠짐을 확인할 수 있다. (첫번째 단락이 영어, 두번째 단락이 한국어, 마지막 단락이 원래 합쳐져있던 데이터임) 

```
head -n 3 ./corpus.shuf.train.*
==> ./corpus.shuf.train.en <==
The head of the Gu shall, concerning the electronic stamps pursuant to Article 3 (1), designate the head of each department of use as a person responsible for the management of the administrative information system, etc. and shall publicly announce necessary matters, such as the name of issuing agency, the serial number, the issuing date, the issuing place, etc., on the Gu Newsletter and the Gu internet website.
The action just taken has forced the preview view to be rebuilt.
Founded in 1999, the Korean Association of Civilization was established as the Chungju branch of the Korean Association for Civilization in December when it was approved as a subsidiary company in 2004.

==> ./corpus.shuf.train.ko <==
구청장은 제3조제1항에 따른 전자증지에 대하여는 행정정보시스템 등의 관리책임자를 각 사용부서의 장으로 하여 발행기관명, 고유번호, 발행개시일, 발행장소 등의 필요한 사항을 구보와 구 인터넷홈페이지에 고시해야 한다.
방금 수행한 작업으로 인해 미리 보기 보기가 재구성되었습니다.
1999년 창립된 한국문인화협회가 2004년 사단법인 인준을 받자 12월 한국문인화협회 충주지부로 발족하였다.

==> ./corpus.shuf.train.tsv <==
구청장은 제3조제1항에 따른 전자증지에 대하여는 행정정보시스템 등의 관리책임자를 각 사용부서의 장으로 하여 발행기관명, 고유번호, 발행개시일, 발행장소 등의 필요한 사항을 구보와 구 인터넷홈페이지에 고시해야 한다.	The head of the Gu shall, concerning the electronic stamps pursuant to Article 3 (1), designate the head of each department of use as a person responsible for the management of the administrative information system, etc. and shall publicly announce necessary matters, such as the name of issuing agency, the serial number, the issuing date, the issuing place, etc., on the Gu Newsletter and the Gu internet website.
방금 수행한 작업으로 인해 미리 보기 보기가 재구성되었습니다.	The action just taken has forced the preview view to be rebuilt.
1999년 창립된 한국문인화협회가 2004년 사단법인 인준을 받자 12월 한국문인화협회 충주지부로 발족하였다.	Founded in 1999, the Korean Association of Civilization was established as the Chungju branch of the Korean Association for Civilization in December when it was approved as a subsidiary company in 2004.
(base) iseunghun@iseunghun-ui-MacBookPro text_generation % 
```
Valid와 test 에 대해서도 동일한 작업을 해주도록 한다. 

```
~ % cut -f1 ./corpus.shuf.valid.tsv > ./corpus.shuf.valid.ko ; cut -f2 ./corpus.shuf.valid.tsv > ./corpus.shuf.valid.en 
~ % cut -f1 ./corpus.shuf.test.tsv > ./corpus.shuf.test.ko ; cut -f2 ./corpus.shuf.test.tsv > ./corpus.shuf.test.en
```
마지막으로 한국어 파일은 한국어만 나오는지, 영어파일은 영어파일만 나오는지 확인해주도록 한다. 

* 한국어 

```
% head -n 3 ./corpus.*.ko   
==> ./corpus.shuf.test.ko <==
가파도 들어가는 배 시간표를 어디에서 볼 수 있나요?
구청장은 주민등록업무담당공무원이 변상책임을 지게 되거나 기타 보험금을 청구할 사유가 발생된 때에는 지체없이 관계보험 회사에 그 뜻을 통지하고 당해 보험금액을 세입금으로 징수하기 위한 필요한 조치를 취하여야 한다.
정부와 한은은 지금까지 채권시장에 외국인 자금이 들어오고 있다는 이유로 한미 금리 역전에 따른 자본 유출 우려가 적다는 입장을 보여왔다.

==> ./corpus.shuf.train.ko <==
구청장은 제3조제1항에 따른 전자증지에 대하여는 행정정보시스템 등의 관리책임자를 각 사용부서의 장으로 하여 발행기관명, 고유번호, 발행개시일, 발행장소 등의 필요한 사항을 구보와 구 인터넷홈페이지에 고시해야 한다.
방금 수행한 작업으로 인해 미리 보기 보기가 재구성되었습니다.
1999년 창립된 한국문인화협회가 2004년 사단법인 인준을 받자 12월 한국문인화협회 충주지부로 발족하였다.

==> ./corpus.shuf.valid.ko <==
이들 단체는 교육부가 진행하는 모든 정책숙려제를 거부한다고 선언했다.
서비스 15주년을 맞이한 ‘메이플스토리’는 지난 여름 업데이트 성과에 힘입어 높은 성과를 기록했으며, 중국 지역에서 서비스 10주년을 맞은 ‘던전앤파이터’ 역시 두 자리 수 이상의 견고한 매출 성장률을 이어갔다.
‘그것이 알고 싶다’ 제작진은 한때 하루 22시간
```

* 영어

```
% head -n 3 ./corpus.*.en
==> ./corpus.shuf.test.en <==
Where can I see the ship schedule for Gapa-do?
When the public official in charge of resident registration duties assumes liability for compensation or any other ground for requesting insurance proceeds arises, the head of the Gu shall notify the relevant insurance company of such fact without delay and take measures necessary to collect the relevant insurance as revenue.
The government and the Bank of Korea maintained position that there is little concern about capital outflow due to the reversal of the US-US interest rate because foreign funds are entering the bond market.

==> ./corpus.shuf.train.en <==
The head of the Gu shall, concerning the electronic stamps pursuant to Article 3 (1), designate the head of each department of use as a person responsible for the management of the administrative information system, etc. and shall publicly announce necessary matters, such as the name of issuing agency, the serial number, the issuing date, the issuing place, etc., on the Gu Newsletter and the Gu internet website.
The action just taken has forced the preview view to be rebuilt.
Founded in 1999, the Korean Association of Civilization was established as the Chungju branch of the Korean Association for Civilization in December when it was approved as a subsidiary company in 2004.

==> ./corpus.shuf.valid.en <==
The groups declared that they would reject all policy deliberations carried out by the Education Ministry.
Maple Story, celebrating its 15th anniversary, scored highly thanks to updates last summer, and Dungeon & Fighter, which celebrated its 10th anniversary in China, has maintained strong sales growth of more than two digits.
The production team of "I Want to Know That" once met a former "heavy-up loader" who had posted videos professionally on more than 10 Webhards for 22 hours a day.
(base) iseunghun@iseunghun-ui-MacBookPro text_generation % 
```

### 3-6 Tokenization(형태소 분석기 적용)

영어(라틴어 기반)의 경우에는 Tokenization을 해줄 필요없이 subword segmentation만 해도 충분하다고 한다. 영어는 띄어쓰기가 매우 잘되어있는 언어이기 때문이다. 반면에 한국어의 띄어쓰기는 정립이 되어있지 않고(문법이 있기는 하지만) 사람마다 제각각이기 때문에, Tokenization을  subword segmentation적용 전단계에 한번 돌려주는 것이 더 낫다고 한다.(Normalizaton의 개념이다, 희소성을 낮추는 역할이라고 생각하면 된다..)  다만 pretrained model 의 경우에는 subword segmentation만 해주는게 대세이긴 한것같다. (어떻게 해주든 성능 자체 차이는 미미할거 같긴 하다.)

**한글** 
이 강의에서는 한글을 위한  형태소 분석기는 mecab 을 사용했다. 가장 대중적으로 쓰이는 형태소 분석이기인것 같다. mecab을 적용하면 다음과 같이 문장이 분절됨을 알수있다. 이때 wakati 라는 옵션을 주지 않으면 POS Tagger 도 같이 출력되기 때문에 꼭 옵션을 부여해주어야 한다. 

```
~ % head -n 5 ./corpus.shuf.train.ko | mecab -O wakati
구청장 은 제 3 조제 1 항 에 따른 전자 증지 에 대하 여 는 행정 정보 시스템 등 의 관리 책임자 를 각 사용 부서 의 장 으로 하 여 발 행기 관명 , 고유 번호 , 발행 개시 일 , 발행 장소 등 의 필요 한 사항 을 구보 와 구 인터넷 홈페이지 에 고시 해야 한다 . 
방금 수행 한 작업 으로 인해 미리 보 기 보 기 가 재 구성 되 었 습니다 . 
1999 년 창립 된 한국 문인화 협회 가 2004 년 사단 법인 인준 을 받 자 12 월 한국 문인화 협회 충주 지부 로 발족 하 였 다 . 
월스트리트 저널 ( WSJ ) 은 소식통 들 을 인용 해 미 연방 검찰 이 중국 정부 와 연계 된 해커 들 의 위법 행위 혐의 를 이르 면 다음 주 중 공표 할 예정 이 라고 지난 7 일 보도 했 다 . 
분야 별 로 교통안전 분야 의 경우 경찰관 서 · 민간단체 등 과 합동 으로 학교 주변 교통안전 캠페인 전개 , 불법 주정차 차량 이동 조치 와 단속 등 등 하굣길 지킴 이 활동 을 실시 했 다 . 
```

이를 각 데이터셋에 적용해주도록 한다. 다만 데이터 사이즈가 크다보니 b 옵션을 활용해 buffer를 충분히 줘야 에러가 날 확률을 줄일수 있다.  다만 위와 같은 형태는  컴퓨터가 보여주기 좋은 형태이고, 자연어생성(즉, 인간이 보기 편하도록 언어가 나열된 형태를 가지도록)을 하기 위해서는 원래 띄어쓰기가 어떻게 되었는지 표시를 해줄 필요가 있다. 이를 위해서 선생님은 후에 detokenization을 위해서 띄어쓰기 된곳에 underscore _ 를 붙여주는 방식으 채택했다고 한다. 이에 대한 함수는 post_tokenize.py 함수를 제공하고 있는데 이는 감사하게도 [김기현님 nlp with pytorch examples 깃허브](https://github.com/kh-kim/nlp_with_pytorch_examples/tree/master/chapter-04)에 올라와 있는듯하다. 다른 쓰이는 함수도 마찬가지로 올라와있는듯. (모든 코드를 면밀하게 확인하지는 않음). 

```
 % cat ./corpus.shuf.test.ko | mecab -O wakati -b 99999 > .corpus.shuf.test.tok.ko 
```

결과물을 확인해보면 다음과 같다. 

```
~% head -n 5 ./corpus.shuf.test.tok.ko
▁가파 도 ▁들어가 는 ▁배 ▁시간표 를 ▁어디 에서 ▁볼 ▁수 ▁있 나요 ?
▁구청장 은 ▁주민 등록 업무 담당 공무원 이 ▁변상 책임 을 ▁지 게 ▁되 거나 ▁기타 ▁보험금 을 ▁청구 할 ▁사유 가 ▁발생 된 ▁때 에 는 ▁지체 없이 ▁관계 보험 ▁회사 에 ▁그 ▁뜻 을 ▁통지 하 고 ▁당해 ▁보험금액 을 ▁세 입금 으로 ▁징수 하 기 ▁위한 ▁필요 한 ▁조치 를 ▁취하 여야 ▁한다 .
▁정부 와 ▁한은 은 ▁지금 까지 ▁채권 시장 에 ▁외국인 ▁자금 이 ▁들어오 고 ▁있 다는 ▁이유 로 ▁한미 ▁금리 ▁역전 에 ▁따른 ▁자본 ▁유출 ▁우려 가 ▁적 다는 ▁입장 을 ▁보여 왔 다 .
▁당사 의 ▁주요 ▁사업 분야 는 ▁해수 담수 ▁및 ▁RO ▁전처 리용 ▁MF ▁입니다 .
▁응 . ▁주로 ▁1 학년 들 이 ▁올 ▁거 고 ▁교수 님 들 이 ▁환영 ▁인사 를 ▁해 주 실 ▁거 야 .
```
참고로 detokenize 예시는 다음과 같다.(지금단계에서는 필요없음). 

```
~ % head -n 5 ./corpus.shuf.test.tok.ko | python ./subword-nmt/detokenizer.py 
가파도 들어가는 배 시간표를 어디에서 볼 수 있나요?
구청장은 주민등록업무담당공무원이 변상책임을 지게 되거나 기타 보험금을 청구할 사유가 발생된 때에는 지체없이 관계보험 회사에 그 뜻을 통지하고 당해 보험금액을 세입금으로 징수하기 위한 필요한 조치를 취하여야 한다.
정부와 한은은 지금까지 채권시장에 외국인 자금이 들어오고 있다는 이유로 한미 금리 역전에 따른 자본 유출 우려가 적다는 입장을 보여왔다.
당사의 주요 사업분야는 해수담수 및 RO 전처리용 MF 입니다.
응. 주로 1학년들이 올 거고 교수님들이 환영 인사를 해주실 거야.
```

 인간이 이해하기 쉽도록 원래대로 되돌려졌다. 

**영어**

영어의 경우 다음과 같은 함수를 사용하여 tokenize를 해준다. 

```
 % cat ./corpus.shuf.test.en | python ./subword-nmt/tokenizer.py | python ./subword-nmt/post_tokenize.py ./corpus.shuf.test.en > ./corpus.shuf.test.tok.en
 ```
 결과는 다음과 같이 확인했을때 원래 띄어쓰기가 있던 자리에 underscore가 붙게 된다. 
 
```
~ % cat ./corpus.shuf.test.en | python ./subword-nmt/tokenizer.py | python ./subword-nmt/post_tokenize.py ./corpus.shuf.test.en > ./corpus.shuf.test.tok.en
(base) iseunghun@iseunghun-ui-MacBookPro text_generation % head -n 5 ./corpus.shuf.test.tok.en
▁Where ▁can ▁I ▁see ▁the ▁ship ▁schedule ▁for ▁Gapa @-@ do ?
▁When ▁the ▁public ▁official ▁in ▁charge ▁of ▁resident ▁registration ▁duties ▁assumes ▁liability ▁for ▁compensation ▁or ▁any ▁other ▁ground ▁for ▁requesting ▁insurance ▁proceeds ▁arises , ▁the ▁head ▁of ▁the ▁Gu ▁shall ▁notify ▁the ▁relevant ▁insurance ▁company ▁of ▁such ▁fact ▁without ▁delay ▁and ▁take ▁measures ▁necessary ▁to ▁collect ▁the ▁relevant ▁insurance ▁as ▁revenue .
▁The ▁government ▁and ▁the ▁Bank ▁of ▁Korea ▁maintained ▁position ▁that ▁there ▁is ▁little ▁concern ▁about ▁capital ▁outflow ▁due ▁to ▁the ▁reversal ▁of ▁the ▁US @-@ ▁US intere▁st ra▁te becau▁se forei▁gn fun▁ds a▁re enteri▁ng t▁he bo▁nd market .
▁Our ▁major ▁business ▁line ▁is ▁seawater ▁freshwater ▁and ▁MF ▁for ▁preprocessing ▁RO .
```
나머지 데이터에 대해서도 적용하도록 한다. (참고로 오래 걸리기 때문에 백그라운드에서 돌아가도록, & 를 끝에 넣어줘서 병렬 처리를 하면 좋다고 한다.) 

```
~ % cat ./corpus.shuf.train.ko | mecab -O wakati -b 99999 | python ./subword-nmt/post_tokenize.py ./corpus.shuf.train.ko > ./corpus.shuf.train.tok.ko &
~ % cat ./corpus.shuf.valid.ko | mecab -O wakati -b 99999 | python ./subword-nmt/post_tokenize.py ./corpus.shuf.valid.ko > ./corpus.shuf.valid.tok.ko &
~ % cat ./corpus.shuf.train.en | python ./subword-nmt/tokenizer.py | python ./subword-nmt/post_tokenize.py ./corpus.shuf.train.en > ./corpus.shuf.train.tok.en &
~ % cat ./corpus.shuf.valid.en | python ./subword-nmt/tokenizer.py | python ./subword-nmt/post_tokenize.py ./corpus.shuf.valid.en > ./corpus.shuf.valid.tok.en & 
```

다음과 같이 행 수가 제대로 맞는지 중간점검하는 작업도 중요하다. 

``` 
~ % wc -l ./corpus.shuf.*.tok.ko

  202408 ./corpus.shuf.test.tok.ko
 1200000 ./corpus.shuf.train.tok.ko
  200000 ./corpus.shuf.valid.tok.ko
 1602408 total
 
~ % wc -l ./corpus.shuf.*.tok.en

  202408 ./corpus.shuf.test.tok.en
 1200000 ./corpus.shuf.train.tok.en
  200000 ./corpus.shuf.valid.tok.en
 1602408 total
```

여기까지만 전처리를 끝내도 되긴하지만, 지금 상태로는 OoV가 발생할 것이다.

### 3-7 Subword Segmentaion(BPE(Byte Pair Encoding) 알고리즘 적용)

BPE 알고리즘을 통해 문서내의 통계를 통해 단어를 쪼개면, OoV(Out of Vacabulary)를 줄일수 있다는 장점이 있다. 이때 다음과 같이 git 을 클론해준다. 원래  있었던 BPE 알고리듬을 김기현님이 한국어 처리(Mecab과 같은 형태소 분석기를 먹인후의)에 좀 더 맞도록 변형한것이라고 한다.  만약에 형태소 분석기를 제외하고 바로 BPE만 먹일거라면, 원 코드를 쓰면 된다. 

```
~ % git clone https://github.com/kh-kim/subword-nmt
2022-05-14 21:27:19.393 xcodebuild[4399:192633] Requested but did not find extension point with identifier Xcode.IDEKit.ExtensionSentinelHostApplications for extension Xcode.DebuggerFoundation.AppExtensionHosts.watchOS of plug-in com.apple.dt.IDEWatchSupportCore
2022-05-14 21:27:19.393 xcodebuild[4399:192633] Requested but did not find extension point with identifier Xcode.IDEKit.ExtensionPointIdentifierToBundleIdentifier for extension Xcode.DebuggerFoundation.AppExtensionToBundleIdentifierMap.watchOS of plug-in com.apple.dt.IDEWatchSupportCore
fatal: destination path 'subword-nmt' already exists and is not an empty directory.
```

**모델 training**

BPE 알고리듬을 배울 함수 learn_bpe 의 옵션은 다음과 같이 확인 가능하다. (asdf 를 빼먹으면, 계속 돌아가는 현상이 있다. 왜그런지는 아직 파악 안해봄.) 

```
% python ./subword-nmt/learn_bpe.py -asdf                 
usage: learn_bpe.py [-h] [--input PATH] [--output PATH] [--symbols SYMBOLS]
                    [--min-frequency FREQ] [--dict-input] [--verbose]
learn_bpe.py: error: unrecognized arguments: --asdf
```

다음과 같이 50000번 수행을 예시로 미리 해보도록 한다. (50000번을 어떤 토큰들을 merging 하는 작업) 

```
% python ./subword-nmt/learn_bpe.py --input ./corpus.shuf.train.tok.en --output bpe.en.model --symbols 50000 --verbose
......(생략)
pair 7557: l ar▁ge</w> -> lar▁ge</w> (frequency 1177)
pair 7558: Y e -> Ye (frequency 1177)
pair 7559: ▁s un -> ▁sun (frequency 1176)
pair 7560: ▁opport unities</w> -> ▁opportunities</w> (frequency 1176)
pair 7561: ▁l isten -> ▁listen (frequency 1176)
pair 7562: ▁expen sive</w> -> ▁expensive</w> (frequency 1176)
....(생략)

```

예를들어서  7559번째에  _S와  un을 합쳐서 _Sun을 단어사전에 저장했다고 생각하면 된다. vocabulary 사이즈의 권장은 20000에서 30000이라고 한다. 그럼에도 verbose 를 50000으로 준 이유는, 통계적인 이유에 의해(유의성에 관한 얘기인듯) 정작 채택되는 단어는 50000 보다 작은 20000에서 30000사이에서 채택되는 경우가 많기 때문이라 한다. (영어의 경우) 

모델을 열어보면 단어들을 합치는 프로세스(merge instruction)를 구경할 수 있다.

```
~ % head bpe.en.model
#version: 0.2
h e</w>
i n
▁t he</w>
t i
e r
e n
e d</w>
a n
o n
~ % tail bpe.en.model 
Go▁ver ▁n
Feb▁ru ary</w>
F os
F ine</w>
Eng land</w>
Em er▁
Ec▁ onomic</w>
EX CO</w>
E st
Dec re▁e</w>
```

한국어의 경우에는 30000 만 줘도 충분하다고 한다. (영어에 비해 복잡하기 때문에) 

```
% python ./subword-nmt/learn_bpe.py --input ./corpus.shuf.train.tok.ko --output bpe.ko.model --symbols 30000 --verbose
...(생략)
pair 29984: ▁겉 옷</w> -> ▁겉옷</w> (frequency 48)
pair 29985: ▁건 양</w> -> ▁건양</w> (frequency 48)
pair 29986: ▁갱 년기</w> -> ▁갱년기</w> (frequency 48)
pair 29987: ▁개인 주의</w> -> ▁개인주의</w> (frequency 48)
...(생략)
```


**모델 apply**

여기서는 "apply_bpe"라는 함수를 사용하도록 하는데,  파라미터는 위와 동일한 방식으로 볼 수 있다. 

```
~ % python  ./subword-nmt/apply_bpe.py --asdf
usage: apply_bpe.py [-h] [--input PATH] --codes PATH [--merges INT]
                    [--output PATH] [--separator STR] [--vocabulary PATH]
                    [--vocabulary-threshold INT] [--glossaries STR [STR ...]]
```

함수 적용 예시는 다음과 같다. 

```
~% head -n 5 ./corpus.shuf.train.ko | python subword-nmt/apply_bpe.py -c ./bpe.ko.model
▁구 청 장 은 ▁제 3 조 제 1 항 에 ▁따 른 ▁전 자 증 지에 ▁대 하 여 는 ▁행정 정보 시스템 ▁등 의 ▁관리 책 임 자 를 ▁각 ▁사 용 부 서 의 ▁장 으로 ▁하 여 ▁발 행 기 관 명 , ▁고 유 번 호 , ▁발 행 개 시 일 , ▁발 행 장소 ▁등 의 ▁필 요한 ▁사 항 을 ▁구 보 와 ▁구 ▁인터 넷 홈 페이 지에 ▁고 시 해야 ▁한 다 .
▁방 금 ▁수행 한 ▁작 업 으로 ▁인해 ▁미리 ▁보기 ▁보 기 가 ▁재 구 성 되 었 습 니 다 .
▁19 9 9 년 ▁창 립 된 ▁한국 문 인 화 협 회가 ▁20 0 4 년 ▁사 단법인 ▁인 준 을 ▁받 자 ▁12 월 ▁한국 문 인 화 협회 ▁충 주 지 부로 ▁발 족 하 였 다 .
▁월 스트 리 트 저 널 ( W S J ) 은 ▁소 식 통 들 을 ▁인 용 해 ▁미 ▁연 방 검 찰 이 ▁중국 ▁정부 와 ▁연 계 된 ▁해 커 들 의 ▁위 법 행위 ▁혐 의 를 ▁이 르 면 ▁다 음 주 ▁중 ▁공 표 할 ▁예 정 이라고 ▁지난 ▁7 일 ▁보 도 했 다 .
▁분 야 별로 ▁교통안전 ▁분 야 의 ▁경우 ▁경찰 관 서 · 민간단체 ▁등 과 ▁합 동 으로 ▁학교 ▁주변 ▁교통안전 ▁캠페인 ▁전 개 , ▁불 법 주 정차 ▁차량 ▁이동 ▁조 치 와 ▁단속 ▁등 ▁등 하 굣 길 ▁지킴이 ▁활동 을 ▁실 시 했 다 .
```

원래 예문은 다음과 같다. 

```
% head -n 5 ./corpus.shuf.train.ko                                                    
구청장은 제3조제1항에 따른 전자증지에 대하여는 행정정보시스템 등의 관리책임자를 각 사용부서의 장으로 하여 발행기관명, 고유번호, 발행개시일, 발행장소 등의 필요한 사항을 구보와 구 인터넷홈페이지에 고시해야 한다.
방금 수행한 작업으로 인해 미리 보기 보기가 재구성되었습니다.
1999년 창립된 한국문인화협회가 2004년 사단법인 인준을 받자 12월 한국문인화협회 충주지부로 발족하였다.
월스트리트저널(WSJ)은 소식통들을 인용해 미 연방검찰이 중국 정부와 연계된 해커들의 위법행위 혐의를 이르면 다음주 중 공표할 예정이라고 지난 7일 보도했다.
분야별로 교통안전 분야의 경우 경찰관서·민간단체 등과 합동으로 학교 주변 교통안전 캠페인 전개, 불법주정차 차량 이동 조치와 단속 등 등하굣길 지킴이 활동을 실시했다.
```
여기서 띄어쓰기가 어떻게 생겨났는지에 대한 기록을 유추할수 있다. (이 예제는 공교롭게도 첫번째 case밖에 없음) 

* underscore 두 개  : 원래 띄어쓰기가 되어있던 경우
* underscore 한 개 : Tokenization 단계에서 띄어쓰기가 된 경우
* underscore 없이 띄어쓰기만 : 지금 띄어쓰기가 생긴것임.  

또한, verbose 하이퍼파라미터를 조절해보면서 내가 원하는 단어가 합쳐지는지 확인해보는것이 좋다. 예를들어 만약 위의 예시와 같이 "미리" 와 "보기"가 합쳐지지 않는게 싫다면 " verbose 사이즈를 키우면 되는식이다. 반면에, 원치 않는 단어가 합쳐져 버렸다면 merging 이 지나치게 많이 일어난 것이기 때문에 verbose 사이즈를 줄이는 것이 좋을것이다. 

train set을 가지고 학습시킨 모델을 그대로 test set 과 validation set 에도 적용시키도록 하자(vocabulary에 조차도 train set만이 영향을 미치도록 하는게 정석이라고 한다.) 

```
# Korean
~ % cat ./corpus.shuf.train.tok.ko | python subword-nmt/apply_bpe.py -c ./bpe.ko.model > ./corpus.shuf.train.tok.bpe.ko ; cat ./corpus.shuf.valid.tok.ko | python subword-nmt/apply_bpe.py -c ./bpe.ko.model > ./corpus.shuf.valid.tok.bpe.ko ; cat ./corpus.shuf.test.tok.ko | python subword-nmt/apply_bpe.py -c ./bpe.ko.model > ./corpus.shuf.test.test.tok.bpe.ko
# English 
~ % cat ./corpus.shuf.train.tok.en | python subword-nmt/apply_bpe.py -c ./bpe.en.model > ./corpus.shuf.train.tok.bpe.en &
[1] 5051 5052
~ % cat ./corpus.shuf.valid.tok.en | python subword-nmt/apply_bpe.py -c ./bpe.en.model > ./corpus.shuf.valid.tok.bpe.en &
[2] 5060 5061
~  % cat ./corpus.shuf.test.tok.en | python subword-nmt/apply_bpe.py -c ./bpe.en.model > ./corpus.shuf.test.tok.bpe.en &
[2] 5066 5067
```

지겹지만, 찐막으로 잘 되었는지 확인만 해주면 된다....

```
% head -n 3 ./corpus.shuf.*.tok.bpe.ko
==> ./corpus.shuf.test.test.tok.bpe.ko <==
▁▁가 파 ▁도 ▁▁들어가 ▁는 ▁▁배 ▁▁시 간 표 ▁를 ▁▁어 디 ▁에서 ▁▁볼 ▁▁수 ▁▁있 ▁나요 ▁?
▁▁구청장 ▁은 ▁▁주민 ▁등록 ▁업무 ▁담당 ▁공무원 ▁이 ▁▁변상 ▁책임 ▁을 ▁▁지 ▁게 ▁▁되 ▁거나 ▁▁기타 ▁▁보험금 ▁을 ▁▁청구 ▁할 ▁▁사유 ▁가 ▁▁발생 ▁된 ▁▁때 ▁에 ▁는 ▁▁지 체 ▁없이 ▁▁관계 ▁보험 ▁▁회사 ▁에 ▁▁그 ▁▁뜻 ▁을 ▁▁통지 ▁하 ▁고 ▁▁당해 ▁▁보험 금액 ▁을 ▁▁세 ▁입금 ▁으로 ▁▁징수 ▁하 ▁기 ▁▁위한 ▁▁필요 ▁한 ▁▁조치 ▁를 ▁▁취 하 ▁여야 ▁▁한다 ▁.
▁▁정부 ▁와 ▁▁한 은 ▁은 ▁▁지금 ▁까지 ▁▁채권 ▁시장 ▁에 ▁▁외국인 ▁▁자금 ▁이 ▁▁들어 오 ▁고 ▁▁있 ▁다는 ▁▁이유 ▁로 ▁▁한미 ▁▁금리 ▁▁역전 ▁에 ▁▁따 른 ▁▁자본 ▁▁유출 ▁▁우려 ▁가 ▁▁적 ▁다는 ▁▁입장 ▁을 ▁▁보 여 ▁왔 ▁다 ▁.

==> ./corpus.shuf.train.tok.bpe.ko <==
▁▁구청장 ▁은 ▁▁제 ▁3 ▁조제 ▁1 ▁항 ▁에 ▁▁따 른 ▁▁전자 ▁증지 ▁에 ▁▁대 하 ▁여 ▁는 ▁▁행정 ▁정보 ▁시스템 ▁▁등 ▁의 ▁▁관리 ▁책임자 ▁를 ▁▁각 ▁▁사용 ▁부서 ▁의 ▁▁장 ▁으로 ▁▁하 ▁여 ▁▁발 ▁행기 ▁관 명 ▁, ▁▁고 유 ▁번호 ▁, ▁▁발행 ▁개시 ▁일 ▁, ▁▁발행 ▁장소 ▁▁등 ▁의 ▁▁필요 ▁한 ▁▁사항 ▁을 ▁▁구보 ▁와 ▁▁구 ▁▁인터넷 ▁홈페이지 ▁에 ▁▁고시 ▁해야 ▁▁한다 ▁.
▁▁방 금 ▁▁수행 ▁한 ▁▁작업 ▁으로 ▁▁인해 ▁▁미리 ▁▁보 ▁기 ▁▁보 ▁기 ▁가 ▁▁재 ▁구성 ▁되 ▁었 ▁습니다 ▁.
▁▁19 99 ▁년 ▁▁창 립 ▁된 ▁▁한국 ▁문 인화 ▁협회 ▁가 ▁▁20 04 ▁년 ▁▁사단 ▁법인 ▁▁인 준 ▁을 ▁▁받 ▁자 ▁▁12 ▁월 ▁▁한국 ▁문 인화 ▁협회 ▁▁충주 ▁지부 ▁로 ▁▁발 족 ▁하 ▁였 ▁다 ▁.

==> ./corpus.shuf.valid.tok.bpe.ko <==
▁▁이 ▁들 ▁▁단체 ▁는 ▁▁교육부 ▁가 ▁▁진행 ▁하 ▁는 ▁▁모든 ▁▁정책 ▁숙 려 ▁제 ▁를 ▁▁거부 ▁한다고 ▁▁선언 ▁했 ▁다 ▁.
▁▁서비스 ▁▁15 ▁주년 ▁을 ▁▁맞이 ▁한 ▁▁‘ ▁메이 플 ▁스토리 ▁’ ▁는 ▁▁지난 ▁▁여름 ▁▁업 데이트 ▁▁성과 ▁에 ▁▁힘 입 ▁어 ▁▁높 ▁은 ▁▁성과 ▁를 ▁▁기록 ▁했으며 ▁, ▁▁중국 ▁▁지역 ▁에서 ▁▁서비스 ▁▁10 ▁주년 ▁을 ▁▁맞 ▁은 ▁▁‘ ▁던 전 ▁앤 ▁파 이터 ▁’ ▁▁역 시 ▁▁두 ▁▁자리 ▁▁수 ▁▁이상 ▁의 ▁▁견 고 ▁한 ▁▁매출 ▁▁성장 ▁률 ▁을 ▁▁이 ▁어 ▁갔 ▁다 ▁.
▁▁‘ ▁그것 ▁이 ▁▁알 ▁고 ▁▁싶 ▁다 ▁’ ▁▁제작 진 ▁은 ▁▁한 때 ▁▁하루 ▁▁22 ▁시간 ▁▁10 ▁여 ▁개 ▁▁웹 ▁하드 ▁에 ▁▁동영상 ▁을 ▁▁전문 ▁적 ▁으로 ▁▁올 렸 ▁던 ▁▁전 직 ▁▁‘ ▁헤 ▁비 업 ▁로 더 ▁’ ▁를 ▁▁만 났 ▁다 ▁.
(base) iseunghun@iseunghun-ui-MacBookPro text_generation % head -n 3 ./corpus.shuf.*.tok.bpe.en
==> ./corpus.shuf.test.tok.bpe.en <==
▁▁W here ▁▁can ▁▁I ▁▁see ▁▁the ▁▁ship ▁▁schedule ▁▁for ▁▁Gap a ▁@-@ ▁do ▁?
▁▁When ▁▁the ▁▁public ▁▁official ▁▁in ▁▁charge ▁▁of ▁▁resident ▁▁registration ▁▁duties ▁▁ass umes ▁▁li ability ▁▁for ▁▁compensation ▁▁or ▁▁any ▁▁other ▁▁ground ▁▁for ▁▁requ esting ▁▁insurance ▁▁proc eeds ▁▁ar ises ▁, ▁▁the ▁▁head ▁▁of ▁▁the ▁▁Gu ▁▁shall ▁▁no tify ▁▁the ▁▁relevant ▁▁insurance ▁▁company ▁▁of ▁▁such ▁▁fact ▁▁without ▁▁delay ▁▁and ▁▁take ▁▁measures ▁▁necessary ▁▁to ▁▁coll ect ▁▁the ▁▁relevant ▁▁insurance ▁▁as ▁▁revenue ▁.
▁▁The ▁▁government ▁▁and ▁▁the ▁▁Bank ▁▁of ▁▁Korea ▁▁maintained ▁▁position ▁▁that ▁▁there ▁▁is ▁▁little ▁▁concern ▁▁about ▁▁capital ▁▁out flow ▁▁due ▁▁to ▁▁the ▁▁revers al ▁▁of ▁▁the ▁▁US ▁@-@ ▁▁US ▁interest ▁rate ▁because ▁foreign ▁funds ▁are ▁enter ing ▁the ▁bond ▁market ▁.

==> ./corpus.shuf.train.tok.bpe.en <==
▁▁The ▁▁head ▁▁of ▁▁the ▁▁Gu ▁▁shall ▁, ▁▁concer ning ▁▁the ▁▁elec tronic ▁▁st amps ▁▁pursu ant ▁▁to ▁▁Article ▁▁3 ▁▁( ▁1 ▁) ▁, ▁▁design ate ▁▁the ▁▁head ▁▁of ▁▁each ▁▁department ▁▁of ▁▁use ▁▁as ▁▁a ▁▁person ▁▁responsible ▁▁for ▁▁the ▁▁management ▁▁of ▁▁the ▁▁administr ative ▁▁information ▁▁system ▁, ▁▁etc. ▁▁and ▁▁shall ▁▁public ly ▁▁announ ce ▁▁necessary ▁▁matters ▁, ▁▁such ▁▁as ▁▁the ▁▁name ▁▁of ▁▁issu ing ▁▁agency ▁, ▁▁the ▁▁ser ial ▁▁number ▁, ▁▁the ▁▁issu ing ▁▁date ▁, ▁▁the ▁▁issu ing ▁▁place ▁, ▁▁etc ▁. ▁, ▁▁on ▁▁the ▁▁Gu ▁▁New sletter ▁▁and ▁▁the ▁▁Gu ▁▁internet ▁▁website ▁.
▁▁The ▁▁action ▁▁just ▁▁taken ▁▁has ▁▁forced ▁▁the ▁▁pre view ▁▁view ▁▁to ▁▁be ▁▁re built ▁.
▁▁F ounded ▁▁in ▁▁19 99 ▁, ▁▁the ▁▁Korean ▁▁Association ▁▁of ▁▁C iv ilization ▁▁was ▁▁established ▁▁as ▁▁the ▁▁Chung ju ▁▁branch ▁▁of ▁▁the ▁▁Korean ▁▁Association ▁▁for ▁▁C iv ilization ▁▁in ▁▁December ▁▁when ▁▁it ▁▁was ▁▁approved ▁▁as ▁▁a ▁▁subsidi ary ▁▁company ▁▁in ▁▁2004 ▁.

==> ./corpus.shuf.valid.tok.bpe.en <==
▁▁The ▁▁groups ▁▁decl ared ▁▁that ▁▁they ▁▁would ▁▁re ject ▁▁all ▁▁policy ▁▁deliber ations ▁▁carried ▁▁out ▁▁by ▁▁the ▁▁Education ▁▁Ministry ▁.
▁▁M aple ▁▁Story ▁, ▁▁celebr ating ▁▁its ▁▁15th ▁▁anniversary ▁, ▁▁scor ed ▁▁highly ▁▁th anks ▁▁to ▁▁up d ates ▁▁last ▁▁summer ▁, ▁▁and ▁▁D ung eon ▁▁&amp; ▁F ighter ▁, ▁which ▁celebr ated ▁its ▁▁10th ▁anniversary ▁in ▁China ▁▁, ▁has ▁maintained ▁strong ▁sales ▁growth ▁of ▁▁more ▁▁than ▁two ▁dig its ▁.
▁▁The ▁▁production ▁▁team ▁▁of ▁▁&quot; ▁▁I ▁W ant ▁to ▁K now ▁▁That ▁▁&quot; ▁once ▁▁met ▁a ▁former ▁&quot; ▁▁heavy ▁@-@ ▁up ▁lo ader ▁&quot; ▁who ▁had ▁posted ▁▁videos ▁professi onally ▁on ▁more ▁than ▁10 ▁W eb h ards ▁for ▁22 ▁hours ▁a ▁day ▁.
```
그렇다면 행 갯수는 ?

```
~ % wc -l ./corpus.shuf.*.tok.bpe.* 
  202408 ./corpus.shuf.test.test.tok.bpe.ko
  202408 ./corpus.shuf.test.tok.bpe.en
 1200000 ./corpus.shuf.train.tok.bpe.en
 1200000 ./corpus.shuf.train.tok.bpe.ko
  200000 ./corpus.shuf.valid.tok.bpe.en
  200000 ./corpus.shuf.valid.tok.bpe.ko
 3204816 total
```

다 잘 되었음을 확인할 수 있다...

***

코딩에 대한 기본기가 있음에도 불구하고 구조를 이해하는데 상당한 시간을 소모해버렸다. 그렇기에 더더욱 정리를 하기 잘했다는 생각이든다. 나중에는 시행착오를 더 줄이도록 해보자. 비록 예제이기 때문에, 크롤링 데이터와 같이 온갖가지 방법으로 더러운 텍스트데이터에 대한 전처리 방법은 아직 리눅스로 할지 모르지만, 큰 Step 을 밟았다고 생각한다.

또한, 파일 이름 짓는 노하우도 배울 수 있었다. 즉, 전처리 순서가 나타나도록 이름을 지어줄것. 이렇게 하면 어떤 단계에서 잘못하더라도, 해당 단계부터 다시 쉽게 할 수 있어서 효율적임을 느꼈다. 




