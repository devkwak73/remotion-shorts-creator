# 영상 제작 규칙

## 기본 파일 생성 코드

### `src/styles.ts`

```typescript
export const CHANNEL = {
  name: '[채널명]',       // setup 인터뷰에서 입력받은 값으로 교체
  accent: '[hex색상]',   // setup 인터뷰에서 선택한 값으로 교체
};

export const COLORS = {
  bg:      '#0A0A0A',
  accent:  CHANNEL.accent,
  text:    '#FFFFFF',
  subtext: '#AAAAAA',
};

export const TYPE_SCALE = {
  hook:    72,
  body:    52,
  caption: 44,
  sub:     36,
};

export const GLOWS = {
  accent: `0 0 8px ${CHANNEL.accent}E6, 0 0 20px ${CHANNEL.accent}99, 0 0 45px ${CHANNEL.accent}4D`,
  green:  '0 0 8px rgba(74,222,128,0.9), 0 0 20px rgba(74,222,128,0.5), 0 0 40px rgba(74,222,128,0.25)',
  red:    '0 0 8px rgba(255,68,68,0.8),  0 0 20px rgba(255,68,68,0.4), 0 0 35px rgba(255,68,68,0.2)',
  blue:   '0 0 8px rgba(56,139,255,0.9), 0 0 20px rgba(56,139,255,0.5), 0 0 40px rgba(56,139,255,0.25)',
};
```

### `src/components/SceneIndicator.tsx`

```tsx
import { COLORS } from '../styles';

interface Props {
  current: number;
  total?: number;
  accentColor?: string;
}

export const SceneIndicator: React.FC<Props> = ({ current, total = 5, accentColor = COLORS.accent }) => (
  <div style={{ position: 'absolute', top: 160, right: 60, display: 'flex', alignItems: 'baseline', gap: 4, zIndex: 10 }}>
    <span style={{ fontSize: 40, fontWeight: 900, color: accentColor, fontFamily: 'Noto Sans KR', textShadow: `0 0 12px ${accentColor}99` }}>
      {String(current).padStart(2, '0')}
    </span>
    <span style={{ fontSize: 24, color: 'rgba(255,255,255,0.3)', fontFamily: 'Noto Sans KR' }}>
      /{String(total).padStart(2, '0')}
    </span>
  </div>
);
```

### `src/components/ChannelBadge.tsx`

```tsx
import { COLORS, CHANNEL } from '../styles';

export const ChannelBadge: React.FC = () => (
  <div style={{ position: 'absolute', top: 160, left: 64, display: 'flex', alignItems: 'center', gap: 10, zIndex: 20 }}>
    <div style={{ width: 6, height: 36, backgroundColor: COLORS.accent, borderRadius: 3, boxShadow: `0 0 8px ${COLORS.accent}CC` }} />
    <span style={{ fontSize: 34, fontWeight: 900, color: COLORS.accent, fontFamily: 'Noto Sans KR', letterSpacing: 2, textShadow: `0 0 8px ${COLORS.accent}E6, 0 0 20px ${COLORS.accent}80` }}>
      {CHANNEL.name}
    </span>
  </div>
);
```

---

## 씬 내부 DOM 레이어 순서

모든 씬에서 아래 순서를 지킨다:

```tsx
<AbsoluteFill style={{ backgroundColor: COLORS.bg, ... }}>
  {/* 1. 배경 이모지 — 맨 뒤 */}
  <div style={{ position: 'absolute', fontSize: 440, opacity: 0.045, pointerEvents: 'none', userSelect: 'none', ... }}>
    이모지
  </div>

  {/* 2. 방사형 그라디언트 오버레이 */}
  <div style={{ position: 'absolute', inset: 0, pointerEvents: 'none', background: 'radial-gradient(...)' }} />

  {/* 3. SceneIndicator — 우상단 고정 */}
  <SceneIndicator current={N} accentColor={테마색상} />

  {/* 4. 실제 콘텐츠 */}
  <div style={{ ... }}>...</div>
</AbsoluteFill>
```

### 씬별 그라디언트 위치

| 씬 번호 | 위치 | 예시 |
|---------|------|------|
| 씬 1 | 좌하단 | `at 15% 90%` |
| 씬 2 | 우상단 | `at 85% 10%` |
| 씬 3 | 하단 중앙 | `at 50% 100%` |
| 씬 4 | 좌우 분할 | 두 개 그라디언트 레이어 |
| 씬 5 (outro) | 중앙 | `at 50% 50%` |

그라디언트 강도: `rgba(테마색상, 0.12~0.18)`, 반경 `60%~80%`

---

## visual 타입별 컴포넌트 패턴

### `bar_chart` — 두 항목 비교 바

```tsx
// 기존 방법 바: 빨강, spring width → ~90~95%
// 새 방법 바:  테마색상, spring width → ~5~15%
// 뱃지: "⚡ XX% 절감" — scale spring으로 등장
// 레이블은 바 오른쪽에 값 표시
const barWidth = interpolate(spring({ fps, frame, config: { damping: 14 } }), [0,1], [0, targetPct]);
```

