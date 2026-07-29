# Figma-호환 퀵레퍼런스 (use_figma 안에서 figma.*)

> 이벤트는 Figma 인터페이스로 통일됨. 아래 `figma.*`가 Penpot으로 번역돼 저작됨(실증). 미지원은 에러로 안내 → `penpot.*` 대체.

```js
// 프레임 + Auto Layout
const card = figma.createFrame(); card.name='Card'; card.resize(320,200);
card.layoutMode='VERTICAL';           // 'HORIZONTAL' | 'VERTICAL'
card.itemSpacing=12;
card.paddingLeft=16; card.paddingRight=16; card.paddingTop=16; card.paddingBottom=16;
card.primaryAxisAlignItems='MIN';     // MIN|CENTER|MAX|SPACE_BETWEEN
card.counterAxisAlignItems='CENTER';
card.layoutSizingHorizontal='FIXED';  // HUG|FIXED (FILL은 FIXED 폴백)
card.layoutSizingVertical='HUG';

const header = figma.createFrame(); header.name='Card/Header'; header.layoutMode='HORIZONTAL';
card.appendChild(header);
const title = figma.createText('제목'); title.name='Card/Header/Title'; header.appendChild(title);

// 디자인 토큰(= Figma Variables)
const col = figma.variables.createVariableCollection('theme');
const primary = figma.variables.createVariable('color/primary', col, 'COLOR'); // '/'→'.' 자동
// primary.setValueForMode(mode, {r,g,b})  // 값 갱신 지원(RGB→hex)

// 컴포넌트
const comp = figma.createComponent(card);
```

**지원 부분집합/미지원 상세**: 이벤트 `figma-compat/README.md`. 아래는 엔진(Penpot) 네이티브 API — fallback/심화용.

---

---
author_id: choesumin
created_at: 2026-07-27T00:00:00+09:00
status: draft
project: harnessthon-1
project_docs_id: penpot-api-cheatsheet
---

# Penpot Plugin API 치트시트 (execute_code용)

> `execute_code` 툴 안에서 `penpot` 전역으로 실행. 아래는 실서버 검증된 스니펫.
> 코드는 함수 본문처럼 취급 → `return`으로 결과 반환. `storage` 객체에 중간결과 저장 가능.
> 상세는 `penpot_api_info({type,member})` / 개요는 `high_level_overview`.

## 기본 규칙
- 반환: `return {...}` (JSON 직렬화 자동). `console.log`는 반환 안 됨.
- 색상: hex 문자열(`'#4f46e5'`) 또는 fill 객체. 토큰은 문자열 value.
- 좌표/크기: `shape.resize(w,h)`, `shape.x/y` 직접 쓰기 가능.

## Board(=Frame) + Auto Layout(flex)
```js
const b = penpot.createBoard(); b.name='Card'; b.resize(320,200);
const fl = b.addFlexLayout();       // FlexLayout
fl.dir = 'column';                  // 'row' | 'column'
fl.rowGap = 12; fl.columnGap = 8;
fl.horizontalPadding = 16; fl.verticalPadding = 16;
fl.alignItems = 'center';           // start|center|end
fl.justifyContent = 'center';       // start|center|end|space-between|...
b.horizontalSizing = 'fix';         // 'fix'(고정) | 'auto'(HUG)
b.verticalSizing = 'auto';
// 자식 추가 (append 후 sizing)
const t = penpot.createText('Title'); t.name='Card/Title'; b.appendChild(t);
```
- Grid는 `b.addGridLayout()`.
- 중첩: board 안에 board를 appendChild 하고 각자 addFlexLayout.

## 디자인 토큰
```js
const cat = penpot.library.local.tokens;          // TokenCatalog
const set = cat.addSet({name:'core'});            // TokenSet
set.addToken({type:'color',   name:'color.primary', value:'#4f46e5'});
set.addToken({type:'spacing', name:'space.md',      value:'12'});
// 테마: cat.addTheme({group:'mode', name:'light'})
```
- type: `color` | `spacing` | `dimension` | `sizing` | `borderRadius` | `fontSize` | `opacity` 등.
- 토큰 적용(재사용): `board.applyToken(token, 'fill')` — 프로퍼티명에 토큰 바인딩.

## 컴포넌트 / 라이브러리
```js
const comp = penpot.library.local.createComponent([board]); // 보드를 컴포넌트화
// 인스턴스: comp.instance() 로 재사용
penpot.library.local.components   // 로컬 컴포넌트 목록
```

## 텍스트
```js
const txt = penpot.createText('Hello'); txt.name='Label';
// 폰트/크기 등은 penpot_api_info({type:'Text'}) 참조
```

## 조회/검증 (하네스 자기점검용)
```js
// 현재 페이지 위계
return penpot.currentPage.root.children.map(s=>({name:s.name,type:s.type,kids:s.children?.length??0}));
// 토큰/컴포넌트 요약
return { tokenSets: penpot.library.local.tokens.sets.map(s=>({name:s.name,n:s.tokens.length})),
         components: penpot.library.local.components.map(c=>c.name) };
```

## 심사 최적화 팁 (채점 6축과 직결)
1. **Auto Layout**: 관련 자식은 반드시 board+flex로. 절대좌표 남발 금지.
2. **네이밍**: `Card/Header/Title` 식 의미 기반. `Board 1` 같은 기본명 금지(감점).
3. **위계**: 2~4단계 깊이로 정리(과도한 평면/과도한 중첩 회피).
4. **토큰**: 색·간격·타이포를 토큰으로 정의하고 **applyToken으로 실제 적용**(정의만 하고 미적용 감점).
5. **컴포넌트**: 반복 요소는 컴포넌트화 후 **인스턴스로 재사용**.

## 아이콘 (양쪽 동일 동작 실증)
아이콘은 **SVG 문자열 → `figma.createNodeFromSvg(svg)`**. Figma·pigma 양쪽에서 동일 렌더(Lucide 등 그대로).
```js
const svg = '<svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="#3C1E1E" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><circle cx="11" cy="11" r="8"/><path d="m21 21-4.3-4.3"/></svg>';
const icon = figma.createNodeFromSvg(svg); icon.name='icon/search'; icon.x=16; icon.y=16;
```
- **컬러링**: SVG의 `stroke`/`fill`을 인라인으로 지정(위처럼). 이 방식이 양쪽에서 가장 안정적.
- Lucide/Heroicons 등 아이콘 라이브러리 SVG를 그대로 넣으면 됨. (Figma=FRAME, pigma=group 반환, 시각 동일)
