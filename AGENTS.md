# Lode Runner (1983) - Claude Code 가이드

1983년 Broderbund Lode Runner의 웹 브라우저 재현판입니다.

## 실행

`index.html`을 웹 브라우저에서 열기

## 프로젝트 구조

```
loderunner/
├── index.html    # 게임 컨테이너 (57줄)
├── style.css    # 반응형 VGA 스타일 (255줄)
├── game.js      # 게임 로직 - 단일 IIFE (6646줄)
└── .claude/skills/ # 커스텀 스킬
```

## 핵심 상수

```javascript
const BASE_TILE_SIZE = 24;
const LEVEL_WIDTH = 28;
const LEVEL_HEIGHT = 16;
const BASE_WIDTH = BASE_TILE_SIZE * LEVEL_WIDTH;   // 672
const BASE_HEIGHT = BASE_TILE_SIZE * LEVEL_HEIGHT; // 384
```

## 게임 상태

| State | 값 | 설명 |
|-------|-----|------|
| TITLE | 0 | 타이틀 화면 |
| PLAYING | 1 | 게임 진행 중 |
| PAUSED | 2 | 일시정지 |
| DYING | 3 | 사망 애니메이션 |
| LEVEL_COMPLETE | 4 | 레벨 클리어 |
| GAME_OVER | 5 | 게임 오버 |
| STUCK | 6 | 스턱 (클리어 불가) |

## 타일 타입

| 값 | 타입 | 설명 |
|----|------|------|
| 0 | EMPTY | 빈 공간 |
| 1 | BRICK | 벽돌 (파괴 가능) |
| 2 | SOLID | 단단한 블록 |
| 3 | LADDER | 사다리 |
| 4 | BAR | 바 (매달리기) |
| 5 | TRAP | 함정 |
| 6 | GOLD | 금괴 |
| 7 | ESCAPE_LADDER | 탈출 사다리 |

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

## 주요 메서드

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

// DEMO 모드
startDemo()
updateDemo()
updateDemoKeyDisplay()
```

## 1983 원작 메카닉

- 150개 Classic 레벨 + 50개 Championship 레벨 구현
- 플레이어: 점프 불가, 무한 낙하
- 벽돌 파기: 좌/우 하단만, 180프레임(~6초) 후 복구
- 가드: 구멍에서 탈출 가능, 금괴 운반/드롭, 리스폰
- 모든 금 수집 → 탈출 사다리 활성화
- 레벨 클리어 → +2000점, +1 목숨

## 코드 수정 시 주의사항

1. **IIFE 구조 유지**: 전역 변수 오염 방지
2. **프레임 기반 타이밍**: setTimeout 대신 frameCount 사용
3. **좌표 시스템**: 타일 좌표 (0~27, 0~15)와 픽셀 좌표 구분
4. **스케일 적용**: 모든 렌더링에 SCALE 또는 TILE_SIZE 사용
5. **원작 충실도**: 새 기능 추가 시 1983 메카닉 우선 고려
