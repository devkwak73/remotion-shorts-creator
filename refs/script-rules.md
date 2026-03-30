# 대본 생성 규칙

## 포맷별 씬 구성

### 📱 숏폼 (30~60초) — 5본편 씬
```
Cover(2초) → Hold(0.3초) → 씬1 title → 씬2 checklist → 씬3 stat → 씬4 table → 씬5 outro
```
- 씬당 약 5~8초, TTS 속도 +25%

### 📺 미디폼 (5~10분) — 챕터 구조
```
인트로 → 챕터1~5 (각: 챕터타이틀 → explain×3~5 → 챕터요약) → 아웃트로
```
- TTS 속도 +20%

### 🎬 롱폼 (10분 이상) — 확장 챕터 구조
```
인트로 → 챕터1~8+ → (3~4챕터마다 중간요약) → 아웃트로
```
- TTS 속도 +15%

---

## 씬 타입 정의

| 타입 | 설명 | 사용 포맷 |
|------|------|-----------|
| `title` | 훅 + `bar_chart` 또는 `big_number` visual | 숏폼 씬1 |
| `content` | `checklist` / `stat_card` / `compare_table` visual | 숏폼 씬2~4 |
| `outro` | CTA 버튼 + 댓글 유도 | 모든 포맷 |
| `chapter_title` | 챕터 제목 카드 | 미디폼/롱폼 |
| `explain` | 설명 씬 — 모든 visual 타입 사용 가능 | 미디폼/롱폼 |
| `chapter_summary` | 챕터 핵심 요약 | 미디폼/롱폼 |

**visual 선택 기준:**
- `bar_chart` — 두 방법의 시간/비용 비교가 핵심일 때 (예: "4시간 vs 30분")
- `big_number` — 임팩트 있는 단일 수치가 핵심일 때 (예: "90% 실패율", "-3,500만원")

---

## 숏폼 대본 JSON 구조

```json
{
  "title": "영상 제목",
  "theme_color": "#hex",
  "cta_keyword": "댓글 키워드",
  "scenes": [
    {
      "type": "title",
      "text": "훅 텍스트\n(2줄)",
      "narration": "나레이션",
      "highlight": "강조단어",
      "visual": "bar_chart",
      "data": {
        "before_label": "기존 방법", "before_value": "4시간",
        "after_label": "자동화",    "after_value": "30분",
        "badge": "⚡ 87% 절감"
      }
    },
    {
      "type": "content",
      "text": "핵심 포인트\n(2줄)",
      "narration": "나레이션",
      "highlight": "강조단어",
      "visual": "checklist",
      "items": ["항목1", "항목2", "항목3", "항목4"]
    },
    {
      "type": "content",
      "text": "핵심 수치\n(2줄)",
      "narration": "나레이션",
      "highlight": "강조단어",
      "visual": "stat_card",
      "stat": {
        "value": "12.3", "unit": "%", "label": "평균 수익률",
        "sub_items": ["항목 A", "항목 B", "항목 C"],
        "badge": "변화 비교 텍스트"
      }
    },
    {
      "type": "content",
      "text": "비교 제목\n(2줄)",
      "narration": "나레이션",
      "highlight": "강조단어",
      "visual": "compare_table",
      "rows": [
        ["항목명", "기존값", "자동화값"],
        ["항목명", "기존값", "자동화값"],
        ["항목명", "기존값", "자동화값"]
      ],
      "total_badge": "총 절약 텍스트"
    },
    {
      "type": "outro",
      "text": "CTA 문구",
      "narration": "아웃트로 나레이션"
    }
  ]
}
```

**visual 옵션:** `bar_chart` | `big_number` | `checklist` | `stat_card` | `compare_table`

---

## 미디폼 / 롱폼 대본 JSON 구조

```json
{
  "format": "mid",
  "title": "영상 제목",
  "theme_color": "#hex",
  "cta_keyword": "댓글 키워드",
  "intro": {
    "hook": "강한 훅 문장",
    "preview": ["오늘 배울 포인트1", "포인트2", "포인트3"]
  },
  "chapters": [
    {
      "title": "챕터 제목",
      "scenes": [
        { "type": "chapter_title", "text": "챕터 제목", "narration": "챕터 소개" },
        { "type": "explain", "text": "설명 제목", "narration": "나레이션", "visual": "checklist", "items": [] },
        { "type": "explain", "text": "설명 제목", "narration": "나레이션", "visual": "stat_card", "stat": {} },
        { "type": "chapter_summary", "points": ["핵심1", "핵심2"], "narration": "챕터 요약" }
      ]
    }
  ],
  "outro": {
    "summary": ["전체 요약1", "요약2", "요약3"],
    "cta": "CTA 문구",
    "narration": "아웃트로 나레이션"
  }
}
```

---

## 씬당 프레임 계산

```
숏폼:
  SCENE_FRAMES = Math.floor(CONTENT_FRAMES / 5)
  마지막 씬 = CONTENT_FRAMES - SCENE_FRAMES * 4  (나머지 프레임 모두)

미디폼 / 롱폼 (씬 타입별 고정값):
  chapter_title:   150 프레임 (5초)
  explain:         300~600 프레임 (10~20초, 내용 밀도에 따라)
  chapter_summary: 180 프레임 (6초)
  intro:           300~450 프레임 (10~15초)
  outro:           450~600 프레임 (15~20초)
```

---

## 대본 작성 지침

1. `narration` 은 화면에 표시되지 않는 TTS 전용 텍스트 — 자연스러운 구어체로 작성
2. `text` 는 화면에 표시 — 짧고 강렬하게, 줄바꿈(`\n`)으로 2줄 구성
3. `highlight` 는 텍스트 중 가장 강조할 단어 1~2개 (테마색상 + 글로우 적용)
4. 수치/통계가 없으면 Claude가 채널 주제에 맞게 그럴듯한 예시 수치 생성
5. `cta_keyword` 는 댓글에 쓸 짧은 단어 (예: "자동화", "계산기", "필터")
