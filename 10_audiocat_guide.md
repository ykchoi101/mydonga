# <span style="color: #1E40AF; border-bottom: 3px solid #3B82F6; padding-bottom: 8px; display: inline-block;">🎙️ OpenAI Whisper 음성 인식 완벽 가이드 (`10_audiocat.py`)</span>

<div style="background-color: #F8FAFC; border-left: 4px solid #3B82F6; padding: 12px 16px; margin: 16px 0; border-radius: 0 8px 8px 0; color: #334155;">
이 문서는 <code>10_audiocat.py</code>에서 사용된 <strong>OpenAI Whisper 기반 음성-텍스트 변환(STT, Speech-to-Text) 파이프라인</strong>의 6단계 흐름과 구조를 시각화한 가이드입니다.
</div>

---

## <span style="color: #0F766E; border-left: 5px solid #0D9488; padding-left: 10px; background-color: #F0FDFA; display: block; border-radius: 0 6px 6px 0; padding-top: 4px; padding-bottom: 4px;">🧭 1. 전체 6단계 파이프라인 흐름도 (Pipeline Flowchart)</span>

음성 파일(`cat.mp3`)이 텍스트로 변환되어 출력되기까지의 6단계 전체 처리 흐름입니다.

```mermaid
flowchart TD
    Start(["🚀 시작: 프로그램 실행 (main)"]) --> Step1["1️⃣ 1단계: 패키지 검증 (validate_whisper)\n- openai-whisper 설치 여부 확인\n- 모듈 충돌 및 함수 유효성 검사"]
    
    Step1 --> Step2["2️⃣ 2단계: 연산 장치 감지 (detect_device)\n- CUDA GPU 존재 여부 확인\n- GPU ➔ FP16 사용 / CPU ➔ FP32 사용"]
    
    Step2 --> Step3["3️⃣ 3단계: Whisper 모델 로딩 (load_whisper_model)\n- base 모델을 메모리에 로드\n- 장치(CPU/GPU)에 모델 배치"]
    
    Step3 --> Step4["4️⃣ 4단계: 오디오 파일 검증 (validate_audio)\n- 'audio/cat.mp3' 파일 존재 여부 확인\n- 지원 확장자(.mp3, .wav 등) 및 파일 크기 체크"]
    
    Step4 --> Step5["5️⃣ 5단계: 음성 인식 및 전사 (transcribe_audio)\n- FFmpeg 오디오 디코딩 ➔ PyTorch 텐서 변환\n- Whisper 인공신경망 추론 (STT)"]
    
    Step5 --> Step6["6️⃣ 6단계: 결과 파싱 및 출력 (print_result)\n- 전체 전사 텍스트 추출\n- 타임스탬프 세그먼트별 구간 텍스트 출력"]
    
    Step6 --> End(["🏁 종료: 최종 텍스트 반환 ('Show me the cat information')"])

    style Start fill:#E0F2FE,stroke:#0284C7,stroke-width:2px,color:#0369A1;
    style End fill:#DCFCE7,stroke:#16A34A,stroke-width:2px,color:#15803D;
    style Step1 fill:#FEF3C7,stroke:#D97706,stroke-width:2px,color:#B45309;
    style Step2 fill:#F3E8FF,stroke:#9333EA,stroke-width:2px,color:#6B21A8;
    style Step3 fill:#E0E7FF,stroke:#4F46E5,stroke-width:2px,color:#3730A3;
    style Step4 fill:#CCFBF1,stroke:#0D9488,stroke-width:2px,color:#115E59;
    style Step5 fill:#FCE7F3,stroke:#DB2777,stroke-width:2px,color:#9D174D;
    style Step6 fill:#FEF9C3,stroke:#CA8A04,stroke-width:2px,color:#854D0E;
```

---

## <span style="color: #1D4ED8; border-left: 5px solid #2563EB; padding-left: 10px; background-color: #EFF6FF; display: block; border-radius: 0 6px 6px 0; padding-top: 4px; padding-bottom: 4px;">🔄 2. 상세 시퀀스 다이어그램 (Sequence Diagram)</span>

파이썬 코드와 외부 라이브러리(PyTorch, Whisper, FFmpeg, 오디오 파일) 간의 상호작용 흐름입니다.

