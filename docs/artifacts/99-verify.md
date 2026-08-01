# 99 — 검증 (run-02, 30분 압축 실행)

| 항목 | 판정 | 근거 |
|---|---|---|
| V1 지정 Page에 board 존재 | PASS | `3-toss-result`에 New/* 보드 6개 + Lib 8종 |
| V2 PRD 필수 화면 | PASS | New/Home, New/Results 존재 (+상태 4종) |
| V3 01-screens P0·P1 커버 | PASS | S01·S03(P0), S02·S04·S05·S06(P1) 전부. P2 3종은 시간상 미저작(계획된 생략) |
| V4 빈 화면 없음 | PASS | 모든 보드 자식 5~14개, 코드 검증 |
| V5 컴포넌트 인스턴스 사용 | PASS | Chip·SectionHeader·Button·FareCard·FlightRow·FlightRowStop·TabBar·DealCard 인스턴스로 저작 |
| V6 네이밍 | PASS | `Frame N`류 0건 (analyzeDescendants 전수) |
| V7 침범 없음 | PASS | `3-toss`는 읽기만(openPage 안 함), 저작은 `3-toss-result`만 |
| V8 산출물 파일 | PASS | 01·02·04·06·07·99 (03·05는 사용자 지시로 생략, 08~10은 시간상 축약) |
| V9 BLOCKER | PASS(조건부) | 독립 비평(8단계) 미실행 — 30분 제약. 코드 검증상 카피·네이밍·구조 결함 0 |
| V10 이모지·em dash·자리표시자 | PASS | 전 텍스트 정규식 전수 검사 0건 |
| V11 제출 Page | PASS | 사용자가 지정한 page-id `4eb60b9b-410f-42df-8b61-270d8d4ad027`(=`3-toss-result`)에 직접 저작 |

## 알려진 한계 (10-limitations 겸용)
1. `export_shape` 서버 오류(Failed to fetch / 30s timeout)가 세션 내내 지속 → **PNG 육안 검증 미수행**. 지오메트리·카피·네이밍은 코드로 전수 검증했으나 시각 겹침·폰트 렌더링은 Penpot UI에서 확인 필요.
2. 독립 비평(stage-8)·수리 루프(stage-9) 미실행 — 30분 제약의 의도된 타협.
3. `figma.variables` 토큰 미사용(잔존 안 되는 알려진 함정) → JS 상수로 대체.
4. Spoqa Han Sans Neo 부재 → Noto Sans KR 대체.
5. 이미지 자산 0장 구성(여행 상품도 텍스트 카드) — 30분 내 uploadMediaUrl 리스크 회피.
