# 미디어 자산 원장 (디자인 개선용)

수집일: 2026-08-03. 소스는 전부 Wikimedia Commons(위키피디아 대표 이미지) — URL 안정적, 라이선스 CC/PD 계열로 해커톤 목업 사용에 안전. 로컬 검증(다운로드 + 육안 확인) 완료, 800px 축소본 준비됨.

## 확보 목록 (검증 완료 7 + 제외 1)

| 키 | 대상 슬롯 | 원본 URL | 육안 확인 |
|---|---|---|---|
| dotonbori | New/Home TripCard 커버 (오사카 여정) | https://upload.wikimedia.org/wikipedia/commons/f/f4/Osaka_Dotonbori_Ebisu_Bridge.jpg | 도톤보리 에비스바시, 적합 |
| osaka | 예비 (오사카 대체컷) | https://upload.wikimedia.org/wikipedia/commons/c/ca/Osaka_Castle_03bs3200.jpg | 오사카성+스카이라인, 적합 |
| fukuoka | DealCard 썸네일 (후쿠오카 특가) | https://upload.wikimedia.org/wikipedia/commons/b/bd/Fukuoka_Skyline_of_Seaside_Momochi.jpg | 모모치 해변 스카이라인 |
| taipei | DealCard 썸네일 (타이베이 특가) | https://upload.wikimedia.org/wikipedia/commons/thumb/e/ea/Taipei_Skyline_2022.06.29.jpg/3840px-Taipei_Skyline_2022.06.29.jpg | 타이베이 101 스카이라인 |
| danang | DealCard 썸네일 (다낭 마감임박) + 담아둔 목적지 | https://upload.wikimedia.org/wikipedia/commons/2/2a/Dragon_Bridge%2C_Da_Nang_during_day_-_20230819_%28cropped%29.jpg | 드래곤브리지, 적합 |
| bangkok | DealCard 썸네일 (방콕 마감임박) | https://upload.wikimedia.org/wikipedia/commons/7/7d/4Y1A1159_Bangkok_%2833536795515%29.jpg | 방콕 시내 |
| sapporo | 담아둔 목적지 썸네일 (삿포로) | https://upload.wikimedia.org/wikipedia/commons/5/54/SapporoCity_Skylines2020.jpg | 삿포로 스카이라인 |
| usj | 제외 | (로고 PNG라 사진 아님) | 부적합 — 투어 행은 텍스트 유지 |

로컬 축소본(800px, sips): 세션 스크래치패드 `small_media_*.jpg` (74~275KB).

## 적용 계획 (Penpot 재연결 후 실행)

1. `penpot.uploadMediaUrl(name, url)` (top-level await 가능, AGENTS 확인사항) → 실패 시 `import_image`(로컬 축소본) 폴백.
2. `New/Home`·`New/Home-Empty` 개선:
   - TripCard: `insertChild(0)`으로 318×150 r12 커버(dotonbori). TripCard는 플레인 보드라 구조 변경 안전.
   - DealCard: 인스턴스에 자식 추가 불가 → **Lib/DealCardMedia** 신규 컴포넌트(64×64 r12 썸네일 + 기존 레이아웃) 생성 후 인스턴스 교체(추가 → 검증 → 구 인스턴스 제거).
   - SavedRow: 좌측 40×40 r10 썸네일(danang/sapporo).
3. fills는 penpot 형식 `{ fillOpacity: 1, fillImage: img }` 만 사용.
4. `New/Results` 계열은 의도적으로 이미지 0장 유지 (라이브러리·배리언트 역량이 순수하게 보이는 화면).

## 차단 상태
2026-08-03 현재 Penpot MCP 플러그인 연결 끊김(`Unable to connect`) — 브라우저에서 `작업` 파일을 열고 MCP 플러그인을 다시 실행해야 적용 가능.
