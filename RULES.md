# Lode Runner (Apple II, 1983) 재구현 명세

이 문서는 이 저장소의 **단일 기준 명세**다. 목표는 Apple II 1983판 Lode Runner를 우선 재현하는 것이다.

## 1. 기준과 우선순위

- 1순위: Apple II 1983 체감/규칙/조작 재현
- 2순위: 현재 코드(`game.js`)와의 일치
- 3순위: modern 편의 기능(데모, 에디터, 자동 교착 탐지)

검증 등급:
- `A`: `game.js` 코드로 직접 확인
- `B`: Apple II/C64 매뉴얼·문서 텍스트 확인
- `C`: 관찰/추정(추가 실측 필요)

기준 시점: 2026-02-15

## 2. 월드/엔진 고정값 (`A`)

```javascript
BASE_TILE_SIZE = 24
LEVEL_WIDTH = 28
LEVEL_HEIGHT = 16
BASE_WIDTH = 672
BASE_HEIGHT = 384
tickRate = 1000 / 30
```

- 좌표계: 타일 좌표(정수) + 엔티티 좌표(부동소수)
- 맵 바깥 조회: `SOLID`로 간주
- 레벨 모드: `classic`(150), `championship`(50)

## 3. 상태 머신 (`A`)

| 상태 | 값 | 설명 |
|---|---:|---|
| `TITLE` | 0 | 타이틀 |
| `PLAYING` | 1 | 플레이 진행 |
| `PAUSED` | 2 | 일시정지 |
| `DYING` | 3 | 사망 연출/지연 |
| `LEVEL_COMPLETE` | 4 | 레벨 완료 지연 |
| `GAME_OVER` | 5 | 게임 오버 |
| `STUCK` | 6 | 교착 감지 상태(Modern 전용) |
| `EDITOR` | 7 | 레벨 에디터(Modern 전용) |

Apple II 타깃에서는 `STUCK`, `EDITOR`를 비활성으로 본다.

## 4. 타일/문자 포맷 (`A`)

| 값 | 타입 | 문자 | 의미 |
|---:|---|---|---|
| 0 | `EMPTY` | `.` | 빈 공간 |
| 1 | `BRICK` | `#` | 파기 가능 벽돌 |
| 2 | `SOLID` | `@` | 파기 불가 고체 |
| 3 | `LADDER` | `H` | 일반 사다리 |
| 4 | `BAR` | `-` | 바(매달림) |
| 5 | `TRAP` | `T` | false floor |
| 6 | `GOLD` | `$` | 금 |
| 7 | `ESCAPE_LADDER` | `E` | 탈출 사다리 |

엔티티 문자:
- `P`: 플레이어 시작 위치
- `E`: 적 시작 위치
- `e`: 숨김 탈출 사다리 seed(엄격 프로파일용)

## 5. Apple II 목표 프로파일 (`A`,`B`)

기본 목표 프로파일: `apple2_strict`

현재 런타임은 프로파일을 `apple2_strict`로 고정한다(타이틀에서 순환 비활성).

```javascript
startLives = 5
allowTrapDig = false
enableDemo = false
enableEditor = false
enableAutoStuck = false
dynamicEscapeLadder = false
classicSpeedLoop = false
inputPreset = 'legacy'
```

의미:
- 목숨 5 시작
- `TRAP` 파기 금지 (`BRICK`만 파기)
- 데모/에디터/자동 교착 탐지 비활성
- 탈출 사다리는 정적 seed(`e`) 공개 우선
- 입력은 `IJKL + UO` 중심

## 6. 그래픽 명세 (`A`,`B`)

### 6.1 팔레트

```javascript
BLACK: '#000000'
WHITE: '#FFFFFF'
RED: '#FF0000'
GREEN: '#00FF00'
ORANGE: '#FF8800'
PURPLE: '#AA00FF'
CYAN: '#00AAAA'
YELLOW: '#FFFF00'
BROWN: '#8B4513'
```

추가 셰이딩 색(`LIGHT_*`, `GRAY`, `BLUE`)은 렌더 가독성용 보강값으로 허용한다.

### 6.2 타일 렌더링 규칙

- `BRICK`: 적갈색 벽돌 패턴
- `SOLID`: 파란 계열 고체 블록
- `LADDER`: 노랑/주황 사다리
- `ESCAPE_LADDER`: 활성 시 점멸 강조
- `BAR`: 회색 수평 바
- `TRAP`: 일반 바닥처럼 보이는 함정 시각
- `GOLD`: 노랑 중심, 하이라이트 포함

### 6.3 엔티티 렌더링 규칙

- 플레이어: 빨강 계열, 이동 프레임 애니메이션
- 적: 빨강/갈색 계열, 갇힘 상태 별도 연출
- 금 보유 상태: 플레이어/적 모두 작은 점멸 인디케이터 허용

## 7. 사운드 명세 (`A`,`B`,`C`)

현재 구현은 WebAudio 기반 다중 톤/노이즈 효과다. Apple II 하드웨어와 파형이 완전히 동일하지는 않지만, 이벤트 매핑은 아래를 기준으로 고정한다.

| 이벤트 | 트리거 |
|---|---|
| `start` | 게임 시작 |
| `gold` | 금 획득 |
| `dig` | 파기 성공 |
| `fill` | 구멍 복구 |
| `trap` | 적이 구멍에 갇힘 |
| `respawn` | 적 리스폰 |
| `fall` | 봉에서 낙하 시작 |
| `step` | 지상 수평 이동 주기 |
| `climb` | 사다리 이동 주기 |
| `bar` | 바 이동 주기 |
| `die` | 플레이어 사망 |
| `escape` | 탈출 사다리 활성 |
| `levelup` | 레벨 클리어 |
| `pause`/`resume` | 일시정지/재개 |
| `gameover` | 게임 오버 |

