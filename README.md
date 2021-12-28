## GAN-based Synthetic Medical Image Augmentation for increased CNN Performance
### 2021 Hallym University Capstone Design

#### Team Members: 이승리, 서영재, 서원진, 최재홍

-----

## 1. Project Overview
![image](https://user-images.githubusercontent.com/65914440/143834784-fe862ba8-fb04-493c-b142-bbf2a94241dd.png)
 먼저 일반 데이터와 기본적인 augmentation을 적용하여 학습을 진행한다. 모델은 U-Net구조를 참고하여 구축하고 성능을 확인한다. Pix2Pix를 통해 생성된 fake image를 train dataset에 추가한다. 그 후 새로 구성한 데이터셋으로 Segmentation 작업을 수행하는데 정확한 비교를 위해 하이퍼 파라미터의 값은 고정하여 학습을 진행한다.

## 2. Dataset

<img src = "https://user-images.githubusercontent.com/65914440/143831739-ddbbe9fa-6ac7-4f11-9e6c-66a33bde7794.png" width = "400" height = "200"> <img src = "https://user-images.githubusercontent.com/65914440/143831332-65a3fa78-e8be-4750-a63f-4f78152b5781.png"  width = "400" height = "200">

　　　　　<Real image : 6400장>  　　　　　　　　　　　　　　　　　　    <Fake image : 1333장>
     
     
 Real image : 캐글 대회에서 제공한 데이터
[**MOAI 2021 Body Morphometry AI Segmentation Online Challenge**](https://www.kaggle.com/c/body-morphometry-kidney-and-tumor/data)

 Fake image : GAN을 이용하여 생성한 이미지.



 


### Data pre-processing

- 전체 데이터를 Train dataset(70%), Validation dataset(20%)와 Test dataset(10%)으로  
  분류했다. Fake data가 추가될 경우에는 train dataset에만 포함되도록 했다.
 
- CT이미지는 신체부위에 따라 WW(Window Width:픽셀값의 범위)와 WL(Window Center:기준이 되는 픽셀값)을 조절해 이미지를 사용합니다. 
  저희는 실험적으로 해보며 학습이 가장 잘되는 WW는 400, WL은 0으로 조정했다.
- 최대-최소 정규화를 했다.

<img src = "https://user-images.githubusercontent.com/65914440/143833289-f15255cd-596a-479c-a463-201e15f58ebf.png" width = "800" height = "400">

- Segmentation 모델의 예측의 채널은 훈련된 클래스의 갯수와 동일합니다. 
  각각의 채널이 하나의 클래스를 대표한다. 
  모든 클래스를 포함한 Label을 클래스별로 채널을 나눠주는 작업이 필요하다. 
  우리는 하나의 채널로 구성되었던 Label을 2채널로 나누어 0채널은 신장, 1채널에는 종
  양으로 구성했다.  

<img src = "https://user-images.githubusercontent.com/65914440/143834221-3b8986f8-929c-4772-a662-e8c8a3f5ccc6.png" width = "350" height = "200"> <img src = "https://user-images.githubusercontent.com/65914440/143834230-0928de6e-ef18-4e5e-bd1a-281df6ad4352.png"  width = "200" height = "200">



## 3. Model

### U-net (our Model)
![image](https://user-images.githubusercontent.com/65914440/143835011-128ade6f-dc06-499d-a11e-a74fd717e8ef.png)

### Contraction Path(수축 단계)
  -  3x3 convolutions을 두 차례씩 반복 padding X
  -  활성화 함수 : ReLU
  -  2x2 max-pooling (stride:2)
  -  Down-sampling 마다 채널의 수를 2배로 늘림
### Expanding Path(팽창 단계)
  -  2x2 convolution (up –convolution)
  -  3x3 convolutions을 두 차례씩 반복 padding x
  -  Up-ConV 를 통한  Up-sampling 마다 채널의 수를 반으로 줄임
  -  활성화 함수 : ReLU 
  -  Up-Conv된 feature map은 Contracting path의 테두리가 Cropped된 
     feature map과 concatenation 함
  -  마지막 레이어에 1x1 convolution 연산
------

## GAN

### Pix2Pix
<img src = "https://user-images.githubusercontent.com/65914440/143835167-8d02e12a-48f8-4f7e-b273-9fb800c70b9b.png" width = "440" height = "300"> <img src = "https://user-images.githubusercontent.com/65914440/143835318-9befb607-e42f-4643-ad3e-a46b04bda5e5.png"  width = "380" height = "300">


　　　　　　　　　　　　　　< Pix2Pix >　　　　　　　　　　　　　　　　　　　　　　　　< Our Case >
- 기존의 GAN은 noise를 사용하여 학습을 진행해 랜덤한 이미지가 나오지만 Pix2Pix는 지도 학습으로 인풋 이미지가 segmentation이미지, 라벨 이미지는 원본 데이터로 학습을 진행한다.

------

## 4. Experiments

### Hyper-Parameter

![image](https://user-images.githubusercontent.com/65914440/143837316-2475ec46-219d-420c-bc1b-d2f5b0862497.png)

- 모든 학습은 위에 있는 하이퍼 파라미터로 Loss는 DiceLoss, optimizer는 Adam, learning rate는 0.001, epoch은 100과 scheduler를 사용하여 학습을 진행했다.

### U-Net

- U-Net은 가벼운 모델과 무거운 모델로 총 2가지의 버전을 만들었는데, 두 모델의 차이는 파라미터가 얼마나 많고 적냐의 차이다.

### Experiment Cases
<img src = "https://user-images.githubusercontent.com/65914440/143837130-90b94fd4-18d6-44d6-9dd8-a49998c9bd1c.png" width = "500" height = "200">


- 우선적으로 직접 구성한 2개의 U-Net으로 실험을 했다. 다른 모델에서도 가짜 데이터 증강으로 인한 성능향상이 적용되는지 확인하기 위해 Pretrain된 FPN으로도 실험을 진행했다.


## 5. Result

### Evaluation Metrics

![image](https://user-images.githubusercontent.com/65914440/143838038-13e1bda4-8f67-4e32-8d3f-4c5d2ce94be6.png)

#### IoU (Intersection over Union)
- 라벨링된 영역과 예측한 영역이 정확히 같다면, 1이되며 그렇지 않을 경우에는 0이 된다.
- 교집합 영역 넓이 / 합집합 영역 넓이로 iou score가 계산된다.
- 최종 Test  성능측정에선 한장마다의 IoU 값을 평균내어 사용함

### Result
![image](https://user-images.githubusercontent.com/90440043/144424001-b901545f-860e-49a8-bf88-b66238529a1f.png)

### Limitations & Future Work

   1) Epoch 수 늘려서 학습한다.
   2) Fake Data 품질을 높인다.
   3) 다른 문제에도 본 기술을 적용한다.

## 6. Conclusion
U-Net의 성능 결과를 보면 성능향상 폭이 상당히 큰 것을 확인할 수 있다. 기존 데이터만 학습을 진행하면 58%로 상당히 저조하지만, 데이터 증강 기법을 사용하는 순간 크게 32%의 성능향상이 있었다. 그러나 모델의 복잡도가 증가할수록 성능은 비교적 비슷하거나, 기하학 변형하지 않을 때 학습 데이터를 그대로 외워버리는 현상이 나오기도 했다. 이런 결과를 보아 모델이 깊어질수록 데이터를 세부적으로 보는데, 생성한 가짜 데이터는 정교하지 않아 큰 성능향상을 이뤄내진 못 한 것으로 보인다. 
   결론적으로 가짜 데이터의 품질을 높인다면 더 높은 성능향상을 이뤄낼 수 있을 것이다.

## 7. Expecting Effect
신장과 종양 형태는 매우 다양하기 때문에 신장과 종양의 형태를 고려한 수술 계획 기술과 수술 결과와의 연관성, 그리고 이러한 기술들의 발전에 대한 관심이 많다. 신장과 신장암의 Segmentation 기술은 이러한 노력과 발전을 위한 유망한 도구로써 사용된다. 

 본 과제물에서는 가상의 신장과 종양 데이터를 생성하고 이를 활용하여 신장 및 종양 검출 모델의 성능을 개선시켰다. 가상 데이터가 신장 종양 검출 모델의 성능을 개선시켰으므로, GAN으로 생성한 가상 데이터가 유의미한 데이터임을 입증하였다. GAN을 통해 부족한 데이터를 증강하는 것 자체가 가치 있는 과정이며, 이를 통해 인공지능 성능들을 높일 수 있다. 
 
 대부분의 유형의 종양을 감지하는 것은 일반적으로 의사에게는 간단하지만, 시각적 평가를 통해 종양 경계를 정의하는 것은 어려운 일이다. 데이터 세트에 대한 GAN 모델을 훈련시켜 종양 세그먼트를 생성하는 것은 분석, 치료 및 수술 중에 의사에게 도움이 될 수 있는 양질의 종양 세그먼트를 생성하는 수단을 제공한다. 
 
 가상 데이터는 본래의 데이터 수가 제한된 상황에서 데이터를 늘리는 데 사용할 수 있으며 향후 인공지능 모델을 훈련하는 데 사용할 수 있다. 이 기술을 더 보완한다면 정확한 모델을 훈련하기 위한 실제 데이터가 충분하지 않은 질병을 감지하기 위한 훈련 모델용 데이터를 생성할 수 있을 것이다. 
 
 본 과제물의 기술을 변형한다면 다양한 데이터 분야에서 개인정보에 제한 없이 데이터를 확보하는 것이 가능하다. 본 기술이 의료 분야에만 국한되어 적용하는 것이 아닌 상대적으로 데이터가 부족한 산업까지 확장하여 상품화를 시킨다면 인공지능 산업에 선점을 기여하여 사회와 인류의 발전을 이루어낼 것이다.

## Data Citation

Nicholas Heller, Niranjan Sathianathen, Arveen Kalapara, Edward Walczak, Keenan Moore, Heather Kaluzniak, Joel Rosenberg, Paul Blake, Zachary Rengel, Makinna Oestreich, Joshua Dean, Michael Tradewell, Aneri Shah, Resha Tejpaul, Zachary Edgerton, Matthew Peterson, Shaneabbas Raza, Subodh Regmi, Nikolaos Papanikolopoulos, Christopher Weight. The kits19 challenge data: 300 kidney tumor cases with clinical context, ct semantic segmentations, and surgical outcomes. arXiv preprint arXiv:1904.00445, 2019. URL https://arxiv.org/abs/1904.00445

## TCIA Citation
Clark K, Vendt B, Smith K, Freymann J, Kirby J, Koppel P, Moore S, Phillips S, Maffitt D, Pringle M, Tarbox L, Prior F. The Cancer Imaging Archive (TCIA): maintaining and operating a public information repository. Journal of Digital Imaging. 2013 Dec;26(6):1045-57. DOI: 10.1007/s10278-013-9622-7

## Data License 

Creative Commons CC-BY-NC-SA license
