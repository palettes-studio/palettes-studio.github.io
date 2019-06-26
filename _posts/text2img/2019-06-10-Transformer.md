---
title: "[Text2img PART2 Prior Knowledge] Transformer"
excerpt: "2019. 06. 26.  Transformer 소개"
search: true
categories: 
  - text2img
---

part4에 등장하는 USE의  Transformer Encoder을 이해하기 위한 사전 지식 문서

**Transformer**라는 개념을 이해하기 위해서 [**attention is all you need 논문**](https://arxiv.org/abs/1706.03762)을 리딩했다. 

해당 논문에서는 Recurrent, Convloution을 사용하지 않고 Attention만을 사용한 간단한 신경망 구조를 통해 기계 번역 분야에서 SOTA를 얻은 방식을 설명한다. 

----------------

### 1. RNN과 LSTM의 한계 

앞으로 언급할 문제는 두가지이다. 

> 1. Paralleization의 한계
> 2. long-term-dependency 문제

RNN모델은 input과 output의 시퀀스의 위치들을 계산하는 것에 탁월하다.  이전 정보와 현재 input 값을 이용하여 새로운 hidden state를 만들어낸다. 그렇기 때문에 RNN은 순차적인 특성을 가지고 있다. 그렇기 때문에 paralleizable한 속성이 부족하다고 한다. 즉, RNN과 LSTM은 시퀀스의 원소를 순회하면서 지금까지 처리한 정보를 state에 저장하는 형태이기 때문에 시퀀스 길이가 길어지면 길어질 수 록 batch로써 풀고자할 때 문제가 된다. ( 연산을 병렬적으로 사용하기 힘들다는 말이다.  왜냐하면 순차적인 정보를 이용하여 학습을 진행하기 때문이다.)

또한 주로 recurrent model은 long-term-dependency 문제가 발생하는데, 해당 문제는 어떤 정보와 다른 정보 사이의 거리가 멀 때 해당 정보를 이용하지 못한다는 것이다. 그러나 attention is all you need 논문에서는 이를 배제하고 attention 매커니즘을 이용하여 encoder와 decoder를 제작한다. 

### 2. Transformer - encoder 

<p align="center">
   <img src="https://user-images.githubusercontent.com/26568793/60103383-79755080-979a-11e9-8820-955b79cce77d.png">
</p>
위 사진처럼 encoder-decoder 구조를 가지고 있는 network를 transformer라 한다. 

자세한 네트워크는 아래 그림처럼 생겼다. 

<p align="center">
   <img src="https://user-images.githubusercontent.com/26568793/58311386-d1d7cc00-7e43-11e9-832d-d9240243c054.png">
</p>
다음과 같이 encoder는 6개의 동일 Layer 스택으로 구성되었으며 하나의 Layer는 multi-head self-attention mechanism과 position별 fully conntected 된 feed-forward network로 구성되어 있다. 이 둘은 residual connection 되어있으며 마지막으로 Layer norm을 거친다.  encoder내의  self-attention layer는 인코더가 특정 단어를 인코드할 때 입력 문장의 다른 단어들을 볼 수 있도록해준다. 

#### Attention
multi-head self-attention mechanism 에서 multi-head는 무슨 개념이고, self-attention은 무슨 개념일까? 

<p align="center">
   <img src="https://user-images.githubusercontent.com/26568793/60107177-7467cf80-97a1-11e9-8da9-d0f139d4d6b6.png">
</p>

 각각의 포지션에 있는 단어들은 self-attention process를 거친다. 그 후 각각 feed-forward NN을 거친다.  
 
 ##### Scaled Dot-Product Attention 
 
<p align="center">
   <img src="https://user-images.githubusercontent.com/26568793/60160864-4762fd80-9831-11e9-82ba-c46f3241fd7b.png">
</p>

<p align="center">
   <img src="https://user-images.githubusercontent.com/26568793/60112621-1b9d3480-97ab-11e9-8428-cf9bdbc14591.png">
</p>

self-attention에서는 인코더에 입력된 input vector 마다 세개의 벡터(Query, Key, Value)를 만든다.  이 벡터들은 학습 프로세스 중에 학습한 것으로 embedding과 세개의 matrices를 곱해 생성된다. 
'query', 'key', 'value' 벡터는 attention을 계산하고 생각하는데 유용한 추상적인 개념이라 볼 수 있다. 
'query'의 경우는 현재 단어이며 'key'(가장 비슷한 key를 찾아줌)와 'value'(key에 해당하는 value를 찾아줌)는 현재query와 가장 유사한 단어를 찾아주는 값이다. 
여기서 output은 value값의 weight sum 값으로 해당 key의 compatibility function을 통해 계산된다고 한다. 

<p align="center">
   <img src="https://user-images.githubusercontent.com/26568793/60114146-2efdcf00-97ae-11e9-835a-3a5fc12b396f.png">
</p>

위 그림은 스코어를 계산하는 과정이다.  즉, 특정 위치에서 단어를 인코딩할 때, 스코어는 입력 문장에 있어 다른 part에 얼마만큼 초점(attention)을 두어야하는지 결정하는 과정이다.  전통적으로 attention weight는 decoder state (query)를 처리할 때 encoder의 hidden states(values)와의 관련성이며, encoder의 또 다른 hidden states(key)와 decoder의 hidden state(query)를 기반으로 계산한다. 

그림의 과정에서 softmax * value의 과정은 계산 과정을 통해 나온 값과 value 값을 곱해 관련없는 단어는 drown-out 하기 위한 과정인 듯 하다. 

따라서 encoding과정은 self-attention으로 입력을 가장 잘 표현하는 input embedding pair (key,value)를 찾는 것이다. 

또한 self-attention은 position에 약하기 때문에 Multi-Head attention이라는 것을 이용하는데 이는 self-attention이 h개 만큼 쌓인 것이라고 볼 수 있다.
또한 초입에 Positional Encoding을 이용하는데 이는 RNN이나 CNN이 아니기 때문에 여전히 position에 대한 정보가 부족할 수 밖에 없는 한계가 있으므로 이를 보완하기 위해 추가하는 것이다.  






### 3. Transformer의 효과



**attention mechanism**은 앞서 언급한 RNN의 문제와는 달리 data의 시퀀스 길이 의존도가 낮다. 따라서 attention mechanism을 이용하는 Transfomer 모델은 input과 output의 global dependency를 잡아내며, 병렬처리를 수월하게 진행하게 해준다. 

순차적인 연산을 줄이고자 하는 목표로 ByteNet등이 생겨났는데, 이들은 모두 hidden representation을 parallel하게 계산하고자, CNN을 활용한다. 그러나 이들은 2개의 positoin 상 멀리 떨어져있는 input - output을 연결하는데에 많은 연산을 필요로 한다(number of operation required). 따라서 distant position에 있는 dependency를 학습하기에는 힘들다. Transformer에서는 attention-weighted position을 평균을 취해줌으로써 effective는 잃었지만, 이 operation이 상수로 고정되어있다. effective에 대해선 Multi-Head Attention으로 이를 극복한다.(section 3.2)

Self-attention은 seq representation을 얻고자 한 sequence에 있는 다른 position을 연결해주는 attention기법이다. 이는 지문이해나 요약등의 과제에서 다양하게 활용되고 있다.

-----------

------------------------------

**해당 문서는 논문과 함께 [링크1](<https://jalammar.github.io/illustrated-transformer/](https://jalammar.github.io/illustrated-transformer/>) 

, [링크2](<http://blog.naver.com/PostView.nhn?blogId=hist0134&logNo=221035988217&redirect=Dlog&widgetTypeCall=true>) 를 많이 참고하여 작성했습니다.**
