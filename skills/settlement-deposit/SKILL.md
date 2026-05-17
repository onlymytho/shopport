---
name: sppt-settlement-deposit
description: |
  통장 입금 내역 CSV를 임포트해서 정산 대사를 완성하는 워크플로우.
  Use when asked to "입금 내역 넣어줘", "통장 CSV 임포트", "실입금 확인", "정산 대사".
---

# settlement-deposit

은행 CSV 파일을 DB에 올려서 입금 예정액 vs 실입금을 자동 대조하는 워크플로우.

## When to use

- "통장 입금 내역 넣어줘", "은행 CSV 임포트해줘"
- "실입금이랑 정산 예정액 비교하고 싶어"
- 대시보드 정산 페이지에서 실입금 컬럼이 비어있을 때

## Prerequisites

- 주문 동기화 완료 (`/sppt-order-collect` 먼저 실행)
- 은행 CSV 파일 준비 (KB·신한·우리·하나 자동 감지)

## Steps

### 1. 파일 확인

사용자에게 CSV 파일 경로를 물어봅니다:

> 입금 내역 CSV 파일 경로를 알려주세요. (예: ~/Downloads/kb_2026_05.csv)

파일 경로를 받으면 진행합니다.

### 2. 미리보기 (dry-run)

실제 임포트 전에 어떤 데이터가 들어오는지 확인합니다:

```bash
sppt settlement deposit <file> --dry-run
```

미리보기 결과를 사용자에게 보여주고 확인을 받습니다:

> N건의 입금 내역이 감지됐습니다. 임포트할까요?

### 3. 날짜 범위 필터 (선택)

사용자가 특정 기간만 넣고 싶다면:

```bash
sppt settlement deposit <file> --from 2026-05-01 --to 2026-06-01 --dry-run
```

범위 필터 없이 전체를 넣어도 됩니다.

### 4. 임포트 실행

확인 후 실제로 저장합니다:

```bash
sppt settlement deposit <file>
```

### 5. 결과 안내

임포트 완료 후:

> N건의 입금 내역을 저장했습니다.
> 대시보드 정산 페이지 → 카페24 → '정산 대사' 패널에서 실입금과 오차를 확인하세요.

### 6. 잘못 넣은 항목 삭제 (선택)

실수로 넣은 항목이 있으면:

```bash
sppt settlement deposit delete --date 2026-05-06 --provider 카페24페이먼
```

## Error handling

- **파일 형식 미지원**: 지원 은행(KB·신한·우리·하나) 확인 후 재시도 안내
- **중복 데이터**: CLI가 자동으로 중복 체크하므로 걱정 불필요
- **날짜 파싱 오류**: `--from`/`--to` 형식 `YYYY-MM-DD` 확인

## Notes

- 같은 파일을 다시 올려도 중복 저장 안 됨
- 임포트 후 대시보드를 새로고침하면 실입금·오차 컬럼이 채워짐
- 매월 말 정산 마감 후 CSV를 받아서 넣는 루틴으로 활용하면 효과적
