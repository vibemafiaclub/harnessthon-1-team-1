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
- **작업 Page 이름** — 매 실행마다 확인한다. 없으면 묻고, 답을 받기 전엔 시작하지 않는다
- (선택) **제출 Page 이름** — 없으면 최종제출로 옮기지 않는다. 임의로 정하지 않는다
- (선택) `run-id` — 없으면 `run-01`, 재실행이면 번호를 올린다

## 참조 문서 (단일 원본 — 각 단계가 읽는다)

| 파일 | 무엇 | 누가 읽나 |
|---|---|---|
| `docs/design-copy-rules.md` | 카피·표기 규칙 (이모지·`—`·자리표시자 방어) | 4·5·7·8 |
| `docs/penpot-troubleshooting.md` | Penpot이 안 될 때 진단·폴백·버리는 순서 | 6·7·9·10 |

## 실행 원칙

1. **각 단계는 반드시 sub agent에게 위임한다.** 오케스트레이터가 직접 저작하지 않는다.
2. 각 sub agent에게 **입력(읽을 파일)·출력(쓸 파일)·작업 Page 이름·run-id**를 명시적으로 넘긴다.
3. **의존관계가 없는 단계는 병렬로** 호출한다. 단, 아래 셋 중 하나라도 해당하면 **순차**다.
   - 한 단계의 출력이 다른 단계의 입력이다
   - 같은 파일 또는 같은 Penpot Page·컴포넌트를 **쓴다**
   - 한 단계가 공통 기준(토큰·화면 목록)을 바꾼다
4. 중간 산출물은 전부 `docs/artifacts/`에 남긴다. 남지 않으면 다음 단계가 읽을 게 없다.
5. 한 단계가 출력 파일을 남기지 못했으면 **다음 단계로 넘어가지 않는다.** 멈추고 보고한다.
6. **"완료했습니다"는 완료가 아니다.** 산출물 파일이 있고, Penpot에 결과가 있고, 그것을
   눈으로(PNG) 확인했을 때만 완료다.
7. **비평(8) 중에는 저작(7·9)을 돌리지 않는다.** 평가 대상 버전이 바뀌면 Finding과 증거가 어긋난다.

## 단계 정의

| # | 단계 | sub agent | 입력 | 출력 | 담당자 | 병렬 가능 |
|---|---|---|---|---|---|---|
| 1 | PRD → 화면 요구사항 | `stage-1-prd-screens` | `docs/PRD.md` | `docs/artifacts/01-screens.md` | TBD | ✅ 2·3과 |
| 2 | 기존 자산 분석·복제 | `stage-2-existing-assets` | `docs/PRD.md`, Penpot 기존 자산 Page(읽기), **작업 Page 이름** | `docs/artifacts/02-existing-assets.md` + 작업 Page의 `Ref/*` 복제본 | TBD | ✅ 1·3과 |
| 3 | 경쟁사 조사 | `stage-3-competitor-research` | `docs/PRD.md` | `docs/artifacts/03-competitors.md` | TBD | ✅ 1·2와 |
| 4 | 디자인 가이드 | `stage-4-design-guide` | `docs/PRD.md`, `01`, `02`, `03`, `design-copy-rules` | `docs/artifacts/04-design-guide.md` | TBD | ❌ |
| 5 | HTML 사전 작업 | `stage-5-html-prototype` | **`04` → `PRD`+`01` → `02` → `design-copy-rules` 순** | `docs/artifacts/05-html/*`, `docs/artifacts/05-html-review.md` | TBD | ❌ |
| 6 | Penpot 라이브러리 | `stage-6-penpot-library` | `02`, `04`, `05-html-review`, `penpot-troubleshooting`, **작업 Page 이름** | `docs/artifacts/06-library.md` + 작업 Page의 `Lib/*` 컴포넌트 | TBD | ❌ |
| 7 | Penpot 화면 저작 | `stage-7-penpot-screens` | `01`, `04`, `05-html-review`, `06`, 두 참조 문서, **작업 Page 이름** | `docs/artifacts/07-screens.md` + 작업 Page의 화면 board | TBD | ❌ |
| 8 | **독립 비평** | `stage-8-design-critique` | `PRD`, `01`, `02`, `04`, `design-copy-rules`, Penpot(읽기), **작업 Page 이름** | `docs/artifacts/08-findings.md` | TBD | ❌ |
| 9 | **수정 · 제출 준비** | `stage-9-repair-release` | `08`, `04`, `06`, 두 참조 문서, **작업 Page 이름** | `docs/artifacts/09-release.md` | TBD | ❌ |
| 10 | 업로드 한계점 분석 | `stage-10-upload-limits` | `05-html-review`, `06`, `07`, `09`, `penpot-troubleshooting` | `docs/artifacts/10-limitations.md` | TBD | ❌ |
| — | **검증 (고정)** | `stage-verify-penpot` | `PRD`, `01`, `08`, `09`, Penpot(읽기), **작업·제출 Page 이름** | `docs/artifacts/99-verify.md` | TBD | ❌ |

