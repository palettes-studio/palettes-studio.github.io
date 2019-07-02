---
title: "[고문서복원]Context Encoder 논문리딩"
excerpt: "2019. 06. 26.  Intro"
search: true
categories: 
  - restoration
---
## Context encoder (한글해석)

[논문주소](https://arxiv.org/pdf/1604.07379.pdf)

#### 1. 논문 요약 및 소개 

이미지의 일부분이 누락되어있어도 사람은 비워져있는 부분을 유추할 수 있다. 해당 논문에서는 context encoder라는 개념을 이용하여 해당 문제를 해결하고자 한다. 

논문에서 제안하는 context encoder는 context 기반의 픽셀 예측을 통해 비지도 학습된 이미지 학습 알고리즘이다. Auto encoder와 유사한 형태이며, 이는 주변 환경에 조건화된 임의 영상 영역의 콘텐츠를 생성하도록 훈련된 복잡한 신경 네트워크이다.  Auto encoder는 input image를 이용하여 장면의 compact한 표현을 얻기 위한 목적으로 low-dimensional 'bottleneck' 레이어를 통과한후 재구성한다. 하지만 이 feature  respresentation은 늘 의미있는 표현을 학습하지 않고 영상내용을 압축할 수 도 있는 한계를 가지고 있다. 

누락된 부분을 채우는 작업은 전체 이미지의 내용을 이해하고 누락된 부분에 대해서 그럴듯한 가설을 만들어야한다. 이를 위해 context encoder를 train할 때 표준 픽셀 단위 재구성 손실과 재구성 및 상대적 손실을 모두 실험했다고한다. (experimented with both *a standard pixel-wise reconstruction loss,*as well as *a reconstruction plus an adversarial loss*)  후자의 경우 출력의 여러 모드를 더 잘 처리 할 수 있기 때문에 훨씬 더 sharper한 결과를 출력한다. 더 나아가 context encoder는 외형뿐만이 아닌 시각 구조의 의미도 학습한다는 것도 발견했으며, semantic inpainting 작업을 할 수 있다는 것을 발견했다.  해당 문제를 CNN을이용해서 해결한다고 한다. 

Denosing Auto encoder는 입력 이미지를 손상시키고 네트워크에서 손상을 복구하도록 요구함으로써 이 문제를 해결한다.  하지만 이러한 corruption(손상)과정에은 매우 지역적이고 low-level 이며 원상태로 복구하기 위해 semantic한 정보를 요구하지 않는다.  이와는 대조적으로, context encoder는 훨씬 더 어려운 작업 즉, 근처 픽셀에서 "힌트"를 얻을 수 없는 이미지의 큰 누락된 영역을 채워야 한다. 이는 장면에 대한 훨씬 더 깊은 의미적 이해와 높은 수준의 특징을 합치는 능력을 필요로 한다.  이는 문맥이 주어진 단어를 예측하여 자연어 문장으로 부터 단어표현을 배우는 word2vec과 유사하다고 볼 수 있다.

context encoder는 이미지의 context를 compact한 latent feature 표현으로 capture하는 Encoder와 누락된 이미지의 컨텐츠를 생성하기 위한 Decoder로 구성된다. Autoencoder와 동일하게 비지도학습으로 진행되고,  해당 이슈를 해결하기위해서 모델이 이미지의 content를 이해하는 것 뿐만이 아닌,  사라진 부분에 대해서 그럴듯한 가설을 생성해야한다. 하지만 이 과제는 본질적으로 multi-modal이다. 누락된 영역을 채우는 동시에 주어진 맥락과 일관성을 유지하는 방법이 여러 가지 있기 때문이다. *context encoder를 공동으로 훈련하여 reconstruction loss과 adversarial loss을  최소화함으로써 이러한 burden(위에서 언급한 이슈)를 제거한다.*  L2라 언급하는 reconstruction loss는 손실된 부분의 문맥적 관계의 구조를 전반적으로 capture하며, 반면에 adversarial loss는 distribution에서 특정 mode를 선택하는 효과가 있다.

또한 *encoder와 decoder를 독립적으로 평가한다. encoder 측면에서는 image patch의 context만 인코딩하고 resulting feature을 사용하여 데이터 세트에서 가장 가까운 인접 컨텍스트를 검색하면 원래 패치와 의미적으로 유사한 패치가 생성된다는 것을 보여준다.*  분류, 개체 감지 및 의미 분할을 포함한 다양한 이미지 이해 작업에 대해 인코더를 미세 조정하여 학습된 형상 표현의 품질을 더욱 검증한다.디코더 측면에서는 해당 방법이 종종 현실적인 영상 내용을 채울 수 있다는 것을 보여준다. 실제로 해당 방식은 semantic hole-filling(즉, 큰 누락 영역)에 대해 합리적인 결과를 제공할 수 있는 최초의 파라메트릭 인페인팅 알고리즘이다. 또한 context encoder는 또한 non parametric 인페인팅 방법으로 가장 가까운 이웃을 계산하는 데 더 나은 시각적 특징으로 유용할 수 있다.



#### 2. 관련 연구 (생략)

------

#### 3.Context encoders for image generation

##### 3.1 context encoder



![image](https://user-images.githubusercontent.com/26568793/60474140-3e4bb380-9cab-11e9-85cd-f0c88a6b6696.png)

#####  (1) Encoder - Decoder pipline

  

######      1-1. Encoder

​     Encoder의 역할은 위의 그림처럼 없어진 부분을 갖고 있는 이미지를 latent vector(6x6x256)로 만든다. 

- Alexnet구조로 5개의 conv + pooling layers로 이루어진다. 

  기존 Alexnet이 분류를 학습했다면 여기서는 scatch로 표현될 수 있는 초기 random weights들로 context를 예측하는 것을 학습했다. 

- Convolution layers가 서로 fully conntected 되어있다. 특정 로컬 영역에 대해서 뉴런들이 보는 것이 아닌 모두 연결되어 가중치를 공유하는 형태 

######      1-2. Channel-wise fully-connected layer

- 기본적으로 그룹과 fully-conntected된 Layer, 각각의 feature map내의 activation에 information propagation을 목적으로 함. 
- 굳이 이 Layer을 사용한 이유는 위에서 Encoder에 대한 방대한 정보를 Decoder에 전달하기 위함. 
- 각 feature map 마다 활성함수를 적용한다. 
- fully connected Layer와의 차이점은 파라미터 개수가 mn^4라는 점 
