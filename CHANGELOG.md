# Changelog

모든 주요 변경사항은 이 파일에 기록됩니다.

형식: [Keep a Changelog](https://keepachangelog.com/ko/1.0.0/)
버전: [Semantic Versioning](https://semver.org/lang/ko/) — v0.x는 breaking changes 허용, v1.0.0부터 하위 호환성 보장

---

## [0.1.10] - 2026-04-22

### Added
- N배송 솔루션 수수료 정산 — 1.5% (주문당 5,000원 상한, VAT 포함)
  - `deliveryAttributeType` (API) + `배송속성` (CSV) 자동 파싱으로 N배송 주문 식별
  - 대상: ARRIVAL_GUARANTEE, SELLER_GUARANTEE, HOPE_SELLER_GUARANTEE
- 브랜드스토어 판매수수료 차등 지원 (2025.06.02~)
  - 스마트스토어: 3% (마케팅 1%) / 브랜드스토어: 4% (마케팅 2%)
- `inflowPath` 기반 판매자마케팅 수수료 자동 판별
  - 네이버 광고, 다나와/에누리, 마케팅링크 경유 시 자동 적용
- 온보딩 완료 후 주문 수집 + 대시보드 자동 실행
- CLI 실행 시 새 버전 업데이트 알림 표시

### Changed
- 정기구독 솔루션 수수료 무료화 반영 (3% → 0%, 2025.06.02~)
- 스토어할인 수수료 차감 적용일 수정 (6/26 → 6/2)
- N배송관 유입수수료 2% 폐지 반영 (2025.06.02~)
- 수수료 계산을 채널 수수료 정책 기반 추정으로 통일 — 이지어드민 '정산금액' 컬럼 의존 제거

### Fixed
- trend platform 설명에서 올리브영 제거 — 미지원 채널
- 정산 discrepancyHints에서 실효성 없는 CLI 플래그 참조 제거, 데이터소스별 한계 안내 추가
- 인코딩 깨짐(U+FFFD) 수정 — setup-skills, settlement 등 3개 파일

---

## [0.1.9] - 2026-04-21

### Added
- 스킬 9종 추가: `cost-calc`, `dashboard`, `order-collect`, `product-sync`, `settlement-report`, `shopport`, `sppt`, `status-check`, `trend-research`
- `sppt setup-skills` — 글로벌 스킬 설치 커맨드 (Claude Code / Codex / OpenCode 자동 등록)
- `sppt order add` — 수동 매출 입력
- `sppt order remove` — 주문 삭제
- 웹 대시보드 (`sppt dashboard`)
  - 정산 페이지: 쿠팡 실정산 API 연동, 스마트스토어 실정산 통합
  - 주문 페이지: 동기화 버튼, 입력 방법 가이드
  - 상품 페이지: 상품/품목 구조 개편
  - 트렌드 페이지: CLI 기록 연동
  - 기간 필터 프리셋 활성화 상태 양방향 연동
- PostgreSQL 데이터 레이어 — operations log, product mapping
- 온채널(onchannel) 소싱 채널 추가
- `product register --all-channels` 전체 채널 동시 등록 지원
- `order route` 소싱 공급사 자동 라우팅
- `status --suppliers` 공급사 신뢰도 스코어 표

### Changed
- 서비스명 전환: rocketsell → shopport (CLI 바이너리: `sppt`)
- 소싱 채널 포지셔닝: 도매꾹 → 도매매 중심으로 변경 (위탁 전용)
- init 온보딩 간소화: 배너·판매자정보·분류규칙·스토어프로파일 통합
- 스마트스토어 주문 조회 24시간 제한 우회 — 일 단위 순회
- CJ Dropshipping USD→KRW 통화 자동 환산

### Fixed
- 대시보드 설정 페이지에서 채널 연결 해제(삭제) 안 되던 버그 수정
- JSON 원자적 쓰기 + 소싱 캐시 TTL 단축
- Cafe24 access token 2시간 만료 자동 갱신
- Apify 트렌드 플랫폼 정리 — 동작 검증된 무신사·화해만 유지

---

## [0.1.0] - 2025-01-01

### Added
- 판매 채널: Cafe24, Coupang, SmartStore, Shopify
- 소싱 채널: 도매매(domeggook), 특가몰(specialoffer), CJ Dropshipping
- Skills: `source-to-register`, `order-fulfill`, `inventory-sync`, `waterfall-source`, `price-monitor`
- `sppt init` 인터랙티브 온보딩 wizard
- Rate limiter (채널별 요청 제한 자동 처리)
- `~/.shopport/config.json` credentials 로컬 저장
