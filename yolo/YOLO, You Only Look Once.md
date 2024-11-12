
- [YOLO 논문 리뷰: You Only Look Once: Uniﬁed, Real-Time Object Detection](https://medium.com/@parkie0517/yolo-%EB%85%BC%EB%AC%B8-%EB%A6%AC%EB%B7%B0-you-only-look-once-uni%EF%AC%81ed-real-time-object-detection-f146af809c57)
- [YoloV5 - Deep learning](https://blog.naver.com/PostView.naver?blogId=intelliz&logNo=222824372526)
- [YOLO v3 논문 리뷰 및 코드구현](https://csm-kr.tistory.com/11)
	-  [Github- darknet](https://github.com/pjreddie/darknet)

----

**Object detection**
디지털 이미지, 비디오 내에서 유의미한 특정 객체를 감지하는 작업

**YOLO: You Only Look Once**
- 빠르다
	- 단일 신경망 구조. 구성이 간단하고 빠름
	- Image Localization과 Image Classification을 하나의 회귀 문제로 보기 때문
- 이미지 전체를 본다
	- 이미지를 한번만 보는 *1-stage detector* 방식을 사용
	- 주변 정보까지 학습, 이미지 전체를 처리. Background error가 낮음
- 객체의 Generalizable Representation을 학습한다.
	- 실제 이미지로부터 일반화될 수 있는 특징을 학습함
	- 새로운 이미지 도메인에서 성능이 저하될 확률이 낮음

# Detection 방식
Input file을 받으면 하나의 convolution network가 
이미지 내에서 찾고자 하는 객체의 bounding box의 위치 정보인 x, y, w, h의 감소가
bounding box의 class 확률을 동시에 계산함

![|550](https://i.imgur.com/ivi5TSW.png)
(1) Input 이미지를 $(s\times s)$ 크기의 Grid로 나눔
- $(448\times448)$ 크기의 이미지. $(s\times s)$ 크기의 Grid 영역으로 나눔
- 한 grid cell 안에 어떤 객체의 중심이 포함되면, 해당 grid cell이 그 객체에 대해 책임을 지고 탐지를 수행함
- 하나의 Bounding box predictor가 ground-truth box와 IoU 점수가 가장 높은 Bounding Box에 관해서 학습을 해야함 

(2) 각 grid cell은 $B$개의 bounding box를 예측. 그 box에 대한 *confidence score*를 같이 예측하며
- Confidence Score
	- $Pr(object)*IOU_{pred}^{truth}$
	- (박스 안에 객체가 포함될 확률) ⨉ (박스가 객체를 얼마나 잘 담고 있는지)
	- $Pr(object)$: 객체가 포함되면 1, 없으면 0
	- IOU(Intersection over union): Ground truth box와 예측 bounding box의 교집합 크기 / 합집합 크기
- Bounding Box
	![|350](https://i.imgur.com/WLTveuN.png)
	- 4개의 값에 대한 예측
	- $(x, y)$ - Bounding box의 중심 값. Grid cell 안에서의 상대적 좌표. 0~1 사이값
	- $(w,h)$ - Bounding box의 너비와 높이. 이미지의 상대적인 길이. 0~1 사이값

(3) 각 Grid Cell은 $c$개 class에 대한 Conditional Class Probabilities를 계산
- 각 Bounding box 안에 있는 객체가 *어떤 class일지*에 대한 조건부 확률
- Conditional Class Probabilities: $Pr(Class_i|Object)$
- 최종적으로, *Class-specific Confidence  Score*
	- Bounding box 안에 특정 객체가 속할 확률, bounding box가 객체를 얼마나 잘 담고 있는가
	- ![](https://i.imgur.com/LaGdTN6.png)
	- (Conditional Class Probailities) ⨉ (Bounding box의 confidence score)


# Architecture
![](https://i.imgur.com/3QtHKPn.png)
- 24개의 Convolutional Layers
	- 이미지로부터 특징을 추출함
- 뒤의 2개의 Fully Connected Layers
	- Output Probabilities와 Coordinates 예측
- 최종 Output은 $7\times7\times30$크기의 tensor

## Training 과정
- 마지막 Layer만 *Linear Activation Function*
- Leaky Rectified Linear Unit(ReLU) Activation Function 
	![|300](https://i.imgur.com/rBU1hhR.png)
- 모델의 output에 대해서 *SSE(Sum Squared Error)* 를 사용함
	- 문제가 존재한다 
	- ① Localization Error(LE)와 Classification Error(CE)의 가중치를 동일하게 적용한다는 문제
	- ② 대부분의 Grid cell에 객체가 포함되지 않음(많은 부분이 배경임)
		Negative sample이 positive sample 보다 많아서 데이터의 불균형 문제가 발생함
	- 따라서
		LE에 대한 가중치는 증가 시키고
		객체를 포함하지 않은 Negative sample에 대한 CE 가중치는 감소 시킴
	- ③ 큰/작은 Bounding box의 가중치를 동일하게 준다는 문제
	- 따라서
		bounding box의 width와 height에 제곱근을 취함
		LE의 증가율이 작아져, 큰 bounding box의 error에 덜 민감해짐

**Training Loss Function**
![|700](https://i.imgur.com/abKdkzn.png)
1. 첫 번째 식은 Bounding Box의 중심 좌표 x, y에 대한 Loss  
2. 두 번째 식은 Bounding Box의 너비, 높이에 대한 Loss  
3. 세 번째 식은 Bounding Box의 Confidence Score에 대한 Loss  
4. 네 번째 식은 Bounding Box의 Confidence Score에 대한 Loss  
5. 다섯 번째 식은 Conditional Class Probability의 Loss 
	올바른 클래스에 대한 예측이면 pi(c) = 1, 아니면 0