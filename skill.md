---
name: remotion-shorts-creator
description: Remotion 기반 한국어 숏폼/미디폼/롱폼 영상 자동 제작 스킬. 아무것도 없는 환경에서 환경설정, 채널 브랜딩 설정, 주제 인터뷰, 대본 AI 생성, TTS, 씬 컴포넌트 제작, MP4 렌더링까지 원스톱 처리. Claude + 인터넷만 있으면 누구나 사용 가능.
triggers:
  - "remotion-shorts-creator"
  - "쇼츠 스킬"
  - "영상 만들어줘"
  - "유튜브 쇼츠 자동화"
  - "숏폼 제작"
  - "동영상 자동 제작"
---

# Remotion Shorts Creator

Claude + 인터넷 연결만으로 한국어 쇼츠/미디폼/롱폼 영상을 자동 제작하는 스킬입니다.

> 제작: **부놈** · dev.kwak73@gmail.com · https://github.com/devkwak73

> **참조 파일** (필요한 단계에서 Read):
> - `refs/script-rules.md` — 대본 구조, 씬 타입, 프레임 계산
> - `refs/video-rules.md` — 컴포넌트 패턴, 색상 테마, 공통 규칙

---

## 실행 모드

| 명령 | 설명 |
|------|------|
| `/remotion-shorts-creator setup` | 최초 환경 구성 + 채널 세팅 |
| `/remotion-shorts-creator new` | 새 영상 제작 (인터뷰 → 대본 → 제작) |
| `/remotion-shorts-creator run "주제"` | 인터뷰 생략, 주제만으로 즉시 제작 |

---

## SETUP 모드

### Phase 1 — 환경 점검

```bash
node --version    # Node.js 18+ 필요
python --version  # Python 3.8+ 필요
pip --version
```

없으면 안내:
- Node.js: https://nodejs.org (LTS 설치)
- Python: https://python.org (PATH 추가 체크 필수)

### Phase 2 — 채널 인터뷰 (AskUserQuestion)

1. 프로젝트를 어느 폴더에 만들까요? (예: `C:\Users\이름\Videos`)
2. 채널 이름은 무엇인가요? (영상 좌상단에 항상 표시)
3. 채널 메인 색상:
   - 🟡 노랑 `#FFE500` / 🔴 빨강 `#FF4444` / 🟢 초록 `#4ADE80`
   - 🔵 파랑 `#388BFF` / 🟣 보라 `#A78BFA` / ✏️ 직접 입력
4. 주요 시청자는? (예: 경매 투자자, 직장인)
5. 채널 핵심 주제는? (예: 부동산 경매, 재테크)
6. 영상 말투: 친근한 구어체 / 전문적 격식체 / 유튜브 훅 스타일

### Phase 3 — 프로젝트 생성

```bash
cd [사용자_지정_경로]
npx create-video@latest my-shorts --yes --blank
cd my-shorts
npm install
npm install @remotion/captions @remotion/google-fonts
pip install edge-tts
mkdir -p public/audio scripts out src/scenes src/components
```

### Phase 3-B — 채널 설정 저장

인터뷰 답변을 스킬 디렉터리의 `.channel-config.json` 에 저장합니다 (Write 도구 사용):

```json
{
  "project_path": "[사용자가 입력한 절대 경로]",
  "channel_name": "[채널명]",
  "theme_color": "[hex 색상]",
  "audience": "[주요 시청자]",
  "topic": "[채널 핵심 주제]",
  "tone": "[말투]"
}
```

> 저장 경로: 이 `skill.md` 파일과 **같은 폴더** (`.claude/skills/remotion-shorts-creator/.channel-config.json`)

### Phase 4 — 기본 파일 생성

> **`refs/video-rules.md` 를 Read한 후** styles.ts, SceneIndicator.tsx, ChannelBadge.tsx 생성

---

## NEW 모드 — 영상 제작

### Step 0 — 채널 설정 로드

> 이 `skill.md` 와 **같은 폴더**의 `.channel-config.json` 을 Read합니다.
> - 파일이 있으면: `project_path`, `channel_name`, `theme_color` 등을 자동으로 불러와서 인터뷰에 기본값으로 사용합니다.
> - 파일이 없으면: "먼저 `/remotion-shorts-creator setup` 을 실행해 주세요." 안내 후 중단합니다.

### Step 1 — 인터뷰 (AskUserQuestion)

