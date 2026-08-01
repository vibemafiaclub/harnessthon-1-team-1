---
name: stage-7-penpot-screens
description: 라이브러리 인스턴스로 PRD가 요구한 화면을 작업 Page에 전부 저작하고 상태 변형·프로토타입 연결·네이밍까지 마무리한다. docs/artifacts/07-screens.md 를 만든다.
---

# 7단계 — Penpot 화면 저작

> 담당자: TBD
>
> **디자인이 실제로 남는 단계다.** 여기가 비면 하네스 전체가 실패다.

## 입력 (이것만 읽는다)

- `docs/artifacts/01-screens.md` — 만들 화면 목록과 상태 변형
- `docs/artifacts/04-design-guide.md` — 토큰
- `docs/artifacts/05-html-review.md` — "Penpot 저작에 넘기는 확정 사항" 표
- `docs/artifacts/06-library.md` — 컴포넌트 compId, 저작 구역 좌표
- 인자: **작업 Page 이름**

## 출력 (이것만 쓴다)

- `docs/artifacts/07-screens.md`
- 작업 Page 안의 화면 board들

## 🔴 작업 Page 게이트 (건너뛰기 금지)

1. **작업 Page 이름을 인자로 받지 못했으면 저작을 시작하지 않는다.** 사용자에게 묻고 답을 기다린다.
2. **기본값으로 첫 Page를 쓰지 않는다.**
3. `중간공유`·`최종제출` 은 결과를 **옮겨 담는** 공용 Page다. 여기서 처음부터 저작하지 않는다.
4. `기존파일`(PRD가 지정한 기존 자산 Page)은 **읽기 전용**이다.
5. **모든 `use_figma` 호출 첫 줄에서 작업 Page를 다시 고정한다.** `openPage` 는 유지되지 않는다.

```js
const PAGE = "{작업 Page 이름}";
const p = penpot.currentFile.pages.find(x => x.name === PAGE);
if (!p) return { error: "page not found: " + PAGE };
penpot.openPage(p);
const TOKENS = { /* 06에서 복사 */ };
const fill = (hex, o = 1) => [{ fillColor: hex, fillOpacity: o }];
```

6. 최초 Page 전환은 **전환만 하는 호출**로 따로 한다. 전환한 그 호출에서 새 노드를 만들면 죽는다.

## 절차

1. 입력 4개를 읽는다. 없으면 **즉시 중단하고 보고한다.** 특히 06의 compId 표가 없으면
   인스턴스를 만들 수 없으므로 6단계로 되돌린다.
2. **배치 계획을 먼저 세운다.** 06이 넘긴 빈 구역에서 시작한다.
   - 화면 간격: `x += screen.width + 200`, 흐름이 바뀌면 `y += screen.height + 300`
   - 02의 참조 복제본 구역, 06의 라이브러리 구역과 **겹치지 않는지 계산으로 확인**한다.
   - 배치 표를 출력에 남긴다. 7단계 재실행 시 같은 자리에 그릴 수 있어야 한다.
3. **화면 이름은 01의 네이밍 규칙을 따른다.** (PRD가 접두사를 지정했으면 그대로.)
   내부 프레임 이름도 **의미 기반**으로 짓는다. `Frame 27` 이 아니라 `StockRow`·`FilterBar` 다.
   이름은 **처음에 확정한다.** 만든 뒤 이름 변경은 플러그인을 멈추게 한다.
4. **화면을 하나씩 만든다. 한 번에 몰아넣지 않는다.**
   화면 1개 = `use_figma` 호출 여러 개 (골격 → 섹션 → 인스턴스 → 텍스트) + **`export_shape` 검증 1회**.
   > 안 보고 쌓으면 마지막에 전부 어긋나 있다. **매 화면마다 PNG를 본다.**
5. **반복 요소는 반드시 인스턴스로 만든다.** 복사·붙여넣기 금지. 재사용이 채점 대상이다.
   ```js
   const comp = penpot.library.local.components.find(c => c.id === COMP_ID); // 이름 아닌 id로 찾는다
   const inst = comp.instance();
   inst.findShapes().find(s => s.name === "Title").characters = row.title;   // 텍스트 오버라이드는 된다
   inst.findShapes().find(s => s.name === "Delta").fills = fill(row.up ? TOKENS.color.success : TOKENS.color.danger);
   ```
   **데이터가 다른 행은 인스턴스 재사용 + `characters`/penpot 형식 `fills` 오버라이드로 처리한다.**
6. **콘텐츠는 05가 확정한 것을 쓴다.** 행 수도 05의 표를 따른다. 3행만 넣고 끝내지 않는다.
   실제 사진이 필요하면 넣을 수 있다 (top-level await 가능):
   ```js
   const img = await penpot.uploadMediaUrl("cover", url);
   rect.fills = [{ fillOpacity: 1, fillImage: img }];
   ```
