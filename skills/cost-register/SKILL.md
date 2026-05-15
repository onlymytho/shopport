---
name: sppt-cost-register
description: |
  SKU 원가 등록·관리 워크플로우.
  Use when asked to "원가 등록", "원가 입력", "매입단가 설정", "변동비 등록", "원가 일괄 등록".
---

# cost-register

SKU별 원가(매입단가·택배비·포장재비·풀필먼트비)를 등록하고 관리하는 워크플로우.
등록된 원가는 클라우드에 자동 동기화되어 대시보드 마진율 계산에 반영됩니다.

## When to use

- "원가 등록해줘", "매입단가 입력하고 싶어"
- "마진율이 안 나와" → 원가 미등록 상태일 때
- 대시보드에서 "원가 등록 필요" 안내를 본 경우
- 소싱 완료 후 원가 확정, 기존 상품 원가 업데이트

## Steps

### 1. 현재 등록 상태 확인

```bash
sppt cost list
```

등록된 원가가 없으면 "저장된 원가 정보가 없습니다." 출력.

### 2. 등록할 SKU 파악

주문 데이터에서 활성 SKU 목록을 뽑아 원가 미등록 건을 확인:

```bash
sppt report --by sku
```

이 목록과 `sppt cost list` 결과를 대조하여 미등록 SKU를 식별합니다.

### 3. 단건 등록

```bash
sppt cost set <skuId> --purchase <매입단가>
```

**전체 옵션:**

| 옵션 | 설명 | 예시 |
|---|---|---|
| `--purchase <원>` | 매입단가 (필수) | `--purchase 5000` |
| `--shipping <원>` | 건당 택배비 | `--shipping 3000` |
| `--packaging <원>` | 포장재비 | `--packaging 500` |
| `--fulfillment <원>` | 풀필먼트 출고비 | `--fulfillment 1200` |
| `--name <상품명>` | SKU 이름 | `--name "기본 티셔츠"` |
| `--channel <채널>` | 채널 한정 원가 (없으면 공통) | `--channel coupang` |
| `--note <메모>` | 비고 | `--note "23년 2차 계약단가"` |

**예시:**
```bash
sppt cost set SKU-001 --purchase 5000 --shipping 3000 --packaging 500 --name "기본 티셔츠"
```

→ 총 변동비 = 매입 5,000 + 택배 3,000 + 포장 500 = **8,500원**

### 4. 일괄 등록

여러 SKU를 연속으로 등록할 때는 사용자에게 SKU별 매입단가를 확인하면서 순차 등록합니다:

```bash
# SKU 목록 확인 후 하나씩
sppt cost set SKU-001 --purchase 5000 --shipping 3000
sppt cost set SKU-002 --purchase 8500 --shipping 3000
sppt cost set SKU-003 --purchase 12000 --shipping 0 --fulfillment 1200
```

채널별로 원가가 다른 경우 `--channel`로 분리 등록:
```bash
sppt cost set SKU-001 --purchase 5000 --channel coupang
sppt cost set SKU-001 --purchase 4500 --channel smartstore
```

### 5. 등록 확인

```bash
sppt cost list
sppt cost show <skuId>
```

### 6. 대시보드 반영 확인

원가 등록 후 대시보드에서 마진율이 표시되는지 확인:

```bash
sppt dashboard
```

정산 페이지의 "원가 · 마진" 카드에 마진율과 원가 커버리지(%)가 표시됩니다.

## 원가 수정·삭제

```bash
# 수정 — 같은 SKU로 다시 set하면 덮어쓰기
sppt cost set SKU-001 --purchase 5500 --shipping 3000

# 삭제
sppt cost remove SKU-001
sppt cost remove SKU-001 --channel coupang   # 채널 한정 원가만 삭제
```

## Notes

- 등록 즉시 클라우드 DB에 자동 동기화 (로그인 상태일 때)
- `--channel` 미지정 시 "공통" 원가로 저장 — 모든 채널에 적용
- 채널별 원가가 있으면 공통보다 우선 적용
- 총 변동비 = 매입단가 + 택배비 + 포장재비 + 풀필먼트비 + 기타비용
- 마진율 = (실수령 - 총변동비) / 실수령 × 100