1. 이번 영상 주제는?
2. 영상 길이:
   - 📱 **숏폼** (30~60초) — 유튜브 쇼츠·릴스·틱톡
   - 📺 **미디폼** (5~10분) — 유튜브 일반 영상
   - 🎬 **롱폼** (10분 이상) — 유튜브 심화 영상
3. 시청자가 댓글에 남길 키워드? (예: "자동화", "계산기")
4. 핵심 수치/통계가 있나요? (없으면 Claude가 생성)
5. 이번 영상 테마 색상 — 채널 기본색 사용? 아니면 변경?

> **⚠️ 필수 규칙**: 어떤 항목이든 사용자가 "직접 입력"을 선택했다면, **반드시 해당 항목을 다시 텍스트로 질문**하고 실제 답변을 받은 후 다음 단계로 넘어가세요. 추측하거나 자동으로 채우는 것은 절대 금지입니다.

### Step 2 — 대본 생성

> **`refs/script-rules.md` 를 Read한 후** 인터뷰 내용을 바탕으로 대본 JSON 생성

### Step 3 — TTS 생성

`scripts/generate-tts-[id].py` 생성 후 실행:

```python
import subprocess, sys, os

narrations = ["씬1 나레이션", "씬2 나레이션", ...]  # 대본 JSON에서 추출
full_narration = " ".join(narrations)

base_dir  = os.path.dirname(os.path.dirname(os.path.abspath(__file__)))
audio_dir = os.path.join(base_dir, "public", "audio-[id]")
os.makedirs(audio_dir, exist_ok=True)
output_mp3 = os.path.join(audio_dir, "voiceover.mp3")
output_vtt = os.path.join(audio_dir, "voiceover.vtt")

# 숏폼 +25% / 미디폼 +20% / 롱폼 +15%
RATE = "+25%"

cmd = ["--voice", "ko-KR-SunHiNeural", "--rate", RATE,
       "--text", full_narration, "--write-media", output_mp3, "--write-subtitles", output_vtt]
try:
    subprocess.run(["edge-tts"] + cmd, check=True)
except FileNotFoundError:
    subprocess.run([sys.executable, "-m", "edge_tts"] + cmd, check=True)

print(f"완료: {output_mp3}")
```

VTT 파싱으로 실제 오디오 길이 확인:

```python
import re, os

def to_ms(t):
    h, m, s = t.split(':')
    return int(h)*3600000 + int(m)*60000 + int(float(s.replace(',','.'))*1000)

# base_dir: 이 스크립트가 scripts/ 폴더에 있다고 가정
base_dir = os.path.dirname(os.path.dirname(os.path.abspath(__file__)))
vtt_path = os.path.join(base_dir, "public", "audio-[id]", "voiceover.vtt")
with open(vtt_path, 'r', encoding='utf-8') as f:
    content = f.read()

matches = re.findall(r'(\d{2}:\d{2}:\d{2}[.,]\d{3}) --> (\d{2}:\d{2}:\d{2}[.,]\d{3})', content)
if matches:
    print(f"총 길이: {to_ms(matches[-1][1])}ms")
```

### Step 4 — 씬 컴포넌트 생성

> **`refs/video-rules.md` 를 Read한 후** 대본 JSON의 `visual` 타입에 맞게 씬 컴포넌트 생성
>
> 파일 위치: `src/scenes/CoverScene[ID].tsx`, `src/scenes/v[ID]/씬파일들.tsx`, `src/scenes/OutroScene[ID].tsx`

### Step 5 — 메인 컴포지션 생성

> **`refs/video-rules.md` 를 Read한 후** `src/Shorts[ID].tsx` 와 `Root.tsx` 업데이트

### Step 6 — 렌더링

```bash
cd [프로젝트_경로]
npx remotion render Shorts[ID] out/video-[ID].mp4 --codec=h264 --crf=18
```

---

## RUN 모드 — 즉시 제작

인터뷰 없이 주제만으로 바로 영상을 만듭니다. Claude가 주제를 분석해서 대본·테마·수치를 모두 자동 결정합니다.

> **먼저** 이 `skill.md` 와 같은 폴더의 `.channel-config.json` 을 Read하여 `project_path` 를 자동으로 불러옵니다. 없으면 setup 실행 안내 후 중단합니다.

1. 주제에서 핵심 메시지 추출 (훅, CTA 키워드 자동 결정)
2. 주제에 맞는 테마 색상 자동 선택 (`refs/video-rules.md` 색상 가이드 참고)
3. `refs/script-rules.md` 를 Read하여 숏폼 대본 JSON 자동 생성
4. NEW 모드의 Step 3~6 동일하게 실행
