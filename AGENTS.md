# Lode Runner (1983) - Claude Code 가이드

1983년 Broderbund Lode Runner의 웹 브라우저 재현판입니다.
원본 Apple II 버전을 가장 충실하게 재현하는 것을 최우선 목표로 합니다.

## 실행 방법

`index.html`을 웹 브라우저에서 열기

## 프로젝트 구조

```
loderunner/
├── index.html    # 게임 컨테이너
├── style.css    # 반응형 스타일
├── game.js      # 게임 로직 - 단일 IIFE
├── RULES.md     # 게임 규칙 및 메카닉 참고문서
└── .claude/     # Claude 설정
```

## 핵심 상수

```javascript
const BASE_TILE_SIZE = 24;
const LEVEL_WIDTH = 28;
const LEVEL_HEIGHT = 16;
const BASE_WIDTH = BASE_TILE_SIZE * LEVEL_WIDTH;   // 672
const BASE_HEIGHT = BASE_TILE_SIZE * LEVEL_HEIGHT; // 384
```

## Apple II 원본 색상 팔레트

```javascript
const COLORS = {
    BLACK: '#000000',      // 배경
    WHITE: '#FFFFFF',      // 플레이어
    RED: '#FF0000',       // 적, 함정
    GREEN: '#00FF00',     // 금, 사다리
    ORANGE: '#FF8800',    // 벽돌
    PURPLE: '#AA00FF',    // 솔리드 블록
    CYAN: '#00AAAA',      // 바 (rope)
    YELLOW: '#FFFF00',    // 금 상자
    BROWN: '#8B4513'      // 추가 색상
};
```

## 게임 상태 (STATE)

| 상태 | 값 | 설명 |
|------|-----|------|
| TITLE | 0 | 타이틀 화면 |
| PLAYING | 1 | 게임 진행 중 |
| PAUSED | 2 | 일시정지 |
| DYING | 3 | 사망 애니메이션 |
| LEVEL_COMPLETE | 4 | 레벨 클리어 |
| GAME_OVER | 5 | 게임 오버 |
| STUCK | 6 | 스턱 (클리어 불가) |
| EDITOR | 7 | 레벨 에디터 |

## 타일 타입 (TILE)

| 값 | 타입 | 문자 | 설명 |
|----|------|------|------|
| 0 | EMPTY | `.` | 빈 공간 |
| 1 | BRICK | `#` | 벽돌 (파괴 가능) |
| 2 | SOLID | `@` | 단단한 블록 (파기 불가) |
| 3 | LADDER | `H` | 사다리 |
| 4 | BAR | `-` | 바 (매달리기) |
| 5 | TRAP | `T` | 함정 (False Floor - 밟으면 추락) |
| 6 | GOLD | `$` | 금괴 |
| 7 | ESCAPE_LADDER | `E` | 탈출 사다리 |

## 조작법

| 동작 | 키 |
|------|-----|
| 이동 (좌/우) | 방향키 / A,D / J,L |
| 사다리 (상/하) | 방향키 위아래 / W,S / I,K |
| 왼쪽 파기 | Z / F1 / U |
| 오른쪽 파기 | X / F2 / O |
| 시작/재개 | ENTER |
| 일시정지 | ESC |
| 레벨 재시작 | R |
| 레벨 에디터 | E (타이틀에서) |
| 음소거 | M |

## 레벨 포맷

```
. = empty    # = brick    @ = solid    H = ladder
- = bar      $ = gold     T = trap     P = player
E = enemy
```

## 1983 원본 메카닉 (핵심 구현 목표)

### 플레이어
- **점프 불가** - 가장 중요한 규칙
- **무한 낙하** - 어떤 높이에서도 데미지 없음
- **금 운반** - 한 번에 1개만 들 수 있음 (원본 규칙)
- **파기** - 좌/우 대각선 하단만, 600프레임(60fps) 후 복구

### 적 (가드)
- 사다리/로프 사용 가능
- **BRICK 위 이동 가능** (플레이어는 파기해야 함)
- 금 1개 소지 가능
- 구멍에서 탈출 가능 (60프레임부터 탈출 시도)
- 金 줍기/드롭 (5% 확률로 줍, 2% 확률로 드롭)
- 리스폰 (상단에서 무작위 위치)
- 플레이어 추적 AI (비최적 경로 찾기)

### 레벨 클리어
- 모든 금 수집 → 탈출 사다리 활성화
- 탈출 사다리로 화면 상단 도달 → 레벨 클리어
- 레벨 클리어 시 +1500점, +1 목숨

## 점수 시스템

| 행동 | 점수 |
|------|------|
| 금 줍기 | 250 |
| 적 구멍에 가두기 (압사) | 75 |
| 레벨 클리어 | 1500 |

## 타이밍 상수 (30fps 기준 - 현재 구현)

```javascript
const HOLE_FILL_TIME = 210;          // 구멍 복구 (~7초) - 원본 Apple II 기준
const ENEMY_TRAPPED_TIME = 150;      // 적 갇힘 (~5초)
const ENEMY_TRAPPED_ESCAPE = 60;     // 적 탈출 시도 (~2초, 남은 프레임)
const GOLD_PICKUP_CHANCE = 0.05;     // 5%
const GOLD_DROP_CHANCE = 0.02;       // 2%
const PLAYER_SPEED = 0.14;           // 플레이어 이동 속도
const ROPE_SPEED_MULTIPLIER = 1.15;  // 로프 이동 15% 빠름 (스피드런 테크닉)
```