> **8단계는 7단계의 자기 보고(`07-screens.md`)를 읽지 않는다.** 제작과 비평을 분리하는 것이
> 이 하네스의 핵심이다. 만든 사람이 스스로 채점하면 같은 실수가 다음 실행에서도 반복된다.

**단계 간 통신은 파일로만 한다.** 대화 맥락으로 값을 넘기지 않는다.
sub agent를 호출할 때는 **읽을 파일 경로 · 쓸 파일 경로 · 작업 Page 이름**을 매번 명시해 넘긴다.

## 실행 순서

0. **preflight.** 여기서 막히면 뒤의 모든 작업이 무의미하다.
   - **작업 Page 이름을 확정한다.** 인자로 받지 못했으면 사용자에게 묻고, **답을 받기 전에는
     어떤 단계도 호출하지 않는다.** 기본값으로 첫 Page를 쓰지 않는다.
   - `docs/PRD.md` 가 있고 비어 있지 않은지 확인한다.
   - `run-id` 를 정한다. **이전 실행 결과가 `docs/artifacts/` 에 남아 있으면
     `docs/artifacts/runs/{이전 run-id}/` 로 옮겨 보존한다.** 덮어쓰면 무엇이 개선됐는지 비교할 수 없다.
   - Penpot MCP가 살아 있는지 확인한다. 실제 저작은 6단계 preflight(사각형 1개)가 맡는다.
1. **`stage-1-prd-screens` · `stage-2-existing-assets` · `stage-3-competitor-research` 를 병렬 호출**
   → `01-screens.md`, `02-existing-assets.md`, `03-competitors.md` 생성 확인
   (2번만 작업 Page 이름이 필요하다. 1·3은 문서만 읽는다.)
2. `stage-4-design-guide` 호출 → `04-design-guide.md` 생성 확인
   - 04에 **TOKENS JS 상수 코드블록**이 들어 있는지 확인한다. 없으면 재실행한다. 6·7단계가 못 쓴다.
3. `stage-5-html-prototype` 호출 → `05-html-review.md` 생성 확인
   - 작업 지시는 **① 디자인 가이드(04) 확인 → ② 작업 지시서(PRD·01) 참고 → ③ HTML 제작** 순이다.
     이 순서를 agent에게 명시해 넘긴다. 가이드 없이 만든 HTML은 기준안이 아니다.
   - review의 "가이드 수정 제안"·"화면 누락 제안"을 읽고 **반영할지 판단한다.**
     반영하기로 했으면 `stage-4` 또는 `stage-1` 을 **해당 제안만 넘겨 재실행**한 뒤 5단계로 돌아온다.
4. `stage-6-penpot-library` 호출 (작업 Page 이름 필수) → `06-library.md` 생성 확인
   - 컴포넌트 **compId 표가 비어 있으면 다음으로 넘어가지 않는다.** 7단계가 인스턴스를 못 만든다.
5. `stage-7-penpot-screens` 호출 (작업 Page 이름 필수) → `07-screens.md` 생성 확인
6. **개선 루프를 돈다** (아래 "개선 루프" 절) — `stage-8-design-critique` → `stage-9-repair-release` → 재비평
7. `stage-10-upload-limits` 호출 → `10-limitations.md` 생성 확인
8. `stage-verify-penpot` 호출 (작업 Page 이름 필수, 제출 Page 이름은 있으면) → 아래 검증 절차로

각 단계가 **출력 파일을 남기지 못했으면 다음 단계로 넘어가지 않는다.** 멈추고 보고한다.

## 개선 루프 (6번 상세)

> 결과를 손으로 고치는 것이 아니라, **문제를 만든 단계를 고치는 것**이 이 하네스의 목적이다.
> 화면만 고치면 다음 실행에서 같은 문제가 다시 나온다.

```
stage-8-design-critique   → 08-findings.md (Finding + 원인 클러스터)
        ↓  BLOCKER 또는 MAJOR 있음?
stage-9-repair-release    → 09-release.md  (클러스터 단위 수정 + 지침 수정 제안)
        ↓
stage-8-design-critique   → 재평가 (resolved / still_failing / regression 판정)
        ↓  BLOCKER 0 이면 빠져나온다
```

**루프 규칙**

1. **원인 클러스터 단위로 고친다.** Finding 4개가 같은 원인이면 한 번 고치고 4개를 함께 재평가한다.
2. **가장 상위 원인부터.** `owner_stage` 가 앞쪽 단계인 클러스터를 먼저 처리한다.
   아래에서 고치면 위쪽 원인이 그 문제를 다시 만들어낸다.
3. **상위 단계 지침이 바뀌면 그 아래는 전부 낡은 결과다.**

   | 바꾼 단계 | 다시 실행할 것 |
   |---|---|
   | `stage-1` 화면 목록 | 4 → 5 → 7 → 8 |
   | `stage-4` 토큰·가이드 | 5 → 6 → 7 → 8 |
   | `stage-6` 라이브러리 | 7 → 8 |
   | `stage-7` 저작 | 8 |
   | `stage-8` 비평 기준 | 8만 (디자인은 안 고쳐도 된다) |

   **매번 전부 다시 돌리지 않는다.** 영향받는 단계와 그 아래만 돌린다.
