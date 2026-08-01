> **리허설 주석 (실제 런과의 차이)** — 이 문서는 stage-4-design-guide 하네스의 *리허설 실행*이다.
> 1) 입력 PRD는 `docs/PRD.md`(현재 심사용 PRD로 교체된 상태)가 아니라 지시에 따라
>    `docs/examples/daangn-stock.md`("관심종목을 담고 매수까지 가는 경험을 설계해주세요" — 당근→증권)를 PRD로 읽었다.
> 2) 정규 입력 `docs/artifacts/02-existing-assets.md`가 이 레포에 없어, 대체 정본
>    `/Users/cpuxp/Curosr/99.Noonoji_automation/하네스톤-준비/당근-실측.md`(Penpot `1-daangn` 페이지를 손으로 실측한 문서)를
>    "02 실측"과 동일한 지위로 사용했다. 이하 토큰 출처 표의 "02 실측"은 전부 이 파일을 가리킨다.
> 3) 공식 디자인 시스템(①)은 웹서치 + `gh api`로 **`daangn/seed-design`(공식 org repo, Apache-2.0)** 확보에 성공했다 —
>    재시도 불필요. 단, seed-design의 실제 계산된 HEX 값(라이트 테마 CSS 변수 값)은 저장소에서 직접 확인하지 못해
>    (`packages/css/base.css`에는 변수 참조만 있고 테마 주입은 런타임/별도 빌드 산출물),
>    **시멘틱 명명 패턴과 상태(Hover/Pressed/Active) 접미사 규칙만** 참고했다. 색의 실제 숫자값은 전부
>    ② 실측(당근-실측.md) 또는 신규가정에서 왔다 — 충돌 규칙("②가 이기면 이건 이겼는지" 여부) 자체가
>    발생하지 않았다(seed-design에서 값을 가져온 적이 없으므로).
> 4) PRD의 증권 화면(11개 인벤토리로 지시받았으나 실제 `04-userflow.md`에는 P0 4 · P1 10 · P2 4 = **18개** 화면
>    인벤토리가 있었다 — 지시문 숫자와 실제 산출물 숫자가 다르다는 점을 그대로 기록하고, 실제 파일 내용(18개)을 썼다.
>
> 나머지 절차·출력 형식은 `.claude/agents/stage-4-design-guide.md` 그대로 따른다.

# 04 — 디자인 가이드 (리허설: 당근→증권)

## A. 사용자

### 페르소나
근거: `docs/rehearsal-daangn-stock/02-persona.md`(GATE PASS, 3명) 그대로 인용·재구성. 새 인물을 만들지 않았다.

| 이름(역할) | 동기 | 행동 패턴 | 이탈 지점 | 근거(PRD 위치) |
|---|---|---|---|---|
| 정하늘 (신규 진입자·스타일 미정) | 관심종목을 일단 모아두고 등락을 훑고 싶다 / 뭘 살지 몰라 오늘 살 만한 걸 찾고 싶다 | 가입 당일, 투자 경험 0년. 아무것도 담지 않은 채 앱을 열어봄 | 관심종목이 비어 있어 아무것도 시작이 안 됨. 투자 스타일을 골라본 적이 없어 뭘 봐야 할지 모름 | PRD §1 "주식을 처음 시작하는 사람이 대상" · §3 스토리1·4 · §5 "가입 첫날" |
| 이도현 (단기수익형·잦은 접속) | 오늘 많이 움직인 종목을 등락률 기준으로 판단해 바로 매수하고 싶다 | 장중 습관적으로 수십 번 접속, 변동성 높은 자산 선호 | 장 시작 직후 시세 미수신으로 불안. 주문이 잔고부족·장마감으로 실패할 수 있음 | PRD §3 스토리2·3 · §4-2 단기수익형 · §5 "시세 미수신"·"주문 실패" |
| 강명숙 (배당·안정형·관심종목 다량) | 등락률보다 배당수익률·거래량·테마로 판단하고 싶다 | 접속 빈도 낮음(하루 1회 이하), 관심종목 최대 200개 보유 | 200개 규모에서 원하는 종목을 못 찾음. 접속이 뜸해 정보가 최신인지 확신이 안 섬 | PRD §3 스토리2·4 · §4-2 배당형·§4-3 테마 · §5 "많게는 200개"·"하루에 한 번도 안 들어오는 사용자" |

### 핵심 시나리오 / 여정
화면 ID는 `04-userflow.md`가 정의한 `New/...` 프레임명을 그대로 쓴다(이 레포에 `01-screens.md`가 없어 대체).

**정하늘 — 신규·스타일 미정 여정**

| # | 사용자 행동 | 화면(ID) | 필요한 정보 | 이탈 위험 |
|---|---|---|---|---|
| 1 | 가입 후 앱을 처음 연다 | `New/Watchlist-Empty` | "여기서 뭘 해야 하는지" 안내 + 다음 행동 CTA 2개 | 빈 화면만 보고 바로 이탈(PRD §1 "첫 화면에서 이탈") |
| 2 | 스타일을 아직 안 골랐지만 발견 탭을 눌러본다 | `New/Discover-NoStyle` | 스타일 무관 공통 정보(오늘의 이슈·인기 테마) | "나랑 상관없는 정보"로 보이면 이탈 |
| 3 | 설정 진입점을 눌러 취향을 답한다 | `New/StyleSetting` | 정보·스타일·테마 칩, 건너뛰기 가능함을 알리는 문구 | 강제 절차처럼 느끼면 중도 이탈 |
| 4 | 맞춤 추천이 반영된 발견 화면으로 돌아온다 | `New/Discover` | 각 추천 카드의 "왜 떴는지" 한 줄 이유 | 이유 없이 종목만 나오면 신뢰 상실 |
| 5 | 추천 종목을 눌러 상세를 본다 | `New/StockDetail` | 현재가·등락·기본 정보 | 정보 과잉으로 압도됨 |
| 6 | 담기(♡)를 눌러 관심종목에 추가한다(확인창 없음) | `New/Watchlist` | 담긴 즉시 리스트에 반영되는 피드백 | 반영이 안 보이면 "담겼는지" 불안 |

**이도현 — 단기수익형·장중 반복 여정**

| # | 사용자 행동 | 화면(ID) | 필요한 정보 | 이탈 위험 |
|---|---|---|---|---|
| 1 | 장 시작 직후 앱을 연다 | `New/PriceLoading` | "곧 들어와요" 류 안내(에러 아님을 표시) | 빈 화면/에러로 오인해 이탈 |
| 2 | 시세 수신 후 관심종목을 등락률순으로 본다 | `New/Watchlist` (정렬 갱신) | 종목별 현재가·등락률(원화 병기) | 오늘 안 움직인 종목이 위에 있으면 이탈 |
| 3 | 가장 많이 움직인 종목 상세로 들어간다 | `New/StockDetail` | 기간 전환 차트, 하단 고정 매수 버튼 | 매수 진입점을 못 찾으면 이탈 |
| 4 | 하단 고정 버튼으로 주문 시트를 연다 | `New/OrderSheet` | 수량·가격 입력, 예상 금액 | 잔고 초과를 입력 중에 못 알아채면 확정 시점에 당황 |
| 5 | 주문을 확정한다 | `New/OrderResult` 또는 `New/OrderFail` | 성공: 체결 요약 / 실패: 사유(잔고부족·장마감)+다음 행동 | 실패 사유를 모르면 "앱이 고장났다"고 오인, 재시도 안 함 |

**강명숙 — 배당·안정형·200개 보유 여정**

| # | 사용자 행동 | 화면(ID) | 필요한 정보 | 이탈 위험 |
|---|---|---|---|---|
| 1 | 오랜만에 접속한다 | `New/Watchlist-Digest` | "그동안 이런 일이 있었어요" 요약 카드 1장 | 200개 변동을 한꺼번에 봐야 하면 압도됨 |
| 2 | 200개 목록을 스크롤한다 | `New/Watchlist-Dense` | 행 높이·구분선이 스케일에서도 안 무너짐 | 레이아웃이 무너지면 원하는 종목을 못 찾고 이탈 |
| 3 | 정렬을 배당수익률 기준으로 바꾼다 | `New/WatchlistSortFilterSheet` | 정렬 기준 4종(등락률/내 수익률/거래량/배당수익률) | 원하는 기준이 없으면 목록이 여전히 무의미 |
| 4 | 관심 테마(반도체 등)를 눌러 구성 종목을 본다 | `New/ThemeDetail` | 테마 등락 요약 + 구성 종목 리스트 | 테마 안에 뭐가 들었는지 안 보이면 이탈 |

---

## B. 아이덴티티와 원칙

### 형용사 3~5개 + 화면에서의 발현
근거: `01-research.md` 브랜드 보이스(어휘 "동네·이웃·가깝고·따뜻한", 어미 "~해요/~하세요") + PRD §1 "부담 없고 익숙한 경험" + §7 "기존 디자인의 톤앤매너를 유지".

| 형용사 | 화면에서의 발현 |
|---|---|
| **다정한** | 실패·에러 문구를 "안 됐어요"가 아니라 "지금은 이래서 못 담았어요 + 다음 행동"으로 쓴다(`New/OrderFail`). 어미는 전부 `~해요/~하세요`, `~습니다` 격식체 미사용 |
| **부담 없는** | 확인 다이얼로그 없이 즉시 반영되는 담기 토글(`New/StockSearch`, `New/Watchlist`). 첫 CTA는 "매수"가 아니라 "담기" |
| **훑는(스캐너블)** | 종목명·로고를 색·숫자보다 먼저 읽히는 위치에 배치(`New/Watchlist` StockRow). 한 화면의 1차 정보는 3개(종목명·현재가·등락률)를 넘지 않는다 |
| **설명이 붙는** | 추천·연결에는 항상 근거 한 줄이 따라온다(`RecommendReasonChip` — `New/Discover`, `New/ThemeDetail`, `New/IssueDetail`) |
| **또렷한(등락 판단만은 예외적으로 선명하게)** | 다른 정보는 저강조지만 상승/하락 색만은 채도 높게 유지 — "부드러움"이 판단을 흐리면 안 됨 |

### 디자인 원칙 (판정 가능한 문장)
1. **한 리스트 행의 1차 색 강조는 등락 배지 하나뿐이다.** 그 외 텍스트는 `text.primary`/`text.secondary`만 쓴다(임의 강조색 금지).
2. **모든 추천·연결 카드에는 `RecommendReasonChip`(이유 한 줄)이 필수다.** 이유 없는 추천 카드는 저작 완료로 보지 않는다.
3. **오버레이로 명시된 것(주문 시트·정렬 시트·운임 등급 등)은 새 화면으로 만들지 않는다.** `BottomSheet` 컴포넌트 위에서만 완결한다.
4. **에러 상태는 항상 "사유 + 다음 행동" 2요소를 갖는다.** 사유만 있고 행동이 없는 에러 화면은 미완성이다.
5. **터치 타깃은 최소 40×40px, 리스트 행 간 여백은 `space.lg`(16px) 이상이다.** 이보다 좁으면 저작 오류로 간주한다.

---

## C. 토큰 — 저작 단계가 그대로 복사해 쓴다

```js
const TOKENS = {
  color: {
    // raw palette — 02 실측(당근-실측.md §11) 기준. carrot600/red500/blue500/green500/amber500은 실측에 없어 신규가정.
    palette: {
      gray900: "#000000",
      gray700: "#5E5E5E",
      gray500: "#8C8C8C",
      gray300: "#C7C7C7",
      gray200: "#DADADA",
      gray150: "#D9D9D9",
      gray100: "#EEEEEE",
      gray50: "#F6F6F6",
      gray40: "#F4F4F4",
      white: "#FFFFFF",
      carrot500: "#FF7E36",   // 실측 §11, 브랜드 주조색(가격/FAB/CTA)
      carrot100: "#FFEBE0",   // 실측 §11, 오렌지 틴트 배경
      carrot600: "#E86A22",   // 신규가정 — pressed용 진한 오렌지(실측엔 눌림 상태 기록 없음)
      accent500: "#4AC1DB",   // 실측 §11 "용도 미상 포인트 컬러" → info로 역할 재배정(가정)
      red500: "#E5361A",      // 신규가정 — 03-competitors 관찰("상승=빨강") 방향만 근거, 정확 HEX는 신규
      blue500: "#2F6FE4",     // 신규가정 — 03-competitors 관찰("하락=파랑") 방향만 근거, 정확 HEX는 신규
      green500: "#2E9E44",    // 신규가정 — success 표현용, 실측/03에 근거 없음
      amber500: "#F5A623",    // 신규가정 — warning 표현용, 실측/03에 근거 없음
    },
    // semantic — 화면 저작은 이것만 쓴다
    bg: "#FFFFFF",
    surface: "#FFFFFF",
    surfaceSunken: "#F6F6F6",
    border: "#EEEEEE",
    divider: "#EEEEEE",
    text: {
      primary: "#000000",
      secondary: "#8C8C8C",
      tertiary: "#5E5E5E",
      disabled: "#C7C7C7",   // 신규가정 — 실측에 비활성 텍스트 사례 없음, gray300 재사용
    },
    brand: "#FF7E36",
    onBrand: "#FFFFFF",
    brandPressed: "#E86A22", // 신규가정
    brandLow: "#FFEBE0",     // 실측 값 재사용(seed-design의 primaryLow 명명 패턴 참고)
    success: "#2E9E44",      // 신규가정
    danger: "#D93025",       // 신규가정 — stock.rise(#E5361A)와 의도적으로 다른 값(등락 신호와 시스템 에러 신호를 혼동시키지 않기 위함)
    warning: "#F5A623",      // 신규가정
    info: "#4AC1DB",         // 실측 값 재사용(역할 신규 배정)
    // PRD 도메인 의미색 — 필수(등락). 03-competitors.md 관찰: 미래에셋 스파크라인·키움 호가 매수/매도잔량 전부 "상승/매수=빨강, 하락/매도=파랑"(한국 증권앱 관행)
    stock: {
      rise: "#E5361A",  // 상승 — 신규가정(방향은 03 근거, 값은 신규)
      fall: "#2F6FE4",  // 하락 — 신규가정(방향은 03 근거, 값은 신규)
      flat: "#5E5E5E",  // 보합 — 신규가정(PRD/03 미언급, 상승/하락만 두면 0% 등락 시 표현 공백이 생겨 보수적으로 추가)
    },
  },
  // 02 실측 §13 "간격 체계" 반복 수치 전부 사용, 8px 배수 체계
  space: { xs: 4, sm: 8, md: 12, lg: 16, xl: 32, xxl: 40, xxxl: 48 },
  // 02 실측 §13 "radius는 4px(카드형) / 100(pill) 두 값만 사용" — sheetTop은 신규 컴포넌트(바텀시트)용 신규가정
  radius: { sm: 4, pill: 100, sheetTop: 16 },
  font: {
    family: "Inter", // 02 실측 §12 — 당근 실측 5프레임 전체 본문 폰트. 리허설이라 penpot.fonts.all 서버 가용성 확인 불가 → 실측 그대로 채택하고 이 사실을 기록
    display: { size: 32, weight: 700, lineHeight: 1.2 }, // 신규가정 — StockDetail 현재가 초대형 표시. 실측 최대폰트(18px) 초과, 03-competitors "상단 초대형 폰트 현재가" 관찰만 근거
    title:   { size: 18, weight: 700, lineHeight: 1.3 }, // 실측, 헤더 타이틀
    heading: { size: 16, weight: 700, lineHeight: 1.4 }, // 실측, 마이페이지 섹션 타이틀
    body:    { size: 16, weight: 400, lineHeight: 1.5 }, // 실측, 리스트 제목·메뉴 항목
    bodyBold:{ size: 14, weight: 700, lineHeight: 1.4 }, // 실측, 강조 텍스트·CTA 버튼
    price:   { size: 15, weight: 700, lineHeight: 1.3 }, // 실측, 리스트 내 가격(#FF7E36)
    caption: { size: 12, weight: 400, lineHeight: 1.4 }, // 실측, 메타 정보(최빈도 101회)
    tabLabel:{ size: 10, weight: 400, lineHeight: 1.2 }, // 실측, 탭바 라벨
    // 위 전 항목의 lineHeight 값 자체는 실측 미기록(정적 프레임 스캔이라 텍스트 paragraph line-height 속성이 없음) — Inter 표준 근사치로 신규가정
  },
  // 신규가정 — 실측 5프레임 전부 flat(box-shadow 미사용, 헤어라인으로만 구분). 오버레이 시트류 최소 elevation 필요해 업계 표준 근사값 사용
  shadow: { sheet: { color: "#000000", opacity: 0.12, blur: 24, offsetY: -4 } },
  screen: { width: 390, height: 844 }, // 02 실측 §0 — 전 프레임 폭 390 고정, 표준(비스크롤) 프레임 높이 844(iPhone 13/14)
};

// Penpot 형식 fill 헬퍼. figma 형식({type:"SOLID", color:{r,g,b}})은 인스턴스에서 막힌다.
const fill = (hex, opacity = 1) => [{ fillColor: hex, fillOpacity: opacity }];
```

### 토큰 출처

| 토큰 | 값 | 출처 (02 실측 / 03 적용 / 신규) | 비고 |
|---|---|---|---|
| palette.gray900 | #000000 | 02 실측 §11 | 본문 텍스트, 177회 |
| palette.gray700 | #5E5E5E | 02 실측 §11 | 서브 텍스트, 15회 |
| palette.gray500 | #8C8C8C | 02 실측 §11 | 메타 텍스트, 85회 |
| palette.gray300 | #C7C7C7 | 02 실측 §11 | 아이콘 보조색, 6회 |
| palette.gray200 | #DADADA | 02 실측 §11 | 아이콘 보조색, 20회 |
| palette.gray150 | #D9D9D9 | 02 실측 §11 | 이미지 placeholder, 35회 |
| palette.gray100 | #EEEEEE | 02 실측 §11 | 헤어라인/칩 테두리, 29+12회 |
| palette.gray50 | #F6F6F6 | 02 실측 §11 | 옅은 배경, 8회 |
| palette.gray40 | #F4F4F4 | 02 실측 §11 | 아바타 placeholder, 6회 |
| palette.white | #FFFFFF | 02 실측 §11 | 배경, 113회 |
| palette.carrot500 | #FF7E36 | 02 실측 §11 | 브랜드 주조색, 17회 |
| palette.carrot100 | #FFEBE0 | 02 실측 §11 | 오렌지 틴트, 5회 |
| palette.carrot600 | #E86A22 | 신규 | pressed 상태용, 실측에 눌림 상태 기록 없음 |
| palette.accent500 | #4AC1DB | 02 실측 §11(값) + 신규(역할) | 실측엔 "용도 미상"으로 기록됨 — info 역할은 이번 단계 배정 |
| palette.red500 | #E5361A | 03 적용(방향) + 신규(값) | 03-competitors "상승=빨강" 관찰, 정확 HEX는 신규 |
| palette.blue500 | #2F6FE4 | 03 적용(방향) + 신규(값) | 03-competitors "하락=파랑" 관찰, 정확 HEX는 신규 |
| palette.green500 | #2E9E44 | 신규 | success 표현, 실측/03 근거 없음 |
| palette.amber500 | #F5A623 | 신규 | warning 표현, 실측/03 근거 없음 |
| color.bg / surface | #FFFFFF | 02 실측 | 전 프레임 배경 |
| color.surfaceSunken | #F6F6F6 | 02 실측 §11 | "옅은 배경" 역할 |
| color.border / divider | #EEEEEE | 02 실측 §11 | 헤어라인 겸용 |
| text.primary | #000000 | 02 실측 | |
| text.secondary | #8C8C8C | 02 실측 | 최빈 보조 텍스트 |
| text.tertiary | #5E5E5E | 02 실측 | 저빈도 서브 텍스트 |
| text.disabled | #C7C7C7 | 신규 | 실측에 비활성 텍스트 사례 없음(당근 마켓 UI에 disabled 버튼류가 없음), gray300 재사용 |
| brand / onBrand | #FF7E36 / #FFFFFF | 02 실측 | CTA "채팅하기" 배색 그대로 |
| brandPressed | #E86A22 | 신규 | seed-design `primaryHover/Pressed` 명명 패턴 참고, 값은 신규 |
| brandLow | #FFEBE0 | 02 실측(값) + ①명명 참고 | seed-design `primaryLow` 패턴 참고해 역할 명명 |
| success | #2E9E44 | 신규 | |
| danger | #D93025 | 신규 | stock.rise와 의도적으로 다른 값(등락 신호 vs 시스템 에러 신호 구분) |
| warning | #F5A623 | 신규 | |
| info | #4AC1DB | 02 실측(값) + 신규(역할) | |
| stock.rise | #E5361A | 03 적용 + 신규 | PRD 도메인 필수 의미색. 03: 미래에셋 스파크라인·키움 매수잔량 관찰 |
| stock.fall | #2F6FE4 | 03 적용 + 신규 | PRD 도메인 필수 의미색. 03: 키움 매도잔량 관찰 |
| stock.flat | #5E5E5E | 신규 | 보합 표현 공백 방지용 추가 |
| space.xs~xxxl (4/8/12/16/32/40/48) | — | 02 실측 §13 | "8px 배수 체계" 반복 수치 표 그대로 |
| radius.sm | 4 | 02 실측 §13 | 카드형 |
| radius.pill | 100 | 02 실측 §13 | 완전 pill/원형 |
| radius.sheetTop | 16 | 신규 | 실측엔 바텀시트 컴포넌트 자체가 없음(신규 화면 `OrderSheet`/`WatchlistSortFilterSheet` 전용) |
| font.family | Inter | 02 실측 §12 | 서버 가용성은 리허설이라 미확인, 실측 그대로 채택 |
| font.display | 32/700 | 신규 | 실측 최대 폰트(18px) 초과, 03 "초대형 현재가" 관찰만 근거 |
| font.title | 18/700 | 02 실측 §12 | |
| font.heading | 16/700 | 02 실측 §12 | |
| font.body | 16/400 | 02 실측 §12 | |
| font.bodyBold | 14/700 | 02 실측 §12 | |
| font.price | 15/700 | 02 실측 §12 | |
| font.caption | 12/400 | 02 실측 §12 | 최빈도(101회) |
| font.tabLabel | 10/400 | 02 실측 §12 | |
| font.*.lineHeight (전체) | 1.2~1.5 | 신규 | 실측은 정적 프레임 스캔이라 line-height 속성 미기록 |
| shadow.sheet | opacity .12/blur 24 | 신규 | 실측 5프레임 전부 flat(box-shadow 미사용) |
| screen.width | 390 | 02 실측 §0 | 전 프레임 공통 |
| screen.height | 844 | 02 실측 §0 | 표준(비스크롤) 프레임 기준, 스크롤 화면은 세로로 확장 |

---

## D. 컴포넌트 스펙

목록은 02 실측 "재사용 레시피 후보 5개" + `04-userflow.md` 화면별 필수 요소에서 도출했다. 사용 화면은 `New/...` 프레임명.

| 컴포넌트 | 용도 | 크기·패딩 (토큰) | 타이포 | 색 (시맨틱) | 가변 부분 | 사용 화면 | 사용 횟수 |
|---|---|---|---|---|---|---|---|
| `Header` (고정 상단바) | 화면 제목 + 우측 아이콘 그룹 | 높이 52(실측), 좌우 패딩 `space.lg`(16), 아이콘 24×24 간격 `space.lg`(16) | title(18/700) | text.primary / bg | 타이틀 텍스트, 우측 아이콘 개수(0~3) | `Watchlist`·`Watchlist-Empty`·`Watchlist-Dense`·`Watchlist-Digest`·`StockDetail`·`Discover`·`Discover-NoStyle`·`StockSearch`·`ThemeDetail`·`ThemeList`·`IssueDetail`·`EarningsCalendar`·`StyleSetting`·`OrderResult`·`OrderFail`(2 variant) | 16 |
| `StockRow` (리스트 행) | 관심종목/검색결과/테마 구성종목 한 줄 | 행 높이 110(실측 썸네일 기준) 또는 72(경량형), 좌우 마진 `space.lg`(16), 썸네일-텍스트 간격 `space.lg`(16) | body(16/400) 종목명 · caption(12/400) 메타 · price(15/700) 가격 | text.primary / text.secondary / brand(가격은 색 대신 숫자, 등락은 `PriceChangeBadge`로 분리) | 로고 이미지, 종목명, 현재가, `PriceChangeBadge` | `Watchlist`·`Watchlist-Dense`·`Watchlist-Digest`·`StockSearch`·`ThemeDetail` | 5 |
| `PriceChangeBadge` (등락 배지) | 상승/하락/보합을 색+기호로 표시 | 내부 패딩 `space.xs`(4)~`space.sm`(8), radius `sm`(4) | bodyBold(14/700) | `color.stock.rise`/`fall`/`flat` | 등락률 숫자, 방향(▲▼–) | `StockRow`(내부) · `StockDetail` · `ThemeDetail` · `ThemeList` · `IssueDetail` · `Discover` | 6 |
| `PriceChart` (가격 차트) | 기간별 가격 흐름 + 기간 전환 | 높이 200(신규가정), 상단 기간 칩 스트립 높이 `space.xxxl`(48 근사) | caption(12/400) 축 라벨 | 라인색 `color.stock.rise`/`fall`(전일 대비), 배경 `surface` | 기간(1일/1주/1개월/…), 크로스헤어 툴팁 | `StockDetail` | 1 |
| `Button/Primary` | 가장 중요한 1차 행동(매수/매도, 주문 확정, 저장, 적용) | 높이 48(신규가정, 터치타깃 기준), radius `sm`(4), 좌우 패딩 `space.lg`(16) | bodyBold(14/700) | bg=`brand`, text=`onBrand` | 라벨 텍스트, 매수/매도 색 분기 여부 | `StockDetail`·`OrderSheet`·`StyleSetting`·`WatchlistSortFilterSheet`·`Watchlist-Empty`·`OrderFail` | 6 |
| `Button/Secondary` | 2차 행동(보조 CTA) | Primary와 동일 크기, fill 없음, stroke 1px `brand` | bodyBold(14/700) | text=`brand`, border=`brand` | 라벨 텍스트 | `Watchlist-Empty`·`OrderFail` | 2 |
| `FilterChip` (Pill) | 정렬·필터·스타일·테마 선택지 | 높이 32(실측), radius `pill`(100), 좌우 패딩 `space.md`(12), 칩 간 간격 `space.sm`(8) | caption(12/400) | 기본 stroke `border`, 선택 시 fill `brandLow`+text `brand` | 라벨, 선택 여부 | `WatchlistSortFilterSheet`·`StyleSetting`·`ThemeList`·`Discover`(테마 섹션) | 4 |
| `BottomSheet` (오버레이 컨테이너) | 화면 위에 올라오는 시트(주문/정렬 등) | 상단 radius `sheetTop`(16), 그림자 `shadow.sheet`, 내부 패딩 `space.lg`(16) | — | bg=`surface`, handle=`palette.gray200` | 내부 콘텐츠, 높이(내용에 따라 auto) | `OrderSheet`·`WatchlistSortFilterSheet` | 2 |
| `AmountField` (수량·가격 입력) | 주문 수량/가격 직접 입력 | 높이 48(신규가정), radius `sm`(4), 패딩 `space.md`(12) | body(16/400) | 기본 border=`border`, 포커스 border=`brand`, 에러 border=`danger` | 입력값, placeholder, 에러 메시지 | `OrderSheet` | 1 |
| `BottomTabBar` + `FAB` | 전역 하단 내비게이션 | 탭바 58(실측)+세이프에어리어 34, 5분할 78px씩, FAB 48×48 radius `pill` | tabLabel(10/400) | 선택 탭 text/icon=`brand`, 비선택=`text.secondary`, FAB bg=`brand` | 탭 라벨 5개(증권 도메인으로 재라벨링: 홈/관심종목/발견/주문내역/나의증권), 선택 상태 | `Watchlist`·`Watchlist-Empty`·`Watchlist-Dense`·`Watchlist-Digest`·`Discover`·`Discover-NoStyle` | 6 |
| `EmptyState` | 데이터 없음(빈 관심종목, 스타일 미설정) 안내 | 중앙 정렬, 아이콘 64×64(신규가정), CTA 버튼 간격 `space.lg`(16) | heading(16/700) 제목 · body(16/400) 설명 | text.primary / text.secondary | 아이콘, 안내 문구, CTA 1~2개 | `Watchlist-Empty`·`Discover-NoStyle` | 2 |
| `SkeletonRow` (`PriceLoading`) | 시세 미수신 구간의 자리 채움 | `StockRow`와 동일 치수, 내부는 회색 블록 | — | bg=`palette.gray100` pulsing(정적 표현은 `gray100` 고정) | 행 개수(리스트 길이에 맞춤) | `PriceLoading`(Watchlist·StockDetail 공통 적용) | 1개 화면 · 2개 상위 화면에서 재사용 |
| `RecommendReasonChip` | 추천/연결 결과의 근거 한 줄 | 높이 24(신규가정), radius `sm`(4), 패딩 `space.xs`~`space.sm` | caption(12/400) | bg=`surfaceSunken`, text=`text.secondary` | 이유 문구(예: "변동성 선호 · 오늘 3.2% 움직임") | `Discover`·`ThemeDetail`·`IssueDetail` | 3 |

---

## E. 컴포넌트 상태

모바일 전용 제품이라 **Hover는 전 컴포넌트 공통 `N/A — 터치 환경`**이다(이하 표에서 반복 표기하지 않고 컴포넌트별로 1회만 명시).

| 컴포넌트 | Default | Hover | Active | Disabled | Error | Loading |
|---|---|---|---|---|---|---|
| `Button/Primary` | bg=`brand`, text=`onBrand` | N/A — 터치 환경 | bg=`brandPressed` | bg=`palette.gray100`, text=`text.disabled` | N/A — 버튼 자체는 에러 상태를 갖지 않음(에러는 `AmountField`/`BottomSheet` 인라인 메시지로 표현) | 라벨→스피너 아이콘 교체, 탭 입력 차단(`OrderSheet` 주문 확정 시) |
| `Button/Secondary` | border=`brand`, text=`brand` | N/A — 터치 환경 | bg=`brandLow` | border=`palette.gray200`, text=`text.disabled` | N/A — Primary와 동일 사유 | N/A — 보조 액션은 로딩을 유발하지 않음(예: "종목 검색" 이동만) |
| `StockRow` | bg=`surface`, text=`text.primary` | N/A — 터치 환경 | bg=`surfaceSunken`(눌림 피드백) | N/A — 관심종목 행은 항상 탭 가능, 비활성 케이스 없음 | N/A — 행 단위 에러 없음(가격 오류 시 `PriceChangeBadge`가 `stock.flat`으로 대체) | `SkeletonRow`로 전체 대체(자체 로딩 표현 없음) |
| `PriceChangeBadge` | 상승=`stock.rise`, 하락=`stock.fall`, 보합=`stock.flat` | N/A — 정보 표시 전용, 탭 불가 | N/A — 상동 | N/A — 상동 | N/A — 상동 | 회색 placeholder bar(`palette.gray100`), 값 노출 안 함 |
| `FilterChip` | border=`border`, fill 없음, text=`text.primary` | N/A — 터치 환경 | 선택됨: fill=`brandLow`, text=`brand`, border=`brand` | N/A — 칩은 항상 선택 가능(옵션 자체를 숨기는 것으로 처리, 반투명 비활성 없음) | N/A — 정적 옵션 | N/A — 정적 옵션, 로딩 없음 |
| `BottomSheet` | bg=`surface`, handle=`palette.gray200` | N/A — 터치 환경 | N/A — 컨테이너 자체는 상호작용 없음 | N/A | 상단에 인라인 경고(`danger` 텍스트+아이콘, 예: "보유 잔고보다 많아요") + 확정 버튼 Disabled 연동 | 내부 `Button/Primary` Loading으로 위임(시트 자체는 로딩 상태 없음) |
| `AmountField` | border=`border`, text=`text.primary` | N/A — 터치 환경 | 포커스: border=`brand` | N/A — 입력은 항상 가능하도록 설계(비활성 케이스 없음) | border=`danger` + 하단 메시지("잔고보다 많아요" 등) | N/A — 필드 자체 로딩 없음 |
| `BottomTabBar` | 비선택: icon/text=`text.secondary` | N/A — 터치 환경 | 선택 탭: icon/text=`brand` | N/A — 탭은 항상 활성 | N/A | N/A — 내비게이션은 로딩되지 않음 |
| `EmptyState` | 아이콘+텍스트+CTA(내부 Button 상태에 위임) | N/A — 정적 안내 블록 | N/A — 자체 상호작용 없음(내부 CTA가 상태를 가짐) | N/A | N/A | N/A — "데이터 없음"이 이미 확정된 상태라 로딩 개념 자체가 없음 |
| `PriceChart` | 라인=당일 등락에 따라 `stock.rise`/`fall` | N/A — 터치 환경(단, 드래그는 Active로 취급) | 드래그 중: 크로스헤어 라인 + 해당 시점 가격 툴팁 | N/A | 시세 자체가 없는 종목: 빈 차트 + "차트 정보가 없어요" 안내(`PriceLoading`과는 별개 — "일시 지연"이 아니라 "데이터 없음") | `SkeletonRow`형 회색 블록으로 대체 |
| `RecommendReasonChip` | bg=`surfaceSunken`, text=`text.secondary` | N/A — 정보 표시, 탭 불가 | N/A | N/A | N/A | N/A — 상위 카드 로딩에 종속(자체 로딩 없음) |
| `Header` | title=`text.primary`, bg=`bg` | N/A — 터치 환경 | 아이콘 탭 시 원형 하이라이트 `palette.gray100` | N/A — 헤더 아이콘은 항상 활성 | N/A | N/A — 헤더는 즉시 렌더 |

---

## F. 반응형

- **기준 해상도: 390×844** (주 타깃: PRD가 다루는 대상은 모바일 앱 단일 폼팩터이며, 02 실측 §0의 5개 기존 프레임 전부 폭 390 고정, 표준 프레임 높이 844를 그대로 계승. 스크롤이 필요한 화면(`StockDetail`, `Watchlist-Dense` 등)은 세로만 늘어난다 — 02 실측의 당근마켓_2/5도 동일 방식으로 1393/1630까지 늘어난 전례가 있다.)

| 구간 | 폭 | 컬럼 | 좌우 여백 | 변화 규칙 |
|---|---|---|---|---|
| 모바일(기준) | 360~430 | 1 | `space.lg`(16) | 기준 그리드. 모든 컴포넌트 스펙은 이 폭 기준 |
| 태블릿(확장, PRD 미요구) | 768~1024 | 2 | `space.xl`(32) | `StockRow`·카드형 컴포넌트를 2컬럼 그리드로 재배치(신규가정 — PRD·02·03 어디에도 태블릿 요구 없음, 축소가 아니라 확장 방향이라 최소 규칙만 기록) |
| 데스크톱 | 미지원 | — | — | PRD 전체가 모바일 앱 시나리오(주문 하단 고정 액션, 스와이프형 탭바 등)라 데스크톱 화면을 만들 근거 자체가 없음 — 축소·확장 규칙을 만들지 않는다 |

---

## G. 가정 (PRD·02실측·03적용에 답이 없어 정한 것)

| 항목 | 정한 값 | 이유 |
|---|---|---|
| `stock.rise`/`stock.fall` 정확 HEX | rise `#E5361A`, fall `#2F6FE4` | 03-competitors.md가 "빨강=상승/파랑=하락" 방향만 관찰했고 정확 색상값은 없음(마케팅 스크린샷 해상도상 표본 색 추출 불가) — 방향은 근거, 값은 신규 |
| `stock.flat`(보합) | `#5E5E5E`(=`palette.gray700`) | PRD·03 어디에도 0%/보합 표현 요구가 명시되지 않았으나, 상승/하락 2색만 두면 등락률 0%일 때 색 표현이 정의되지 않는 공백이 생겨 보수적으로 추가 |
| `text.disabled` | `#C7C7C7`(=`palette.gray300`) | 02 실측 5프레임(당근 마켓)에는 비활성 버튼/텍스트 UI 자체가 없음 — 기존 아이콘 보조색을 재사용 |
| `danger`를 `stock.rise`와 다른 값으로 분리 | `#D93025` | 상승(긍정적 신호)과 시스템 에러(부정적 신호)가 같은 빨강 계열이면 `New/OrderFail` 화면에서 "이 빨강이 상승인지 에러인지" 혼동 위험 — 의도적으로 채도·톤을 달리함 |
| `success`/`warning` | `#2E9E44` / `#F5A623` | 02 실측·03 관찰 어디에도 근거 없음. seed-design 명명 최소 집합(success/danger/warning/info)을 채우기 위한 신규 정의 |
| `info` 역할을 `accent500`(#4AC1DB)에 배정 | 02 실측 §11에서 "용도 미상"으로 기록된 값을 재사용 | 새 HEX를 늘리기보다 이미 실측된 값의 역할만 확정하는 편이 팔레트 일관성에 유리하다고 판단 |
| `brandPressed`(#E86A22) | 신규 | 02 실측은 정적 프레임 스캔이라 pressed/hover 같은 상호작용 상태를 원천적으로 담지 못함. seed-design의 `primaryPressed` 명명 패턴만 참고해 값은 새로 정함 |
| `radius.sheetTop`(16) | 신규 | 02 실측 5프레임에는 바텀시트 컴포넌트가 존재하지 않음(이번에 PRD가 처음 요구하는 신규 컴포넌트) — 업계 표준 바텀시트 라운드 근사값 |
| `shadow.sheet` | 신규 | 02 실측 5프레임 전부 flat 디자인(구분선만 사용, box-shadow 사용 사례 0건). 오버레이 시트는 배경과 분리되는 최소 elevation이 필요해 신규 정의 |
| `font.display`(32/700, 현재가 초대형 표시) | 신규 | 02 실측의 최대 폰트는 18px(헤더 타이틀)뿐. 03-competitors.md의 "상단 초대형 폰트 현재가"(미래에셋 M-STOCK) 관찰만 근거로 신규 스케일 추가 |
| 모든 `font.*.lineHeight` | 1.2~1.5 근사치 | 02 실측은 Penpot 도형 트리 순회로 얻은 값이라 텍스트의 fontSize/weight/color는 잡히지만 paragraph line-height 속성은 기록되지 않음 — Inter 표준 관행치로 근사 |
| `BottomTabBar` 라벨 5개(홈/관심종목/발견/주문내역/나의증권) | 신규(구조·치수는 02 실측 그대로 승계) | 02 실측 원문 라벨은 마켓 도메인(홈/동네생활/내 근처/채팅/나의당근) — 증권 도메인 의미로 교체하되 5분할·78px·아이콘24+라벨10 구조는 그대로 재사용(PRD §7 "톤앤매너 유지") |
| `PriceChart`/`AmountField`/`Button` 높이(200 / 48 / 48) | 신규 | 02 실측 5프레임에 차트·입력필드·범용 버튼 컴포넌트가 존재하지 않음(당근 마켓 UI에는 이런 폼 요소가 없음) — 터치 타깃 최소 40px 원칙(B절 원칙5) + 일반적 모바일 폼 관행에서 역산 |
| 태블릿 브레이크포인트 규칙 | 신규(2컬럼) | PRD·02·03 전부 모바일 단일 폼팩터만 다룸 — F절 반응형 항목 공란을 남기지 않기 위해 최소 확장 규칙만 기록 |
