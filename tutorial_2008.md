# 음성인식용 HTK 사용법 

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
```
 0 4500000 sil사
 4500000 5000000 N
 5000000 5500000 AA
    …
 12500000 13000000 A
 13000000 13500000 sil
 
 (100ns단위)
```

* Xwaves 레이블 형식
```
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
```
 
 
## 3. 트랜스크립션(Transcription) 파일 작성
 
인식할 문장들을 디코딩 단위로 나누어 리스트를 작성한다.
* wlist (인식 단위 리스트)

>> $sort words | uniq > wlist

인식하고자하는 단어리스트들을 나열한 파일로서 한 라인의 단위는 인식하고자하는 디코딩 단위와 발음사전과 일치하여야 한다.
* words.mlf
```
 #!MLF!#
 "*/s0001.lab”
 나
 는
 학교
 에
 가
 ㄴ다
 .
 "*/s0002.lab"
 나
 ....
```

인식하고자하는 단어리스트들에 대응하는 폰열(Phone list)을 나열한 파일로서 한 라인에는 폰 하나씩 적는다. <br>
문장의 시작에는 반드시 sil로 시작하고, 문장의 끝에도 sil로 끝난다.
파일의 끝에는 마침표(.)로 문장의 끝을 구분한다.
* phones0.mlf
```
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
```

phones0.mlf에 적힌 폰열(phone list)은 같다. <br>
일반적으로 사전 단위의 구분에 따라 Q를 삽입하거나, 어절단위에 Q 삽입, 끊어읽기에 따라 Q를 삽입하여 스크립트를 작성한다.<br>
어떻게 구분짓는 것이 인식률에 가장 좋은지는 아직 실험결과를 얻지 못한 상태이다.
일반적으로 영어권에서는 단어단위가 끊어읽기 단위이기 때문에 한국어에서 고려되어야하는 사항과는 다르다.
* phones1.mlf 
```
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
```


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

발음 사전의 표제어와 words.mlf에 라인별로 기재된 단어를 매치하여 phones0.mlf와 phones1.mlf를 생성하는 명령어이다. 
사전에는 해당 표제어에 대한 발음열 리스트가 기재되어 있고, 영어권의 경우, 단어 단위가 디코딩 단위이므로 일대일 매칭으로 트랜스크립션을 얻을 수 있다.
한국어의 경우 디코딩 단위에 따라서 이러한 사전과 단어열의 일대일 매칭이 올바른 학습용 발음열 트랜스크립션을 얻기 어렵기 때문에, 한국어에 맞는 학습용 발음열 생성기가 필요하다.

* mkphones0.led
```
 EX
 IS  sil  sil
 DE  Q
```

* mkphones1.led
```
 EX
 IS  sil  sil
```

>> $ HLEd -l '*' -d dict -i phones0.mlf mkphones0.led words.mlf

>> $ HLEd -l '*' -d dict -i phones1.mlf mkphones1.led words.mlf


## 6. 음성 특징 추출 (Feature Extraction) : step5 - Coding the Data
사람의 음성을 인식하기 위해서는 음성의 특징들을 뽑아 사용합니다. 
입력된 음성 파일로부터 특징 추출 과정을 수행하는 단계로 일반적으로 LPC나 MFCC를 사용합니다. 
HTK에서 제공하는 feature 타입은 책을 참고하세요.

* 예) codetr.scp
<pre>
 /user/speech/DB/s0001.wav   /user/speech/DB/s0001.mfc
 /user/speech/DB/s0002.wav   /user/speech/DB/s0002.mfc
 ...                         ...
</pre>

* config파일
```
 # Coding parameters
 TARGETKIND=MFCC_0
 TARGETRATE=100000.0
 SAVECOMPRESSED=T
 SAVEWITHCRC=T
 WINDOWSIZE=250000.0
 USEHAMMING=T
 PREEMCOEF=0.97
 NUMCHANS=26
 CEPLIFTER=22
 NUMCEPS=12
 SOURCERATE=625
 TARGETLABEL=HTK
 TARGETFORMAT=HTK
 NONUMESCAPES=T 
```

__부가설명__

