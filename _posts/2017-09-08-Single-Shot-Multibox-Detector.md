
Single Shot MultiBox Detector
================

## 빠르고 강력한 Detector
SSD가 등장하기 전까지 많이 사용되던 대표적인 detector는 Faster R-CNN이다.([Faster R-CNN 논문](https://arxiv.org/pdf/1506.01497.pdf))

이 Faster R-CNN은 Detecting을 위해

   1. 들어온 image의 **후보 영역**을 열심히 뽑아서! (Region Proposal Network)
   2. 뽑은 부분의 특징을 **resampling** 하고
   3. 그 뒤에 높은 성능의 classifier를 이용하는 것이다.
   
   (비슷한 시기에 나온 detector들은 대부분 이 과정을 거치게 된다.)
   >![Faster R-CNN](https://raw.githubusercontent.com/sunshineatnoon/Paper-Collection/master/images/faster-rcnn.png "Faster R-CNN")

하지만 Faster R-CNN은 전작인 R-CNN을 열심히 개선했음에도 불구하고 너무 느려서 (7 FPS with mAP 73.2% ) 실시간 영상분석에 사용할 수는 없었다. 

YOLO 는 빠르긴 했지만 그만큼 성능을 포기하게 되었다.(45 FPS with mAP 63.4%)
 
하지만 SSD는 후보 영역 추출 과정과 resampling 과정을 제거한 방식을 이용하여 높은 정확성과 빠른 속도를 모두 얻어냈다. (59 FPS with mAP 74.3%) 

(위의 모든 성능 측정은 VOC2007 test를 기반으로 한다)

이 포스트에서는 SSD가 어떻게 이런 놀라운 성능을 가질 수 있게 되었는지 공부해볼 것이다.


----------


## SSD 가 빠르고 좋은 이유는 뭐지?

SSD는 전부 새로 만든 구조가 아니다. 원래 잘 만들어졌던 feed-forward convolutional network에서 feature map을 뽑아내는 과정까지를 하나의 기본 구조로 가지고, 여러 보조적인 몇 가지 구조만을 추가한 것이다.

논문에서는 *[VGG-16 network](https://arxiv.org/pdf/1409.1556.pdf)*에서 conv5_3까지를 잘라서 기본 구조로 사용하였다. 현재 2017년 9월 7일 기준으로 6020회의 인용이 있는 network인 만큼 feature map을 놀랍도록 잘 뽑아낸다!

사실 Faster R-CNN도 그렇고, 잘 되던 네트워크를 가져다 쓰는 건 많은 논문에서 하고 있다. 하지만 SSD가 특히 좋은 성능을 보여준 이유는 바로  **Single-shot learning** 과 **보조 구조**에 있다.

SSD의 Framework를 천천히 살펴보면서 SSD가 가지고 있는 특징을 살펴보자.

#### 1) *single-shot detector*

Single-shot detector는 말 그대로 **사진의 변형 없이 그 한 장으로 훈련, 검출**을 하는 detector를 의미한다.

그 의미를 알기 위해 다른 deep learning detector를 먼저 살펴보자.

보편적으로 사용되고 있는 Deep-learning Detector 들은 처음 훈련시킨 크기만을 입력으로 받을 수 있다. 대표적으로 VGG-16, Alexnet에서 주로 224X224 크기의 이미지를 입력으로 받는다.

>![VGG-16](/assets/vgg16.png ) 
>
>224X224 크기의 이미지를 입력으로 받아, 그 결과를 1000 labels에 대한 확률로 반환해준다.
>(출처: http://www.datalearner.com)

즉 원하는 사진에서 객체가 있는지 없는지 확인하기 위해서는 그림을 224X224로 자르거나 변형켜서 알아내야 한다. 이러한 과정을 위해 *Image Pyramid & Sliding Window* 라던가, *Region Proposal Network(Faster R-CNN)*등으로 입력 이미지를 변형시켜 네트워크 집어넣게 된다. 

문제는 위의 처리 과정을 통해 얻은 여러가지 sample 들을 하나씩 network에 넣어서 하나씩 검출 해야 한다는 것이다. 여러 장의 정보를 처리해야하기 때문에 그만큼 네트워크를 많이 돌게 되고, 횟수 차이에 의한 속도의 저하가 일어나게 된다.

위와 같은 이유로 Single-shot detector가 상대적으로 검출 속도가 빠른 것이다.

#### 2) *Multi-scale feature maps for detection*

하지만 single-shot learning을 위해서는 큰 문제 하나를 해결해야 한다.

**단! 한장의! 사진**만을 가지고 여러가지 크기의 물체를 검출해야 한다.

SSD는 이러한 문제를 기본 구조 뒤에 보조 구조를 붙여 얻은 *Multi-scale feature maps* 을 이용해서 해결하였다.

>![SSD-Framework](http://www.cs.unc.edu/~wliu/papers/ssd.png)
> *멍멍이*와 *야옹이*는 크기가 **두 배정도** 다르기에 
> 크기가 다른 feature map들에서 찾아내게 된다. 

위 사진에서 멍멍이의 크기는 사진의 1/3 가량인데 반해, 고양이는 대략 1/6 정도로 둘의 크기 차이가 크다. 이를 한가지 feature map에서 구하려면 bounding box의 크기차이가 크기 때문에, box의 크기 추정부터 위치 추정까지 많은 과정이 필요할 것이다.

하지만 SSD 는 그렇지 않고, feature map을 여러 개의 크기로 만들어서, 큰 map에서는 작은 물체의 검출을, 작은 map에서는 큰 물체의 검출을 하게 만들었다. 후술하겠지만, 이러한 방식은 위치 추정 및 입력 이미지의 resampling을 없애면서도 정확도 높은 결과를 도출하게 된다. 

#### 3) *Convolutional predictors for detection* & *Default boxes and aspect ratios*

> ![SSD-architecture](/assets/ssdarchitecture.png )

이 것이 SSD 의 architecture이다. 기본 구조나 보조 구조에서 얻은 feature map들은 각각 다른 convolutional filter에 의해 결과값을 얻게 된다. 

*m* x *n* 을 *p* 채널을 가지고 있는 feature map은, 각 위치 마다 *3* x *3* x *p* kernel들 을 적용할 수 있으며, 각 **kernel(filter) 은 카테고리 점수나 bounding box offset 점수**를 알려주게 된다.

한 가지 예를 들어보자. bounding box offset이란 각 cell(feature map 한 칸)을 기준으로 한 상대적 위치와 박스의 크기를 의미한다. 이를 위해 필요한 정보는 x, y, width, height 로 4개이다. 즉 이들을 표현하기 위해 한 cell 당 4개의 filter들로 표현할 수 있게 되는 것이다.

마찬가지로 카테고리 점수는 각 label 마다 얼마만큼의 가능성이 있는 것인지 표시하는 것이기에, 한 카테고리당 하나의 filter로 표현할 수가 있다. 만약 c개의 카테고리가 있었다면, 한 cell 당 (c + 4)개의 filter로 표현 할 수 있다는 것이다.

....만! 사실 offset에서 실제 bounding box의 크기를 말하지 않는다. 실제 박스와 예측 박스가 정확히 일치하는 것이 제일 좋겠지만, 너무나도 어렵고 비효율적이다. 그래서 *aspect ratio*라는 개념을 도입하였다.
이는 **상자가 가질만한 비율을 몇 가지만 추려서 정답은 이 안에 있어! 라고 추정하는 것**이다. (이 비율들을 가진 상자들을 default boxes라 부른다.)

이는 위 사진에 잘 표현되어 있다. 약간의 정확도를 포기하지만 빠르고 나쁘지 않은 성능을 보여준다. 그래서 만약 aspects ratio가 k개라 했을 때 실제로 한 cell 당 (c + 4) x k 개의 filter를 적용하게 된다.

논문에서는 5개의 feature map에 대해서, 전부 다른 filter들을 사용함으로 크기에 상관없이 검출할 수 있는 환경을 만들었다. 이러한 방식 덕분에 위치 추정 및 resampling 없이도 높은 정확도를 가진 detector가 완성된 것이다.

------------------------------

## SSD 훈련하기 

위에서 구조에 대해서 알아보았다. 이제 중요한 것은 **어떻게 훈련을 시키는가** 이다.
많은 논문에서도 종종 언급되는 문제가 훈련 방법이다. 인공신공망이라는 이름에 안 어울리게 훈련을 대충하면 결과도 대충 나온다.다행히도 대부분의 논문에는 저자들이 피땀흘려 얻어낸 훈련법이 있기 때문에, 자세히 읽어보는 것이 좋다.

SSD 를 훈련시킬 때 중요한 점은 물체 뿐만 아니라 위치가 정확하게 표시된 dataset이 필요하다는 것이다. 그렇기에 다른 detector와는 살짝 다른 방식으로 훈련을 하게 된다.

#### 1) *Matching Strategy*
SSD가 예측한 박스와 실제 박스가 일치하는 지를 확인하는 것은 매우 중요한 문제이다. 이럴 때 가장 많이 사용하는 것은 **jaccard overlap**, 혹은 **intersection over Union(IOU)** 이다. SSD에서는 이 값이 일정 값(threshold, 본 논문에서는 0.5)를 넘기기만 하면 일단 일치한다고 가정한다. 
>![jaccard overlap](/assets/jaccard_overlap.png)

다른 detector들의 구조에서는 IOU가 가장 큰 상자만을 사용하는 경우가 많은 데, 굳이 threshold를 넘는 상자를 모두 선택하는 이유는 두 가지가 있다. 

첫째는 **높은 정확도를 가진 상자를 한꺼번에 여러번 학습**시킴으로써 하나만 고르는 것보다 학습이 잘 된다고 한다. 

둘째는 Hard Negative mining을 위해서이다. SSD는 negative sample을 따로 받지 않는다. 여기서 나온 0.5 이상의 상자들 중, 점수가 상대적으로 낮은 것 들을 negative로 학습시킨다. 최대 3:1 비율로 negative, positive로 나누며 이러한 학습이 최적화가 빠르고 훈련이 좀 더 안정화 됨을 (실험적으로)알았기 때문이다.

#### 2) *Training objective*
>![SSD Loss Function](/assets/ssd_loss_function.png)
>
> N: 검출된 박스의 개수 ( N=0 일 시에 loss를 0으로 설정함)
>
> g: ground truth box, 실제 박스의 변수들을 의미한다.
>
> d: default box, default box
>
> c: category, 말 그대로 카테고리
>
> l: predicted boxes, 예상된 박스의 변수들을 의미한다.
>  
> cx, cy: offset of center
>
> w,h : width and height

SSD 는 위치와 카테고리를 같이 학습한다. 따라서 loss 역시 그 두 개를 동시에 고려하여야 한다. 가장 위의 식은 전체 Loss를 의미하며, 이 값은 위치에 따른 loss (L_loc)과 카테고리의 자신감에 따른 loss(L_conf) 를 합친 것이다. 자세한 것은 논문을 살펴보면 좋다!

-----------------------------------------

##  결과

> ![ssd 결과](/assets/result_voc2007.png)
> 
>입력 이미지가 커질수록 해상도가 높아지고,검출 할 수 있는 한도가 늘어나 수치가 더 좋게 나온다. 
SSD300과 SSD512 의 차이는 그 결과이다. 
 
실제로 test를 돌려보면 위와 같은 결과가 나온다. 기존의 detection 방식과 많은 차이가 있음에도 불구하고 속도는 우월하고, 성능도 최신 detector들 만큼 뽑을 수 있다는 점은 놀랍다.

논문을 자세히 살펴보면 
지금 현재는 YOLO의 새 버전인 YOLO9000이 성능과 속도를 모두 따라잡았기에 이 모델을 그대로 쓰는 것은 필요 없어졌다.
하지만 SSD를 통해 다른 좋은 네트워크도 Single-shot Learning이 가능함을 보여주었고, 
