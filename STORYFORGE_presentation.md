# StoryForge
> **아이디어만 있으면 누구나 자기만의 스토리 쇼츠를 만든다**

캐릭터 설정 하나로 스토리·이미지·음성·BGM이 모두 AI로 생성되어 쇼츠 한 편이 완성되는 웹 애플리케이션.
그림을 못 그려도, 글쓰기가 서툴러도, 음악을 몰라도 — 내 캐릭터가 살아 움직이는 영상을 만들 수 있다.

---

## 핵심 한 줄 요약

> **"내가 만든 캐릭터의 성격과 말투가 AI 스토리에 반영되고, 그 캐릭터가 영상 전체에서 일관되게 등장한다"**

---

## 전체 작동 원리

```mermaid
flowchart TD
    A([사용자 시작]) --> B[캐릭터 설정 입력]

    subgraph 입력방식 [선택 방식]
        B --> C1[완전 자동\n장르·분위기만 선택]
        B --> C2[반자동\n원하는 항목만 입력\n나머지는 AI 자동 완성]
    end

    C1 --> D[Claude AI\n스토리 + 장면 + 대사 생성]
    C2 --> D

    D --> E[Stable Diffusion\n장면별 이미지 생성\nIP-Adapter로 캐릭터 일관성 유지]
    D --> F[ElevenLabs\n캐릭터 성격에 맞는\nAI 음성 생성]
    D --> G[Suno AI\n장면 분위기에 맞는\nBGM 자동 생성]

    E --> H[ffmpeg + Whisper\n이미지 + 음성 + BGM 합성\n자막 자동 싱크]
    F --> H
    G --> H

    H --> I([쇼츠 완성\n미리보기 + 다운로드])
```

---

## 캐릭터 설정 구조

사용자는 원하는 항목만 입력하면 된다. 비워두면 AI가 자동으로 채운다.

```mermaid
flowchart LR
    subgraph 사용자_입력 [사용자 입력 선택사항]
        A1[이름·나이·직업]
        A2[성격 키워드\n활발함·냉소적·겁쟁이]
        A3[말투\n반말·존댓말·고어체]
        A4[외모 묘사\n파란 단발·안경·교복]
        A5[그림체 스타일\n애니메이션풍·웹툰풍]
        A6[장르·배경·분위기]
        A7[스토리 힌트\n원하는 방향 메모]
    end

    subgraph AI_처리 [AI 자동 완성]
        B1[비워둔 항목\nAI가 자동 생성]
        B2[입력한 항목\n그대로 반영]
    end

    A1 & A2 & A3 & A4 & A5 & A6 & A7 --> AI_처리
    AI_처리 --> C[완성된 캐릭터 설정\n→ 스토리 생성에 반영]
```

---

## AI 파이프라인 상세

```mermaid
flowchart TD
    subgraph Claude_AI [① Claude AI — 스토리 생성]
        S1[캐릭터 설정 입력]
        S2[장면 수 결정\n기본 8장면]
        S3[각 장면별 출력\n장면 묘사 + 대사 + 분위기 + BGM 힌트]
        S1 --> S2 --> S3
    end

    subgraph SD_AI [② Stable Diffusion — 이미지 생성]
        I1[장면 묘사\n→ 이미지 프롬프트 변환]
        I2[IP-Adapter 적용\n첫 캐릭터 이미지를\n레퍼런스로 고정]
        I3[장면마다 동일한\n캐릭터 외모 유지]
        I1 --> I2 --> I3
    end

    subgraph EL_AI [③ ElevenLabs — 음성 생성]
        V1[대사 텍스트 입력]
        V2[캐릭터 성격에 맞는\n목소리 선택]
        V3[감정 표현 반영\n음성 출력]
        V1 --> V2 --> V3
    end

    subgraph Suno_AI [④ Suno AI — BGM 생성]
        B1[장면 분위기 텍스트 입력]
        B2[오리지널 BGM 생성\n장면과 완벽히 매칭]
        B1 --> B2
    end

    subgraph Compose [⑤ ffmpeg + Whisper — 영상 합성]
        C1[이미지 시퀀스]
        C2[음성 파일]
        C3[BGM 파일]
        C4[Whisper 자막 싱크]
        C5[9:16 쇼츠 완성]
        C1 & C2 & C3 --> C4 --> C5
    end

    Claude_AI --> SD_AI
    Claude_AI --> EL_AI
    Claude_AI --> Suno_AI
    SD_AI & EL_AI & Suno_AI --> Compose
```