SOURCEFORMAT=NOHEAD (헤더가 없는 경우) <br>
SOURCEFORMAT=WAVEFORM (웨이브 형식인 경우) <br>
NONUMESCAPES = T는 프린트할때(WriteString, ReWriteString) 한글이 8bit 숫자로 바뀌지 않도록 하는 옵션.

* Configuration File (16bit, 16KHz 음성 파일 기준)
```
 TARGETKIND=MFCC_0_D_A
 #SOURCEFORMAT=NOHEAD
 ...
```

>> HCopy -T 1 -C config -S codetr.scp


## 7. Monophone 생성 단계

각 명령어를 실행하기 전에 다음 파일 및 필요한 디렉토리 hmm0 ~ hmm15, lib를 미리 생성해야 한다. 
'mkdir ~'...  HTK 자체에서 자동으로 디렉토리를 생성해 주지 않는다. 
디렉토리가 없어도 실행은 되지만, 지정한 디렉토리에 계산된 값을 저장할 위치를 찾지 못한 상태로 프로그램이 종료되면 다시 실험을 수행해야하므로 반드시 디렉토리가 있는지 없는지 확인한 다음, 실험을 수행한다. 
또한 실험에 사용되는 파라메터 파일 및 스크립트가 필요하다.

* Master Macro File(MMF)
* proto
```
~o <VecSize> 39 <MFCC_0_D_A>
~h "proto"
<BeginHMM>
   <NumStates> 5
   <State> 2
      <Mean> 39
        0.0  0.0  0.0  0.0  …
      <Variance> 39
        1.0  1.0  1.0  1.0  …
   <State> 3
      <Mean> 39
        0.0  0.0  0.0  0.0  …
      <Variance> 39
        1.0  1.0  1.0  1.0  …
   <State> 4
      <Mean> 39
        0.0  0.0  0.0  0.0  …
      <Variance> 39
        1.0  1.0  1.0  1.0  …
   <TransP> 5
        0.0  1.0  0.0  0.0  0.0
        0.0  0.6  0.4  0.0  0.0
        0.0  0.0  0.6  0.4  0.0
        0.0  0.0  0.0  0.7  0.3
        0.0  0.0  0.0  0.0  0.0
<EndHMM>
```

* hmm0/hmmdefs
```
~h “sil”
<BeginHMM>
   <NumStates> 5
   <State> 2
      <Mean> 39
        0.0  0.0  0.0  0.0  …
      <Variance> 39
        1.0  1.0  1.0  1.0  …
   <State> 3
      <Mean> 39
        0.0  0.0  0.0  0.0  …
      <Variance> 39
        1.0  1.0  1.0  1.0  …
   <State> 4
      <Mean> 39
        0.0  0.0  0.0  0.0  …
      <Variance> 39
        1.0  1.0  1.0  1.0  …
   <TransP> 5
        0.0  1.0  0.0  0.0  0.0
        0.0  0.6  0.4  0.0  0.0
        0.0  0.0  0.6  0.4  0.0
        0.0  0.0  0.0  0.7  0.3
        0.0  0.0  0.0  0.0  0.0
<EndHMM>

~h “IY”
<BeginHMM>
<NumStates> 5
   <State> 2
      <Mean> 39
        0.0  0.0  0.0  0.0  …
      <Variance> 39
        1.0  1.0  1.0  1.0  …
…
```

* train.scp
<pre>
/user/speech/DB/s0001.mfc
/user/speech/DB/s0002.mfc
…
</pre>

__초기 mean과 variance 값 계산__

>> HCompV -C config1 -f 0.01 -m -S train.scp -M hmm0 proto

scan a set of data file / global mean과 variance 계산 
모든 gaussian에 같은 mean과 variance 할당

HCOMPV 수행 후, hmm0/vFloors를 복사하고 수정하여 macros를 작성한다.

* hmm0/vFloors (예)
```
<Variance> 39
 1.361697e-01 5.078497e-02 … 3.495922e-04 
```
* macros
```
~o
<VecSize> 39
<MFCC_0_D_A>
~v varFloor1
<Variance> 39
 1.361697e-01 5.078497e-02 … 3.495922e-04
```

#### Monophone 모델 학습 (반복)

>> HERest -T 1 -C config1 -I phones0.mlf -t 250.0 150.0 1000.0 -S train.scp -H hmm0/macros -H hmm0/hmmdefs -M hmm1 monophones0 

