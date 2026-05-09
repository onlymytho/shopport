---
name: sppt-product-parse
description: |
  주문 상품명을 SKU/카테고리/제품라인으로 해체하고, 채널옵션→SKU 매핑까지 완료하는 워크플로우.
  Use when asked to "상품명 분석", "SKU별 정리", "미매핑 상품 확인", "매핑 설정".
---

# product-parse

주문 데이터의 상품명 → SKU 카탈로그 구축 → 채널옵션 매핑까지의 전체 워크플로우.

## When to use

- 주문 임포트 후 상품 카탈로그를 구축하고 싶을 때
- 미매핑 상품이 있는지 확인하고 싶을 때
- 채널 옵션별 수량/세트 구성을 설정하고 싶을 때

## Prerequisites

- 주문 데이터가 임포트되어 있어야 함 (`sppt import <file>` 또는 API 동기화)

## Steps

### 1. 상태 확인

```bash
sppt product unmapped
```

미매핑 상품명이 몇 건인지 확인. 전부 매핑되어 있으면 Step 7로 건너뜁니다.

### 2. 분류 규칙 확인

```bash
sppt product rules
```

- **규칙이 있으면**: 내용을 읽고 이해합니다.
- **규칙이 없으면**: 사용자에게 "상품 분류 규칙이 있나요?" 질문 → 없으면 상품명에서 자체 판단.

규칙 파일에 다음이 포함되면 카탈로그 + 매핑 모두 정확해집니다:

1. **카테고리/라인 분류**: 키워드 → 카테고리/라인 매핑
2. **SKU 체계**: SKU 코드 생성 규칙
3. **세트 구성**: 세트 SKU의 단품 구성 (예: `CKRS-SET2 = CKRS-WASH × 1 + CKRS-LOTN × 1`)
4. **수량 옵션 패턴**: 옵션명 → 수량 (예: `"2set" → ×2`)

규칙 저장: Write 툴로 `/tmp/product-rules.md` 작성 → `sppt product rules set /tmp/product-rules.md`

### 3. 상품명 해체

미매핑 상품명을 분류 규칙에 따라 해체합니다.

- **규칙 우선**: `product-rules.md` 규칙을 최우선 적용
- **MECE**: 모든 상품이 하나의 카테고리에 정확히 배정
- **모호한 경우**: 자의적 분류 대신 사용자에게 질문

### 4. 결과 저장

엔트리 배열을 `/tmp/product-catalog-entries.json`에 저장 후 임포트:

```bash
sppt product catalog import /tmp/product-catalog-entries.json
```

```json
[
  {
    "sku": "AB-MC20",
    "brand": "알텐바흐",
    "productLine": "316Ti",
    "category": "냄비",
    "productName": "멀티쿠커 편수",
    "option": "20cm",
    "rawNames": ["[알텐바흐] 316Ti 올인원 멀티쿠커 편수 20cm"],
    "confidence": "llm"
  }
]
```

### 5. 검증

`sppt product catalog`로 저장 확인 → `sppt product unmapped`로 미매핑 0건 확인.

### 6. 채널옵션 → SKU 매핑 (enrichment)

카탈로그 구축 후, 채널 옵션별 SKU 구성(수량, 세트 분해)을 매핑합니다.

```bash
sppt product mapping --json    # pending 매핑 목록 확인 (전부 confirmed면 완료)
```

규칙 파일의 세트 구성/수량 패턴을 참고하여 각 항목의 SKU 구성을 설정:

```bash
sppt product mapping --edit 1 --skus "CKRS-LOTN:2"                                    # 단품 2개
sppt product mapping --edit 2 --skus "CKRS-WASH:3:component,CKRS-LOTN:3:component"    # 세트 × 3
```

설정 후 `sppt product mapping --json`으로 결과 검증 → 사용자 승인 → `sppt product mapping --confirm all`.
`--confirm` 시 경고가 출력되면 즉시 중단하고 사용자에게 보고.

#### 타입 선택 기준

`--skus` 형식: `SKU:수량[:타입[:정가]]` — 타입 생략 시 primary.

| 타입 | 용도 | 매출 배분 |
|------|------|-----------|
| primary | 단품 (구성품 1개일 때) | 매출 100% |
| component | 세트 구성품 (2개 이상) | 정가 비율 or 균등 배분 |
| gift | 사은품 | 매출 0원, 수량만 집계 |
| packaging | 포장재 | 매출 0원, 수량만 집계 |
| sample | 샘플 | 매출 0원, 수량만 집계 |

세트 구성품이 2개 이상이고 정가(`listPrice`)를 설정하면 정가 비율로 매출 배분, 미설정 시 균등(1/N) 배분.

#### 판단 원칙

1. **수량 추출**: "2개"/"2set"/"2ea"/"2개입" → ×2. 단독 숫자는 모호 → 사용자 확인
2. **세트 분해**: 규칙에 구성 정보가 있으면 구성품 SKU로 분해 (타입: component)
3. **포장 옵션**: SKU 구성에 영향 없음 (포장재 SKU가 있으면 packaging 타입으로 추가)
4. **모호한 경우**: 사용자에게 질문 (자의적 판단 금지)
5. **대량 매핑**: 20개 이상이면 패턴별 그룹핑 후 일괄 처리

## Tips

- 규칙 파일은 자연어로 자유롭게 작성. 이 스킬이 읽고 해석합니다.
- 매핑된 상품명은 캐시되어 다음에는 자동 적용됩니다.
- 새 상품 추가 시 `sppt product unmapped`로 확인 후 추가 해체.
