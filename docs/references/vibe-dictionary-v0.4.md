# Vibe Dictionary — UI 컴포넌트 분류체계 v0.4 (하네스용 발췌)

> stage-5 (디자인 시스템) agent가 컴포넌트 **명명·분류·상태 정의** 기준으로 로드한다.
> 원문 사상: "디자인이 개발의 메타 언어". 기준점 = Apple HIG · Google Material · NN/g · WCAG.
> V0/Lovable/Bolt 출력물은 기준으로 삼지 않는다 (표준 아님).

## 3층 구조

```
[1층] 인터랙션 목적 (원론)  →  [2층] 패턴 원형 (HIG/Material)  →  [3층] 플랫폼 구현 어휘
```

## 디자인 토큰 원칙 (MUI 수준 시멘틱 구조)

| 토큰 유형 | 패턴 | 예 |
|---|---|---|
| 색상 | main / light / dark / contrastText | `primary.main`, `primary.light` |
| 상태색 | error / warning / success / info | 기본 제공 |
| 타이포 | variant 체계 | h1–h6, body1, body2, caption |
| 간격 | 일관 스케일 | `spacing(1) = 8px` |
| 반경 | 단계 정의 | sm, md, lg, xl |

- 확장 규칙: 새 색 `brand` 추가 → `brand.main/light/dark/contrastText` 4종 전부.
- **Penpot 저작 시 토큰은 JS 상수 객체로** (AGENTS.md 함정: `figma.variables.*` 안 남음).

## 컴포넌트 vs 스타일 분리

- 컴포넌트 목록에 스타일 조합을 넣지 않는다. ❌ "GlassCard" / ✅ "Card + surface:transparent"
- 모양(색·크기·간격·폰트) = 토큰. 동작(hover·click) = 컴포넌트 상태 변형.

## 카테고리 (Part 1 기본 컴포넌트 — 하네스에서 주로 쓸 것)

| 카테고리 | 핵심 컴포넌트 (이 과제 관련) |
|---|---|
| Typography | Heading(h1–h6), Paragraph, Label, Caption, List |
| Container | Box, Divider, Accordion, ScrollArea |
| Card | BaseCard, OutlinedCard, FilledCard, ActionCard, SplitCard |
| Data Display | Table, List, Statistic, Badge, Tag, Progress, **Empty**, **Skeleton**(로딩) |
| In-page Nav | **Tabs**(차트 기간 전환), **SegmentedControl**, Steps, Pagination |
| Input & Control | **Button**, IconButton, Input, Select, Checkbox, Radio, **Switch(토글)**, Slider, InputNumber, **Chip**(필터/테마), Form |
| Layout | Grid, Flex, Layout(Header/Content/Footer), Affix(하단 고정) |
| Overlay & Feedback | Modal, **Sheet(Bottom)**, Tooltip, Toast, Alert, Spinner, **Result**(주문 성공/실패) |
| Navigation (Global) | **NavigationBar(하단)**, TopAppBar |

경계 판정 정본: Chip/Tag→Input(선택 액션이 주목적), Empty→Data Display,
Accordion→Container, Steps→In-page Nav.

## 상태 축 (컴포넌트 변형 정의 기준)

- **보편성**: Canonical(HIG/Material 표준) / Common / Unusual / Experimental
- **상호작용성**: Static / Reactive / Interactive / Animated
- **밀도**: Minimal / Compact / Default / Expanded

**Material 상태값 세트 (Interactive 컴포넌트당)**: `enabled / pressed(selected) / disabled`
(+ 정보 변형: 예: Button primary/secondary, Switch on/off). 버튼과 토글은 별개 컴포넌트.

## 하네스 적용 규칙

1. Penpot 컴포넌트 이름 = `{카테고리}/{컴포넌트}/{변형}` (예: `Input/Button/Primary-Enabled`)
   — 컴포넌트 이름은 파일 전역이므로 팀원 충돌 방지를 위해 프리픽스 필수.
2. 새 컴포넌트가 필요하면 이 분류 안에서 자리를 찾는다. 없으면 합성으로 정의
   (예: `StockListRow = List + Statistic + Badge`).
3. 값(색·폰트·간격)은 이 문서가 아니라 `05a-tokens.md`(기존파일 실측)에서 온다.
   이 문서는 **구조와 이름**만 준다.
