# CLAUDE.md

Claude Code (claude.ai/code)가 이 프로젝트에서 작업할 때 참고하는 가이드입니다.

## 프로젝트 개요

1983년 Broderbund Lode Runner의 웹 브라우저 재현판. 원작 Apple II/DOS 버전 기반, VGA 스타일 벡터 그래픽 사용.

## 실행 방법

`index.html`을 웹 브라우저에서 열기

## 아키텍처

### 파일 구조
| 파일 | 설명 | 크기 |
|------|------|------|
| `index.html` | 게임 컨테이너, 오버레이 UI | ~58줄 |
| `style.css` | 반응형 VGA 스타일, CSS 변수 | ~256줄 |
| `game.js` | 전체 게임 로직 (단일 IIFE) | ~6600줄 |

### 핵심 상수
```javascript
const BASE_TILE_SIZE = 24;
const LEVEL_WIDTH = 28;
const LEVEL_HEIGHT = 16;
const BASE_WIDTH = 672;   // 28 * 24
const BASE_HEIGHT = 384;  // 16 * 24
```

### 게임 상태 (STATE)
```javascript
TITLE: 0        // 타이틀 화면
PLAYING: 1      // 게임 진행 중
PAUSED: 2       // 일시정지
DYING: 3        // 사망 애니메이션
LEVEL_COMPLETE: 4  // 레벨 클리어
GAME_OVER: 5    // 게임 오버
STUCK: 6        // 스턱 (클리어 불가)
```

### 타일 타입 (TILE)
```javascript
EMPTY: 0        // 빈 공간
BRICK: 1        // 벽돌 (파괴 가능)
SOLID: 2        // 단단한 블록
LADDER: 3       // 사다리
BAR: 4          // 바 (매달리기)
TRAP: 5         // 함정 (벽돌처럼 보임)
GOLD: 6         // 금괴
ESCAPE_LADDER: 7 // 탈출 사다리
```

### 주요 클래스/메서드

#### LodeRunner 클래스
```javascript
// 초기화
constructor()
setupResponsiveCanvas()
setupInput()
loadLevel(levelNum)

// 게임 루프
gameLoop()
update()
render()

// 플레이어
updatePlayer()
collectGold(x, y)
dig(digX, digY)
playerDie()

// 적 AI
updateEnemies()
respawnEnemy(e)

// 스턱 감지
checkStuck()
findReachablePositions(startX, startY)
getPossibleMoves(x, y)
triggerStuck(reason)

// 렌더링 (벡터 그래픽)
drawTile(x, y, tile)
drawBrick(px, py)
drawPlayer(p)
drawEnemy(e)
drawRoundRect(x, y, w, h, r)
drawCircle(x, y, r)
createGradient(x1, y1, x2, y2, color1, color2)
createRadialGradient(x, y, r1, r2, color1, color2)
```

## 1983 원작 메카닉

- 150개 Classic 레벨 + 50개 Championship 레벨 구현
- 플레이어는 점프 불가, 무한 낙하 가능
- 벽돌만 파기 가능 (좌/우)
- 구멍은 180프레임(~6초) 후 복구
- 가드는 구멍에서 탈출 가능, 플레이어는 불가
- 플레이어는 가드 머리 위에 서기 가능
- 가드는 금괴 1개 운반/드롭
- 가드 사망 시 상단 랜덤 위치에서 리스폰
- 모든 금괴 수집 → 탈출 사다리 활성화
- 레벨 클리어 → +2000점, +1 목숨

## 조작법

| 동작 | 키 |
|------|-----|
| 이동 | 방향키 / I,J,K,L / W,A,S,D |
| 왼쪽 파기 | Z / U / F1 |
| 오른쪽 파기 | X / O / F2 |
| 시작/재개 | ENTER |
| 일시정지 | ESC |
| 레벨 재시작 | R |

## 레벨 포맷

```
. = empty    # = brick    @ = solid    H = ladder
- = bar      $ = gold     T = trap     P = player
E = enemy
```

28x16 그리드 (일부 레벨은 바닥이 없는 특수 구조)

## 구현된 기능

### 벡터 그래픽
- 해상도 독립적 렌더링
- 그라디언트, 라운드 사각형, 원형
- devicePixelRatio 지원 (Retina)

### 반응형 스케일링
- 브라우저 크기에 맞게 0.5x ~ 2.5x 스케일
- CSS 변수 (`--scale`, `--game-width`, `--game-height`)

### 향상된 적 AI
- 우선순위 기반 이동 결정
- 사다리/바에서 지능적 이동
- 랜덤 요소로 교착 방지
- 구멍에 빠지면 해당 위치에 고정 (아래층 낙하 불가)

### 스턱 감지 시스템
- BFS 기반 경로 탐색
- 도달 불가능한 금괴 감지
- 구멍에 갇힘 감지
- 층간 이동 불가 감지
- 탈출 사다리 도달 불가 감지

### 사망 오버레이
- 스턱 원인 설명 표시
- ENTER로 계속 진행

## 커스텀 스킬

`.claude/skills/` 폴더의 프로젝트 전용 스킬:

| 스킬 | 용도 |
|------|------|
| `/commit` | Git 커밋 (한글, 타입별) |
| `/level` | 새 레벨 생성 |
| `/test-play` | 테스트 체크리스트 |
| `/optimize` | 성능 최적화 분석 |
| `/review` | 코드 리뷰 |
| `/debug` | 버그 디버깅 |

## 코드 수정 시 주의사항

1. **IIFE 구조 유지**: 전역 변수 오염 방지
2. **프레임 기반 타이밍**: setTimeout 대신 frameCount 사용
3. **좌표 시스템**: 타일 좌표 (0~27, 0~15)와 픽셀 좌표 구분
4. **스케일 적용**: 모든 렌더링에 SCALE 또는 TILE_SIZE 사용
5. **원작 충실도**: 새 기능 추가 시 1983 메카닉 우선 고려
