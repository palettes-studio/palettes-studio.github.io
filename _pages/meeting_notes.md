---
permalink: /meeting_notes/
title: "MeetingNotes"
excerpt: "매주 진행되는 맥락 회의 노트"
modified: 2019-06-09
search: true
header:
  image: /assets/images/LAB_CONTEXT.png

---

## 이전 기수 회의 노트보기

- [2기](https://www.ai-lab.kr/labs/maegrag-raebjang-ganguram)
- [3기](https://www.ai-lab.kr/labs/maegrag-raebjang-ganguram-1)
- [4기](https://www.ai-lab.kr/labs/maegrag-raebjang-ganguram-2)

## 이번 기수 회의 노트 

[5기 링크](https://www.ai-lab.kr/labs/maegrag-raebjang-ganguram-3)

### [1차 미팅]

첫 미팅을 가졌습니다! 5기에는 새로운 연구원인 박산희연구원과 채민기 연구원이 합류하게 되었습니다.

이번 기수에는 이전 기수의 활동 경험을 토대로 메인 프로젝트로 정하여 진행하기로 하였습니다.  메인 프로젝트 주제는 **'고문서 복원 프로젝트'** 입니다.

**1차 미팅**에서 논의된 사항은 다음과 같습니다.

- **데이터 관련** 

  조선왕조실록 데이터 크롤링 하기로 계획 

- **모델 관련** 

  이것저것 찾아보는 중...

- **미팅 장소 관련**
  매주 7시30분 부터 합정 센터에서 미팅을 가지기로 함. 

------

### [2차 미팅]

두번째 미팅을 가졌습니다. 고성능컴퓨터지원 사업에서 지원 받은 서버가 열렸습니다.

- **데이터 관련**

  조선왕조실록 데이터 크롤링이 완료 되었습니다.  서버에 대략 10만장의 데이터가 저장되었으며 사진 사이즈는 3700 * 2400입니다. 

- **이미지 전처리** 

  이미지 해상도가 상당히 커 학습을 돌리기 위해 이미지 처리 방향에 대해서 논의해보았습니다.  Batch 이미지 크기 후보군을 정했으며 다음과 같은 사이즈가 후보군으로 나왔습니다. 

  ​                                       [512x512 / 384x384 / 256x256]

  또한 조선왕조실록 이미지의 테두리를 처리할 방법을 논의하였고 김준화연구원님께서 처리코드를 작성해주셨습니다.

  1. img read
  2. img resize to 512 x512   
  3. 행 별 픽셀 값을 더해서 검정 테두리 LINE의 행 번호 위 아래 얻음.                          
  4. 512X512와 원본 이미지의 W,H에 일차 변환으로 비례해서 행 번호 위 아래를 구함.  

- **다음 미팅까지..**

  공부해보고자 하는 모델 하나 찾아오기 (단 찾았을때 공유 하고 중복 안됨) 

  두번째 아무거나 GPU에 돌려놓고 오기

  논문 스터디 시작하고 공부해보고자하는 모델의 베이스, 모르는 논문부터 공부해오기로 했습니다. 

- **이번 기수 목표**

  이미지 복원 -> 이미지 OCR -> 자연어 문맥 -> 없는거나 너무 깨진부분 채워넣기

------
### [3차 미팅]
중간 발표 전 모임입니다. 발표 자료 작성 및 실험 결과를 공유했습니다. 

- **발표 자료 작성**
  1. Context 5기 계획
  2. 진행 이슈
  3. 향후 계획 
  
- **실험 결과 공유**
  수집된 데이터셋과 context encoder 베이스 라인으로 center masking을 이용하여 train 진행 
  
  ![image](https://user-images.githubusercontent.com/26568793/61384670-061fb400-a8ec-11e9-82bb-b40d27cd9270.png)
  
------
### [4차 미팅]
랩장이 강우람님에서 김인수님으로 변경되었습니다. 
논문 리뷰와 산희님이 진행한 베이스모델 설명, 그리고 향후 연구 방향에 대해 논의하였습니다. 

- **모델 어디까지 진행되었나?** 
  1. context encoder 논문 리딩 --> 공식코드(우리가 본 샘플 이미지) --> 모델에 partial conv 붙이는중
  2. 모델 개선 사항 
      * input => 256 결과로 바꿔볼 것. 
      * base line, context encoder // U-net구조?  
      * 정사각형의 마스크 모양 => 자유 모양 마스크로 변형 
      * convolution => partial convolution 바꿔볼 것 => 현재 베이스라인의 변화 확인 
      * convolution system에 attention 추가해볼 것.
      
- **다음주까지**
  1. Partial Convolution
     해당 논문을 읽고 DOCS에 정리 및 colab에 코드 주석을 달고 있습니다. 
     해당 문서를 토대로 블로그에 포스팅할 예정입니다. 
     * Convolution에 마스크를 적용한 형태임
     * Loss가 6개나 되는 Loss를 합쳐서 구성.
     * 이중 Style loss는 다른 논문에서 왔다.
     
   2. 각자 
     * 인수님: 방향잡기
     * 은영님: partial conv 공부
     * 준화님: partial conv 파일 공유하여 주석달면서 공부, Ltotal 문서 정리 
     * 미래님: 진도 따라잡기, 블로그 글 작성하기, 회의록 올리기
     * 산희님: 베이스 모델 개선
     
- **앞으로의 방향**
코딩, 논문 리딩 위주로 진행됩니다. 못하면 함께 모여서(온/오프) 논문 읽고 디스커션

------
### [5차 미팅]
오늘의 회의내용 :
#Pconv2d 코드주석리뷰, #ImageInpainting_Partialconvolution논문_구글닥스 정리내용 리뷰
#아카이브 안내,#다음주에_각자_더_공부할_내용_논의
• Pconv2d코드 주석 리뷰
- raw_out = W * (X ⊙ M)
- self.mask_ratio = sum(1)/sum(M)
- output : 위 두 개를 곱한 것
• Further Research
- Partialconvolution 논문 전체 코드 분석(generator,discriminator, loss, unet구조 등)
- loss 중 Pconv와 밀접하게 연결된 Loss항은 어딘가?
• 아카이빙을 기다립니다!
Image inpainting_Pconv 논문 코드에 주석달기
colab 주소 :
• 다음주까지
1. Pconv 의 다음의 두 개 코드에 주석 달기
(1)loss
https://github.com/bobqywei/inpainting-partial-conv/blob/master/loss.py
우리가 논문에서 봤던 ＊각종loss＊들을 코드로 구현해둬서 연구가치가있어요
(2)network
https://github.com/bobqywei/inpainting-partial-conv/blob/master/partial_conv_net.py
(우리가 본 conv2d 코드와 다르게 느껴지는데 forward 부분은 비슷! 그리고 U-net *네트웍* 구조 부분 코드도 있고 더 좋아요.)
2. 각자 공부하기
미래 : 코드 리뷰 & 로스 적용해 볼 수 있으면 적용해 보겠습니다.
준화 : 코드 주석달기
인수 : 코드 주석달기, 서버접속... 서버세팅..
3. 아카이빙하기
-• 앞으로 방향 :
*Pconv 논문 코드* 연구하고 context encoder 베이스라인 모델에 본격적으로 뛰어들어 pconv 붙여보자!!!!
• 아카이브
1. 블로그
2. colab
3. 공식 깃헙
4. 우리 맥락랩 서버의 코드
5. 구글닥스
https://docs.google.com/document/d/1TBi2wajWoNI_fNP5Upjcinx9cQSEbfyypKUptCd5SYE/edit
6. 위 모든 링크들을 정리하는 스프레드시트

------
### [7차 미팅]

오늘의 회의 내용 :
#Pconv2d 도깨비 공부방 리뷰, #아카이빙의 중요성, #캐글 코드는_서버에_있다, #unet+Pconv_코드도_서버에_있습니다, #심지어Pconv_context_encoder_코드도_서버에서_개발중, #산희님..진짜..실행력과..야망을_존경해요, #이러다산희님준화님두분이다개발해버리면어쩌나,#고문서복원_코드개발_지금_막차타세요,#캐글 21일남았는데_실화인가요?,#저에게허드렛일을주십시오(??),이번주도_불타올라봅시다
• 지난번 모임후 연구진전
Pconv loss 코드 이해 중.
perceptual loss와 style loss는 vgg16을 이용해 이미지 feature를 분석하는 코드입니다.
각종 로스중 partial conv와 가장 관련이 깊은 것은 L_valid, *l_hole*로 판단됩니다.
다만 다른 로스들도 중요. 특히 style loss같은 경우 pconv로 복원한 이미지의 스타일을 따집니다.
준화님께서 loss 코드에 주석 달아주심,
캐글 진행중(준화님이 더 해볼만한거 알려 주시기로함)
• 다같이 블로깅합시다!
[Harvard] Annotated Transformer-
http://nlp.seas.harvard.edu/2018/04/03/attention.html
이런 형태로 논문+코드 합쳐 블로그에 정리하면 > 이해한 게 잘 간직될듯.
사실.. 블로그 쓰기 기초 튜토리얼이 필요.. @mirae 미래님 언제한번 짧게 시연해주세여!
• 고문서 관련 정리
기수말 깔쌈한 마무리를 어떻게 하면좋을까
1. 고문서(그림) -> 단순 inpainting 적용 가능할 듯
>16장의 고문서 그림으로 시도해보겠음 @sanheepark
2. 고문서(글자) -> 훼손 정도 작으면 이것도 inpainting 적용 가능할 듯
>조그맣게 마스크 뚫어서 해보겠음  @sanheepark
3. 고문서(글자) 훼손정도 크면 OCR 또는 글자 Detection해서 index로 추출. 
>OCR, AR, GAN, .... 아직 모름!
• 더 공부해 볼 거리 : chinese OCR
https://github.com/A-bone1/Attention-ocr-Chinese-Version
• 이번주 : kaggle부터 집중을 해보자.
목표는 앙상블과 허드렛일(data augmentatn)로 분류정확도 90% 뚫기
허드렛일 시켜주세요 @Insoo Kim
• 그러면 고문서복원은요?
1) context encoder : 산희님께서 결과 파일 공유해주심. pconv + unet으로 베이스라인 전환
2) pconv +  u-net : 산희님께서 결과 파일 공유해 주실 예정임
3) nvidia pconv + context encoder : 준화님께서 하고 계시다고함
:context/context encoder ~ /-p.py 붙은 파일 서버에서 확인해 보시면 돼요
• 다들 이번주 뭐해요?
@sanheepark 산희님 : 글자단위 마스크 해보기,
@Insoo Kim 인수님: 전.. 약 6일간 휴가입니다. 산희님 코드 주석을 달고 캐글을 하겠습니다. 준화님 허드렛일주세요.
@Junhwa 준화님 : Pconv + context encoder, @Insoo Kim 인수김에게 허드렛일주기
@mirae 미래님: 블로그 글쓰는 기초 튜토리얼 정리, efficient 논문 리딩

------
### [8차 미팅]
08.02 회의록
1. kaggle 차종 분류 대회
- 대회종료 D-14, 준화님이 올리신 csv 파일이 리더보드 24등을 달리고 있습니다 /
- 단일모델 가장 성능이 잘 나오는 것은 fixresnet 모델. /
+ 제가 Inception v3이 train 성능이 이상하게 잘나온다고 주장해서 모두를 설레게 하는 해프닝이 있었는데 08.03 오전 test해본 결과는 역시 fixresnet이 우월했습니다. (민망ㅋㅋㅋ)
2. 고문서 복원 > Neurlips workshop 논문 제출 제안
- 유명 학회인 Neurlips(Nips의 새이름) 의 Workshop중 하나인
- ML for Creativity & Design workshop에서는 Creativity와 Design 관련 머신러닝을 이용한 논문을 모집합니다.(제출데드라인 : 9.9)
- 고문서 복원 프로젝트도 여기 제출함직한 주제를 다룹니다.
- 위에 준화님이 올려주신 논문들이 작년 Creativity & Design workshop 논문들이니 참고하시면 돼요
- 경험과 동기부여, 커리어 등을 위해서 최고의 기회! 가볍게 논의후 도전하기로 했습니다!
- 특이사항 : 작성시 LaTeX(논문작성프로그램)를 이용해야 합니다.
- 다음 오프모임(8/11,일 오후 3-4시 시작)때 다같이 논문에 대해 논의하고 LaTeX도 공부할게요!
- 오시기로 하신 분들 반드시 오프라인으로 참여해 주세요!   (현재 준화,산희,인수 참여예정)
3. 나갈만한 대회들 모음
- 빵빵한 서버를 갖고 있으니, 어느 대회든 참여해 보자는 이야기가 제안되었습니다
- Dacon, Kaggle 등에서 나갈 만한 대회가 있으면 제안해 주세요~
- + Funda 에서 하는 상점매출 예측 대회도 8.30까지인데, 데이터 다루기를 공부수 있을만한 주제네요!
- 이중 퓨처파이낸스 AI챌린지 대회는
- 개발을 요하지 않고 가볍게 참가해볼 만한 가치가 있는 아이디어 챌린지로,
- 참여의사가 있으신 분들을 모아 참여해 볼까해요.
- 대회 채널도 만들었습니다.
- 더 참여하실 분들은 저에게 참여 의사를 밝혀주세요! (현재 준화, 산희, 인수 참여예정)
- 다음주 오프라인 모임에 만나면 이 대회도 논의, 작성후 마무리해서 제출할 거예요!
4. 이번주에 할일
- 캐글 대회에 신경써서 마무리! 각자 모델 돌려보고 앙상블 해보구요.
- 고문서복원 프로젝트를 들어가기 위해 각자 공부를 해오기로 해요
- 저는 부족한 부분인 Context encoder를 읽으려구요/
- 페이퍼 읽고 고문서복원에 쓰기위한 재료나 insight를 모아보는 한 주로 합시다!
5. 블로그 / 깃헙
- 회의록은 역시 블로그에 정리해야 제맛.. 정리해서 블로그에 회의록을 올려보아요
- 깃헙엔 큰 활동들을 정리해서 업데이트