### `big_number` — 대형 카운팅 숫자

```tsx
// 0 → 목표값 카운팅 (interpolate)
// fontSize: 130~160, 테마색상 + GLOWS.accent
// 서브 텍스트 (단위, 설명)
// 하단 임팩트 뱃지
const countVal = interpolate(frame, [fps*0.3, fps*2.5], [0, targetValue], { extrapolateLeft: 'clamp', extrapolateRight: 'clamp' });
```

### `checklist` — 4항목 리스트

```tsx
// 각 항목 spring 스태거: frame - i * 7
// translateX: (1 - spring) * -60 → 0 (좌측 슬라이드인)
// 좌측: 테마색상 원형 뱃지 (번호 또는 이모지)
// 카드: borderLeft 4px 테마색상, 등장 후 boxShadow 글로우
const itemEnter = spring({ fps, frame: frame - i * 7, config: { damping: 14 } });
```

### `stat_card` — 통계 카드

```tsx
// 상단 헤더 레이블
// 카드: 테마색상 테두리 + boxShadow, scale spring 등장
// 대형 카운팅 숫자 + 단위
// 세부 항목 3개: ✓ 값 (테마색상 + glow)
// 하단 변화 비교 뱃지
const cardEnter = spring({ fps, frame: frame - fps * 0.2, config: { damping: 12 } });
```

### `compare_table` — 3행 비교표

```tsx
// 헤더: 항목명 | 기존(빨강) | 자동화(테마색상)
// 각 행 spring 스태거 (i * 8프레임), translateX 좌측 슬라이드인
// 기존값: color '#FF4444', textDecoration 'line-through', opacity 0.7
// 새값: 테마색상 + textShadow glow
// 하단 합계 뱃지: 테마색상 배경, 볼드
const rowEnter = spring({ fps, frame: frame - i * 8 - fps * 0.3, config: { damping: 14 } });
```

### `outro` — CTA 씬

```tsx
// 상단 6px 라인 (테마색상)
// 펄스 원 배경 (frame % (fps*0.9) 사이클, opacity 0.08~0.12)
// 사회적 증거 칩 "댓글 받는 중" (초록 점 + 텍스트)
// CTA 버튼: 테마색상 배경, 펄스에 맞춰 boxShadow 강도 변동
// 바운스 화살표 👇 (frame % (fps*0.6) 사이클)
```

---

## 커버 씬 구성 요소

```tsx
// 1. 배경 이모지 (opacity 0.035, 테마 관련 이모지)
// 2. 그라디언트 (테마색상, 상단 중앙 at 50% 0%)
// 3. 파티클 8개 (테마색상 원, spring 스태거 i*4프레임)
// 4. 서브 태그 칩 (카테고리 레이블, rounded pill)
// 5. 메인 타이틀 2줄 — 2번째 줄 테마색상 + glow
// 6. 구분선 (interpolate 너비 0→80%)
// 7. 하단 뱃지 (아이콘 + 핵심 메시지)
// ※ 채널명은 ChannelBadge 전역 컴포넌트로 표시 — 커버에 중복 금지
```

---

## 메인 컴포지션 파일 템플릿

`src/Shorts[ID].tsx`:

```tsx
import { AbsoluteFill, Audio, Sequence, staticFile, useCurrentFrame, interpolate } from 'remotion';
import { loadFont } from '@remotion/google-fonts/NotoSansKR';
import { COLORS } from './styles';
import { ChannelBadge } from './components/ChannelBadge';
import { CoverScene[ID] }  from './scenes/CoverScene[ID]';
import { TitleScene[ID] }  from './scenes/v[ID]/TitleScene[ID]';
// ... 씬 임포트

loadFont();

const FPS           = 30;
const COVER_FRAMES  = 60;   // 커버 2초
const HOLD_FRAMES   = 9;    // 전환 홀드 0.3초
const CONTENT_START = COVER_FRAMES + HOLD_FRAMES; // 69

const TOTAL_DURATION_MS = /* parse-vtt 결과값 */;
const CONTENT_FRAMES    = Math.ceil((TOTAL_DURATION_MS / 1000) * FPS);
const SCENE_FRAMES      = Math.floor(CONTENT_FRAMES / 5); // 숏폼 기준

export const TOTAL_FRAMES_[ID] = CONTENT_START + CONTENT_FRAMES;

export const Shorts[ID]: React.FC = () => {
  const frame = useCurrentFrame();
  const progress = interpolate(frame, [0, TOTAL_FRAMES_[ID] - 1], [0, 100], {
    extrapolateLeft: 'clamp', extrapolateRight: 'clamp',
  });

  return (
    <AbsoluteFill style={{ backgroundColor: '#0A0A0A' }}>
      {/* 오디오 — 커버+홀드 이후 시작 */}
      <Sequence from={CONTENT_START}>
        <Audio src={staticFile('audio-[id]/voiceover.mp3')} />
      </Sequence>

      {/* 커버 */}
      <Sequence from={0} durationInFrames={COVER_FRAMES}>
        <CoverScene[ID] durationInFrames={COVER_FRAMES} />
      </Sequence>

      {/* 본편 씬 1~5 */}
      <Sequence from={CONTENT_START}                      durationInFrames={SCENE_FRAMES}><TitleScene[ID]   durationInFrames={SCENE_FRAMES} /></Sequence>
      <Sequence from={CONTENT_START + SCENE_FRAMES}       durationInFrames={SCENE_FRAMES}><ListScene[ID]    durationInFrames={SCENE_FRAMES} /></Sequence>
      <Sequence from={CONTENT_START + SCENE_FRAMES * 2}   durationInFrames={SCENE_FRAMES}><StatScene[ID]    durationInFrames={SCENE_FRAMES} /></Sequence>
      <Sequence from={CONTENT_START + SCENE_FRAMES * 3}   durationInFrames={SCENE_FRAMES}><TableScene[ID]   durationInFrames={SCENE_FRAMES} /></Sequence>
      <Sequence from={CONTENT_START + SCENE_FRAMES * 4}   durationInFrames={CONTENT_FRAMES - SCENE_FRAMES * 4}>
        <OutroScene[ID] durationInFrames={CONTENT_FRAMES - SCENE_FRAMES * 4} />
      </Sequence>

      {/* 전역 오버레이 */}
      <ChannelBadge />
      <div style={{ position: 'absolute', bottom: 0, left: 0, right: 0, height: 5, backgroundColor: 'rgba(255,255,255,0.08)' }}>
        <div style={{ height: '100%', width: `${progress}%`, backgroundColor: COLORS.accent, boxShadow: `0 0 12px ${COLORS.accent}CC`, borderRadius: '0 3px 3px 0' }} />
      </div>
    </AbsoluteFill>
  );
};
```

