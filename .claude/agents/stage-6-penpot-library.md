---
name: stage-6-penpot-library
description: 작업 Page에 디자인 토큰을 적용한 공통 컴포넌트 라이브러리를 Penpot 네이티브로 만들고 이름·id 매핑을 남긴다. docs/artifacts/06-library.md 를 만든다.
---

# 6단계 — Penpot 라이브러리 (토큰 + 공통 컴포넌트)

> 담당자: TBD
>
> HTML을 가져오지 않는다. **Penpot에서 편집 가능한 네이티브 구조로 새로 짓는다.**

## 입력 (이것만 읽는다)

- `docs/artifacts/04-design-guide.md` — TOKENS 코드블록과 컴포넌트 스펙
- `docs/artifacts/05-html-review.md` — 확정 사항 표
- `docs/artifacts/02-existing-assets.md` — 폰트 가용성, 참조 복제본 구역 좌표
- 인자: **작업 Page 이름**

## 출력 (이것만 쓴다)

- `docs/artifacts/06-library.md`
- 작업 Page 안의 컴포넌트 마스터들 (`Lib/` 접두사)

## 🔴 작업 Page 게이트 (건너뛰기 금지)

1. **작업 Page 이름을 인자로 받지 못했으면 저작을 시작하지 않는다.** 사용자에게 묻고 답을 기다린다.
2. **기본값으로 첫 Page를 쓰지 않는다.** 추측해서 고르지 않는다.
3. `중간공유`·`최종제출` 은 공용 Page다. 여기서 처음부터 저작하지 않는다.
4. `기존파일`(PRD가 지정한 기존 자산 Page)은 **읽기 전용**이다. 수정 금지.
5. **`penpot.openPage()` 는 다음 호출까지 유지되지 않는다.** 모든 스크립트 **첫 줄**에서 작업 Page를
   다시 고정한다. 아래 프리앰블을 매 `use_figma` 호출에 붙인다.

```js
const PAGE = "{작업 Page 이름}";
const p = penpot.currentFile.pages.find(x => x.name === PAGE);
if (!p) return { error: "page not found: " + PAGE };
penpot.openPage(p);
```

6. **Page를 전환한 그 호출 안에서 새 Page의 노드를 새로 만들면 죽는다.**
   최초 전환은 **전환만 하는 호출**로 따로 하고, 저작은 그다음 호출부터 한다.

## 절차

1. 입력 3개를 읽는다. 없으면 **즉시 중단하고 보고한다.**
2. `high_level_overview` 를 읽는다. 작업 Page를 게이트대로 고정한다.
3. **폰트를 먼저 확정한다.** 04가 지정한 폰트가 서버에 실제로 있는지 다시 확인한다.
   없으면 **조용히 대체되므로** 여기서 잡지 않으면 끝까지 모른다.
   ```js
   return penpot.fonts.all.filter(f => f.name.includes(FAMILY)).map(f => f.name);
   ```
4. **좌표 구역을 정한다.** 02가 적어둔 참조 복제본 구역과 겹치지 않게 라이브러리 구역을 잡는다.
   (예: 라이브러리 `y = 0` 대, 화면 저작은 `y = 1200` 대 — 7단계에 이 좌표를 넘긴다.)
5. **토큰은 JS 상수로 굴린다.** 04의 `TOKENS` 코드블록과 `fill()` 헬퍼를 **그대로 복사해서**
   매 스크립트 상단에 둔다.
   > `figma.variables.*` 는 성공 응답만 오고 토큰이 실제로 남지 않는다. 시도하지 않는다.
   > 점수는 **컴포넌트 재사용**으로 받는다.
6. **컴포넌트를 하나씩 만든다. 한 번에 몰아넣지 않는다.** 컴포넌트 1개 = `use_figma` 호출 1~2개.
   각 컴포넌트마다:
   - 컨테이너 board 생성 → auto layout 설정 → 자식 배치 → 토큰 적용 → 컴포넌트화 → **export 확인**
   - **이름을 처음에 확정한다.** 만든 뒤 이름 변경·자식 remove는 플러그인을 멈추게 한다.
     잘못 만들었으면 **고치지 말고 새 이름으로 새로 만든다.**
   - 이름 규칙: `Lib/{그룹}/{이름}` (예: `Lib/Button/Primary`). 의미 기반으로. `Frame 27` 금지.
7. **저작 함정 — 아래를 그대로 지킨다.**

   | 하려는 것 | 이렇게 한다 |
   |---|---|
   | 색 채우기 | `node.fills = [{ fillColor: "#RRGGBB", fillOpacity: 1 }]` (penpot 형식). figma 형식은 인스턴스에서 막힌다 |
   | 가로 크기 정책 | `node.horizontalSizing = "fix" \| "auto"` (`primaryAxisSizingMode` 등 figma 사이징은 안 먹는다) |
   | 텍스트 높이 | `text.growType = "auto-height"`. `"fixed"` 로 두면 글자가 잘린다 |
   | 가변 텍스트 칸 | **고정 폭 + 텍스트 정렬.** hug(자동 폭)은 텍스트를 갈아끼워도 위치가 안 따라온다 |
   | 하단 고정 요소 | `layoutGrow` Spacer는 폭 1로 되돌아간다. **Spacer 높이를 계산해서 명시** |
   | 비-오토레이아웃 부모에 붙이기 | `appendChild` 후 `c.x = parent.x + dx; c.y = parent.y + dy` 로 직접 위치 |
   | 반투명 오버레이 | `fillOpacity: 0.4` 스크림은 렌더링에서 사라질 때가 있다. **뒤 보드의 `opacity` 를 낮춘다** |
   | 자식 순서 교정 | `board.insertChild(index, node)` (지우기보다 안전하다) |
   | 실제 사진 | `const img = await penpot.uploadMediaUrl(name, url)` → `rect.fills = [{ fillOpacity:1, fillImage: img }]` (top-level await 가능) |

