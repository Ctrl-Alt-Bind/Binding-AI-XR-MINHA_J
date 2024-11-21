

## c# 활용 방안 고민하기
- [c# 기반 배포 가능한 딥러닝 객체 감지 프로그램 개발(feat. YOLOv5) #1](https://blog.hbsmith.io/c-%EA%B8%B0%EB%B0%98-%EB%B0%B0%ED%8F%AC-%EA%B0%80%EB%8A%A5%ED%95%9C-%EB%94%A5%EB%9F%AC%EB%8B%9D-%EA%B0%9D%EC%B2%B4-%EA%B0%90%EC%A7%80-%ED%94%84%EB%A1%9C%EA%B7%B8%EB%9E%A8-%EA%B0%9C%EB%B0%9C-feat-yolo-v5-1-98581e397aa4)
	- CPU 동작 가능한 모델 선정
	- Python 기반 추론 코드 분석
	- 학습된 모델을 어떻게 c#에서 사용할 수 있는가?
		Python 기반 모델 학습 → 모델 export → 다른 프로그램 언어에서 로드 후 사용 함수 구현 (설치형 프로그램)
- [c# 기반 배포 가능한 딥러닝 객체 감지 프로그램 개발(feat. YOLOv5) #2](https://blog.hbsmith.io/c-%EA%B8%B0%EB%B0%98-%EB%B0%B0%ED%8F%AC-%EA%B0%80%EB%8A%A5%ED%95%9C-%EB%94%A5%EB%9F%AC%EB%8B%9D-%EA%B0%9D%EC%B2%B4-%EA%B0%90%EC%A7%80-%ED%94%84%EB%A1%9C%EA%B7%B8%EB%9E%A8-%EA%B0%9C%EB%B0%9C-feat-yolo-v5-2-3310b8d81a82)
	- 학습한 모델을 OnnxRuntime을 활용
	- c#에서 불러온 후, 추론하는 코드 구현
-  [https://github.com/singetta/OnnxSample](https://github.com/singetta/OnnxSample)  

- ONNX Runtime on Unity
	- [Medium | ONNX Runtime on Unity](https://medium.com/@asus4/onnx-runtime-on-unity-a40b3416529f)
	- [Github | onnxruntime-unity](https://github.com/asus4/onnxruntime-unity)
	- [Github | onnxruntime-unity-example](https://github.com/asus4/onnxruntime-unity-examples)
- ONNX Deep Learning in Unity
	- [Youtube | How to load a Keras model and transform it into ONNX so it can be used in unity](https://youtu.be/7ndUGBzGVvg?si=OMpYsMZf6D2jiPFF)
	- [Youtube | Load a simple ONNX Deep Learning model in Unity for your own game](https://www.youtube.com/watch?v=R9I9prRUiEo)

- Barracuda → Sentis인가 보군
	- [Tistory | 유니티 바라쿠다 튜토리얼 (StyleTransfer-AdaIN)](https://pnltoen.tistory.com/entry/Unity-Barracuda-%EC%9C%A0%EB%8B%88%ED%8B%B0-%EB%B0%94%EB%9D%BC%EC%BF%A0%EB%8B%A4-%ED%8A%9C%ED%86%A0%EB%A6%AC%EC%96%BC-StyleTransfer-AdaIN)
	- [Tistory | Unity Barracuda 적용 사례 정리(튜토리얼 수준)](https://velog.io/@ji1kang/Unity-Barracuda-%EC%A0%81%EC%9A%A9-%EC%82%AC%EB%A1%80-%EC%A0%95%EB%A6%AC)