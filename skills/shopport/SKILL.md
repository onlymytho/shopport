---
name: shopport
description: |
  shopport CLI 도움말 + 사용 가능한 스킬 목록. /sppt와 동일한 역할.
  Use when asked to "shopport", "shopport 도움말", "shopport 뭐야".
---

# shopport

> `/sppt` 스킬과 동일합니다. shopport CLI 도움말 + 스킬 목록을 보려면 아래를 실행하세요.

먼저 CLI 도움말을 확인합니다:

```bash
sppt --help
```

## 사용 가능한 스킬

| 스킬 | 설명 |
|---|---|
| `/sppt` | shopport 전체 도움말 + 스킬 목록 |
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
| `/sppt-settlement-deposit` | 통장 입금 CSV 임포트 → 정산 대사 |
| `/sppt-policy-ask` | PG사·판매채널 수수료·정산 정책 Q&A |
| `/sppt-cost-calc` | 원가/마진/수수료 계산 |
| `/sppt-cost-register` | SKU 원가 등록·관리 |
| `/sppt-trend-research` | 트렌드 키워드 리서치 |
| `/sppt-dashboard` | 운영 대시보드 띄우기 |
| `/sppt-status-check` | 전채널 연결 + 건강 체크 |
| `/sppt-update` | CLI 버전 업데이트 + 릴리즈 노트 확인 |

## Quick Start

```bash
sppt init              # 채널 온보딩
sppt status            # 연결 상태 확인
sppt context           # AI용 스토어 브리핑
sppt order list cafe24 # 주문 조회
sppt order exclude <id> --reason "테스트"  # 매출 집계에서 제외
sppt order excluded    # 제외된 주문 목록
sppt report --by category --explain  # 수량 기여 분해
sppt analytics cafe24 visitors --period 7d # 카페24 방문자수 조회
sppt analytics cafe24 visitpaths --type domains --period 7d # 카페24 유입경로 조회
sppt product mapping --confirmed     # 확정 SKU 매핑 조회
sppt crm cafe24 groups                # cafe24 회원등급 조회
sppt crm cafe24 customers --limit 30  # cafe24 회원목록 조회
sppt coupon list cafe24 --name "재구매" # 카페24 쿠폰 목록 + 다운로드 URL
sppt coupon issues cafe24 <coupon_no> # 쿠폰 발급내역 조회
sppt coupon revoke cafe24 <member_id> <coupon_no> --confirm # 회원 쿠폰 회수
sppt coupon link cafe24 <coupon_no> --domain birthtii.com # 쿠폰 다운로드 URL 조립
sppt settlement deposit <file.csv>   # 은행 입금 내역 CSV 임포트 (KB·신한·우리·하나)
sppt settlement deposit delete --date YYYY-MM-DD --provider <입금처>  # 입금 내역 삭제
```

## 쿠폰

```bash
sppt coupon list cafe24 [--name <검색어>] [--json]
sppt coupon get cafe24 <coupon_no> [--json]
sppt coupon issues cafe24 <coupon_no> [--member <member_id>] [--order-status <status>] [--json]
sppt coupon revoke cafe24 <member_id> <coupon_no> [--confirm]
sppt coupon link cafe24 <coupon_no[,coupon_no...]> [--domain <domain>] [--copy]
sppt coupon create <channel> --name <name> --type <type> --value <N>
sppt coupon issue <channel> <couponId> --members <id1,id2,...>
sppt coupon support
```

## CRM

```bash
sppt crm cafe24 groups [--json]
sppt crm cafe24 customers [--limit N] [--group-no <no>] [--json]
sppt crm cafe24 customer <member_id> [--json]
sppt crm cafe24 group assign <member_id> --group-no <no> [--fixed] [--dry-run]
sppt crm balance
sppt crm sms send --content "본문" --to <phone>
```

## 분석/리포트

```bash
sppt analytics cafe24 visitors [--period 7d] [--device total] [--format day] [--json]
sppt analytics cafe24 visitpaths [--type domains|ads|keywords] [--period 7d] [--json]
```

판매 채널: cafe24, coupang, smartstore, shopify, toss-shopping, 29cm, musinsa
소싱 채널: domemae, domeggook, specialoffer, cjdropshipping, 1688
물류 채널: dada-erp, ezadmin, onewms, sabangnet, boxhero, ecount-erp, douzone-wehago, playauto, argo, poomgo
