- Yolo v8 모델 만들기
    - coco dataset
- C# 또는 다른 환경에서 활용하는 방법 알아보기
- 참고
	- https://velog.io/@choonsik_mom/Object-Detection-with-yolo-NAS-zpetis4o
	- https://developer-lionhong.tistory.com/62


-----

- [Github | yolov8-object-tracking](https://github.com/RizwanMunawar/yolov8-object-tracking?tab=readme-ov-file)
- [Medium | Train YOLOv8 on Custom Data?](https://muhammadrizwanmunawar.medium.com/train-yolov8-on-custom-data-6d28cd348262)
- [Tistory | YOLO v8 사용하기](https://geunuk.tistory.com/463)
- [Tistory | Pytorch(GPU)를 위한 Python 환경 구축 - conda 외 가상환경](https://qlsenddl-lab.tistory.com/59)
- [Tistory | YOLO v8 사용하기(ultralytics)](https://geunuk.tistory.com/463)
- [Youtube | YOLOv8 커스텀 데이터 학습하기](https://www.youtube.com/watch?v=em_lOAp8DJE)

----

## YOLOv8
Yolov8 모델이 object detection과 segmantion을 동시에 지원함

Object Detection
``` python
from ultralytics import YOLO

# Load a pre-trained model
model = YOLO("yolov8n.py")

# Train the model
model.train(data="data.yaml", epochs=100, imgsz=640)

# Predict with the model
results = model.predict(source="test.jpg")
```

Image Segmentation
``` python
from ultralytics import YOLO

# Load a pre-trained model
model = YOLO("yolov8n-seg.pt")

# Train the model
model.train(data="data-seg.yaml", epochs=100, imgsz=640)

# Predict with the model
results = model.predict(source="test.jpg")
```


## YOLOv8 학습 프로세스 (custom data)
**Data Preparation**
- 이미지, 정답 데이터로 이루어진 데이터들
- 예시)
	![](https://i.imgur.com/c5Be2sW.png)


**Loading Data**
- `wget`, `curl` 등 명령어로 Dataset을 Colab으로 다운로드

**Make YAML file**
- Custom data를 학습하기 위해서 필요한 데이터 정보를 가지고 있음
- YAML
	- 1. 이미지와 정답이 저장되어 있는 디렉토리 정보
	- 2. 인식(Detection)하고 싶은 클래스 종류와 대응되는 각각의 이름
- 예시) ![](https://i.imgur.com/qWht0nB.png)

**Install YOLOv8**
``` python
# yolov8 실행에 필요한 라이브러리 설치 및 depenency 체크
pip install ultralytics
```

**Train model**
``` python
from ultralytics import YOLO

model = YOLO("yolov8n-seg.pt") # Load a pre-trained model
model.train(data='mydata.yaml', epochs=10) # mydata.yaml 참조하여 학습 (파인튜닝)
```

**Prediction**
``` python
results = model.predict(source='/content/test/') # predict on test images
```



https://github.com/neowizard2018/neowizard/blob/master/DeepLearningProject/YOLOv8_Object_Detection_Roboflow_Aquarium_Data.ipynb