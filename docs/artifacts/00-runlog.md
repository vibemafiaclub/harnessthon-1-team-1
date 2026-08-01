
## stage-1-research
- [START] $(date '+%Y-%m-%d %H:%M:%S') 도메인 리서치 시작. 입력=docs/PRD.md
- [START-TS] 2026-08-01 15:41:07 실제 타임스탬프 (위 줄의 $(date)는 이스케이프 실수, 무시)
- [결정] 도메인 키워드=증권/주식 초보 투자(MTS·관심종목·매수), 모회사=당근(당근마켓). 검색은 모회사명 없이 도메인 키워드로만 수행, 모회사 검색은 소구점 추출용으로만 분리 사용.
- [결정] 도메인 리포트 폴백 불필요 — 오픈서베이 '금융 투자 트렌드 리포트 2026'(2026-03-09)이 도메인 정합 리포트로 확인되어 그대로 채택.
- [결정] 1년 초과 소스(토스증권 MZ 70% 통계 2022 byline, 대학내일20대연구소 2022) 폐기(R3 상당 처리, 발행일 확인 후 배제).
- [END] 2026-08-01 15:44:21 산출물=docs/artifacts/01-research.md 작성 완료. 소구점 5개(S1~S5). GATE=PASS.

## stage-2-persona
- [START] 2026-08-01 15:46:15 페르소나 작성 시작. 입력=docs/PRD.md + docs/artifacts/01-research.md(GATE PASS) + docs/references/persona-nng.md
- [소비자 검증] 01-research.md를 "이걸로 일할 수 있는가"로 검사 → WARN 3건(투자 스타일별 정량 데이터 원자료 게이트로 부재/증권 도메인 verbatim 소구점 부재/PRD §5 상황 데이터는 01이 아니라 PRD가 출처) 기록, BLOCK 없음(재실행 요청 안 함). 상세: docs/artifacts/feedback/01-feedback.md
- [결정] 페르소나 3명 채택 — PRD §3 유저 스토리 4개(모으기/판단/주문/발견)를 2명으로는 못 덮음(판단 스토리의 "등락률 vs 수익률 vs 거래량" 다양성, 발견 스토리의 "스타일 미정 vs 테마 탐색" 다양성 때문에 3번째 페르소나 강명숙 추가)
- [결정] 근거 없는 필드(성별·구체 직업 등)는 창작하지 않고 `근거 부족`으로 표기 — persona-nng.md 금지 조항 준수
- [전문가 렌즈 자체 검수] 3명 페어와이즈 비교 결과 전부 화면 설계 차이 발생 확인 → 병합 없이 3명 유지
- [END] 2026-08-01 15:52:30 산출물=docs/artifacts/02-persona.md 작성 완료. 페르소나 3명(정하늘·이도현·강명숙). 커버리지 맵 미커버 스토리 0건. GATE=PASS.
## [stage-3-competitors] 2026-08-01

- 입력: docs/PRD.md (당근마켓팀 증권서비스, 투자초심자 대상)
- 선정: 토스증권(MAU 410~550만, 언론보도) · 미래에셋증권 M-STOCK(360만) · 키움증권 영웅문S#(343만) ·
  삼성증권 mPOP(284만) · 한국투자증권 MTS(265만) — 후4개는 와이즈앱·리테일 2026-01 표본조사 공통 출처
- 화면 수집 폴백 사다리: ① uibowl.io 실패(빈 콘텐츠) → ② Mobbin MCP 실패(결제벽, "requires a paid plan")
  → ③ 앱스토어 공식 스크린샷 성공(iTunes Search/Lookup API, country=kr) — 5개 앱 총 38장 다운로드,
  전량 크기 확인(100KB~900KB, 깨짐 없음) → ④ 블로그 텍스트 리서치 보조 1건(토스증권 매수 주문 시트,
  이미지 미확보라 텍스트 인용으로 대체) → ⑤ 유튜브 프레임 미시도(③으로 충분)
