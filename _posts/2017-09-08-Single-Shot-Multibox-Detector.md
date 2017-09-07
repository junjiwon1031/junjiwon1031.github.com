
Single Shot MultiBox Detector
================

## 빠르고 강력한 Detector
SSD가 등장하기 전까지 많이 사용되던 대표적인 detector는 Faster R-CNN이다.([Faster R-CNN 논문](https://arxiv.org/pdf/1506.01497.pdf))

이 Faster R-CNN은 Detecting을 위해

   1. 들어온 image의 **후보 영역**을 열심히 뽑아서! (Region Proposal Network)
   2. 뽑은 부분의 특징을 **resampling** 하고
   3. 그 뒤에 높은 성능의 classifier를 이용하는 것이다.
   
   ![Faster R-CNN](https://raw.githubusercontent.com/sunshineatnoon/Paper-Collection/master/images/faster-rcnn.png "Faster R-CNN")


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

Single-shot detector는 말 그대로 **사진 한장으로 훈련, 검출**을 하는 detector를 의미한다.

그 의미를 알기 위해 다른 deep learning detector를 먼저 살펴보자.

보편적으로 사용되고 있는 Deep-learning Detector 들은 처음 훈련시킨 크기만을 입력으로 받을 수 있다. 대표적으로 VGG-16, Alexnet에서 주로 224X224 크기의 이미지를 입력으로 받는다.

>![VGG-16](http://www.datalearner.com/resources/blog_images/7c1008aa-1c42-45f0-83c7-88c6a0440677.png) 
>
>224X224 크기의 이미지를 입력으로 받아, 그 결과를 1000 labels에 대한 확률로 반환해준다.

즉 원하는 사진에서 객체가 있는지 없는지 확인하기 위해서는 그림을 224X224로 자르거나 변형켜서 알아내야 한다. 이러한 과정을 위해 Image Pyramid & Sliding Window 라던가, Region Proposal Network등으로 입력 이미지를 변형시켜 네트워크 집어넣게 된다.
        - 이미지를 
 다양한 크기의 객체를 검출하기 위해서 크기를 키우고 줄여서 각각의 사진에서 네트워크를 이용하여 검출한다.  

하지만 여러 장의 정보를 처리해야하기 때문에 그만큼 네트워크를 더 돌아야하고, 속도의 저하가 일어나게 된다.

YOLO, SSD와 같은 Detector들이 상대적으로 빠른 속도를 가지고 있는 것은 이러한 이유이다.

2) *Multi-scale feature maps for detection*

근데 Single-shot이 좋았으면 다른 논문에서도 single-shot을 사용했을 것이다.

보통의 경우 입력의 크기를 맞추어 training을 하게 된다. ( 여기서는 alexnet에서 사용되었던 \\(224 \times 224 \\)를 input으로 받는다고 가정해보자) 

CNN의 특징 때문에 테스트를 할때에도 같은 Input을 주어서 테스트를 해야한다. 사진을 넣으면 Sliding Windows나 Region Proposal를 통해 특정 부분을 잘라내어 \\(224 \times 224 \\)로 변형시킨 뒤 그 부분만을 확인하게 된다. Detector가 확인할 수 있는 것은 해당 사진이 훈련되어 있는 물체인가 아닌가 만을 확인하기 때문이다.
