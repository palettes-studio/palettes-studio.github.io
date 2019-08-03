---
title: "[고문서복원] Partial Convolutions 논문 & 코드 리뷰 "
excerpt: "2019. 08. 03. Partial Convolution"
search: true
categories: 
  - restoration

---

## Image Inpainting for Irregular Holes Using Partial Convolutions

[논문링크](https://arxiv.org/pdf/1804.07723.pdf)

[code](https://github.com/MathiasGruber/PConv-Keras/blob/master/libs/pconv_model.py)

------

#### 1. 초록

<center>키워드 : 이미지 인페인팅, 파셜 콘볼루션  </center>

이미지 빈칸 채우기를 위해 전통적 Convolution이 아닌 partial convolution 제안한다. convolution이 유효 픽셀에만 적용되도록 제한하는 어떤 마스킹을 거치게 함. 이후 재 노멀라이즈된 것이 partial -. 포워드 패스의 한 부분으로서 현 레이어의 다음 레이어로 갈 때 업데이트된 마스크를 자동으로 생성하게 하는 메커니즘도 제안한다.



#### 2. Related Work

- Non-learning Based model
  1. 이웃한 픽셀을 이용하여 채워 넣는 기법 
  2. 구멍이 작은 경우에만 가능
  3. texture의 variance가 작아야 가능 
  4. computing cost가 매우 큼 실시간 처리 어려움 
- Deep learning Based model 
  1. 일정한 placeholder 값을 가지고 hole의 값을 초기화하는게 일반적임논문에서는 크게 context encoder와 Semantic Image Inpainting with Deep Generative Model 소개 



#### 3. Partial Convolutional Layer

- **input : 원본 이미지, 마스크된 이미지**

- **output : output feature & output mask** 

  

![image](https://user-images.githubusercontent.com/26568793/62411617-8fdfb900-b630-11e9-8c2a-35f2e3c92c83.png)



- **x’ : output feature**

![image](https://user-images.githubusercontent.com/26568793/62411693-282a6d80-b632-11e9-9baf-0fdcaab81c91.png)

1. W: convolution filter weights

2. b: bias

3. X : picture values (input으로 들어오는 feature) 

4. M : binary Mask (input으로 들어오는 Mask - 우리가 만들게 되는 hole)  

   → hole vs non hole : hole 은 mask된 부분으로 binary 값으로 0                                                                      non-hole은 마스크 되지 않은 부분으로 binary 값은 1

   

<p align="center">
   <img src="https://user-images.githubusercontent.com/26568793/62411915-91f84680-b635-11e9-94e5-1a36ef4c610b.png">
</p>

<p align="center">[흰색이 non-hole (1)  / 검은 부분이 hole(0) ]</p>

slide하며 convolution을 진행할 때  non-hole인 부분이 조금이라도 있다면 상위의 연산을 진행,  (즉, 이미지와 마스크가 동시에 들어올 때) convolution에 모두 마스크만 들어와 있다면 아무것도 하지 않고 pass 함. 

<p align="center">
   <img src="https://user-images.githubusercontent.com/26568793/62411929-bf44f480-b635-11e9-9ed5-dd42d784a40d.png">
</p>

해당 연산을 반복적으로 진행함에 따라 hole인 부분이 상위 연산으로 채워지기 때문에 점점 hole이 채워지는 것을 볼 수 있음. 

sum(1) / sum(M)  (scaling factor) : 마스크가 쓰이지 않은 부분의 픽셀에 곱해서 조정. 픽셀 값의 노말라이즈를 담당.

- **m’ : output mask**

<p align="center">
   <img src="https://user-images.githubusercontent.com/26568793/62411950-059a5380-b636-11e9-83d8-40afc845b332.png">
</p>

sum(M) > 0 : hole 아닌 부분이 1개라도 포함되어 있는 경우 채워짐.

- **partial conv 이해** 

<p align="center">
   <img src="https://user-images.githubusercontent.com/26568793/62411958-295d9980-b636-11e9-95cf-b53ea29726cb.png">
</p>

#### 3.2 Network의 적용 

Binary Mask의 경우 사이즈가 CxHxW : image 의 피쳐 사이즈와 동일, 고정된 conv layer를 통해서 동일한 kernel 사이즈의 Pconv 구현. Netwok의 경우 U-net base임을 설명

<p align="center">
   <img src="https://user-images.githubusercontent.com/26568793/62411982-793c6080-b636-11e9-88f1-7e379c592b56.png">
</p>

- 3 Channel의 마스크와 이미지로 구성 
- 좌우 : partical conv / 상하 : batch norm
- 마지막 Decoder 단계에 input 이미지를 concat하는 것을 볼 수 있는데, 이 과정의     의미는 input이미지의 정보를 output 이미지에 전달하기 위함.
- Nearest neighbor up-sampling 이용 
- 이미지 바운더리에 Pconv와 함께 typical한 Pconv를 행했음을 설명, 이미지 밖의 픽셀들이 또다른 구멍으로 인식되지 않게 주의



#### 3.3 Loss function 

Loss 함수의 목적은 pixel단위의 복원 정확도와 주변과 구멍이 자연스럽게 이어지는 지에 대한 것들이다. total Loss는 각종 Loss들의 조합들이다. 하나씩 살펴보자.

<p align="center">
   <img src="https://user-images.githubusercontent.com/26568793/62412023-2b742800-b637-11e9-8345-158f47b4149d.png">
</p>

- Iin  : input image with hole

- Iout  : output feature(image), network prediction(output) (모델이 낸 결과)

- M: initial binary mask
- Igt  : Ground truth image
- Icomp  = Iout but with the non-hole pixels directly set to ground truth
- 𝛹(크사이) : activation map of the nth selected layer



- **총 6개의 Loss를 조합** 

  ```python
  # 초기 값 
  import torch
  import os
  import torch.nn as nn
  import numpy as np
  
  from torchvision import models
  from torchvision import transforms
  from places2_train import Places2Data, MEAN, STDDEV
  from PIL import Image
  
  LAMBDAS = {"valid": 1.0, "hole": 6.0, "tv": 2.0, "perceptual": 0.05, "style": 240.0} # 각각의 loss들에 대한 가중치 
  ```

  

  1.  **L valid : non hole loss**

     ![image](https://user-images.githubusercontent.com/26568793/62412141-552e4e80-b639-11e9-9bfa-c888a3419846.png)

     Non-hole인 부분에 대한 loss 계산복원한 이미지, 원본이미지(손상되지 않은 이미지) → 손상되지 않은 부분에 대한 loss 값

  2.  **L hole : hole loss**

     ![image](https://user-images.githubusercontent.com/26568793/62412150-860e8380-b639-11e9-906b-18cde80ae6b6.png)

     hole 인 부분에 대한 loss 계산손상된 부분을 얼마나 잘 복원했는가에 대한 loss인듯 

  3.  **L perceptual**

     ![image](https://user-images.githubusercontent.com/26568793/62412156-9a528080-b639-11e9-8fc6-292653b2c24a.png)

     - I out : 모델을 타고 나오는 결과 이미지

     - I comp : I out에서 non-hole부분을 원래 input pixel값으로 바꾼 것 (I out경우에는 non hole부분이 바뀔 수 있어서로 보임)

       => 두 픽셀간의 L1 loss, 그러니까 절대값의 차이를 줄이는 방향으로 학습

       

     CNN(VGG-16)의 POOL, POOL2, POOL3의 activation map을 사용하여 loss를 계산.

     ```python
     def perceptual_loss(h_comp, h_out, h_gt, l1):
     	loss = 0.0
     
     	for i in range(len(h_comp)):
              # perceptual loss는 style loss와 다르게 gram 식을 거치지 않으므로 바로 h_out, h_gt, h_comp를 통해 l1로스 구함.
     		loss += l1(h_out[i], h_gt[i])
             # l1 = nn.L1Loss : Mean absolute loss
     		loss += l1(h_comp[i], h_gt[i]) 
      
     	return loss
     ```

     

  4.  **L style**

     ![image](https://user-images.githubusercontent.com/26568793/62412171-b81fe580-b639-11e9-82bd-1d62cc6ed77e.png)

     kn: normalization factor for the nth selected layer개념적으로 보면 모델이 만들어낸 이미지를 학습된 vgg모델에 태워서 만들어낸 이미지의 style을 학습하고 원래 ground truth 이미지를 태워서 만들어낸 이미지의 style과 차이를 줄이는 것을 학습한다.여기도 마찬가지로 composition 부분을 따로 한번 더 loss과정을 거치는데, 원래의 output이미지에 non hole부분은 gt로 바꾼 부분을 학습된 vgg거친 것과 gt가 거친 것과의 차이를 학습한다.
     => 여기서 p는 layer의 한 층을 의미하는 것으로 보인다.CpxCp : Gram matrix   Kp :normalization factor 1 /CpHpWp  for pth selected layer 

     ```python
     # 모델이 만든 이미지를 학습된 VGG모델에 태워서 만들어낸 이미지 style을 학습하고 
     # 원래 groud_truth 이미지를 태워서 만들어낸 이미지의 style과의 차이를 줄이는 것을 학습
     # style_loss(fs_composed_output, fs_output, fs_ground_truth, self.l1) * LAMBDAS["style"] 
     # 이때 fs_~ 를 모르니까 class CalculateLoss의 forward를 먼저 보자.
     
     def style_loss(h_comp, h_out, h_gt, l1): # h_comp : fs_composed_output, h_out : fs_output, h_gt : fs_ground_truth, l1 : self.l1 
     	loss = 0.0
     
     	for i in range(len(h_comp)):
     		loss += l1(gram_matrix(h_out[i]), gram_matrix(h_gt[i])) # K_n 값을 gram_matrix에서 바로 구했으니 L_style_out의 loss를 바로 구함
     		loss += l1(gram_matrix(h_comp[i]), gram_matrix(h_gt[i])) #K_n 값을 gram_matrix에서 바로 구했으니 L_style_comp의 loss를 바로 구함
                                                                 # L_style_out + L_style_comp는 가중치가 둘다 120이므로 한번에 더해서 loss로 반환
     	return loss
     
     def gram_matrix(feature_matrix): # function style_loss 와 같이 보자.
     	(batch, channel, h, w) = feature_matrix.size() #feature_matrix 텐서 shape을 반환
     	feature_matrix = feature_matrix.view(batch, channel, h * w) # view는 reshape과 같은 함수 즉 batch x channel x h*w를 갖는 shape으로 변환
     	feature_matrix_t = feature_matrix.transpose(1, 2) # transpose는 전치, 두번 째 차원과 세번 째 차원 바꿔줌.
     
     	# batch matrix multiplication * normalization factor K_n
     	# (batch, channel, h * w) x (batch, h * w, channel) ==> (batch, channel, channel)
     	gram = torch.bmm(feature_matrix, feature_matrix_t) / (channel * h * w)  # bmm은 batch matrix-matrix product, 즉 행렬 곱
                                                                               # 논문 정의를 보면 K_n이 (1/channel * h*w)로 normalize factor -> 이 부분이 K_n
     
     	# size = (batch, channel, channel)
     	return gram # gram 반환
     ```

     

  5.  **L tv: 경계**

     면의 차이를 줄이도록 학습하는 loss (total variation loss)    hole인 부분에서 가장 인접해있는 경계만 보는 것     P: hole인 부분에서 non-hole과 붙어 있는 경계 

     - 삼지창같은 기호는 activation map을 씌워서 나오는 결과를 표현한 기호라 보면 됨. 

       ```python
       # computes TV loss over entire composed image since gradient will not be passed backward to input
       def total_variation_loss(image, l1): # image = composed_output
           # shift one pixel and get loss1 difference (for both x and y direction)
           # image[batch, channel, h,w]
           
           loss = l1(image[:, :, :, :-1] - image[:, :, :, 1:]) + l1(image[:, :, :-1, :] - image[:, :, 1:, :])
           
           # [::::] 행렬 편집 예제
           """
           ex) a = torch.range(1,16)
               a = view(2,2,4) 2x4 행렬 2개인 3차원 행렬
               
               a[0] : 1 2 3 4   a[1]  : 9 10 11 12
                      5 6 7 8          13 14 15 16
               
               b = a[:,:,:-1]
               b.shape : [2,2,3] 2x3 행렬 2개인 3차원 행렬  
               b[0] : 1 2 3    b[1]  : 9 10 11 
                      5 6 7           13 14 15         
               
               c = a[:,:,1:]
               c.shape : [2,2,3] 2x3 행렬 2개인 3차원 행렬  
               c[0] :  2 3 4    c[1]  :  10 11 12 
                       6 7 8             14 15 16         
                       
               d = a[:,:-1,:]
               d.shape : [2,1,4] 1x4 행렬 2개인 3차원 행렬  
               d[0] : 1 2 3 4   d[1]  : 9 10 11 12
               
               e = a[:,1:,:]
               e.shape : [2,1,4] 1x4 행렬 2개인 3차원 행렬  
               e[0] : 5 6 7 8   e[1]  : 13 14 15 16                         
                                
           """
           return loss
       ```

       나머지 코드 

       ```python
       class CalculateLoss(nn.Module):
       	def __init__(self):
       		super().__init__()
       		self.vgg_extract = VGG16Extractor() #함수 VGG16Extractor() 반환 
       		self.l1 = nn.L1Loss() # L1 로스 사용
       
       	def forward(self, input_x, mask, output, ground_truth): 
       		composed_output = (input_x * mask) + (output * (1 - mask)) # compose_output 정의 I_comp
       
       		fs_composed_output = self.vgg_extract(composed_output) # 크사이(I_comp)
       		fs_output = self.vgg_extract(output)                   # 크사이(I_out)
       		fs_ground_truth = self.vgg_extract(ground_truth)       # 크사이(I_gt)
       
       		loss_dict = dict() # key- value 를 받는 dict로 loss_dict 선언
       
       		loss_dict["hole"] = self.l1((1 - mask) * output, (1 - mask) * ground_truth) * LAMBDAS["hole"]
       		loss_dict["valid"] = self.l1(mask * output, mask * ground_truth) * LAMBDAS["valid"]
       		loss_dict["perceptual"] = perceptual_loss(fs_composed_output, fs_output, fs_ground_truth, self.l1) * LAMBDAS["perceptual"]
       		loss_dict["style"] = style_loss(fs_composed_output, fs_output, fs_ground_truth, self.l1) * LAMBDAS["style"]
       		loss_dict["tv"] = total_variation_loss(composed_output, self.l1) * LAMBDAS["tv"]
       
       		return loss_dict
       
       
       # Unit Test
       if __name__ == '__main__':
       	cwd = os.getcwd() # 현재 위치 반환
       	loss_func = CalculateLoss() #calculateLoss 함수 호출하고 return으로 
       
       	gt = Image.open(cwd + "/test_256/Places365_test_00000050.jpg") # ground truth 이미지 호출
       	mask = Image.open(cwd + "/mask/mask_512.jpg") # mask 이미지 호출
       
       	img_transform = transforms.Compose([transforms.ToTensor(), transforms.Normalize(MEAN, STDDEV)]) # 텐서 변환, 노말라이즈
        	mask_transform = transforms.ToTensor() #텐서변환만
       
       	gt = img_transform(gt.convert("RGB")) #ground truth 이미지를   rgb채널 이미지로 변환 후 텐서변환, 노말라이즈
       	mask = img_transform(mask.convert("RGB")) # mask 이미지를 rgb채널 이미지로 변환 후 텐서 변환후 
       	img = gt * mask # img는 ground truth * mask
       
       	img.unsqueeze_(0) #차원 삽입
       	mask.unsqueeze_(0)#차원 삽입
       	gt.unsqueeze_(0)#차원 삽입
       
       	loss_out = loss_func(mask, img, gt)
       
       	for key, value in loss_out.items():
       		print("KEY:{} | VALUE:{}".format(key, value))
       ```

       

#### 4.1 Irregular Mask Dataset 

- 실험적 특이성 (Partial convolution을 사용한 이유 중 하나는, 마스크에 대한 실험과도 관련이 있어보인다.)
  - 55,116 masks 데이터셋 for training , 24,866 for testing, random dilation, rotation and cropping , 512 x 512 
  - 기준마스크들도 종류가 있음, 먼저 구멍이 경계선과 멀리 떨어져 나있는 것, 가까이에 같이 뚫려 있는 것.

#### 4.2 Training Process

- dataset설명 - 논문 참고
- batch size 6, V100 single, imagenet classification initializer를 사용한 것으로 보임
- Batch normalization Issue:  배치마다 전체 픽셀에 대해서 평균과 분산을 맞춰주게 되는데 구멍들이 이 부분에 혼란을 가져오기 때문에 무시했다고 되어있음. 하지만 구멍들이 점차 채워지기 때문에 okay!

