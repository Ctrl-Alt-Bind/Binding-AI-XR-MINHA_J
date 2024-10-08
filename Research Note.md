*Last Updated 24/10/05*

---
## Open source
- Whisper (https://github.com/Macoron/whisper.unity)
	- Context 판단
- MediaPipe Unity plugin (https://github.com/homuler/MediaPipeUnityPlugin)
	- 인체 골격 따는
- OpenAI Unity (https://github.com/srcnalt/OpenAI-Unity)
- YOLO Unity ([unity/sentis-yolotinyv7 at main](https://huggingface.co/unity/sentis-yolotinyv7/tree/main))
- ***Sentis***
	- Sentis가 뭐지.. > [[Research Note#Open source#Sentis]]
	- MusicGen (https://huggingface.co/unity/sentis-MusicGen)
	- BodyPixSentis (https://github.com/keijiro/BodyPixSentis)

### Sentis
문서 작성(.hwp/.word)와 같은 많은 편집툴이 존재하나, 공용 포맷인 *.pdf*가 존재.
이처럼 Machine Learning에도 [ONNX(Open Neural Network Exchange)](https://learn.microsoft.com/ko-kr/windows/ai/windows-ml/get-onnx-model)라는 포맷 존재.
Unity는 이 포맷을 추론(Interface)할 수 있도록 Sentis package를 제공하는 것.

- https://unity.com/kr/products/sentis
- [Tistory-Unity Sentis에 대한 글이 몇 개 있음](https://pnltoen.tistory.com/tag/%EC%9C%A0%EB%8B%88%ED%8B%B0%20Sentis)
	- [Tistory-Unity Sentis 소개 및 설치](https://pnltoen.tistory.com/entry/Unity-Sentis-%ED%8A%9C%ED%86%A0%EB%A6%AC%EC%96%BC-Unity-Sentis-%EC%86%8C%EA%B0%9C-%EB%B0%8F-%EC%84%A4%EC%B9%98-AdaIN-%EC%83%98%ED%94%8C-%EC%86%8C%EA%B0%9C)
- [Gihub-sentis samples](https://github.com/Unity-Technologies/sentis-samples/tree/main/BlazeDetectionSample/Hand)
- [Medium-ONNX 너 누구야!WHO ARE YOU?](https://medium.com/@enerzai/onnx-%EB%84%88-%EB%88%84%EA%B5%AC%EC%95%BC-who-are-you-5c1435b997e2)
- Unity Korea
	- [Unity AI 기술 Sentis를 활용해 제작 가능한 AI 콘텐츠 에시 만나보기](https://www.youtube.com/watch?v=0GZ4KJAspJM)
	- [Unity Sentis와 Hugging Face로 게임에 적합한 AI 모델 찾기](https://unity.com/kr/blog/games/hugging-face-ai-models-and-more-sentis-updates)
#### 샘플 코드 확인해봐야지  
> DigitRecognitionSample \  
> Editor version: 2023.2.0b17 \  
> [Youtube- Unity Sentis project sample: Build an escape room with a digit detection](https://www.youtube.com/watch?v=IofX0CAYdmU)  
  
##### 영상 내용 정리  
- How to implement that in c#?  
  - NN를 사용하여 여러 숫자 이미지에 대해 네트워크를 훈련시키고, 플레이어의 이미지를 분류할 수 있다  
  - Runtime으로 network는 플레이어의 스케치를 분석하고 어떤 숫자인지 분류해냄

**Code (MNISTEngine.cs)**
`GetMostLikelyDigitProbability()`

``` c#
// Sends the image to the neural network model and returns the probability that the image is each particular digit.  
public (float, int) GetMostLikelyDigitProbability(Texture2D drawableTexture)  
{  
    inputTensor?.Dispose();  
  
    // Convert the texture into a tensor, it has width=W, height=W, and channels=1:      
	inputTensor = TextureConverter.ToTensor(drawableTexture, imageWidth, imageWidth, 1);  
    
    // run the neural network:  
    engine.Execute(inputTensor);  
    
    // We get a reference to the output of the neural network while keeping it on the GPU  
    TensorFloat result = engine.PeekOutput() as TensorFloat;  
    
    // convert the result to probabilities between 0..1 using the softmax function:  
    var probabilities = ops.Softmax(result);  
    var indexOfMaxProba = ops.ArgMax(probabilities, -1, false);  
    
    // We need to make the result from the GPU readable on the CPU  
    probabilities.MakeReadable();  
    indexOfMaxProba.MakeReadable();  
  
    var predictedNumber = indexOfMaxProba[0];  
    var probability = probabilities[predictedNumber];  
  
    return (probability, predictedNumber);  
}
```
- 플레이어 스케치를 Texture2D로 받아온 다음 신경망을 호출. Sentis를 사용하여 추론 수행
- engine.Execute() 하면 CPU/GPU인 Backend model을 취함


----

## Unity Plugin
[Unity Documentation- 플러그인](https://docs.unity3d.com/kr/current/Manual/Plugins.html)
### Managed Plug-ins
<u>관리되는</u> .NET assembly
	VS과 같은 툴로 Unity 외부에서 생성하여 DDL(Dynamically Linked Library)로 컴파일
<u>.NET 라이브러리</u>가 지원하지 않는 기능에는 엑세스 할 수 없음
	Microsoft에서 개발한 소프트웨어 프레임워크. 다양한 애플리케이션을 개발할 수 있는 환경을 제공. 이때 재사용 가능한 코드 집합이 라이브러리 예) `System`, `System.IO`, `System.Net`


### Native Plug-ins
플랫폼 별 native code library
원래는 Unity에서 사용할 수 없는 OS 호출, 타사 코드 라이브러리와 같은 기능에 access 할 수 있음

- [Unity Documentation- 네이티브 플러그인](https://docs.unity3d.com/kr/2022.3/Manual/NativePlugins.html)
- 사용하기 위해서는
	- C 기반 언어로 함수를 작성. 필요한 기능을 access
	- 라이브러리로 compile
	- Unity에서 Native library의 함수를 호출하는 c# 스크립트를 생성

``` C#
using UnityEngine; 
using System.Runtime.InteropServices; 

class ExampleScript : MonoBehaviour { 
	#if UNITY_IPHONE 
	// On iOS plugins are statically linked into 
	// the executable, so we have to use __Internal as the 
	// library name. 
	[DllImport ("__Internal")] 
	
	#else 
	// Other platforms load plugins dynamically, so pass the 
	// name of the plugin's dynamic library. 
	[DllImport ("PluginName")] 
	
	#endif private static extern float ExamplePluginFunction (); 
	
	void Awake () 
	{ 
		// Calls the ExamplePluginFunction inside the plugin 
		// And prints 5 to the console 
		print (ExamplePluginFunction ()); 
	} 
}
```

[예시-OpenCV 사용하기 #2.플러그인 만들기](https://dreamfuture.tistory.com/28)