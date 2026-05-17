---
name: sppt
description: |
  shopport CLI 도움말 + 사용 가능한 스킬 목록.
  Use when asked to "sppt", "shopport 뭐 할 수 있어?", "스킬 목록", "커맨드 알려줘".
---

# shopport (sppt)

> AI를 위한 이커머스 운영도구 — 사장님이 뽑으려던 운영 담당자, shopport가 대신합니다.

## Quick Start

```bash
sppt --help          # CLI 전체 도움말
sppt status          # 채널 연결 상태 확인
sppt context         # AI용 스토어 브리핑
```

## 사용 가능한 스킬

| 스킬 | 설명 |
|---|---|
| `/sppt-daily-report` | 매일 아침 매출/주문 브리핑 |
| `/sppt-order-collect` | 멀티채널 주문 한번에 모아보기 |
| `/sppt-order-fulfill` | 주문 확인 → 소싱 발주 → 송장 등록 |
| `/sppt-source-to-register` | 소싱 검색 → 마진 계산 → 판매 채널 등록 |
| `/sppt-waterfall-source` | 3개 소싱채널 워터폴 가격 비교 |
| `/sppt-product-sync` | 소싱→판매 배치 상품 동기화 |
| `/sppt-product-parse` | 상품명 → SKU/카테고리/제품라인 해체 |
| `/sppt-inventory-sync` | 멀티채널 재고 분배 + 동기화 |
| `/sppt-price-monitor` | 소싱가 변동 감지 → 마진 체크 → 판매가 조정 |
| `/sppt-weekly-analysis` | 전주 대비 매출 변화 원인 분석 + 채널 수수료 최적화 |
| `/sppt-settlement-report` | 정산 현황 조회 + 수익 요약 |
| `/sppt-policy-ask` | PG사·판매채널 수수료·정산 정책 Q&A |
| `/sppt-cost-calc` | 원가/마진/수수료 계산 |
| `/sppt-cost-register` | SKU 원가 등록·관리 |
| `/sppt-trend-research` | 트렌드 키워드 리서치 |
| `/sppt-dashboard` | 운영 대시보드 띄우기 |
| `/sppt-status-check` | 전채널 연결 + 건강 체크 |
| `/sppt-update` | CLI 버전 업데이트 + 릴리즈 노트 확인 |

## CLI 커맨드 요약

### 설정
- `sppt init [channel]` — 채널 온보딩 (`--json`, `--local`, `--terminal`)
  - 비대화형 채널 연결 (AI 에이전트용 — 플래그로 크리덴셜 전달 시 프롬프트 없이 바로 저장+테스트):
    - `sppt init smartstore --client-id <id> --client-secret <secret>`
    - `sppt init coupang --vendor-id <id> --access-key <key> --secret-key <secret> [--vendor-user-id <id>]`
    - `sppt init shopify --store-domain <domain> --access-token <token> [--api-version <ver>]`
    - `sppt init toss-shopping --client-id <id> --client-secret <secret>`
    - `sppt init 29cm --client-id <id> --client-secret <secret> --partner-key <key>`
    - `sppt init musinsa --api-key <key> --api-secret <secret>`
- `sppt auth login` — 브라우저 OAuth 로그인
- `sppt auth logout` — 세션 초기화
- `sppt auth whoami` — 현재 로그인 계정 확인
- `sppt auth cafe24 --mall <id> --client-id <id> --client-secret <secret>` — Cafe24 토큰 재발급
- `sppt config list|get|set|remove` — 설정 관리
- `sppt setup-skills` — Claude Code 스킬 설치 (`uninstall`로 제거)

### 상태/모니터링
- `sppt status` / `sppt health` — 채널 연결 헬스체크 (`--suppliers`, `--check`)
- `sppt context [--json]` — AI용 스토어 브리핑
- `sppt dashboard` / `sppt dash` — 웹 대시보드 (`--local`)
- `sppt cap [channel]` — 채널별 API 기능 지원 현황
- `sppt budget [set <channel> <limit>]` — API 사용량/제한 관리
- `sppt log [--days N]` — 작업 이력