4. **명령이 끝난 것은 해결이 아니다.** Finding의 `acceptance` 를 **새 결과에서 확인**해야 `resolved` 다.
   확인 못 한 것은 `needs_recheck` 로 두고 8단계가 다시 판정한다.
5. **최대 3회.** 같은 Finding이 2회 연속 `still_failing` 이면 루프를 멈추고 **사용자에게 보고한다.**
   무한히 돌리지 않는다.
6. **회귀를 본다.** 해결된 Finding만 보지 말고, 이번 수정으로 **새로 생긴 문제(`regression`)** 를
   8단계가 찾게 한다. 새 토큰이 대비를 깨는 식의 일이 실제로 일어난다.
7. 루프를 빠져나온 뒤 **9단계의 "지침 수정 제안"을 사용자에게 보여준다.**
   agent 파일 수정은 **담당자가** 한다. 오케스트레이터가 자동으로 고치지 않는다.

## 제출 (사람 승인 게이트)

> 🔴 **채점 대상은 제출 Page뿐입니다.** 개인 Page에만 있으면 결과가 없는 것과 같습니다.
> 그러나 **에이전트가 임의로 옮기면 남의 결과를 덮습니다.**

넷 다 충족될 때만 `stage-9-repair-release` 에 이동을 지시한다.

1. `08-findings.md` 의 BLOCKER가 **0건**
2. 모든 화면 PNG를 확인했고 잘림·겹침이 없다
3. **사람이 옮기라고 명시적으로 지시했다**
4. **제출 Page 이름을 인자로 받았다**

하나라도 빠지면 옮기지 않고 **무엇이 막혔는지 사용자에게 보고한다.** 조건을 스스로 만들어내지 않는다.
`중간공유`·`최종제출` 은 **옮겨 담는 곳**이다. 거기서 처음부터 저작하지 않는다.

## 마지막 단계 — 검증 (고정, 삭제 금지)

모든 단계가 끝나면 **항상** `stage-verify-penpot` 을 호출한다.

- 지정 Page에 board/frame이 1개 이상 있는가
- PRD가 요구한 화면이 전부 있는가
- 각 단계 산출물이 `docs/artifacts/`에 남아 있는가
- **BLOCKER Finding이 0건인가**
- **화면 텍스트에 이모지·`—`·자리표시자가 0인가** (세어서 판정)
- **제출 Page 인자를 받았다면, 결과가 그 Page에 있는가**

결과는 `docs/artifacts/99-verify.md`. **실패 항목이 있으면 완료를 선언하지 않는다.**

| FAIL 항목 | 되돌릴 단계 |
|---|---|
| V1 board 없음 / V4 빈 화면 / V5 인스턴스 미사용 / V6 네이밍 | `stage-7-penpot-screens` |
| V2·V3 화면 누락 — 01에는 있는데 안 만들어짐 | `stage-7-penpot-screens` |
| V2·V3 화면 누락 — 01에 아예 없음 | `stage-1-prd-screens` → 이후 4~7 재실행 |
| V5 컴포넌트 자체가 없음 | `stage-6-penpot-library` → `stage-7` |
| V8 산출물 파일 없음 | 그 파일의 담당 단계 |
| V9 BLOCKER 잔존 / V11 제출 Page 비어 있음 | `stage-9-repair-release` |
| V10 AI티 (이모지·`—`·자리표시자) | `stage-7-penpot-screens` (문안 자체가 문제면 `stage-5`) |
| V7 침범(기존 자산 Page 수정 / 공용 Page 저작) | **재실행하지 말고 즉시 사용자에게 보고한다** |

되돌린 단계를 재실행한 뒤 **검증을 다시 돌린다.** 재실행 후에도 실패하면
무엇이 왜 비었는지 사용자에게 보고한다. **자동 재실행은 같은 단계에 대해 1회까지만** 한다.

## 완료 조건

- Penpot 파일의 **지정된 Page**에 화면이 실제로 만들어져 있다
- 각 단계의 중간 산출물이 `docs/artifacts/`에 남아 있다
- `08-findings.md` 의 **BLOCKER가 0건**이다
- `docs/artifacts/99-verify.md` 가 전 항목 통과다
- 제출 Page 인자를 받았다면 결과가 **그 Page에** 있다

## 완료 보고 형식

사용자에게 아래를 보고한다. **"완료했습니다"만 말하지 않는다.**

1. 만들어진 화면 목록과 **Page 이름** (몇 개, 어디에)
2. 8단계 5축 점수 (위계 / 유저 고려 / 기획 방향 / 레퍼런스다움 / 기본 완성도)
3. 남은 Finding: BLOCKER {n} / MAJOR {n} / MINOR {n}
4. **9단계의 지침 수정 제안** — 다음 실행 전에 담당자가 반영할 것
5. 제출 상태: 이동 완료 / 이동 대기(막힌 조건)
