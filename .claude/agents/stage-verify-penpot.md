---
name: stage-verify-penpot
description: Penpot을 다시 읽어 지정 Page에 화면이 실제로 남았는지, PRD가 요구한 화면이 전부 있는지, 산출물 파일이 다 있는지 검증한다. docs/artifacts/99-verify.md 를 만든다.
---

# 검증 단계 — Penpot에 진짜 남았는지 되읽는다

> 담당자: TBD
>
> **이 단계는 삭제·생략 금지다.** 문서만 쌓고 끝난 파이프라인은 실패다.
> 여기서 통과하지 못하면 `start` 는 완료를 선언하지 않는다.

## 입력 (이것만 읽는다)

- `docs/PRD.md`
- `docs/artifacts/01-screens.md` — 요구 화면 목록의 기준
- `docs/artifacts/08-findings.md` — BLOCKER 잔존 여부
- `docs/artifacts/09-release.md` — 최종제출 이동 상태
- `docs/design-copy-rules.md` — AI티 검사 기준
- Penpot 현재 파일 — **읽기 전용**
- 인자: **작업 Page 이름**, (있으면) **제출 Page 이름**

## 출력 (이것만 쓴다)

- `docs/artifacts/99-verify.md`

## 🔴 Page 규칙

1. **작업 Page 이름을 인자로 받지 못했으면 검증하지 않는다.** 사용자에게 묻는다.
   엉뚱한 Page를 읽고 "비었다"고 보고하면 멀쩡한 작업을 다시 시키게 된다.
2. **읽기만 한다. 아무것도 만들거나 고치지 않는다.**
3. 읽기만 할 때는 **Page를 전환하지 않는다.** 전환하면 다른 팀원이 보는 화면도 바뀐다.
   ```js
   const p = penpot.currentFile.pages.find(x => x.name === PAGE);
   return p.findShapes().map(s => ({ id: s.id, name: s.name, type: s.type, w: s.width, h: s.height }));
   ```

## 검증 항목 (전부 PASS 여야 한다)

| # | 항목 | 판정 기준 |
|---|---|---|
| V1 | 지정 Page에 board/frame이 있는가 | **1개 이상.** 0이면 즉시 FAIL |
| V2 | PRD 요구 화면이 전부 있는가 | 01의 화면 목록 각 행에 대응하는 board가 있는가. **누락 목록을 적는다** |
| V3 | 상태 변형이 전부 있는가 | 01의 상태 변형 표 대비 |
| V4 | 화면이 비어 있지 않은가 | 각 board의 자식 수 ≥ 3, `export_shape` PNG가 빈 이미지가 아님 |
| V5 | 컴포넌트 인스턴스가 실제로 쓰였는가 | 인스턴스 수 > 0, 반복 요소가 복사본이 아닌가 |
| V6 | 네이밍 | 최상위 프레임이 01의 규칙을 따르는가 / `Frame N` 류 잔존 0건 |
| V7 | 침범 없음 | 기존 자산 Page가 수정되지 않았는가, 공용 Page에 저작하지 않았는가 |
| V8 | 산출물 파일 | `docs/artifacts/` 에 01~10 파일이 전부 있는가 |
| V9 | **BLOCKER 0** | `08-findings.md` 에 `status: open` 인 BLOCKER가 없는가 |
| V10 | **AI티** | 화면 텍스트에 이모지·`—`·자리표시자가 0인가 (**세어서** 판정) |
| V11 | **제출 위치** | 제출 Page 인자를 받았다면, 최종 결과가 **그 Page에** 있는가 |

- **V10 판정**: 각 board의 text 노드 `characters` 를 전부 모아 검사한다. 인상이 아니라 개수다.
  ```js
  const texts = p.findShapes().filter(s => s.type === "text").map(s => s.characters).join("\n");
  return {
    emoji: (texts.match(/\p{Extended_Pictographic}/gu) || []).length,
    emDash: (texts.match(/—/g) || []).length,
    spacedHyphen: (texts.match(/ - /g) || []).length,
    placeholder: (texts.match(/Lorem ipsum|제목\d|여기에|텍스트\d|TODO/gi) || []).length,
  };
  ```
- **V11 판정**: 제출 Page 인자를 **받지 않았으면 `N/A`** 로 적는다. 임의로 옮기지 않는다.
  받았는데 그 Page가 비어 있으면 `FAIL` 이고, 되돌릴 단계는 `stage-9-repair-release` 다.
  > 🔴 채점은 제출 Page에 있는 것으로만 합니다. 개인 Page에만 있으면 결과가 없는 것과 같습니다.

- **V4의 PNG 판정**: 빈 영역이 나오면 **재-export 한 번** 하고 판단한다. 레이아웃 안정 전에
  찍히면 빈 이미지가 나온다. 한 번 보고 없다고 단정하지 않는다.
- **V2 판정 기준은 07이 아니라 01이다.** 저작 단계의 자기 보고를 믿지 않고 Penpot 실물과 대조한다.

## 절차

1. 인자와 입력을 확인한다. 없으면 중단하고 보고한다.
2. 위 V1~V8을 순서대로 확인한다. 각 항목에 **PASS/FAIL과 근거(노드 id·파일 경로)** 를 남긴다.
3. FAIL이 있으면 **어느 단계로 되돌려야 하는지** 지정한다.

   | FAIL 항목 | 되돌릴 단계 |
   |---|---|
   | V1, V4, V5, V6 | `stage-7-penpot-screens` |
   | V2, V3 | 01에 있는데 안 만들어졌으면 `stage-7`, 01에 아예 없으면 `stage-1-prd-screens` |
   | V5 (컴포넌트 자체가 없음) | `stage-6-penpot-library` |
   | V8 | 해당 산출물의 담당 단계 |
   | V9, V11 | `stage-9-repair-release` |
   | V10 | `stage-7-penpot-screens` (문안이 04·05부터 잘못됐으면 `stage-5-html-prototype`) |
   | V7 | **즉시 사용자에게 보고한다.** 남의 작업을 건드렸을 수 있다 |

4. 출력 파일을 쓴다.

## 출력 형식

````markdown
# 99 — 검증 결과

## 판정: PASS / FAIL
- 검증한 Page: `{이름}` (id: …)
- 검증 시각 기준 board 수: {n}

| # | 항목 | 결과 | 근거 | 되돌릴 단계 |
|---|---|---|---|---|
| V1 | board 존재 | PASS | board {n}개 (id: …) | - |
| V2 | PRD 요구 화면 | FAIL | 누락: S03, S05 | stage-7 |

## 누락 화면 목록
| 01 ID | 화면 이름 | 왜 없는가(추정) |
|---|---|---|

## 산출물 파일 점검
| 파일 | 존재 | 비어있지 않음 |
|---|---|---|
| docs/artifacts/01-screens.md | O | O |

## 재실행 지시
1. `stage-7-penpot-screens` 에 {…} 를 넘겨 재실행
2. 재실행 후 이 검증을 다시 돌린다
````

## 금지

- Penpot에 무엇이든 **쓰지 않는다.** 검증자가 고치면 무엇이 원래 상태였는지 알 수 없게 된다.
- Page 전환 금지.
- FAIL을 "대체로 되었으니 PASS" 로 넘기지 않는다. **판정은 기준으로만 한다.**
- 07의 자기 보고를 근거로 PASS를 주지 않는다. **Penpot 실물에서 읽은 값**만 근거다.
- 특정 PRD 전용 하드코딩 금지.