### 60fps 기준 (원본)
- HOLE_FILL_TIME: 420 (~7초)
- ENEMY_TRAPPED_TIME: 300 (~5초)
- ENEMY_TRAPPED_ESCAPE: 60 (탈출 시도 시작)

## 속도 증가

- 150레벨 이후 15% 속도 증가
- 최대 2.0배 (200%)

## 주요 메서드 목록

### 초기화
- `constructor()` - 게임 초기화
- `setupResponsiveCanvas()` - 반응형 캔버스 설정
- `setupInput()` - 입력 시스템 설정
- `setupToggle()` - 토글 버튼 설정
- `setupTouchControls()` - 터치 컨트롤 설정

### 레벨 관리
- `loadLevel(levelNum)` - 레벨 로드
- `startGame()` - 게임 시작
- `loadEditorLevel()` - 에디터 레벨 로드

### 게임 루프
- `gameLoop()` - 메인 게임 루프
- `update()` - 게임 상태 업데이트
- `render()` - 렌더링

### 플레이어
- `updatePlayer()` - 플레이어 업데이트
- `collectGold(x, y)` - 금 줍기 (1개만 들 수 있도록 수정)
- `dig(digX, digY)` - 벽돌 파기
- `playerDie()` - 플레이어 사망
- `updateDigEffects()` - 파기 효과 업데이트

### 적 AI
- `updateEnemies()` - 적 업데이트
- `respawnEnemy(e)` - 적 리스폰
- `checkCollisions()` - 충돌 검사

### 구멍 메카닉
- `updateHoles()` - 구멍 상태 업데이트

### 스턱 감지
- `checkStuck()` - 스턱 상태 검사
- `findReachablePositions(startX, startY)` - 도달 가능한 위치 찾기
- `triggerStuck(reason)` - 스턱 트리거

### 데모 모드
- `startDemo()` - 데모 모드 시작
- `updateDemo()` - 데모 모드 업데이트
- `updateDemoKeyDisplay()` - 데모 키 표시 업데이트

### UI
- `updateUI()` - UI 업데이트
- `updateTitleMessage()` - 타이틀 메시지 업데이트

### 렌더링
- `renderTitleScreen(ctx)` - 타이틀 화면 렌더링
- `drawPlayer(p)` - 플레이어 렌더링 (스틱 피겨)
- `drawEnemy(e)` - 적 렌더링
- `drawTile(x, y, tile)` - 타일 렌더링

## 코드 수정 시 주의사항

1. **IIFE 구조 유지**: 전역 변수 오염 방지, 모든 코드를 IIFE로 감싸야 함
2. **프레임 기반 타이밍**: requestAnimationFrame 사용, frameCount로 프레임 관리
3. **좌표 시스템**: 타일 좌표 (0~27, 0~15)와 픽셀 좌표 구분
4. **스케일 적용**: 모든 렌더링에 SCALE 또는 TILE_SIZE 사용
5. **원작 충실도**: 새 기능 추가 시 1983 메카닉 우선 고려

## 수정 필요 사항 (완료)

1. **구멍 복구 시간**: 210 프레임 (30fps) - ✅ 완료
2. **플레이어 금 운반**: 1개만 가능 - ✅ 완료
3. **색상 팔레트**: Apple II 원본 적용 - ✅ 완료
4. **적 BRICK 위 이동**: 가능 - ✅ 완료
5. **금 줍기/드롭 확률**: 5%/2% - ✅ 완료

## 레벨 데이터

- 150개 클래식 레벨 + 50개 챔피언십 레벨 (총 200레벨)
- 레벨 데이터는 game.js에 문자열 배열로 저장
- 레벨 포맷 문자 사용: `. # @ H - $ T P E`

## 웹 구현 고려사항

### 반응형 캔버스
- Canvas 2D API 사용
- 28×16 타일 × 24px = 672×384 (베이스 해상도)
- window 크기에 맞게 스케일링
- 가로/세로 비율 4:3 유지

### 터치 컨트롤
- 화면 좌/우측 터치 영역으로 이동
- 별도 파기 버튼 배치
- 모바일에서도 원활한 조작

### 접근성
- 키보드만으로 완전 플레이 가능
- ESC로 일시정지

### 성능
- 60fps 권장 (프레임 기반 타이밍)
- requestAnimationFrame 사용

## 적 AI 구현 가이드

### 2단계 우선순위 시스템

```javascript
// 1단계: 같은 층에서 플레이어로 바로 이동
if (enemy.y === player.y && hasClearPath(enemy, player)) {
    moveTowardPlayer();
    return;
}

// 2단계: 플레이어 층에 접근
// 하 → 상 → 좌 → 우 순서로 평가
let bestDirection = evaluateDirections();
move(bestDirection);
```

### 고려사항
- 플레이어가 파낸 구멍은 무시하고 들어감
- 네이티브 함정(TRAP)은 장애물로 취급
- 사다리에서 경로 재계산 시 무한 루프 주의
- **BRICK 타일은 적만 통과 가능** (`isSolid`는 SOLID만, `isSolidForPlayer`는 BRICK+SOLID)
- 플레이어는 BRICK을 통과할 수 없으며 파기해야 함