### 상품
- `sppt product list <channel> [--limit N] [--search <term>]` — 상품 목록
- `sppt product list [channel] --from-orders` — 주문 기반 크로스채널 상품 조회
- `sppt product get <channel> <productId>` — 상품 상세
- `sppt product register <channel> --file <path>` — 상품 등록 (`--all-channels`로 전채널 동시 등록)
- `sppt product update <channel> <productId> --file <path>` — 상품 수정
- `sppt product status <channel> <productId>` — 상품 상태 확인
- `sppt product delete <channel> <productId>` — 상품 삭제
- `sppt product sync <from> <to> <id> [--markup N]` — 단건 채널 간 복사
- `sppt product sync-batch <from> <to> [--ids ...] [--all] [--markup N]` — 배치 복사
- `sppt product keyword get|set|support` — 검색 키워드/태그 관리
- `sppt product seo get|set|support` — SEO 메타데이터 관리
- `sppt product image get|set|add|delete|support` — 이미지 관리
- `sppt product catalog [--local]` — 상품 카탈로그 브라우징
- `sppt product unmapped [--local]` — 미매핑 상품 목록
- `sppt product rules [set <file>] [--local]` — 분류 규칙 조회/설정
- `sppt product parse [--file <mapping.csv>]` — 매핑 파일에서 카탈로그 파싱
- `sppt product mapping [--edit N] [--confirm N|all] [--confirmed] [--json]` — SKU 매핑 관리
- `sppt product candidates` — 매핑 후보 조회

### 주문/송장
- `sppt order list|get|confirm|cancel|return <channel>` — 주문 관리
- `sppt order add <channel> --product "name" --price N [--qty N] [--date DATE]` — 수동 주문 추가
- `sppt order remove <orderId>` — 수동 주문 삭제
- `sppt order delete [--ids ...] [--channel <ch>] [--from DATE] [--to DATE]` — 벌크 삭제
- `sppt order exclude <channelOrderId> [--reason "사유"]` — 매출 집계에서 제외
- `sppt order include <channelOrderId>` — 제외 해제
- `sppt order excluded` — 제외된 주문 목록
- `sppt invoice register <channel> <orderId> <carrier> <tracking>` — 송장 등록

### 재고
- `sppt inventory sync <channel> <productId> <qty> [--delta] [--location <id>]` — 재고 수량 업데이트
- `sppt inventory plan <totalStock> [sku] [--buffer N] [--ratio ...]` — 채널별 재고 분배 계획
- `sppt inventory bundle add <bundleSku> <componentSku> --qty N` — 세트 구성 등록
- `sppt inventory bundle list [bundleSku]` — 세트 구성 조회
- `sppt inventory bundle remove <bundleSku> <componentSku>` — 세트 구성 삭제

### 소싱
- `sppt source search <channel> <keyword> [--page N]` — 소싱 상품 검색
- `sppt source search <channel> <keyword> --waterfall [--strategy first|best]` — 멀티채널 워터폴 검색
- `sppt source detail <channel> <productId> [--market-check]` — 소싱 상품 상세
- `sppt source register <channel> <productId> <sellingChannel> [--markup N] [--dry-run]` — 소싱→판매채널 등록
- `sppt source market-check <keyword> [--price N]` — 시장 경쟁도 체크
- `sppt source verify-supplier <channel> <productId>` — 공급사 신뢰도 검증
- `sppt source categories <channel>` — 소싱 채널 카테고리 탐색
- `sppt source history [--tier 🟢|🟡|🔴] [--channel C] [--limit N]` — 소싱 이력
- `sppt source route <orderId> --keyword <kw> [--qty N] [--strategy first|best]` — 최적 공급사 라우팅
- `sppt source score record <sourceChannel> --on-time [--return]` — 공급사 성과 기록

