# Lode Runner (1983) - 에이전트 작업 가이드

이 문서는 이 저장소에서 코드 작업할 때 참고할 구현 기준입니다.
기준 소스는 `game.js`입니다.

## 실행

- `index.html`을 브라우저에서 열어 실행

## 프로젝트 구조

```text
loderunner/
├── index.html
├── style.css
├── game.js
├── README.md
├── RULES.md
└── AGENTS.md
```

## 핵심 상수

```javascript
const BASE_TILE_SIZE = 24;
const LEVEL_WIDTH = 28;
const LEVEL_HEIGHT = 16;
const BASE_WIDTH = BASE_TILE_SIZE * LEVEL_WIDTH;   // 672
const BASE_HEIGHT = BASE_TILE_SIZE * LEVEL_HEIGHT; // 384
```

## 색상 팔레트 (기본)

```javascript
const COLORS = {
  BLACK: '#000000',
  WHITE: '#FFFFFF',
  RED: '#FF0000',
  GREEN: '#00FF00',
  ORANGE: '#FF8800',
  PURPLE: '#AA00FF',
  CYAN: '#00AAAA',
  YELLOW: '#FFFF00',
  BROWN: '#8B4513'
};
```

## 상태값

| 상태 | 값 |
|------|----|
| `TITLE` | 0 |
| `PLAYING` | 1 |
| `PAUSED` | 2 |
| `DYING` | 3 |
| `LEVEL_COMPLETE` | 4 |
| `GAME_OVER` | 5 |
| `STUCK` | 6 |
| `EDITOR` | 7 |

## 타일값

| 값 | 타입 | 문자 |
|----|------|------|
| 0 | `EMPTY` | `.` |
| 1 | `BRICK` | `#` |
| 2 | `SOLID` | `@` |
| 3 | `LADDER` | `H` |
| 4 | `BAR` | `-` |
| 5 | `TRAP` | `T` |
| 6 | `GOLD` | `$` |
| 7 | `ESCAPE_LADDER` | `E` |

## 현재 구현 메카닉

플레이어:
- 점프 불가
- 낙하 데미지 없음
- 금 1개만 운반
- 좌/우 대각선 아래 BRICK만 파기 가능

적:
- BRICK 위 이동 가능
- 금 1개 소지 가능
- 구멍에 빠지면 탈출 시도
- 금 줍기/드롭 확률: 5% / 2%

클리어:
- 모든 금 수집 후 탈출 사다리 활성화
- 상단 도달 시 클리어 (+1500, +1 목숨)

## 타이밍/속도 상수 (30fps)

```javascript
const HOLE_FILL_TIME = 210;          // ~7초
const ENEMY_TRAPPED_TIME = 150;      // ~5초
const ENEMY_TRAPPED_ESCAPE = 60;     // 탈출 시도 시작
const GOLD_PICKUP_CHANCE = 0.05;
const GOLD_DROP_CHANCE = 0.02;
const PLAYER_SPEED = 0.14;
const ROPE_SPEED_MULTIPLIER = 1.15;
```

60fps 환산:
- `HOLE_FILL_TIME`: 420
- `ENEMY_TRAPPED_TIME`: 300

## 점수

| 행동 | 점수 |
|------|------|
| 금 획득 | 250 |
| 적 처리 | 75 |
| 레벨 클리어 | 1500 |

## 조작

| 동작 | 키 |
|------|-----|
| 이동 | 방향키 / `A,D` / `J,L` / `W,S` / `I,K` |
| 왼쪽/오른쪽 파기 | `Z/X`, `U/O`, `F1/F2` |
| 시작/재개 | `ENTER` |
| 일시정지 | `ESC` |
| 재시작 | `R` |
| 음소거 | `M` |
| 모드 전환(타이틀) | `C` |
| 에디터 토글 | `E` |
| 디버그 레벨 이동 | `Shift+N`, `Shift+P` |

## 레벨

- 클래식 150 + 챔피언십 50 (총 200)
- 문자 포맷: `. # @ H - $ T P E`

## 코드 수정 시 주의

1. IIFE 구조 유지
2. 프레임 기반 타이밍 유지
3. 타일 좌표/픽셀 좌표 구분
4. 반응형 스케일 처리 유지
5. 원작 메카닉 우선