## `Root.tsx` 업데이트 패턴

```tsx
import { Shorts[ID], TOTAL_FRAMES_[ID] } from './Shorts[ID]';

// RemotionRoot 내부에 추가:
<Composition
  id="Shorts[ID]"
  component={Shorts[ID]}
  durationInFrames={TOTAL_FRAMES_[ID]}
  fps={30}
  width={1080}
  height={1920}
  defaultProps={{}}
/>
```

---

## 색상 테마 가이드

| 색상 | HEX | 어울리는 주제 |
|------|-----|--------------|
| 🟡 노랑 | `#FFE500` | 자동화, 시간 절약, 효율 |
| 🔴 빨강 | `#FF4444` | 주의, 실수, 위험, 손해 |
| 🟢 초록 | `#4ADE80` | 수익, 성공, 성장, 돈 |
| 🔵 파랑 | `#388BFF` | 정보, 데이터, 분석, IT |
| 🟣 보라 | `#A78BFA` | 프리미엄, 전략, 브랜딩 |
| 🟠 주황 | `#FB923C` | 에너지, 도전, 부동산 |
| 🩷 핑크 | `#F472B6` | 라이프스타일, 뷰티, 육아 |

---

## 배경 이모지 가이드

주제별 추천 이모지 (opacity 0.045, fontSize 440~460):

| 카테고리 | 이모지 |
|----------|--------|
| 부동산/경매 | 🏠 🏢 🔑 |
| 수익/돈 | 💰 💵 📈 |
| 자동화/IT | ⚡ 🤖 💻 |
| 시간/효율 | ⏱ 🚀 ⚙️ |
| 위험/경고 | ⚠️ 🚨 🛡️ |
| 분석/데이터 | 📊 📋 🔍 |
| 계산/수학 | 🧮 📐 🔢 |

---

## 고정 규칙 (모든 영상 공통)

| 항목 | 값 |
|------|-----|
| 채널명 위치 | `top: 160, left: 64` |
| 씬 번호 위치 | `top: 160, right: 60` |
| 하단 진행 바 | `height: 5px, bottom: 0`, 테마색상 |
| TTS 음성 | `ko-KR-SunHiNeural` |
| TTS 속도 | 숏폼 `+25%`, 미디폼 `+20%`, 롱폼 `+15%` |
| 해상도 | `1080 × 1920` |
| 프레임레이트 | `30fps` |
| 인코딩 | `h264, crf=18` |
| 커버 | `60프레임(2초) + 9프레임(0.3초) 홀드` |
| 폰트 | `Noto Sans KR` — `loadFont()` 반드시 호출 |

---

## 트러블슈팅

| 문제 | 해결 |
|------|------|
| `edge-tts` 명령 없음 | `python -m edge_tts` 로 대체 (fallback 코드 필수) |
| 폰트 미적용 | `loadFont()` 호출 확인, `fontFamily: 'Noto Sans KR'` 명시 |
| 오디오 없음 에러 | `public/audio-[id]/voiceover.mp3` 경로 확인 |
| `borderTop` on AbsoluteFill 무효 | 내부 별도 div에 `position: absolute, top: 0` 적용 |
| 렌더 느림 | `--concurrency=4` 옵션 추가 |
| VTT 파싱 0개 | 타임스탬프 `,` vs `.` 구분자 확인 → 정규식 `[.,]` 사용 |