---

## 기술 스택

| 분류 | 기술 | 역할 |
|------|------|------|
| 프론트엔드 | React + Vite | 웹 UI 전체 |
| 백엔드 | FastAPI (Python) | REST API 서버 |
| 스토리 생성 | Claude Sonnet API | 캐릭터 설정 반영 스토리·대사·장면 묘사 |
| 이미지 생성 | Stable Diffusion 3.5 | 장면별 이미지 생성 |
| 캐릭터 일관성 | IP-Adapter | 장면마다 동일한 캐릭터 외모 유지 |
| 음성 생성 | ElevenLabs | 캐릭터 성격별 AI 음성 |
| BGM 생성 | Suno AI | 장면 분위기 맞춤 오리지널 BGM |
| 영상 합성 | ffmpeg | 이미지+음성+BGM → 최종 영상 |
| 자막 | Whisper | 음성 타임스탬프 추출 → 자막 싱크 |
| DB | PostgreSQL | 캐릭터·프로젝트 저장 |

---

## 시중 서비스와의 차별점

### 경쟁 서비스 비교

```mermaid
quadrantChart
    title 시중 서비스 포지셔닝
    x-axis 낮은 커스텀 --> 높은 커스텀
    y-axis 단순 영상 --> 캐릭터 스토리
    quadrant-1 StoryForge 목표 영역
    quadrant-2 캐릭터 중심 but 자동화 약함
    quadrant-3 단순 자동화
    quadrant-4 자동화 but 캐릭터 없음
    StoryForge: [0.85, 0.90]
    Katalist: [0.60, 0.65]
    StoryShort AI: [0.30, 0.20]
    MagicLight: [0.25, 0.45]
    Kling AI: [0.50, 0.35]
```

### 기능별 비교표

| 기능 | StoryForge | StoryShort | Katalist | MagicLight |
|------|:----------:|:----------:|:--------:|:----------:|
| 오리지널 캐릭터 생성 | ✅ | ❌ | ⚠️ 부분 | ❌ |
| 캐릭터 성격→대사 반영 | ✅ | ❌ | ❌ | ❌ |
| 캐릭터 외모 일관성 유지 | ✅ IP-Adapter | ❌ | ✅ | ❌ |
| AI 스토리 자동 생성 | ✅ | ✅ | ❌ | ⚠️ 템플릿 |
| AI 음성 생성 | ✅ | ✅ ElevenLabs | ❌ | ✅ |
| AI BGM 생성 | ✅ Suno | ❌ | ❌ | ❌ |
| 완전 자동 + 커스텀 동시 | ✅ | ❌ | ❌ | ❌ |
| 모든 요소 AI 생성 | ✅ | ⚠️ | ❌ | ⚠️ |

> ✅ 지원 | ⚠️ 부분 지원 | ❌ 미지원

### StoryForge만의 핵심 차별점 3가지

**1. 캐릭터가 스토리를 만든다**
기존 서비스는 스토리를 만들고 거기에 이미지를 붙인다.
StoryForge는 캐릭터의 성격·말투·관계가 스토리 자체를 결정한다.
같은 "판타지 로맨스"여도 캐릭터 설정에 따라 완전히 다른 이야기가 나온다.

**2. 장면마다 같은 캐릭터가 나온다**
기존 AI 영상 서비스의 가장 큰 문제는 "캐릭터 드리프트" — 장면마다 캐릭터 얼굴이 달라지는 것.
StoryForge는 IP-Adapter로 첫 생성 이미지를 레퍼런스로 고정해서 이 문제를 해결한다.

