# K.-N.Lee's : 음성인식용 HTK 사용법 

written by K.-N.Lee (최초 작성: 1998년 / egloos: 2008년)

-----
Steve Young, Dan Kershaw, Julian Odell, Dave Ollason, Valtcho Valtchev, Phil Woodland의 
/The HTK Book Version 2.2/의 3장에 나와 있는 튜토리얼을 기반으로 작성하였습니다.

## 1. 음성 녹음

우선 인식 실험을 수행하기 위해서는 학습에 필요한 음성이 필요하다. 
일반적으로 구축하고자하는 목적에 맞게 텍스트를 준비하고, 발성 음성을 녹음하는 작업이 수반된다. 
HTK에서는 녹음 툴로 HSLab이라는 명령어가 제공된다.

>> $HSLab  -C  HAUDIO  noname  


## 2. 레이블 파일 작성
일반적으로 학습을 하기 위해서는 레이블링된 파일이 필요하다. 
물론, embeded training(HTK의 HERest)을 하는 경우, 시간 정보열이 요구되지 않으므로 올바른 학습용 발음열을 입력으로 넣어주면 된다. 
대부분 레이블링된 파일은 모델의 초기값을 생성하는 경우 많이 사용된다. (HTK의 HRest)

레이블링 툴로는 보통 Xwaves를 사용하며 레이블링한 파일은 HTK 형식으로 변환되어져야 한다.
(But, 학습시 레이블 파일을 사용하는 경우에는 레이블 파일의 형식에 따라 변환을 하여야 하나 명령어 옵션을 찾아보면, 다른 형식들을 지원하는 것이 있으므로 확인)

* HTK 레이블 형식
<pre>
 0 4500000 sil사
 4500000 5000000 N
 5000000 5500000 AA
    …
 12500000 13000000 A
 13000000 13500000 sil
 
 (100ns단위)
</pre>

* Xwaves 레이블 형식
<pre>
 signal s0001
 type 0
 comment created using xlabel Thu Jan 22 
 19:27:57 1998
 font - misc-*-bold-*-*-*-15-*-*-*-*-*-*-*
 separator ;
 nfields 1
 #
    0.450000   -1 sil
    0.500000   -1 N
    …
    1.300000   -1 A
    1.350000   -1 sil
 </pre>
 
 
## 3. 트랜스크립션(Transcription) 파일 작성
 
인식할 문장들을 디코딩 단위로 나누어 리스트를 작성한다.
* wlist (인식 단위 리스트)

>> $sort words | uniq > wlist

인식하고자하는 단어리스트들을 나열한 파일로서 한 라인의 단위는 인식하고자하는 디코딩 단위와 발음사전과 일치하여야 한다.
* words.mlf

인식하고자하는 단어리스트들에 대응하는 폰열(Phone list)을 나열한 파일로서 한 라인에는 폰 하나씩 적는다. <br>
문장의 시작에는 반드시 sil로 시작하고, 문장의 끝에도 sil로 끝난다.
파일의 끝에는 마침표(.)로 문장의 끝을 구분한다.
* phones0.mlf
<pre>
#!MLF!#
"*/s0001.lab”
sil
N
AA
N
WW
N
H
AA
KQ
KK
OW
EY
K
AA
N
D
A
sil
.
"*/s0002.lab"
N
....
</pre>

phones0.mlf에 적힌 폰열(phone list)은 같다. <br>
일반적으로 사전 단위의 구분에 따라 Q를 삽입하거나, 어절단위에 Q 삽입, 끊어읽기에 따라 Q를 삽입하여 스크립트를 작성한다.<br>
어떻게 구분짓는 것이 인식률에 가장 좋은지는 아직 실험결과를 얻지 못한 상태이다.
일반적으로 영어권에서는 단어단위가 끊어읽기 단위이기 때문에 한국어에서 고려되어야하는 사항과는 다르다.
* phones1.mlf 
<pre>
#!MLF!#
"*/s0001.lab”
sil
N
AA
N
WW
N
Q
H
AA
KQ
KK
OW
EY
Q
K
AA
N
D
A
sil
.
"*/s0002.lab"
N
....
</pre>


## 4. 발음사전(Pronunciation Dictionary) 만들기

HTK에서 제공하는 HDMan 명령어는 영어 음성사전 작성에 사용되는 것으로 사전 생성에는 해당 단어가 어떻게 발음되어지는지 폰 열들이 기록되어진다. <br>
이런한 사전 생성 방식은 한국어 발음사전을 만들기에는 부적합하다. 
한국어의 경우 디코딩 단위(사전의 엔트리 단위 = lexical 단위)가 영어권과 다르며, 어절간이나 단어사이에 일어나는 발음변이 현상을 반영하는 방법이 고려되어져야 한다. 
이렇게 고려되어진 한국어 음운변화 현상을 반영하여 한국어 음소 Set에 맞는 사전 생성 프로그램을 작성하여 사용하여야 한다.

인식단위가 적은 경우, 직접 손으로 사전을 작성해도 되지만, 인식 어휘가 크게 증가하는 경우 자동 생성 프로그램이 필요하게 된다. 
공개된 발음생성 프로그램이 있느냐는 질문을 많이 받는데, 사실 국내에서는 찾아보기 힘들다. 
다만 관련 논문들을 참조하여 직접 작성하는 것을 권장한다. 
일반적으로 대학 연구실에서는 각기 연구실 내에서 사용하는 프로그램이 있으나 공개용은 구하기 힘들고, 
직접 문의를 하거나 대상 어휘를 주고 사전생성 프로그램 결과를 받는 방법은 가능할 듯하다.

* 발음사전 예제
<pre>
 -----------------
 표제어  PLU List
 -----------------
 간다   K AA N T AA
 나    N AA
 는    N WW N
 에    EY
 온다   OW N T AA
 있다   IY TQ TT AA
 집    Z IY PQ
 학교   HH AA KQ K JO
</pre>


## 5.  HLED 명령어와 발음사전을 이용하여 트랜스크립션작성하기


## 6. 음성 특징 추출 (Feature Extraction) : step5 - Coding the Data


## 7. Monophone 생성 단계


## 8. Silence와 Short-pause 결합


## 9. Tied-State Triphone 생성 단계


## 10. Word network 생성 방법

## 11. 인식 단계


## 12. 인식 결과 평가

