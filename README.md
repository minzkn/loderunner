# Lode Runner (1983) - 웹 브라우저 재현판

1983년 Broderbund 원작 Lode Runner를 웹 브라우저에서 재현한 프로젝트입니다.
핵심 목표는 Apple II 원작 메카닉을 최대한 충실하게 유지하는 것입니다.

## 실행

`index.html`을 브라우저에서 열면 바로 실행됩니다.

```bash
# 선택: 로컬 서버 실행
python -m http.server 8080
# http://localhost:8080
```

## 현재 구현 요약

- 클래식 150레벨 + 챔피언십 50레벨 (총 200레벨)
- 고정 업데이트 30fps (`tickRate = 1000 / 30`)
- 캔버스 기준 해상도 672x384 (28x16 타일, 타일 24px)
- 반응형 스케일링 + 모바일 터치 컨트롤
- 데모 모드 (무입력 30초)
- 레벨 에디터 내장

## 조작

| 동작 | 키 |
|------|-----|
| 이동 (좌/우) | 방향키 / A,D / J,L |
| 사다리 (상/하) | 방향키 위아래 / W,S / I,K |
| 왼쪽 파기 | Z / U / F1 |
| 오른쪽 파기 | X / O / F2 |
| 시작/재개/일시정지 | ENTER |
| 일시정지 토글 | ESC |
| 레벨 재시작 (목숨 1 소모) | R |
| 사운드 토글 | M |
| 게임 모드 전환 (타이틀) | C |
| 규칙 프로파일 전환 (타이틀) | V |
| 레벨 에디터 토글 | E |
| 디버그 레벨 이동 | Shift+N / Shift+P |

에디터 모드:
- `1~8`: 타일 선택
- `P`: 플레이어 배치
- `E`: 적 배치
- `S`: 저장
- `L`: 로드
- `ESC`: 에디터 종료

## 점수/클리어

| 행동 | 점수 |
|------|------|
| 금 획득 | +250 |
| 적 처리 (구멍 복구 시) | +75 |
| 레벨 클리어 | +1500 + 목숨 1 |

레벨 클리어 조건:
1. 모든 금 수집
2. 탈출 사다리 활성화
3. 화면 상단 도달

## 핵심 메카닉

- 점프 불가
- 낙하 데미지 없음
- 플레이어 금 운반은 1개 제한
- 파기 대상은 프로파일별로 다름 (`modern`: BRICK/TRAP, strict: BRICK)
- 적은 BRICK 위를 통과 가능, 플레이어는 불가
- 함정(`T`)은 밟으면 떨어지는 False Floor

규칙 프로파일:
- `modern` (기본): 목숨 99, 자동 교착 감지, 데모/에디터 활성, 동적 탈출 사다리
- `apple2_strict`: 목숨 5, 데모/에디터/자동 교착 비활성, 레거시 입력
- `c64_strict`: 목숨 5, 레거시 입력 + `Ctrl+A` 중단

## 주요 상수 (현재 구현)

```javascript
const GOLD_PICKUP_CHANCE = 0.05;
const GOLD_DROP_CHANCE = 0.02;

const ENEMY_TRAPPED_TIME = 150;   // 30fps 기준 약 5초
const ENEMY_TRAPPED_ESCAPE = 60;  // 탈출 시도 시작
const HOLE_FILL_TIME = 210;       // 30fps 기준 약 7초

const PLAYER_SPEED = 0.14;
const ROPE_SPEED_MULTIPLIER = 1.15;
```

속도 증가:
- 클래식 모드에서 레벨 150 이후 `+15%`
- 최대 `2.0x`

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

## 참고

- 레벨 데이터는 `game.js` 내 문자열 배열로 포함됨
- `RULES_CHECKLIST.md` 에 RULES 기반 수동 회귀 체크리스트 제공
- 원작 Lode Runner 저작권은 권리자(Tozai Games 등)에 있음
