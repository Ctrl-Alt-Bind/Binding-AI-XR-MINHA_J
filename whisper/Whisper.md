- https://github.com/Macoron/whisper.unity/blob/master/ProjectSettings/ProjectVersion.txt
	- Unity 2021.3.3f1
- [Whisper를 이용한 한국어 음성 인식 사용해보기](https://velog.io/@hwang9u/Speech-Whisper%EB%A5%BC-%EC%9D%B4%EC%9A%A9%ED%95%9C-%ED%95%9C%EA%B5%AD%EC%96%B4-%EC%9D%8C%EC%84%B1-%EC%9D%B8%EC%8B%9D-%EC%82%AC%EC%9A%A9%ED%95%B4%EB%B3%B4%EA%B8%B0)


# Whisper.unity
## WhisperSteam
> - 스트리밍 방식으로 음성 인식을 처리
> - 마이크 입력을 실시간으로 받아서, 텍스트로 변환하는 기능을 제공


``` c#
public delegate void OnStreamResultUpdatedDelegate(string updatedResult);  
public delegate void OnStreamSegmentUpdatedDelegate(WhisperResult segment);   public delegate void OnStreamSegmentFinishedDelegate(WhisperResult segment);   public delegate void OnStreamFinishedDelegate(string finalResult);
```

**Event**
``` c#
public event OnStreamResultUpdatedDelegate OnResultUpdated;`
	//stream transcription를 업데이트하는 경우 발생
	//string updatedResult: stream의 시작~현재까지의 전체 stream transcription 결과를 포함

public event OnStreamSegmentUpdatedDelegate OnSegmentUpdated;
	// 현재 segment transcript를 업데이트할 때 발생
	// WhisperResult segment
		// 현재 segment transcription의 결과를 포함함

public event OnStreamSegmentFinishedDelegate OnSegmentFinished;
	// 현재 segment transcript를 완료할 때 발생
	// WhisperResult segment
		// 완료된 segment transcription의 결과를 포함함

public event OnStreamFinishedDelegate OnStreamFinished;
	// stream transcription를 완료하고, 새로운 stream을 시작할 수 있을 때 발생함
	// string finalResult
		// stream의 최종 stream transcription 결과를 포함함
```

**Function**
``` c#
public void StartStream()
	// 새로운 stream 음성 인식을 시작
	// 마이크 입력을 받아 stream을 생성

public async void AddToStream(AudioChunk chunk)
	// streaming 중, 새로운 audioChunk를 추가하는 기능
	// async 방식으로 처리
	// streaming이 진행 중이라면 오디오 데이터를 buffer에 추가하고,
	// await UpdateSlidingWindow()

public async void StopStream()
	// 현재 진행 중인 streaming transcription을 중지
	// Steaming 중지, 마지막 UpdateSlidingWindow()
	// OnStreamFinished 이벤트 발생

private async Task UpdateSlidingWindow(bool forceSegmentEnd = false)
	// 슬라이딩 윈도우 방식으로 처리, 비동기 음성인식
	// 충분한 데이터가 buffer에 쌓이면, transcription을 진행함
	// _wrapper.GetTextAsync(buffer, _param.Frequency, _param.Channels, _param.InferenceParam);
```


## WhisperWrapper
> Whisper model을 wrapping하여 음성 인식 기능을 제공하는 Class

``` C#
public delegate void OnNewSegmentDelegate(WhisperSegment text);  
public delegate void OnProgressDelegate(int progress);
```

**Event**
``` c#
public event OnNewSegmentDelegate OnNewSegment;
	// 새로운 text segment가 transcription될 때 발생함
public event OnProgressDelegate OnProgress;
	// transcription 진행 상황이 Update할 때 발생함
```

**Function**
``` C#
public WhisperResult GetText(AudioClip clip, WhisperParams param)
public async Task<WhisperResult> GetTextAsync(AudioClip clip, WhisperParams param)

public WhisperResult GetText(float[] samples, int frequency, int channels, WhisperParams param)
	// Audio buffer를 입력받아, 이를 transcription하고 text 결과를 반환함
	// 이는 동기적으로 실행: transcription이 완료될 때까지 호출된 thread를 blocking
	// InferenceWhisper(readySamples, nativeParams) 로 실제로 음성 인식 수행
	// WhisperNative.whisper_full_n_segments(_whisperCtx);
	// 완료 후, Sengment List와 langID를 포함한 WhisperResult 객체를 생성&반환

public async Task<WhisperResult> GetTextAsync(float[] samples, int frequency, int channels, WhisperParams param)

private unsafe bool InferenceWhisper(float[] samples, WhisperNativeParams param)
	// 주어진 Audio data를 Native Whisper engine으로 전달하여 처리
	// WhisperNative.whisper_full(_whisperCtx, param, samplesPtr, samples.Length);



```


## WhisperManager
> Unity scene에서 Whisper 모델의 생명주기를 관리함

**Event**
``` c#
public event OnNewSegmentDelegate OnNewSegment;
	// audio로부터 새로운 text segment가 transcription될 때 발생함
public event OnProgressDelegate OnProgress;
	// transcription 진행 상황이 Update할 때 발생함
```

**Method**
``` c#
private void Update()
	// Main thread dispatcher를 업데이트

public async Task InitModel()
	// whisper model과 기본 parameter를 load

public async Task<WhisperResult> GetTextAsync(AudioClip clip)
public async Task<WhisperResult> GetTextAsync(float[] samples, int frequency, int channels)

public async Task<WhisperStream> CreateStream(int frequency, int channels)
public async Task<WhisperStream> CreateStream(MicrophoneRecord microphone)
```




# Whisper.cpp
Unity에서 `InferenceWhisper()`를 통해 실질적으로 음성인식을 수행하는 것
해당 함수에서 호출하는 Native code를 확인해보려고 함

```C++
int whisper_full(
        struct whisper_context * ctx,
	    struct whisper_full_params params,
	    const float * samples,
	    int   n_samples) 
{
    return whisper_full_with_state(ctx, ctx->state, params, samples, n_samples);
}
```

`whisper_full_with_state` 내용
>음성 데이터를 입력 받아, 이를 텍스트로 변환하는 과정을 수행
>음성 데이터 전처리, 해당 데이터에 대해 text를 추론하는 과정을 포함함
- 주요 변수
	- `ctx` : 모델과 관련된 전역 상태 정보
	- `state`: 음성 인식 작업 상태를 관리하는 struct. 이전 처리 결과, 현재 decoder 상태 등 저장
	- `params`: 사용자 설정값 struct. 
	- `samples`: 입력 음성 데이터
	- `n_samples`: 입력된 음성 데이터의 sample 수
	- `result_all`: 최종 결과를 담는 vector
- 주요 단계
	- 1. 결과 초기화 및 전처리
		- 입력된 음성 데이터를 받아 해당 데이터를 *Log mel spectrogram*으로 변환
	- 2. 자동 언어 감지
	- 3. Temperature값 설정
	- 4. 디코더 초기화
	- 5. 디코딩 프로세스
		- 음성 데이터의 spectrogram을 디코딩하여 텍스트로 변환
		- 각각의 결과에 대해서 확률 값을 계산, 가장 높은 것을 choose
	- 6. 결과 저장 및 반환
		`result_all.push_back({ tt0, tt1, text, {} , speaker_turn_next });
`