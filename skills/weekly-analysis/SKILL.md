---
name: sppt-weekly-analysis
version: 0.2.0
description: |
  전주 대비 매출 변화 원인 분석 + 채널 수수료 최적화 컨설팅 워크플로우.
  Use when asked to "주간 분석", "왜 매출이 줄었어", "채널별 원인 분석", "지난 주 비교",
  "매출이익률 분석", "수수료율 변화", "원인 분석해줘".
triggers:
  - 주간 분석
  - 원인 분석
  - 왜 매출이
  - 전주 대비
  - 채널 분포 변화
  - 수수료율 변화
  - 매출이익률 분석
  - weekly analysis
allowed-tools:
  - Bash
  - Read
---

# weekly-analysis

## Data Collection

```bash
sppt report compare --by channel --period 7d --json
sppt report compare --by product --period 7d --top 10 --json
sppt settlement summary --json   # 기본값 30일 — 단기보다 채널 누락 방지
sppt cost list --json             # 없으면 Step 4 skip
```

## Analysis Steps

### Step 1: 채널 수수료율 분석

`settlement summary` 데이터로 아래를 순서대로 수행한다.

1. **채널별 수수료율 랭킹** — 낮은 순 정렬, 표본 < 5건 채널은 신뢰도 낮음 표기
2. **현재 실효수수료율** = Σ(채널_비중 × 채널_수수료율)
3. **믹스 최적화 시뮬레이션** — 수수료 높은 채널 N%p를 낮은 채널로 이동 시 월/연간 절감액 계산
4. **집중 리스크** — 단일 채널 비중 > 70% 시 경고, > 90% 시 위험
5. **전주 대비 변화** (데이터 있을 때만) — 채널믹스 효과 vs 채널내 수수료율 변화로 분해

### Step 2: 채널별 매출 변화 분해

`report compare --by channel` diff 배열에서 직접 읽는다.

- 채널별 매출 증감, volumeEffect / priceEffect 요약
- 채널 점유율 변화 (이전주 비중 → 이번주 비중)

### Step 3: Top Mover 상품 귀속

`report compare --by product` diff 배열 상위 항목을 채널 변화에 귀속시킨다.

- 매출 변화 상위 5개 상품: revenueChange, volumeDiff%, priceDiff%
- 특이값 (신규 등장 또는 완전 소멸 상품) 별도 언급

### Step 4: 공헌이익률 분석 (원가 있을 때만)

`cost list`에 데이터가 있을 때만 수행한다.

- 이번주 vs 지난주 공헌이익률 변화
- 상품 믹스 효과 vs 단가 변화 효과로 분해

## AI Interpretation

위 데이터를 바탕으로 각 변화에 대해:
1. 가능한 원인 (내부/외부 구분)
2. 우선순위 조치 (즉각 / 단기 / 검토)
3. 수수료 최적화: 낮은 채널 확대 방법, 집중 리스크 분산 전략, 절감이 실질적인 금액이 되는 매출 임계값

## Error Handling

- **이번주 매출이 이전주 대비 -80% 이상**: 데이터 수집 공백 가능성 → `sppt order pull` 재실행 안내 후 분석
- **원가 미등록**: Step 4 skip
- **채널 1개만**: 수수료 비교 skip, 단가/볼륨 분석 집중
- **settlement 표본 부족** (채널별 < 5건): 수수료율 신뢰도 낮음 표기, `--days 90` 권장
