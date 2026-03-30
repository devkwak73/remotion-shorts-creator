# Remotion Shorts Creator — Claude Code Skill

Claude Code + 인터넷 연결만으로 한국어 유튜브 쇼츠/미디폼/롱폼 영상을 자동 제작하는 스킬입니다.

환경설정부터 대본 AI 생성, TTS, Remotion 씬 컴포넌트 제작, MP4 렌더링까지 원스톱으로 처리합니다.

---

## 설치 방법

### 방법 1 — git clone (업데이트 자동)

```bash
# Windows
git clone https://github.com/devkwak73/remotion-shorts-creator "%USERPROFILE%\.claude\skills\remotion-shorts-creator"

# Mac / Linux
git clone https://github.com/devkwak73/remotion-shorts-creator ~/.claude/skills/remotion-shorts-creator
```

### 방법 2 — ZIP 다운로드

1. 이 페이지 상단 **Code → Download ZIP** 클릭
2. 압축 해제 후 폴더를 아래 경로로 복사:
   - Windows: `C:\Users\[이름]\.claude\skills\remotion-shorts-creator\`
   - Mac/Linux: `~/.claude/skills/remotion-shorts-creator/`

---

## 필수 조건

- [Claude Code](https://claude.ai/code) 설치: `npm install -g @anthropic-ai/claude-code`
- Node.js 18+
- Python 3.8+
- `pip install edge-tts`

---

## 사용 방법

Claude Code 실행 후:

```
/remotion-shorts-creator setup   # 최초 환경 구성 + 채널 세팅
/remotion-shorts-creator new     # 새 영상 제작 (인터뷰 → 대본 → 제작)
/remotion-shorts-creator run "주제"  # 주제만 입력하면 바로 제작
```

---

## 파일 구조

```
remotion-shorts-creator/
├── skill.md              ← YAML 헤더 + 실행 오케스트레이션
└── refs/
    ├── script-rules.md   ← 대본 생성 규칙 (JSON 구조, 씬 타입, 프레임 계산)
    └── video-rules.md    ← 영상 제작 규칙 (컴포넌트 패턴, 색상 테마, 공통 규칙)
```

---

## 지원 포맷

| 포맷 | 길이 | 용도 |
|------|------|------|
| 📱 숏폼 | 30~60초 | 유튜브 쇼츠, 인스타 릴스, 틱톡 |
| 📺 미디폼 | 5~10분 | 유튜브 일반 영상 |
| 🎬 롱폼 | 10분 이상 | 유튜브 심화/강의 영상 |

---

## 업데이트

```bash
cd ~/.claude/skills/remotion-shorts-creator
git pull
```

---

## 제작자

**부놈** — 부동산 경매 자동화 유튜브 채널
📧 dev.kwak73@gmail.com
🐙 https://github.com/devkwak73
