---
name: start
description: PRD를 입력받아 단계별 sub agent를 순서대로 호출해 Penpot 디자인을 완성하는 하네스 진입점. "/start", "시작해줘", "디자인 만들어줘", "PRD 실행", "하네스 돌려줘" 등에 트리거된다.
---

# start — 하네스 진입점

> 🔒 **공용 파일입니다. 수정하려면 조장 승인이 필요합니다.**
>
> 단계 표를 바꾸면 단계 간 계약이 바뀝니다. 각 단계 agent 파일(`.claude/agents/stage-*.md`)은
> 담당자가 자유롭게 고도화하되, **입출력 파일 경로는 이 표와 일치해야 합니다.**

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
| 1 | PRD → 화면 요구사항 | `stage-1-prd-screens` | `docs/PRD.md` | `docs/artifacts/01-screens.md` | TBD | ✅ 2·3과 |
| 2 | 기존 자산 분석·복제 | `stage-2-existing-assets` | `docs/PRD.md`, Penpot 기존 자산 Page(읽기), **작업 Page 이름** | `docs/artifacts/02-existing-assets.md` + 작업 Page의 `Ref/*` 복제본 | TBD | ✅ 1·3과 |
| 3 | 경쟁사 조사 | `stage-3-competitor-research` | `docs/PRD.md` | `docs/artifacts/03-competitors.md` | TBD | ✅ 1·2와 |
| 4 | 디자인 가이드 | `stage-4-design-guide` | `docs/PRD.md`, `01`, `02`, `03` | `docs/artifacts/04-design-guide.md` | TBD | ❌ |
| 5 | HTML 사전 작업 | `stage-5-html-prototype` | `01`, `02`, `04` | `docs/artifacts/05-html/*`, `docs/artifacts/05-html-review.md` | TBD | ❌ |
| 6 | Penpot 라이브러리 | `stage-6-penpot-library` | `02`, `04`, `05-html-review`, **작업 Page 이름** | `docs/artifacts/06-library.md` + 작업 Page의 `Lib/*` 컴포넌트 | TBD | ❌ |
| 7 | Penpot 화면 저작 | `stage-7-penpot-screens` | `01`, `04`, `05-html-review`, `06`, **작업 Page 이름** | `docs/artifacts/07-screens.md` + 작업 Page의 화면 board | TBD | ❌ |
| 8 | 업로드 한계점 분석 | `stage-8-upload-limits` | `05-html-review`, `06`, `07` | `docs/artifacts/08-limitations.md` | TBD | ❌ |
| — | **검증 (고정)** | `stage-verify-penpot` | `docs/PRD.md`, `01`, `07`, Penpot(읽기), **작업 Page 이름** | `docs/artifacts/99-verify.md` | TBD | ❌ |

**단계 간 통신은 파일로만 한다.** 대화 맥락으로 값을 넘기지 않는다.
sub agent를 호출할 때는 **읽을 파일 경로 · 쓸 파일 경로 · 작업 Page 이름**을 매번 명시해 넘긴다.

## 실행 순서

0. **작업 Page 이름을 확정한다.** 인자로 받지 못했으면 사용자에게 묻고, **답을 받기 전에는
   어떤 단계도 호출하지 않는다.** 기본값으로 첫 Page를 쓰지 않는다.
1. **`stage-1-prd-screens` · `stage-2-existing-assets` · `stage-3-competitor-research` 를 병렬 호출**
   → `01-screens.md`, `02-existing-assets.md`, `03-competitors.md` 생성 확인
   (2번만 작업 Page 이름이 필요하다. 1·3은 문서만 읽는다.)
2. `stage-4-design-guide` 호출 → `04-design-guide.md` 생성 확인
   - 04에 **TOKENS JS 상수 코드블록**이 들어 있는지 확인한다. 없으면 재실행한다. 6·7단계가 못 쓴다.
3. `stage-5-html-prototype` 호출 → `05-html-review.md` 생성 확인
   - review의 "가이드 수정 제안"·"화면 누락 제안"을 읽고 **반영할지 판단한다.**
     반영하기로 했으면 `stage-4` 또는 `stage-1` 을 **해당 제안만 넘겨 재실행**한 뒤 5단계로 돌아온다.
4. `stage-6-penpot-library` 호출 (작업 Page 이름 필수) → `06-library.md` 생성 확인
   - 컴포넌트 **compId 표가 비어 있으면 다음으로 넘어가지 않는다.** 7단계가 인스턴스를 못 만든다.
5. `stage-7-penpot-screens` 호출 (작업 Page 이름 필수) → `07-screens.md` 생성 확인
6. `stage-8-upload-limits` 호출 → `08-limitations.md` 생성 확인
7. `stage-verify-penpot` 호출 (작업 Page 이름 필수) → 아래 검증 절차로

각 단계가 **출력 파일을 남기지 못했으면 다음 단계로 넘어가지 않는다.** 멈추고 보고한다.

## 마지막 단계 — 검증 (고정, 삭제 금지)

모든 단계가 끝나면 **항상** `stage-verify-penpot` 을 호출한다.

- 지정 Page에 board/frame이 1개 이상 있는가
- PRD가 요구한 화면이 전부 있는가
- 각 단계 산출물이 `docs/artifacts/`에 남아 있는가

결과는 `docs/artifacts/99-verify.md`. **실패 항목이 있으면 완료를 선언하지 않는다.**

| FAIL 항목 | 되돌릴 단계 |
|---|---|
| V1 board 없음 / V4 빈 화면 / V5 인스턴스 미사용 / V6 네이밍 | `stage-7-penpot-screens` |
| V2·V3 화면 누락 — 01에는 있는데 안 만들어짐 | `stage-7-penpot-screens` |
| V2·V3 화면 누락 — 01에 아예 없음 | `stage-1-prd-screens` → 이후 4~7 재실행 |
| V5 컴포넌트 자체가 없음 | `stage-6-penpot-library` → `stage-7` |
| V8 산출물 파일 없음 | 그 파일의 담당 단계 |
| V7 침범(기존 자산 Page 수정 / 공용 Page 저작) | **재실행하지 말고 즉시 사용자에게 보고한다** |

되돌린 단계를 재실행한 뒤 **검증을 다시 돌린다.** 재실행 후에도 실패하면
무엇이 왜 비었는지 사용자에게 보고한다. **자동 재실행은 같은 단계에 대해 1회까지만** 한다.

## 완료 조건

- Penpot 파일의 **지정된 Page**에 화면이 실제로 만들어져 있다
- 각 단계의 중간 산출물이 `docs/artifacts/`에 남아 있다
- `docs/artifacts/99-verify.md` 가 전 항목 통과다