8. **컴포넌트화하고 id를 즉시 기록한다.**
   ```js
   const comp = penpot.library.local.createComponent([node]);   // 미지원이면 figma.createComponent 계열로 대체 시도
   return { name: node.name, compId: comp.id, mainId: comp.mainInstance?.id ?? node.id };
   ```
   **컴포넌트 이름은 파일 전역이다.** 옆 팀원 Page의 동명 컴포넌트가 잡힌다.
   그래서 7단계가 이름만으로 찾으면 안 된다 — **여기서 기록한 id가 유일한 안전장치다.**
   id를 못 남긴 컴포넌트는 만들지 않은 것과 같다.
9. **상태 변형**: 04 E절의 상태 중 화면에 실제로 등장하는 것만 만든다.
   처음부터 다시 짓지 말고 **`shape.clone()` → 덮을 것만 얹는다.** 이름은 `Lib/Button/Primary--Disabled` 처럼.
10. **텍스트 재계산**: 라이브러리를 다 만든 뒤, `growType === "auto-height"` 인 텍스트를 전부 찾아
    `resize` 로 한 번 재계산시킨다. hHug 프레임 안 텍스트가 아래가 잘린 채 굳는 것을 막는다.
    ```js
    p.findShapes().filter(s => s.type === "text" && s.growType === "auto-height")
      .forEach(t => t.resize(t.width, t.height));
    ```
11. **검증**: 컴포넌트 그룹마다 `export_shape` 로 PNG를 본다. 빈 영역이 나오면 **재-export 한 번**
    한 뒤에 판단한다. 잘림·겹침·색 누락이 보이면 그 자리에서 고친다. 안 보고 쌓지 않는다.
12. 출력 파일을 쓴다.

## 출력 형식

````markdown
# 06 — Penpot 라이브러리

## 저작 환경
- 작업 Page: `{이름}` (id: …)
- 라이브러리 좌표 구역: x {…}~{…}, y {…}~{…}
- **7단계 화면 저작 구역(비어 있음): x {…} 부터, y {…} 부터** ← 겹치면 안 된다
- 확정 폰트: `{family}` (서버 존재 확인됨 / 대체됨: 원본 `{…}`)

## 컴포넌트 매핑 — 7단계는 이 표의 id로 찾는다

| 이름 | compId | mainInstanceId | 용도 | 크기 | 오버라이드 가능 | 상태 변형 |
|---|---|---|---|---|---|---|
| Lib/Button/Primary | … | … | 1차 액션 | 343×48 | characters, fills | --Disabled, --Loading |

- 오버라이드 가능: 인스턴스에서 바꿀 수 있는 것만 적는다.
  **`characters`(텍스트) 오버라이드는 된다. `fills` 는 penpot 형식이면 된다.**

## 토큰 상수 (7단계가 그대로 복사해 쓴다)

```js
const TOKENS = { /* 04에서 확정된 최종본 그대로 */ };
const fill = (hex, opacity = 1) => [{ fillColor: hex, fillOpacity: opacity }];
```

## 인스턴스 생성 스니펫 (7단계용)

```js
const comp = penpot.library.local.components.find(c => c.id === COMP_ID);  // 이름이 아니라 id로 찾는다
const inst = comp.instance();
inst.name = "…";
// 텍스트 오버라이드
inst.findShapes().find(s => s.name === "Label").characters = "…";
```

## 검증 로그
| 컴포넌트 | export 결과 | 재-export | 남은 문제 |
|---|---|---|---|

## 8단계로 넘기는 문제 (여기서 못 고친 것)
| 항목 | 발생 문제 | 시도한 대응 | 결과 |
|---|---|---|---|
````

## 금지

- `figma.variables.*` 로 토큰 등록 시도 금지. 성공 응답만 오고 남지 않는다.
- 만든 컴포넌트의 **이름 변경·자식 remove 금지.** 플러그인이 멈춘다. 새 이름으로 새로 만든다.
- 컴포넌트를 **이름만으로** 조회하지 않는다. 이름은 파일 전역이라 남의 Page 것이 잡힌다. id로 찾는다.
- 한 번의 `use_figma` 호출에 라이브러리 전체를 몰아넣지 않는다. 작게 쪼개고 매번 검증한다.
- `기존파일` Page 수정 금지. 공용 Page(`중간공유`·`최종제출`)에서 저작 금지.
- 5단계 HTML을 import 하거나 그대로 옮기지 않는다. 네이티브로 새로 짓는다.
- 특정 PRD 전용 하드코딩 금지 — 컴포넌트 목록·값은 전부 04에서 읽는다.
- 다른 단계의 출력 파일을 쓰지 않는다.
