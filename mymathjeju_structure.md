# 🗺️ mymathjeju.py 구조 및 실행 흐름 가이드

이 문서는 [mymathjeju.py](file:///c:/workAi/work9langchain/mymathjeju.py) 파일의 내부 구성 요소와 LangChain 도구 호출(Tool Calling) 동작 과정을 직관적으로 파악할 수 있도록 **색상별 영역 구분과 친절한 한글 설명**으로 구성된 다이어그램 문서입니다.

---

## 🎨 1. 전체 아키텍처 및 모듈 구성도

각 영역별로 기능에 맞는 **고유 색상**이 적용되어 있어 한눈에 구조를 파악할 수 있습니다:
- 🔵 **파란색 (Client)**: 사용자 질문 입력 및 히스토리 관리
- 🟣 **보라색 (LangChain Agent)**: LLM 판단 및 파이프라인 제어
- 🟢 **초록색 (수학 도구)**: 파이썬 math 연산 도구
- 🟠 **주황색 (제주도 도구)**: 제주 여행/날씨/맛집 안내 도구
- 🔴 **분홍색 (저장소 & 출력)**: 최종 답변 출력 및 JSON 파일 영구 저장

```mermaid
flowchart TB
    %% ======================= 서브그래프 정의 =======================
    subgraph SG_USER["🔵 1. 사용자 인터페이스 (입력 및 대화 기록)"]
        UserNode["👤 <b>사용자 질문 입력</b><br/>(예: 수학 연산 또는 제주도 질문)"]
        HistoryNode["💬 <b>대화 히스토리 (chat_history)</b><br/>HumanMessage / AIMessage 누적"]
    end

    subgraph SG_AGENT["🟣 2. LangChain AI 에이전트 파이프라인"]
        PromptNode["📝 <b>프롬프트 템플릿 (ChatPromptTemplate)</b><br/>• 시스템 역할 부여 (친절한 AI 가이드)<br/>• 이전 대화 기록 + 사용자 질문 합성"]
        LLMNode["🧠 <b>LLM 추론 엔진 (ChatOpenAI)</b><br/>• OpenRouter / gpt-4o-mini 모델<br/>• 사용자의 의도를 분석해 도구 호출 결정"]
        ExecutorNode["⚙️ <b>에이전트 실행기 (AgentExecutor)</b><br/>• 도구 실행 결과 취합 및 추가 추론 제어<br/>• 최종 답변(output) 및 과정(intermediate_steps) 생성"]
    end

    subgraph SG_MATH["🟢 3-A. 수학 계산 도구 (Math Tool)"]
        MathSchema["📋 <b>입력 검증 (MathQuery)</b><br/>• 연산자 (add, subtract, abs, sqrt, pow 등)<br/>• 계산할 숫자 (num1, num2)"]
        MathFunc["🧮 <b>수학 계산 함수 (math_tool)</b><br/>• Python math 내장 모듈 연산 실행<br/>• 계산 결과 문자열 반환"]
    end

    subgraph SG_JEJU["🟠 3-B. 제주 가이드 도구 (Jeju Tool)"]
        JejuSchema["📋 <b>입력 검증 (JejuQuery)</b><br/>• 카테고리 (날씨, 명소, 맛집, 팁)<br/>• 지역 (서귀포, 애월 등) 및 날짜"]
        JejuFunc["🌴 <b>제주 정보 함수 (jeju_tool)</b><br/>• 카테고리별 제주 가이드 텍스트 매칭<br/>• 맞춤 여행 정보 반환"]
    end

    subgraph SG_OUTPUT["🔴 4. 결과 출력 및 JSON 데이터 저장"]
        OutNode["🤖 <b>최종 답변 출력 (Console)</b><br/>도구 실행 로그 + AI 최종 정리 답변"]
        JsonNode[("💾 <b>data2/jejumath.json</b><br/>일시, 질문, 처리결과 누적 저장")]
    end

    %% ======================= 연결선 정의 =======================
    UserNode --> PromptNode
    HistoryNode --> PromptNode
    PromptNode --> LLMNode
    LLMNode --> ExecutorNode

    ExecutorNode -->|1. 수학 연산 필요 시| MathSchema
    MathSchema --> MathFunc
    MathFunc -->|계산 결과 전달| ExecutorNode

    ExecutorNode -->|2. 제주 정보 필요 시| JejuSchema
    JejuSchema --> JejuFunc
    JejuFunc -->|제주 정보 전달| ExecutorNode

    ExecutorNode --> OutNode
    OutNode -->|대화 갱신| HistoryNode
    OutNode -->|자동 파일 저장| JsonNode

    %% ======================= 스타일 및 색상 지정 =======================
    classDef clientStyle fill:#E0F2FE,stroke:#0284C7,stroke-width:2px,color:#0369A1;
    classDef agentStyle fill:#F3E8FF,stroke:#7C3AED,stroke-width:2px,color:#5B21B6;
    classDef mathStyle fill:#DCFCE7,stroke:#16A34A,stroke-width:2px,color:#15803D;
    classDef jejuStyle fill:#FEF3C7,stroke:#D97706,stroke-width:2px,color:#B45309;
    classDef outStyle fill:#FFE4E6,stroke:#E11D48,stroke-width:2px,color:#9F1239;

    class UserNode,HistoryNode clientStyle;
    class PromptNode,LLMNode,ExecutorNode agentStyle;
    class MathSchema,MathFunc mathStyle;
    class JejuSchema,JejuFunc jejuStyle;
    class OutNode,JsonNode outStyle;

    style SG_USER fill:#F0F9FF,stroke:#0284C7,stroke-width:1.5px,stroke-dasharray: 4 4;
    style SG_AGENT fill:#FAF5FF,stroke:#7C3AED,stroke-width:1.5px,stroke-dasharray: 4 4;
    style SG_MATH fill:#F0FDF4,stroke:#16A34A,stroke-width:1.5px,stroke-dasharray: 4 4;
    style SG_JEJU fill:#FFFBEB,stroke:#D97706,stroke-width:1.5px,stroke-dasharray: 4 4;
    style SG_OUTPUT fill:#FFF1F2,stroke:#E11D48,stroke-width:1.5px,stroke-dasharray: 4 4;
```

---

## ⚡ 2. 한눈에 보는 데이터 처리 흐름 (좌 ➔ 우 흐름도)

```mermaid
flowchart LR
    Q["👤 <b>사용자 질의</b><br/>'abs(2 - 17) 계산해줘'"] 
    --> AI{"🧠 <b>AI 판단</b><br/>어떤 도구가 필요한가?"}

    AI -->|수학 계산| T1["🧮 <b>math_tool</b><br/>operation: 'abs'<br/>num1: 2, num2: 17"]
    AI -->|제주 가이드| T2["🌴 <b>jeju_tool</b><br/>category: 'food'<br/>location: '서귀포'"]

    T1 --> R1["📊 <b>연산 결과</b><br/>15.0"]
    T2 --> R2["🍊 <b>특산물 정보</b><br/>흑돼지, 감귤 등"]

    R1 --> Final["🤖 <b>AI 최종 답변</b><br/>자연스러운 한국어 설명"]
    R2 --> Final

    Final --> Save[("💾 <b>data2/jejumath.json</b><br/>저장 완료!")]

    style Q fill:#E0F2FE,stroke:#0284C7,stroke-width:2px,color:#0369A1;
    style AI fill:#F3E8FF,stroke:#7C3AED,stroke-width:2px,color:#5B21B6;
    style T1 fill:#DCFCE7,stroke:#16A34A,stroke-width:2px,color:#15803D;
    style T2 fill:#FEF3C7,stroke:#D97706,stroke-width:2px,color:#B45309;
    style R1 fill:#DCFCE7,stroke:#16A34A,stroke-width:1.5px,color:#15803D;
    style R2 fill:#FEF3C7,stroke:#D97706,stroke-width:1.5px,color:#B45309;
    style Final fill:#FFE4E6,stroke:#E11D48,stroke-width:2px,color:#9F1239;
    style Save fill:#F1F5F9,stroke:#475569,stroke-width:2px,color:#1E293B;
```

---

## ⏱️ 3. 상세 실행 시퀀스 다이어그램 (순서도)

```mermaid
sequenceDiagram
    autonumber
    actor User as 👤 사용자 (CLI)
    participant Agent as ⚙️ AgentExecutor (파이프라인)
    participant LLM as 🧠 OpenRouter (GPT-4o-mini)
    participant Tool as 🛠️ 전용 도구 (Math / Jeju)
    participant DB as 💾 jejumath.json (저장소)

    Note over User, Agent: 1단계: 사용자 요청 수신
    User->>Agent: "제주도 서귀포 특산물 및 맛집 알려줘"
    
    Note over Agent, LLM: 2단계: 의도 파악 및 도구 호출 결정
    Agent->>LLM: 프롬프트 전달 (시스템 지침 + 질문 + 도구 목록)
    LLM-->>Agent: 도구 호출 결정 (jeju_tool, category='food', location='서귀포')

    rect rgb(254, 243, 199)
        Note over Agent, Tool: 3단계: 도구 실행 (Tool Execution)
        Agent->>Tool: jeju_tool() 실행
        Tool-->>Agent: "🍊 [서귀포] 추천 특산물 및 맛집: 흑돼지 구이, 감귤..."
    end

    Note over Agent, LLM: 4단계: 최종 종합 답변 생성
    Agent->>LLM: 도구 실행 결과를 포함하여 최종 정리 요청
    LLM-->>Agent: "서귀포의 대표적인 특산물은 흑돼지와 감귤입니다..."

    Note over Agent, DB: 5단계: 결과 출력 및 영구 저장
    Agent->>DB: save_to_jejumath_json(질문, 답변) 기록 누적
    Agent-->>User: 🤖 콘솔 출력 (도구 실행 로그 + AI 최종 답변)
```

---

## 📌 4. 모듈별 기능 및 데이터 매핑표

| 모듈 구분 | 파일 내 이름 | 색상 | 핵심 역할 및 설명 |
| :--- | :--- | :---: | :--- |
| **Pydantic 스키마** | `MathQuery` | 🟢 | 수학 연산 타입(`operation`) 및 숫자(`num1`, `num2`) 유효성 검사 및 `calculate()` 실행 |
| **Pydantic 스키마** | `JejuQuery` | 🟠 | 제주 정보 카테고리(`weather`, `food` 등) 유효성 검사 및 `get_jeju_info()` 실행 |
| **LangChain 도구** | `math_tool` | 🟢 | `@tool` 데코레이터를 적용한 수학 연산기 (사칙연산, `abs`, `round`, `sqrt`, `pow`) |
| **LangChain 도구** | `jeju_tool` | 🟠 | `@tool` 데코레이터를 적용한 제주도 맞춤 여행 가이드 |
| **AI 파이프라인** | `create_agent_pipeline` | 🟣 | OpenRouter 기반 `gpt-4o-mini` 모델과 도구들을 바인딩하여 `AgentExecutor` 생성 |
| **실행 및 기록** | `process_query` | 🔵 | 사용자 질문 처리, 대화 히스토리 갱신, 콘솔 로그 출력 |
| **JSON 저장소** | `save_to_jejumath_json` | 🔴 | 질의응답 결과를 `data2/jejumath.json` 파일에 시간순 누적 기록 |