**3. 완전 자동이거나 완전 커스텀이거나**
기존 서비스는 완전 자동(선택지 없음) 또는 직접 다 설정하는 방식 둘 중 하나.
StoryForge는 원하는 것만 입력하고 나머지는 AI가 채우는 유연한 구조를 제공한다.

---

## StoryForge가 해결하는 문제

```mermaid
flowchart LR
    subgraph 기존_문제 [기존의 문제]
        P1[웹툰·쇼츠를 만들고 싶지만\n그림을 못 그린다]
        P2[스토리 아이디어는 있지만\n글쓰기가 서툴다]
        P3[내 캐릭터가 있지만\n영상으로 만들 수단이 없다]
        P4[기존 AI 도구는\n범용 콘텐츠만 만들어준다]
    end

    subgraph StoryForge_해결 [StoryForge 해결]
        S1[AI가 스토리에 맞는\n그림을 자동 생성]
        S2[캐릭터 설정만 입력하면\nAI가 스토리를 완성]
        S3[내 캐릭터가 영상 전체에\n일관되게 등장]
        S4[캐릭터 설정이 반영된\n나만의 오리지널 영상]
    end

    P1 --> S1
    P2 --> S2
    P3 --> S3
    P4 --> S4
```

---

## 수요 및 시장성

```mermaid
flowchart LR
    subgraph 타겟_사용자 [타겟 사용자]
        U1[그림 못 그리는\n웹툰 작가 지망생]
        U2[오리지널 캐릭터가 있는\n창작자·팬아트 작가]
        U3[쇼츠 콘텐츠를 만들고 싶은\n1인 크리에이터]
        U4[이야기를 영상으로\n만들고 싶은 누구나]
    end

    subgraph 시장_트렌드 [시장 트렌드]
        T1[숏폼 콘텐츠 시장\n폭발적 성장 중]
        T2[AI 크리에이터 도구\n수요 급증]
        T3[1인 미디어 시대\n제작 도구 민주화]
    end

    타겟_사용자 --> 수요확인[명확한 수요]
    시장_트렌드 --> 수요확인
    수요확인 --> 결론[누구나 쓸 수 있는\n오리지널 콘텐츠 제작 도구]
```

---

## 개발 로드맵

```mermaid
gantt
    title StoryForge 개발 일정
    dateFormat  YYYY-MM-DD
    section Phase 1 백엔드 기반
    FastAPI 초기화 + DB 연동     :p1, 2026-04-07, 4d
    캐릭터 설정 저장 API          :p2, after p1, 3d
    Claude API 스토리 생성        :p3, after p2, 5d
    React 초기화 + 캐릭터 폼 UI  :p4, after p3, 5d

    section Phase 2 이미지 + 음성
    Stable Diffusion 이미지 생성  :p5, after p4, 5d
    IP-Adapter 캐릭터 일관성      :p6, after p5, 4d
    ElevenLabs TTS 연동           :p7, after p6, 3d

    section Phase 3 BGM + 영상
    Suno AI BGM 생성              :p8, after p7, 3d
    ffmpeg 영상 합성              :p9, after p8, 4d
    Whisper 자막 싱크             :p10, after p9, 3d

    section Phase 4 완성도
    WebSocket 진행 상황           :p11, after p10, 3d
    UI/UX 완성도                  :p12, after p11, 5d
    전체 테스트 + 버그 수정       :p13, after p12, 4d
    발표 준비                     :p14, after p13, 3d
```

---

## 발표 시연 시나리오

1. **캐릭터 설정 입력** — "겁쟁이 고등학생 마법사, 파란 단발, 존댓말 사용, 공포 코믹 장르" 입력
2. **AI 스토리 생성** — Claude가 캐릭터 성격이 반영된 8장면 스토리 + 대사 자동 생성
3. **이미지 생성** — 각 장면에 맞는 이미지 생성, 모든 장면에서 동일한 캐릭터 등장 확인
4. **음성 + BGM** — 캐릭터 말투에 맞는 AI 음성, 공포 코믹 분위기 BGM 자동 생성
5. **쇼츠 완성** — 영상 합성 후 자막 포함 쇼츠 다운로드

> 버튼 몇 번으로 오리지널 캐릭터의 쇼츠가 완성된다.
