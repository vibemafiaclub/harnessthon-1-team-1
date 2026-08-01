---
name: start
description: PRD를 입력받아 단계별 sub agent를 순서대로 호출해 Penpot 디자인을 완성하는 하네스 진입점. "/start", "시작해줘", "디자인 만들어줘", "PRD 실행", "하네스 돌려줘" 등에 트리거된다.
---

# start — 하네스 진입점

> 🔒 **공용 파일입니다. 수정하려면 조장 승인이 필요합니다.**
>
> ⚠️ **이 파일은 비어 있는 뼈대입니다.** 설계 세션에서 팀이 정한 단계를
> 여기에 채워 넣으세요. 미리 채워두지 않은 이유는, 단계를 나누는 일 자체가
> 오늘의 핵심 학습이기 때문입니다.
>
> 👉 단계가 확정됐다면 손으로 쓰지 말고 **`/scaffold-harness`** 를 조장이 실행하세요.
> 아래 표·실행순서와 `.claude/agents/stage-*.md`를 한 번에 만들어줍니다.

## 입력

- `docs/PRD.md` — 만들어야 할 것의 명세
- 작업 Page 이름 — **매 실행마다 확인한다.** 없으면 묻고, 답을 받기 전엔 시작하지 않는다

## 실행 원칙

1. **각 단계는 반드시 sub agent에게 위임한다.** 오케스트레이터가 직접 저작하지 않는다.
2. 각 sub agent에게 **입력(읽을 파일)·출력(쓸 파일)·작업 Page 이름**을 명시적으로 넘긴다.
3. **의존관계가 없는 단계는 병렬로** 호출한다.
4. 중간 산출물은 전부 `docs/artifacts/`에 남긴다. 남지 않으면 다음 단계가 읽을 게 없다.
5. 한 단계가 출력 파일을 남기지 못했으면 **다음 단계로 넘어가지 않는다.** 멈추고 보고한다.

## 단계 정의

| # | 단계 | sub agent | 입력 | 출력 | 담당자 | 병렬 가능 |
|---|---|---|---|---|---|---|
| 1 | 도메인 리서치 | `stage-1-research` | `docs/PRD.md` | `01-research.md` | 미정(리허설: 최병찬) | 3, 5a와 병렬 |
| 2 | 페르소나 | `stage-2-persona` | PRD + `01` | `02-persona.md` | 미정(리허설: 최병찬) | — |
| 3 | 경쟁사 분석 | `stage-3-competitors` | `docs/PRD.md` | `03-competitors.md` + `images/` | 미정(리허설: 최병찬) | 1, 5a와 병렬 |
| 4 | 유저플로우/화면 인벤토리 | `stage-4-userflow` | PRD + `02` + `03` | `04-userflow.md` | 미정(리허설: 최병찬) | — |
| 5a | 토큰 실측 [접점 A: 읽기] | `stage-5a-tokens` | PRD + Penpot 기존 자산 Page | `05a-tokens.md` | 미정(리허설: 최병찬) | 1, 3과 병렬 |
| 5b | 컴포넌트 저작 [접점 B: 쓰기] | `stage-5b-components` | `04` + `05a` + 작업 Page 인자 | `05b-components.md` + Penpot 컴포넌트 | 미정(리허설: 최병찬) | — |
| 6 | 화면 저작 [접점 B: 쓰기] | `stage-6-author` | `04` + `05b` + `05a` + `01`·`03`(소구점) + 작업 Page 인자 | `06-author.md` + PNG + Penpot board | 미정(리허설: 최병찬) | 화면 단위 분할 가능 |
| 7 | 검증 (고정) | `stage-verify-penpot` | `04` + Penpot 작업 Page | `99-verify.md` | 고정 | — (맨 끝) |

## 실행 순서

0. **작업 Page 이름을 확보한다.** 없으면 묻고, 답 전엔 시작하지 않는다.
   런로그 `docs/artifacts/00-runlog.md`에 런 시작을 기록한다.
1. `stage-1-research` + `stage-3-competitors` + `stage-5a-tokens` **3개 병렬 호출**
   → `01`·`03`·`05a` 생성 확인
2. `stage-2-persona` 호출 (`01` GATE 판정 후) → `02` 생성 확인
3. `stage-4-userflow` 호출 (`02`·`03` GATE 판정 후) → `04` 생성 확인
4. `stage-5b-components` 호출 (`04`·`05a` GATE 판정 후, 작업 Page 인자 전달) → `05b` 생성 확인
5. `stage-6-author` 호출 (`04`·`05b` GATE 판정 후, 작업 Page 인자 전달) → `06` + PNG 생성 확인
6. `stage-verify-penpot` 호출 (작업 Page 인자 전달) → `99-verify.md`
   실패 항목 → 담당 단계 **1회만** 재실행 → 재-verify

### GATE 분기 규칙 (다음 단계 호출 전, 앞 산출물의 `## GATE` 절만 읽는다)

- `PASS` → 다음 단계 호출
- `PARTIAL` → 다음 단계를 저신뢰 모드로 진행(있는 것만 사용) + 런로그 기록.
  단 **G4·G5b의 PARTIAL은 P0 범위만으로 축소 진행**
- `FAIL` → 해당 단계 1회 재실행. 재실행 후에도 FAIL이면 그 브랜치만 멈추고 병렬 형제는 계속

### 공통 런타임 (요약 — 정본: `docs/harness-failure-modes.md`)

- 모든 단계는 런로그 append (R1) · 타임박스 초과 시 PARTIAL 저장 후 진행 (R2)
- 재시도 1회, 항목 단위 SKIPPED (R3) · 한 단계 실패가 전체를 죽이지 않는다 (R4)
- artifacts 파일 = 체크포인트. 재실행 시 PASS 산출물은 건너뛴다 (R5)
- 실행 중 질문 금지 — 보수적 기본값 + 런로그 기록 (R6) · P0 우선 (R7)

## 마지막 단계 — 검증 (고정, 삭제 금지)

모든 단계가 끝나면 **항상** `stage-verify-penpot` 을 호출한다.

- 지정 Page에 board/frame이 1개 이상 있는가
- PRD가 요구한 화면이 전부 있는가
- 각 단계 산출물이 `docs/artifacts/`에 남아 있는가

결과는 `docs/artifacts/99-verify.md`. **실패 항목이 있으면 완료를 선언하지 않는다.**
해당 단계를 다시 호출하고, 재실행 후에도 실패하면 무엇이 왜 비었는지 사용자에게 보고한다.

## 완료 조건

- Penpot 파일의 **지정된 Page**에 화면이 실제로 만들어져 있다
- 각 단계의 중간 산출물이 `docs/artifacts/`에 남아 있다
- `docs/artifacts/99-verify.md` 가 전 항목 통과다