### 정산
- `sppt settlement calc [--channel C] [--from DATE] [--to DATE]` — 정산 계산
- `sppt settlement list [channel] [--days N] [--status S]` — 정산 목록
- `sppt settlement summary [channel] [--days N] [--from DATE] [--to DATE]` — 채널별 정산 합계
- `sppt settlement fee list|set|remove` — 수수료 규칙 관리
- `sppt settlement fee-check <channel> [--annual-revenue N] [--payment METHOD]` — 적용 수수료율 확인
- `sppt settlement hints <channel>` — 정산 불일치 사유 조회
- `sppt settlement refund <channel> <orderId> --amount N --commission N --pg N` — 환불 공제 계산
- `sppt settlement schedule [--from DATE] [--to DATE]` — 예정 정산일 조회
- `sppt settlement detail [--from DATE] [--to DATE] [--provider NAME]` — 입금 상세 내역
- `sppt settlement reconcile <channel>` — 정산 대사
- `sppt settlement deposit <file> [--bank ...] [--dry-run]` — 은행 입금 CSV 임포트 (KB·신한·우리·하나)
- `sppt settlement deposit delete --date <날짜> --provider <입금처>` — 입금 내역 삭제
- `sppt settlement pg-compare <channel> --amount N [--annual-revenue N]` — PG 수수료율 비교
- `sppt settlement policy search <term> [--channel C] [--type T]` — 정산 정책 검색

### 원가
- `sppt cost set <skuId> --purchase <N> [--shipping N] [--packaging N] [--fulfillment N]` — 원가 설정
- `sppt cost list [--channel C]` — 원가 목록
- `sppt cost show <skuId> [--channel C]` — 원가 상세
- `sppt cost remove <skuId> [--channel C]` — 원가 삭제

### 가격/쿠폰
- `sppt price set <channel> <productId> <price> [--sale-price N]` — 가격 변경
- `sppt price variant shopify <variantId> <price> [--compare-at N]` — Shopify 옵션가 변경
- `sppt price support` — 채널별 가격 기능 지원 현황
- `sppt coupon create <channel> --name <name> --type <type> --value <N>` — 쿠폰 생성
- `sppt coupon issue <channel> <couponId> --members <id1,id2,...>` — 쿠폰 발급
- `sppt coupon support` — 채널별 쿠폰 기능 지원 현황

### 분석/리포트
- `sppt trend keyword <keyword> [--volume] [--trend] [--months N]` — 검색량 + 트렌드
- `sppt trend compare <kw1> <kw2> [kw3...]` — 키워드 비교
- `sppt trend platform <platform> [--category C] [--limit N]` — 전문몰 랭킹
- `sppt trend coverage [--min-volume N]` — 카탈로그 키워드 커버리지 분석
- `sppt report revenue [--period <period>]` — 크로스채널 매출 요약
- `sppt report --by <category|productline|product|channel|sku>` — 다차원 매출 리포트
  - `--explain` — SKU별 수량 기여 분해 (직접주문 vs 세트전개)
  - `--include-excluded` — 제외된 주문도 포함하여 집계
- `sppt report compare --by <dim> --period <period>` — 기간 대비 비교 분석
- `sppt report daily [--period N] [--top N]` — 일간 리포트
- `sppt report weekly [--period N] [--top N]` — 주간 리포트

### 물류
- `sppt logistics status` — 물류 채널 연결 상태
- `sppt logistics orders <channel> [--since DATE] [--status STATUS]` — 물류 주문 조회
- `sppt logistics ship <channel> <orderId> <carrierCode> <trackingNumber>` — 출고 등록
- `sppt logistics bulk <channel> --file shipments.json` — 벌크 출고 등록
- `sppt logistics stock <channel> [--sku SKU1,SKU2]` — 재고 수량 조회

### 유틸리티
- `sppt import <file>` — CSV/엑셀 임포트
- `sppt migrate cloud` — 로컬 데이터 → 클라우드 DB 마이그레이션
- `sppt issue "title" [--body "description"]` — GitHub Issue 버그 리포트 생성

## 판매 채널
cafe24, coupang, smartstore, shopify, toss-shopping, 29cm, musinsa

## 소싱 채널
domemae (위탁), domeggook (사입), specialoffer, cjdropshipping, 1688

## 물류 채널
dada-erp, ezadmin, onewms, sabangnet, boxhero, ecount-erp, douzone-wehago, playauto, argo, poomgo
