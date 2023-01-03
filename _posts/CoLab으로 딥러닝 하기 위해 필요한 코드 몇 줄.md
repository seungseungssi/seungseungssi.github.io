---
title: CoLab으로 딥러닝 하기 위해 필요한 코드 몇 줄
tags: colab 파이썬 딥러닝
---

가난한 자를 위한 유일무이한 선택지는 구글 코랩을 쓰기 위해서는 
다음과 같은 코드 복붙하다 싶이 하자

```
# 구글 코랩으로 부터 드라이브 import
from google.colab import drive
import os 

# drive mount 시키기
drive.mount("/content/gdrive")
# 경로 지정하기
os.chdir("/content/gdrive/My Drive/Colab Notebooks/폴더 이름") 
```

추가로 구글 코랩은 몇시간 놔두면 저절로 중단되는 치명적 단점이 있는데 
돈을 질러서 유료계정을 쓰던지 다음과 같은 꼼수 절차를 밟도록 하자 
(그런데 어차피 12시간이면 꺼진다고 하기에 임시방편일 뿐이다.)


**비활성화방지 > F12 > 개발자도구. 콘솔창에 다음을 입력 > 엔터**

function ClickConnect(){
console.log(“Working”); 
document.querySelector(“colab-toolbar-button#connect”).click() 
}
setInterval(ClickConnect,60000)
