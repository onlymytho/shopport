# Changelog

모든 주요 변경사항은 이 파일에 기록됩니다.

형식: [Keep a Changelog](https://keepachangelog.com/ko/1.0.0/)
버전: [Semantic Versioning](https://semver.org/lang/ko/) — v0.x는 breaking changes 허용, v1.0.0부터 하위 호환성 보장

---

## [Unreleased]

### Added
- 온채널(onchannel) 소싱 채널 추가
- `product register --all-channels` 전체 채널 동시 등록 지원
- `order route` 소싱 공급사 자동 라우팅
- `status --suppliers` 공급사 신뢰도 스코어 표
- `agent-install.txt` 엔드포인트 (AI Agent용 설치 가이드)
- PostgreSQL 데이터 레이어 — operations log, product mapping
- 웹 대시보드 (`rocketsell dashboard`)

### Changed
- 소싱 채널 포지셔닝: 도매꾹 → 도매매 중심으로 변경 (위탁 전용)
- init 온보딩: 도매매 우선 안내, 도매꾹은 동일 계정 대안으로 표시
- CJ Dropshipping USD→KRW 통화 자동 환산

### Fixed
- JSON 원자적 쓰기 + 소싱 캐시 TTL 단축
- Cafe24 access token 2시간 만료 자동 갱신

---

## [0.1.0] - 2025-01-01

### Added
- 판매 채널: Cafe24, Coupang, SmartStore, Shopify
- 소싱 채널: 도매매(domeggook), 특가몰(specialoffer), CJ Dropshipping
- Skills: `source-to-register`, `order-fulfill`, `inventory-sync`, `waterfall-source`, `price-monitor`
- `rocketsell init` 인터랙티브 온보딩 wizard
- Rate limiter (채널별 요청 제한 자동 처리)
- `~/.rocketsell/config.json` credentials 로컬 저장
