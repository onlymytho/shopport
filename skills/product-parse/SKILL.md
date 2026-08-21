---
name: sppt-product-parse
description: |
  주문 상품명을 CatalogProduct/SourceOffering 기준으로 분류하고, 후보 검토 후 채널옵션 매핑까지 완료하는 워크플로우.
  Use when asked to "상품명 분석", "SKU별 정리", "미매핑 상품 확인", "매핑 설정".
---

# product-parse

주문 데이터의 상품명 → SourceOffering → CatalogProduct 분류 미리보기 → 후보 검토·적용까지의
워크플로우. 클라우드 카탈로그의 권위는 현대 CatalogProduct API에 있으며, 레거시
`ProductCatalogEntry` 파일 임포트나 `/api/sync/products` 쓰기는 사용하지 않습니다.

## When to use

- 주문 임포트 후 미매핑 SourceOffering을 확인하고 싶을 때
- 상품명을 CatalogProduct 후보로 분류하고 싶을 때
- 후보를 검토하고 승인된 분류를 적용하고 싶을 때
- 채널 옵션의 현재 CatalogProduct 매핑을 확인하고 싶을 때

## Prerequisites

- 주문 데이터가 임포트되어 있어야 함 (`sppt import <file>` 또는 API 동기화)
- 클라우드 인증이 되어 있어야 함 (`sppt auth login` 또는 agent token)

## Steps

### 1. 미매핑 상태 확인

```bash
sppt product unmapped
```

미매핑 SourceOffering의 채널, 상품명, 옵션명, raw SKU를 확인합니다. 전부 매핑되어 있으면
분류 실행을 만들지 않고 종료합니다.

### 2. 분류 기준 확인

로컬 규칙 파일이 있으면 먼저 읽습니다.

```bash
sppt product rules --local
```

규칙 파일에 다음이 포함되면 후보 검토가 정확해집니다.

1. 카테고리·제품라인 분류 기준
2. 상품명 정규화·동의어 규칙
3. 세트 구성과 수량 옵션 패턴
4. raw SKU와 기존 CatalogProduct option의 대응 규칙

분류 실행 자체는 서버의 CatalogProduct/SourceOffering 규칙과 revision 검증을 따릅니다.

### 3. 분류 미리보기 생성

```bash
sppt product parse
```

출력된 실행 ID를 기록합니다. 이 명령은 CatalogProduct를 즉시 만들거나 기존 매핑을 바꾸지
않고, 미매핑 SourceOffering을 대상으로 `preview` classification run만 생성합니다.

### 4. 후보 확인

```bash
sppt product parse status <runId> --json
```

각 결과의 source 상품명·옵션·raw SKU와 후보의 `candidateKind`, 제안 상품명, 대상 option,
신뢰도를 확인합니다.

- `existing_option`: 기존 CatalogProduct option으로 연결할 후보
- `new_catalog_product`: 새 CatalogProduct로 만들 후보
- `pending` 후보만 검토 대상입니다.
- 이름·SKU·옵션이 모호하면 자의적으로 승인하지 말고 사용자에게 질문합니다.

### 5. 후보 검토

확실한 후보만 승인하고, 잘못된 후보는 거절하거나 판단을 보류합니다.

```bash
sppt product parse accept <runId> <candidateId>
sppt product parse reject <runId> <candidateId> --reason "상품명이 다름"
sppt product parse defer <runId> <candidateId> --reason "사용자 확인 필요"
```

하나의 결과에서 후보를 승인하면 다른 후보는 서버가 자동으로 거절합니다.

### 6. 승인 후보 적용

사용자에게 승인 대상과 결과를 보여주고 확인받은 뒤 실행합니다.

```bash
sppt product parse apply <runId>
```

적용은 CatalogProduct 생성·수정과 SourceOffering 매핑을 현대 API의 멱등성·mapping revision
검증 아래 처리합니다. 적용 후 다음으로 재확인합니다.

```bash
sppt product unmapped
sppt product catalog
```

### 7. 채널옵션 매핑 확인

세트 구성, 수량, BOM, 매출 배분처럼 상품 option 구성이 필요한 경우에는 Product Catalog
대시보드의 계보·매핑 화면에서 확인·수정합니다. 레거시 `sppt product mapping --confirm`으로
현대 CatalogProduct 매핑을 확정하지 않습니다.

## 로컬 호환 임포트

실험용 로컬 카탈로그 파일을 검증하는 경우에만 아래 명령을 사용할 수 있습니다.

```bash
sppt product parse --local --file mapping.csv
```

이 경로는 `~/.shopport` 로컬 파일만 변경하며 클라우드 CatalogProduct를 변경하지 않습니다.
클라우드 상품 분류에는 사용하지 않습니다.

## 판단 원칙

1. raw SKU가 있으면 이름보다 SKU·option identity를 우선 확인합니다.
2. "2개", "2set", "2ea", "2개입"은 수량 후보로 해석하되 세트 구성은 규칙과 함께 확인합니다.
3. 기존 상품과 신규 상품을 구분할 근거가 없으면 승인하지 않습니다.
4. 후보 적용 전 반드시 사용자에게 승인 목록을 보여줍니다.
5. 적용 후 미매핑이 남으면 남은 SourceOffering만 대상으로 다시 실행합니다.