7. **상태 변형은 `clone()` 으로 만든다.** 빈 상태·로딩·에러·모달을 처음부터 다시 짓지 않는다.
   ```js
   const variant = base.clone();
   variant.name = base.name + "--Empty";
   variant.x = base.x + (TOKENS.screen.width + 200);
   ```
   **모달·시트는 뒤 화면 board의 `opacity` 를 낮춰서 표현한다.** 반투명 스크림 사각형을 덮으면
   렌더링에서 사라질 때가 있다.
8. **화면 간 프로토타입 연결**을 시도한다. (`figma.*` 미지원이면 `penpot.*` 로 대체를 시도한다.)
   연결이 지원되지 않으면 **억지로 우회하지 말고** 흐름 순서를 화면 이름·배치로 표현하고
   그 사실을 8단계로 넘긴다.
9. **레이아웃 마무리 — 전 화면 저작이 끝난 뒤 한 번에 돌린다.**
   ```js
   p.findShapes().filter(s => s.type === "text" && s.growType === "auto-height")
     .forEach(t => t.resize(t.width, t.height));   // hHug 안 텍스트 잘림 해제
   ```
   그다음 **모든 화면을 다시 `export_shape` 한다.** 빈 이미지는 재-export 한 번 더 하고 판단한다.
10. **커버리지 대조**: 01의 화면 목록·상태 변형 표와 실제 만든 board를 1:1로 대조한다.
    **빠진 것이 있으면 이 단계는 끝나지 않았다.** 못 만든 것은 이유와 함께 남긴다.
11. 출력 파일을 쓴다.

## 저작 함정 (6단계와 동일 — 어기면 시간을 통째로 날린다)

| 하려는 것 | 이렇게 한다 |
|---|---|
| 색 | `fills = [{ fillColor:"#RRGGBB", fillOpacity:1 }]` — penpot 형식. figma 형식은 인스턴스에서 막힌다 |
| 크기 정책 | `node.horizontalSizing = "fix" \| "auto"` |
| 텍스트 | `growType = "auto-height"`. 고정 폭 + `"fixed"` 는 글자가 잘린다 |
| 가변 텍스트 칸 | 고정 폭 + 정렬. hug 칸은 텍스트를 갈아끼워도 위치가 안 따라온다 |
| 하단 고정 | Spacer 높이를 **계산해서 명시** (`layoutGrow` 는 폭 1로 되돌아간다) |
| 비-오토레이아웃 부모 | `appendChild` 후 `c.x = parent.x + dx; c.y = parent.y + dy` |
| 자식 순서 | `board.insertChild(index, node)` |
| 오버레이 | 뒤 board의 `opacity` 를 낮춘다 |
| 잘못 만든 컴포넌트/노드 | 이름 변경·자식 remove 금지. 새 이름으로 새로 만든다 |

## 출력 형식

````markdown
# 07 — Penpot 화면 저작 결과

## 저작 환경
- 작업 Page: `{이름}` (id: …)
- 저작 구역: x {…}~{…}, y {…}~{…} (02 참조구역·06 라이브러리구역과 비겹침 확인: OK)

## 화면 인벤토리

| 01 ID | board 이름 | board id | 좌표 | 크기 | 쓴 컴포넌트(compId) | 인스턴스 수 | export 확인 |
|---|---|---|---|---|---|---|---|

## 상태 변형

| 상위 board | 변형 이름 | id | 만든 방법(clone/신규) | export 확인 |
|---|---|---|---|---|

## 커버리지 대조 (01 대비)

| 01 ID | 요구 | 만들어짐 | 비고 |
|---|---|---|---|
**미구현: {n}건** ← 0이 아니면 이 단계는 미완료다. 이유를 아래에 적는다.

## 프로토타입 연결
| 출발 | 도착 | 트리거 | 결과(성공/미지원) |
|---|---|---|---|

## 네이밍 정리
- 최상위 프레임 규칙: {01에서 읽은 규칙}
- 의미 없는 이름(`Frame N`) 잔존: {n}건 ← 0이어야 한다

## 8단계로 넘기는 문제
| 항목 | 발생 문제 | 영향 | 시도한 대응 | 결과 |
|---|---|---|---|---|
````

## 금지

- 반복 요소를 복사·붙여넣기로 만들지 않는다. **인스턴스**로 만든다.
- 컴포넌트를 이름으로만 찾지 않는다. 이름은 파일 전역이다. **06의 compId** 로 찾는다.
- 한 호출에 여러 화면을 몰아넣지 않는다. 화면마다 export 검증한다.
- 자리표시자 텍스트(`Lorem ipsum`, `제목1`) 금지.
- `기존파일` 수정 금지. 공용 Page 저작 금지. 참조 복제본 구역·라이브러리 구역 침범 금지.
- 특정 PRD 전용 하드코딩 금지 — 화면 목록·개수·이름은 전부 01/05에서 읽는다.
- 다른 단계의 출력 파일을 쓰지 않는다.