>> HERest -T 1 -C config1 -I phones0.mlf -t 250.0 150.0 1000.0 -S train.scp -H hmm1/macros -H hmm1/hmmdefs -M hmm2 monophones0

>> HERest -T 1 -C config1 -I phones0.mlf -t 250.0 150.0 1000.0 -S train.scp -H hmm2/macros -H hmm2/hmmdefs -M hmm3 monophones0


 * 학습하고자하는 문장수가 많은 경우, 
 
파라메터의 값이 일정 임계치를 넘지 못해서 생기는 에러가 발생한다.(HTK Error solution 참고) <br>
이런 경우 병렬로 학습 문장들을 나누어서 학습한다. <br>
아래 예는 train-part1.scp와 train-part2.scp로 나누어 병렬 학습 후, HMM 모델을 합치는 방법이다.

>> HERest -T 1 -C config1 -I phones0.mlf -t 250.0 150.0 1000.0 -S train-part1.scp -H hmm0/macros -H hmm0/hmmdefs -M hmm1 -p 1 monophones0 

>> HERest -T 1 -C config1 -I phones0.mlf -t 250.0 150.0 1000.0 -S train-part2.scp -H hmm0/macros -H hmm0/hmmdefs -M hmm1 -p 2 monophones0

>> HERest -T 1 -C config1 -t 250.0 150.0 1000.0 -H hmm0/macros -H hmm0/hmmdefs -M hmm1 -p 0 monophones0 hmm1/*.acc


## 8. Silence와 Short-pause 결합

우선 hmm3/hmmdef와 hmm3/macros를 hmm4/ 디렉토리로 복사한다.
hmm4/hmmdefs에서 ‘sil’의 state3 부분을 ‘Q’의 state2로 복사하고, 다음과 같이 수정한다. (그림 3.9 참조)

* sil.hed
```
AT 2 4 0.2 {sil.transP}
AT 4 2 0.2 {sil.transP}
AT 1 3 0.3 {Q.transP}
TI silst {sil.state[3],Q.state[2]}
```


## 9. Tied-State Triphone 생성 단계

* mktri.led
```
WB  Q
WB  sil
TC
```

wintri.mlf는 학습에서 사용된 triphone list로 aligned.mlf를 이용하여 재구성된 트랜스크립션 파일이다. 
HLEd를 사용하면 wintri.mlf와 triphones0가 자동으로 생긴다.

>> HLEd -n triphones0 -l '*' -i lib/wintri.mlf mktri.led aligned.mlf 
 
>> % mktrihed < monophones0 > mktri.hed

*  mktri.hed (예)
```
CL triphones0
TI T_IY {(*-IY+*,IY+*,*-IY).transP}
TI T_EY {(*-EY+*,EY+*,*-EY).transP}
TI T_EH {(*-EH+*,EH+*,*-EH).transP}
…
TI T_L {(*-L+*,L+*,*-L).transP}
TI T_R {(*-R+*,R+*,*-R).transP}
TI T_LI {(*-LI+*,LI+*,*-LI).transP}
```

>> HHEd -B -H hmm9/macros -H hmm9/hmmdefs -M hmm10 mktri.hed monophones1

트라이폰 모델 학습

>> HERest -T 1 -C config1 -I lib/wintri.mlf -s stats -t 250.0 150.0 1000.0 -S train.scp -H hmm10/macros -H hmm10/hmmdefs -M hmm11 triphones0

>> HERest -T 1 -C config1 -I lib/wintri.mlf -s stats -t 250.0 150.0 1000.0 -S train.scp -H hmm11/macros -H hmm11/hmmdefs -M hmm12 triphones0

* tree.hed - Question set (예) / 자음의 경우, 음의 분류기준에 따라 state-tying.
```
RO 100.0 stats
TR 0
QS "L_Stop-Beg"  {P-*,B-*,PQ-*,PP-*,PH-*} 
QS "L_Stop-Mid"  
{D-*,TQ-*,T-*,TT-*,TH-*} 
QS "L_Stop-End"  {K-*,G-*,KQ-*,KK-*,KH-*} 
QS "R_Stop-Beg"  {*+P,*+B,*+PQ,*+PP,*+PH} 
QS "R_Stop-Mid"  {*+D,*+TQ,*+T,*+TT,*+TH} 
QS "R_Stop-End"  {*+K,*+G,*+KQ,*+KK,*+KH}
QS "L_Affr"      {TS-*,JH-*,TST-*,CH-*} 
QS "L_Fric"      {S-*,SH-*,SS-*,SHS-*} 
QS "L_Nasal"     {M-*,N-*,NI-*,NX-*} 
QS "L_Liquid"    {R-*,L-*,LI-*} 
QS "R_Affr"      {*+TS,*+JH,*+TST,*+CH} 
QS "R_Fric"      {*+S,*+SH,*+SS,*+SHS} 
QS "R_Nasal"     {*+M,*+N,*+NI,*+NX} 
QS "R_Liquid"    {*+R,*+L,*+LI} 
QS "L_IY"       {IY-*} 
… 
QS "L_R"        {R-*} 
QS "L_LI"       {LI-*} 
QS "R_IY"       {*+IY}
```

Question을 가지고 tiedlist를 만드는 과정

>> HHEd -H hmm12/macros -H hmm12/hmmdefs -M hmm13 tree.hed triphones0 > log

Tiedlist 트라이폰 모델을 학습하는 과정

>> HERest -T 1 -C config1 -I lib/wintri.mlf -s stats -t 250.0 150.0 1000.0 -S train.scp -H hmm13/macros -H hmm13/hmmdefs -M hmm14 tiedlist

>> HERest -T 1 -C config1 -I lib/wintri.mlf -s stats -t 250.0 150.0 1000.0 -S train.scp -H hmm14/macros -H hmm14/hmmdefs -M hmm15 tiedlist


## 10. Word network 생성 방법

FSN wdnet 작성: 의미문법 gram 작성 (HTK book 참조)

>> HParse  -C  config1  gram  wdnet

back-off bigram : 통계적 언어모델

>> HLStats -b bigfn -o wlist words.mlf

>> HBuild -n bigfn wlist wdnet


## 11. 인식 단계

일반적으로 학습 데이터와 테스트 데이터를 분리하여 인식 실험을 한다. 
간단하게 동작이 잘 되는지를 확인하기 위해 학습 데이터를 이용해 확인하는 경우도 있다. 
테스트 데이터를 선정하는 방법 또한 다양하다. 
선별된 테스트 데이터는 학습시 발생한 트라이폰 리스트에 없는 경우가 생겨 네트웍 loading시 에러가 나는 경우가 발생한다. <br>
이때에는 tiedlist를 만드는 부분부터 다시 확인하여 테스트 데이터 내에 생길 수 있는 트라이폰 리스트가 모두 포함될 수 있도록 한다.

>> HVite -C config1 -T 33 -H hmm15/macros -H hmm15/hmmdefs -S train.scp -l '*'-o ST  -i recout.mlf(출력화일) -w wdnet(언어모델 네트웍) -t 200(beam threshold) -p 0.0 -s 5.0 dict(인식사전) tiedlist(HMM리스트)

>> HVite -C config1 -T 33 -H hmm15/macros -H hmm15/hmmdefs -S test.scp -l '*'-o ST  -i recout-test.mlf -w wdnet -t 250(beam threshold) -p 0.0 -s 5.0 dict tiedlist

Option 설명 : <br>
  -t : Beam threshold 설정, default는 무한 대여서 인식 시간이 무지 오래걸립니다. 경험치로 200~250 정도면 큰 차이없이 결과를 얻을 수 있을겁니다. 
  숫자가 적을수록 빔 폭이 작은 대신 인식속도는 빨라집니다.
  -s : Langage Scale Factor 
  -p : Insertion Penalty
  

## 12. 인식 결과 평가

testref.mlf는 인식할 문장 순서대로 정답을 기술한 내용이 들어있어야 한다. (words.mlf와 비슷) <br>
recout.mlf는 인식결과 파일로서, SILENCE 등은 제외하고 인식결과를 평가한다. (-z SILENCE) <br>
(단, 이 프로그램을 돌리기 위해서는 위 HVITE에서 -o ST 옵션이 주어져야만 됨.)

>> HResults  -I testref.mlf  tiedlist  recout1.mlf

