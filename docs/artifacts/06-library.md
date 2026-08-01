# 06 — Penpot 라이브러리 (run-02, 30분 압축 실행)

작업 Page: `3-toss-result` (id `4eb60b9b-410f-42df-8b61-270d8d4ad027`)
Lib 원본 영역: x=1400 컬럼. 토큰은 `figma.variables` 함정 회피를 위해 JS 상수(04 참조)로 일관 적용.

## 컴포넌트 (8종)

| 이름 | componentId | 용도 | 오버라이드 포인트(텍스트 name) |
|---|---|---|---|
| Lib/Chip | e97cef55-e0e7-8094-8008-6a5800b7879d | 조건 좁히기 칩 | Label. 적용중 상태는 fills 블루+Label 흰색 |
| Lib/SectionHeader | e97cef55-e0e7-8094-8008-6a58017e79de | 섹션 헤더 | Title, Action |
| Lib/PrimaryButton | e97cef55-e0e7-8094-8008-6a5801b52e18 | 주 CTA | Label |
| Lib/FareCard | e97cef55-e0e7-8094-8008-6a5801bc6797 | 운임 등급 카드 | FareName, FarePrice, Cond1~3. 선택은 blue stroke 2 |
| Lib/FlightRow | e97cef55-e0e7-8094-8008-6a583ad616bc | 직항 항공편 행 | DepTime, ArrTime, Duration, Airline, Price, DepCode, ArrCode |
| Lib/FlightRowStop | e97cef55-e0e7-8094-8008-6a583ada2c37 | 경유 항공편 행 (타임라인 바 포함) | 위 + StopLabel, StopType |
| Lib/TabBar | e97cef55-e0e7-8094-8008-6a583ade5b73 | 하단 이동 | TabLabel×5 |
| Lib/DealCard | (Lib 영역 y=950) | 특가·마감 카드 | Dest, DealSub, PillLabel, DealPrice. 마감계는 Pill red 오버라이드 |

## 인스턴스 오버라이드 규약
- 텍스트: `storage.setTexts(inst, {name: chars})` — name 기반 매칭
- 색: **penpot 형식** `{fillColor, fillOpacity}` 만 사용 (figma 형식은 인스턴스에서 막힘 — AGENTS 함정표 준수)

## 특이사항
- `figma.variables` 미사용(토큰 잔존 안 됨) → JS 상수 + 컴포넌트로 재사용 점수 확보
- 폰트: Spoqa Han Sans Neo 서버 부재 → Noto Sans KR 확정
