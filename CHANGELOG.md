# Changelog

모든 주요 변경사항은 이 파일에 기록됩니다.

형식: [Keep a Changelog](https://keepachangelog.com/ko/1.0.0/)
버전: [Semantic Versioning](https://semver.org/lang/ko/) — v0.x는 breaking changes 허용, v1.0.0부터 하위 호환성 보장

---

## [Unreleased]

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