- 산출물: docs/artifacts/03-competitors.md (선정표·유저플로우 4과업·소구점 원장 C1~C19·수집소스기록·
  SELF-CHECK·GATE) + docs/artifacts/images/{toss,miraeasset,kiwoom,samsung,hantu}/*.png|jpg
- SKIPPED(재시도 후에도 미확보, R3): (1) 5개 앱 공통 주문결과/실패 화면(잔고부족·장마감) —
  마케팅 스크린샷 구조적 한계, 4단계에서 PRD 5절 기준 직접 설계 필요 (2) 토스증권 발견 탭 내부화면
- Penpot 도구 미사용. GATE = PASS (G3 충족, 위 2건 SKIPPED 명시)

## stage-4-userflow
- [START] 2026-08-01 유저플로우 작성 시작. 입력=docs/PRD.md + docs/artifacts/02-persona.md(GATE PASS) + docs/artifacts/03-competitors.md(GATE PASS, SKIPPED 2건 인계)
- [소비자 검증] 02·03 모두 사용 가능 판정 — BLOCK/WARN 없음, feedback 파일 신규 생성 안 함
- [결정] 포지션 문장: 01-research.md 브랜드보이스(동네·이웃·가깝고)를 증권 도메인에 이식 — "낯선 종목→내가 가까이서 지켜보는 관심종목"
- [결정] 환원 규칙 4종을 PRD 전 문장(§3·§5·§6)에 적용 — ①결과확인 1건 ②진입점 3건 ③정렬/필터진입점 1건 ④빈상태·로딩·실패 6건(§5 전 문장 소화, 해당없음 2건 포함)
- [결정] 03의 SKIPPED 2건 인계 처리 — (1)주문결과/실패 화면은 New/OrderResult로 직접 설계(성공/실패 2상태) (2)토스 발견 탭 내부 미확보는 경쟁 벤치마크 없이 PRD §6-3 요구요소만으로 New/Discover 독자 설계
- [결정] 화면 11개(P0 4 · P1 7 · P2 0) — 20개 미만이라 P2 컷 불필요, 근거 없는 화면(관심종목 검색/추가 전용 화면 등) 생성 안 함
- [전문가 렌즈 자체 검수] "지우면 끊기는 유저스토리" 질문으로 11개 전부 검증, P2 강등 대상 없음
- [END] 2026-08-01 산출물=docs/artifacts/04-userflow.md 작성 완료. 포지션 문장 1개 · 환원체크리스트 11행 · 화면인벤토리 11개(P0 4/P1 7/P2 0) · 플로우맵 4과업. GATE=PASS.

## stage-4-userflow (v2 · 재실행 확장)
- [START] 2026-08-01 16:00 재실행. 사유=(a) 3단계 SKIPPED 2건의 흡수를 문서에 명시적으로 남길 것 (b) 환원 규칙을 PRD §3·§5뿐 아니라 §4·§6 전 문장으로 확대 적용할 것. 입력=docs/PRD.md + docs/artifacts/02-persona.md(GATE PASS) + docs/artifacts/03-competitors.md(GATE PASS). 참조=01-research.md(포지션 근거 한정)
- [소비자 검증] v1은 02·03을 "문제 없음"으로 통과시켰으나 재검사 결과 양쪽 모두 WARN 발견 → feedback 파일 2개 신규 작성. 02=W1 PRD §4-2 투자스타일 4종 중 '성장형' 대표 페르소나 부재(StyleSetting 선택지는 유지하되 화면 우선순위 미정의) · W2 강명숙 우려사항 후반부가 02 자체 `근거 부족` 표기. 03=W1 주문결과/실패 화면 벤치마크 0개 · W2 토스 발견 탭 내부 미확보 · W3 C4가 텍스트 인용뿐(OrderSheet 시각 레퍼런스 없음). BLOCK 0건, 재실행 요청 안 함
- [결정] 포지션 문장 유지·보강 — 01-research 브랜드보이스(동네·이웃·가깝고·따뜻한) + 소구점 S1 verbatim을 근거로 명시. "낯선 종목 데이터 → 내가 가까이서 지켜보는 관심종목"
- [결정] 환원 규칙 적용 범위 확대: v1은 §3·§5·§6만 훑어 11행이었으나, v2는 §4-1(정보 4종)·§4-2(스타일)·§4-3(테마 2질문)·§4 말미 지시문·§7까지 포함해 **21행**으로 확장. 화면 미생성 5건도 "해당없음+사유"로 처리(빈 행 0)
- [결정] 규칙② 확대 적용 2건을 보수적 기본값으로 채택(R6, 리스크표 R-1/R-2에 기록) — (a) §6-3 "호재/악재와 그로 인해 움직인 종목의 **연결**"의 "연결"을 진입점으로 해석 → New/IssueDetail 생성 (b) §3-1 "관심 종목을 **모아두고**"의 담기 경로 → New/StockSearch 생성(§6 미명시라 P1)
- [결정] 3단계 SKIPPED 2건 흡수 — ① 주문결과/실패는 New/OrderResult(성공) + New/OrderFail(잔고부족·장마감 2 variant)로 **분리 생성**(실패는 사유가 본문이라 성공 레이아웃의 상태 변형으로 안 담김). 근거=PRD §3-3·§5 원문 + 당근 권유체(사용자 비난 금지). ② 토스 발견 탭 내부 미확보는 New/Discover를 PRD §6-3 요소 6개만으로 독자 설계 + 과밀 회피용 전체보기 목적지 4개(ThemeList·ThemeDetail·EarningsCalendar·IssueDetail)로 분산. 04-userflow.md에 전용 인계 표로 명시. **4단계 시점 미해결 SKIPPED=0건**
- [결정] 화면 18개(P0 4 · P1 10 · P2 4) — v1 11개 대비 +7. 20개 미만이라 P2 컷 미적용. 근거 못 댄 화면(계좌개설·포트폴리오·뉴스 전체피드·커뮤니티)은 생성 안 함
- [전문가 렌즈 자체 검수] 18행 전부 "지우면 끊기는 유저스토리" 질문 통과 검사 → 답 못한 4개(ThemeList·EarningsCalendar·Watchlist-Dense·Watchlist-Digest)를 **P2로 강등**
- [결정] 저신뢰 2건 GATE 절에 명시(화면은 생성, GATE 영향 없음) — OrderSheet의 "자산 대비 비중" 아이디어(근거=C4 텍스트 리서치뿐) · Discover 정보 위계(경쟁사 내부 미확보 상태 결정, 5단계 export_shape 재검증 권고)
- [결정] 5·6단계 인계: 저작 순서 P0→P1→P2, 타임박스 부족 시 P2부터 컷. 상태/실패 화면은 shape.clone() 후 덮어쓰기 전제. RecommendReasonChip(추천 이유 한 줄)을 Discover·ThemeDetail 추천 카드의 필수 슬롯으로 컴포넌트 계약에 못박음
- [END] 2026-08-01 16:05:48 산출물=docs/artifacts/04-userflow.md(전면 개정) + docs/artifacts/feedback/02-feedback.md(신규) + docs/artifacts/feedback/03-feedback.md(신규). 포지션 1문장 · 환원체크리스트 21행 · 화면 18개(P0 4/P1 10/P2 4) · 플로우맵 4과업 + 페르소나별 여정 3개 · 리스크표 R-1~R-5. Penpot/use_figma/MCP 미사용. GATE=PASS.
