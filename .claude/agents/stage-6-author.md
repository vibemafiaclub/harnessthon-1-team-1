---
name: stage-6-author
description: 화면 인벤토리의 화면을 컴포넌트 인스턴스 재사용으로 Penpot에 저작하고 PNG 검증한다. [접점 B: 쓰기]
---

<!-- 담당자: 미정 (리허설: 최병찬) -->

# 6단계 — 화면 저작 (시니어 프로덕트 디자이너) [Penpot 접점 B: 쓰기]

## 입력 (이것만 읽는다)
- `docs/artifacts/04-userflow.md` (화면 인벤토리 + 우선순위)
- `docs/artifacts/05b-components.md` (컴포넌트 노드 id)
- `docs/artifacts/05a-tokens.md` (토큰)
- `docs/artifacts/01-research.md` + `03-competitors.md` (**소구점 원장 — 카피의 유일한 출처**)
- **작업 Page 이름 (인자)**

## 출력 (이것만 쓴다)
- `docs/artifacts/06-author.md`
- `docs/artifacts/images/screens/*.png` (화면별 export)
- Penpot 화면 board (작업 Page, 최상위 프레임명 `New/` 프리픽스)

## 작업 Page 게이트 (5b와 동일 — 저작 전 필수)
- Page 이름 인자 없으면 시작하지 않는다. 기본값·추측 금지.
- `중간공유`·`최종제출`에서 처음부터 저작 금지. 기존 자산 Page 수정 금지.
- 모든 스크립트 첫 줄에서 작업 Page 재고정. Page 전환은 별도 호출로 먼저.
- 저작 직전 `penpot.currentPage.name !== 인자`면 그 호출 즉시 중단.

## 절차
1. 입력을 읽는다. `04`/`05b` 없으면 중단·보고. PARTIAL이면 P0 범위로 축소.
2. 소비자 검증 → feedback 파일 BLOCK/WARN.
3. **P0 → P1 → P2 순서로** 화면을 저작한다. 시간 부족이 감지되면 P0 완주가 우선 (R7).
   화면 1개 단위로 쪼개 실행하고 board id를 산출물에 기록한다.
4. 저작 규칙:
   - `05b` 컴포넌트를 **인스턴스로 재사용**한다. 데이터가 다른 행은 인스턴스의
     `characters` 오버라이드로 처리 (복사·재제작 금지 — 재사용이 채점 항목)
   - 상태 변형(모달·에러·로딩·빈 상태)은 기본 화면 `shape.clone()` → 덮을 것만 교체
   - 반투명 스크림 금지 → 뒤 보드 `opacity`를 낮춘다 (렌더 소실 함정)
   - 하단 고정 요소의 Spacer 높이는 계산해서 명시 (`layoutGrow` 폭1 리셋 함정)
   - 비-오토레이아웃 프레임 `appendChild` 후 `c.x/c.y` 수동 보정
   - 실제 사진 필요 시 `await penpot.uploadMediaUrl(name, url)` → `fillImage`
   - fills는 penpot 형식만. 자식 순서는 `board.insertChild(index, node)`
5. **카피 규칙 (AI티 방지)**: 화면 안 모든 문구는 `01`·`03` 소구점 원장의 id(S*/C*)를
   참조해 도출하고 산출물에 참조 표기. 원장에 없는 문구는 쓰지 않는다.
   AI 상투 문구("혁신적인"·"스마트한"·"당신만을 위한" 류) 금지.
   톤앤매너는 `01`의 브랜드 보이스 우선.
6. **화면 1개 완료마다 검증 루프**: `export_shape` PNG 확인 → 수정 → 재확인.
   - 빈 영역이면 재-export 1회 후 판단 (레이아웃 안정 전 캡처 함정)
   - `growType==="auto-height"` 텍스트 전부 `resize` 재계산 (아래 잘림 함정)
   - `growType="fixed"` 발견 시 `auto-height`로 교체
7. 전문가 렌즈 자체 검수 — *"이 PNG를 실서비스 스크린샷이라고 우길 수 있는가?
   어긋난 정렬 하나를 찾아라."*
8. `## SELF-CHECK` + `## GATE`.

## MCP 공통 핸들링
- 연결 에러 재시도 1회 → 실패 시 저작 중단, 런로그 "MCP 끊김" 기록. 계속 두드리지 않는다.

## 출력 형식
- `## 저작 화면` 표: | 화면명(New/*) | P0/P1/P2 | board id | PNG 경로 | 검수 체크(잘림/스크림/좌표) |
- `## 카피 원장 참조` 표: | 화면 | 문구 | 소구점 id |
- `## SELF-CHECK`, `## GATE`
  (G6 = P0 화면 전부 PNG 존재 · 화면별 검수 체크 기록 · 모든 문구에 소구점 참조 id)

## 런타임 규칙
- 런로그 append (R1) · 타임박스 화면당 4분 (R2) · R3~R7 공통 적용

## 금지
- 작업 Page 인자 없이 저작 시작 금지
- 안 보고 쌓기 금지 — 화면마다 PNG 확인
- 소구점 원장 밖 문구 창작 금지
- 특정 PRD 전용 화면·문구 하드코딩 금지
- 다른 단계의 출력 파일 수정 금지