```mermaid
sequenceDiagram
    autonumber
    actor User as 👤 사용자
    participant Main as 💻 main()
    participant Validator as 🔍 검증 모듈
    participant Torch as ⚡ PyTorch
    participant Whisper as 🧠 Whisper Model
    participant FFmpeg as 🎵 FFmpeg / Audio

    User->>Main: python 10_audiocat.py 실행
    
    Main->>Validator: validate_whisper()
    Validator-->>Main: ✅ 패키지 정상 확인
    
    Main->>Torch: torch.cuda.is_available() 감지
    Torch-->>Main: 🖥️ CPU (FP16 비활성화)
    
    Main->>Whisper: whisper.load_model("base", device="cpu")
    Whisper-->>Main: ✅ 모델 인스턴스 반환
    
    Main->>Validator: validate_audio("audio/cat.mp3")
    Validator-->>Main: ✅ 파일 존재 및 확장자 확인
    
    Main->>Whisper: model.transcribe(audio_path, fp16=False)
    Whisper->>FFmpeg: 오디오 디코딩 및 리샘플링 (16kHz)
    FFmpeg-->>Whisper: 오디오 텐서 데이터 전달
    Note over Whisper: 음향 특징(Mel-spectrogram) 추출<br/>인코더-디코더 신경망 추론
    Whisper-->>Main: 전사 결과 Dictionary (text, segments, language)
    
    Main->>Main: print_result() 포맷팅
    Main-->>User: 🔤 "Show me the cat information" 화면 출력
```

---

## <span style="color: #7C2D12; border-left: 5px solid #EA580C; padding-left: 10px; background-color: #FFF7ED; display: block; border-radius: 0 6px 6px 0; padding-top: 4px; padding-bottom: 4px;">🧩 3. 시스템 아키텍처 및 의존성 관계도</span>

```mermaid
graph LR
    subgraph Core ["🐍 Python 3.12 (.venv)"]
        Script["10_audiocat.py"]
    end

    subgraph AI_Libs ["🧠 핵심 인공지능 라이브러리"]
        WhisperLib["openai-whisper\n(음성 인식 신경망)"]
        TorchLib["PyTorch\n(텐서 및 딥러닝 연산)"]
    end

    subgraph External ["⚙️ 시스템 외부 도구 / 파일"]
        FFmpegBin["FFmpeg\n(오디오 파일 디코더)"]
        AudioFile["audio/cat.mp3\n(오디오 원본 소스)"]
    end

    Script --> WhisperLib
    Script --> TorchLib
    WhisperLib --> TorchLib
    WhisperLib --> FFmpegBin
    Script --> AudioFile
    FFmpegBin --> AudioFile

    style Core fill:#EFF6FF,stroke:#3B82F6,stroke-width:2px;
    style AI_Libs fill:#FDF4FF,stroke:#C084FC,stroke-width:2px;
    style External fill:#FFFBEB,stroke:#F59E0B,stroke-width:2px;
```

---

## <span style="color: #6D28D9; border-left: 5px solid #7C3AED; padding-left: 10px; background-color: #F5F3FF; display: block; border-radius: 0 6px 6px 0; padding-top: 4px; padding-bottom: 4px;">📖 4. 6단계 함수별 상세 설명</span>

### <span style="color: #4338CA;">① `validate_whisper()` — 패키지 유효성 검사</span>
- **역할:** 잘못된 서드파티 `whisper` 패키지가 아닌 OpenAI 공식 `openai-whisper`가 설치되어 있는지 확인하고, 프로젝트 내에 동일한 이름의 `whisper.py` 파일이 있어 발생하는 모듈 충돌(Shadowing)을 방지합니다.

---

### <span style="color: #4338CA;">② `detect_device()` — 연산 장치(GPU/CPU) 자동 감지</span>
- **역할:** NVIDIA CUDA 그래픽카드가 있으면 GPU를 사용하고, 없으면 CPU를 선택합니다. CPU에서는 `FP16` 연산 시 발생하는 불필요한 경고를 방지하기 위해 `FP32` 모드로 자동 전환합니다.

---

### <span style="color: #4338CA;">③ `load_whisper_model()` — Whisper 모델 가중치 로드</span>
- **역할:** Whisper 모델(`base`, `small`, `medium`, `large` 등) 중 `base` 모델 가중치(약 140MB)를 다운로드/로드하여 메모리에 올립니다.

---

### <span style="color: #4338CA;">④ `validate_audio()` — 오디오 파일 유효성 검증</span>
- **역할:** 지정된 파일 경로(`audio/cat.mp3`)가 실제 존재하는지 확인하고, 지원 가능한 오디오 형식(`.mp3`, `.wav`, `.m4a`, `.flac` 등)인지 검사합니다.

---

### <span style="color: #4338CA;">⑤ `transcribe_audio()` — 음성 인식(STT) 추론</span>
- **역할:** 내부적으로 `FFmpeg`를 사용하여 오디오를 16,000Hz 모노 신호로 변환한 뒤, Whisper 신경망을 거쳐 음성을 텍스트로 변환합니다.

---

### <span style="color: #4338CA;">⑥ `print_result()` — 결과 출력 및 세그먼트 파싱</span>
- **역할:** 최종 추출된 전체 텍스트와 함께 각 발화 구간(예: `[0.00s → 3.00s]`)의 타임스탬프 정보를 깔끔한 표 형태로 화면에 출력합니다.
