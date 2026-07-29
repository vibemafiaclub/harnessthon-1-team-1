---
name: penpot-design
description: PRD로부터 디자인 토큰·컴포넌트 라이브러리와 화면을 use_figma(MCP)로 저작한다. Figma API(figma.*)를 그대로 사용. "디자인 만들어줘", "PRD 디자인", "figma 저작", "토큰/컴포넌트 세팅" 등에 트리거.
---

# penpot-design

PRD를 입력받아 **토큰 → 컴포넌트 → 화면** 순으로 저작한다. `use_figma` MCP 툴로 코드를 실행하며, 코드 안에서 **`figma.*`(Figma Plugin API)**를 쓴다. (엔진은 Penpot이지만 인터페이스는 Figma로 통일. `penpot.*`도 병행 가능.)

## 사전조건
- Figma-호환 MCP 실행 중(운영자 배포) + 브라우저 Penpot 플러그인 Connected + `claude mcp add --transport http penpot http://localhost:4401/mcp`.
- 지원 `figma.*` 부분집합·미지원 목록: 이벤트 `figma-compat/README.md`. 미지원 호출은 명확한 에러로 안내됨 → 그때 `penpot.*`로 대체.

## 절차
1. **토큰 먼저**: 색/간격/타이포/라운드를 `figma.variables.createVariableCollection`+`createVariable`.
2. **컴포넌트**: 반복요소를 `figma.createFrame()`(+autolayout) → `figma.createComponent(frame)`.
3. **화면 조립**: 컴포넌트 인스턴스를 autolayout 프레임에 배치, 위계·네이밍 정리.
4. **자기점검**: 위계/토큰/컴포넌트 조회로 검증(`penpot.*` 읽기 병행 가능).

## 채점 축 (최적화 목표) — 2트랙

**A. 디자인 완성도 — 전 참가자 채점, 4항목 × 1~5점 → TOP 3**
화면만 보고 판단되는 것들이다. 저작 결과가 여기서 평가된다.

| 항목 | 5점 기준 |
|---|---|
| 레이아웃·정렬 | 그리드·여백이 일관되고 정렬이 맞다 |
| 타이포·컬러 | 제목/본문 위계가 명확하고 색이 조화롭다 |
| 완성도·디테일 | 실제 서비스 화면이라 해도 믿을 만하다 |
| PRD 충족도 | PRD가 요구한 화면·요소가 전부 있다 |

**B. 하네스 설계 완성도 — 심사위원(DRI) 채점, 5항목 → Best 1 (TOP 3 제외 후 선정)**
단계 분할의 타당성 · 단계 간 계약의 명료성 · **재현성(반하드코딩)** ·
**지침의 구체성** · 협업의 흔적.

> **토큰·컴포넌트 재사용·의미기반 네이밍·Frame 위계·Auto Layout은 점수 항목이 아니라
> 위 둘을 동시에 끌어올리는 수단이다.** 토큰과 컴포넌트로 저작하면 A(일관성)가 오르고,
> 하드코딩 없이 PRD를 읽어 저작하면 B의 재현성이 오른다.
> 심사용 PRD는 미공개다 — 특정 PRD 전용 스크립트는 그 자리에서 무너진다.

## API 스니펫
`cheatsheet.md` 참조 (figma.* autolayout/변수/컴포넌트 실증 코드 + penpot.* 대응).

## use_figma 규칙
함수 본문처럼 작성 → `return`으로 결과. console.log 반환 안 됨. 10연산 이하로 쪼개 점진 실행+검증. 실패 시 에러 읽고 수정 후 재시도.