엄격 재현 목표:
- 음색 단순화(펄스/노이즈 중심)
- 동시발음 최소화
- 이벤트 우선순위 정리(이동음 과다 중첩 방지)

## 8. 플레이어 규칙 (`A`,`B`)

- 점프 없음
- 낙하 데미지 없음
- 기본 속도: `PLAYER_SPEED = 0.14`
- 바 이동 속도: `PLAYER_SPEED * ROPE_SPEED_MULTIPLIER(1.15)`
- `DOWN`으로 바에서 자발적 하강 가능
- 바 자동 잡기 허용(수평 접근/낙하 중)

False Floor(`TRAP`):
- 플레이어는 1프레임 지연 후 관통
- 적은 즉시 관통

## 9. 파기(dig) 규칙 (`A`,`B`)

- 입력: `DIGL`, `DIGR`
- 목표: 플레이어 좌/우 대각선 아래 1칸
- 파기 가능: `BRICK` (`apple2_strict`), `TRAP`는 Modern에서만 허용
- 파기 불가 조건: 목표 타일 위가 `LADDER`/`ESCAPE_LADDER`/`BAR`이거나, 플레이어 옆칸이 고체(`BRICK`/`SOLID`)인 경우

타이밍 상수:

```javascript
HOLE_FILL_TIME = 210
ENEMY_TRAPPED_TIME = 150
ENEMY_TRAPPED_ESCAPE = 60
```

## 10. 구멍/갇힘/복구 (`A`,`B`)

- 구멍 만료 시 `BRICK`으로 복구
- 복구 시 적이 구멍 좌표에 있으면 적 사망(+75) 후 리스폰
- 복구 시 플레이어가 구멍 좌표에 있으면 즉사
- 플레이어는 구멍에서 기본적으로 못 올라옴
- 적은 일정 시간 후 탈출 시도

## 11. 금/탈출 (`A`,`B`)

플레이어:
- 금 획득 점수: `+250`
- 보유량 최대 1 (`player.goldCount`)
- 사망 시 보유 금 드롭 시도

적:
- 보유량 최대 1 (`hasGold`)
- 획득 확률: `GOLD_PICKUP_CHANCE = 0.05`
- 드롭 확률: `GOLD_DROP_CHANCE = 0.02`
- 획득 쿨다운: 30프레임

탈출:
- 모든 금 수집 시 탈출 사다리 활성
- 활성 후 상단 도달 시 레벨 완료

## 12. 탈출 사다리 생성 규칙 (`A`,`B`,`C`)

Apple II 목표:
- 레벨 내 숨김 seed(`e`)를 `ESCAPE_LADDER`로 공개

대체(백업) 규칙:
- seed가 없을 때만 동적 생성(사다리 열 우선, 없으면 바 열)

## 13. 적 AI 규칙 (`A`,`C`)

- 기본 추적: 수평 우선 + 사다리 수직 추적
- 바/사다리/낙하 상태별 이동 제약 공유
- 구멍 갇힘, 탈출, 리스폰 동작 보장
- 금 보유 시 드롭/탈취 규칙 유지

참고:
- 원작 내부 우선순위의 프레임 단위 완전 일치는 추가 실측 필요

## 14. 충돌/사망 (`A`,`B`)

- 플레이어-적 근접 중첩 시 사망
- 단, 갇힌 적과의 접촉은 예외
- 플레이어가 적 머리 위(발판 상황)인 경우 생존
- 적이 금 보유 중 접촉 시, 플레이어가 금 탈취(+250)하고 생존

## 15. 점수/진행 (`A`,`B`)

| 항목 | 점수 |
|---|---:|
| 금 획득 | 250 |
| 적 포획/구멍 사망 | 75 |
| 레벨 클리어 | 1500 |

진행:
- 시작 목숨: 5 (`apple2_strict`)
- 레벨 클리어 시 목숨 +1

## 16. 조작 명세 (`A`,`B`)

### 16.1 Apple II 우선 조작

- 이동: `I`(위), `J`(좌), `K`(아래), `L`(우)
- 파기: `U`(좌), `O`(우)
- 시작/재개: `ENTER`
- 일시정지: `ESC`
- 재시작(목숨 소모): `R`

### 16.2 보조 조작 (호환)

- 파기 대체: `F1`/`F2`
- 파기 보조: `Z`/`X` (웹 접근성 보조 입력)
- 프로파일 순환(타이틀): `V`
- 모드 전환(타이틀): `C`
- 사운드 토글: `M`

`V`는 현재 빌드에서 프로파일 고정으로 비활성이다.
Modern 전용 기능(`E`, `Shift+N/P`, 자동 STUCK 등)은 Apple II 엄격 재현 범위에서 제외한다.

## 17. 재구현 시 필수 유지사항

1. IIFE 구조 유지
2. 프레임 기반 타이밍 유지(30fps tick)
3. 타일 좌표/픽셀 좌표 분리
4. 반응형 스케일 유지
5. Apple II 규칙 우선

## 18. 원작 대비 남은 갭 (`C`)

- Apple II 실제 음색/채널 동작 1:1 재현 미완료
- 적 AI 내부 우선순위 완전 일치 검증 미완료
- 일부 레벨의 숨김 탈출 사다리 원본 데이터 검증 필요

## 19. 출처

- Lemon64 manual text: https://www.lemon64.com/doc/lode-runner/355
- Apple2Games wiki: https://apple2games.com/wiki/Lode_Runner
- GameFAQs Apple II FAQ: https://gamefaqs.gamespot.com/appleii/577310-lode-runner/faqs/7328
- C64-Wiki: https://www.c64-wiki.com/wiki/Lode_Runner
