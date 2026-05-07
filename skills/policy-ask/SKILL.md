---
name: sppt-policy-ask
description: |
  PG사·판매채널 정산·수수료 정책 Q&A.
  Use when asked about "수수료율", "정산 주기", "지급보류", "D+", "영세 중소",
  "토스페이먼츠", "카페24페이먼츠", "이니시스", "정책", "언제 정산돼".
---

# policy-ask

PG사·판매채널 정책 DB를 검색해 수수료·정산·지급보류 질문에 답변.

## 채널 키

| 질문 대상 | `--channel` 값 |
|---|---|
| 토스페이먼츠 | `toss-payments` |
| 카페24페이먼츠 | `cafe24-payments` |
| KG이니시스 | `kg-inicis` |
| 네이버페이 | `naver-pay` |
| 카페24 운영·결제 FAQ | `cafe24` |
| 스마트스토어 | `naver-smartstore` |
| 쿠팡 | `coupang` |

## Steps

### 1. 채널·키워드 결정
- 특정 PG사/채널 언급 → 해당 채널만
- "PG사별 비교", 채널 미언급 → 관련 채널 전체 순회
- 핵심 키워드 2~3단어 추출 (예: "수수료율", "정산 주기", "지급보류")

### 2. 채널별 검색
```bash
sppt settlement policy search "<키워드>" --channel <채널키> --limit 5
```

비교 질문은 채널당 limit 3으로 순회:
```bash
sppt settlement policy search "수수료율" --channel toss-payments --limit 3
sppt settlement policy search "수수료율" --channel cafe24-payments --limit 3
sppt settlement policy search "수수료율" --channel kg-inicis --limit 3
```

### 3. 결과 합성
- 채널별로 구분해서 답변, 숫자(%, D+N)는 강조
- 출처 채널 명시
- DB에 없는 내용은 추측 금지

## Notes
- **반드시 `--channel` 명시** — 미지정 시 naver FAQ(4000건)로 편향됨
- `fee_rate` 레코드는 등급별로 분리되어 있어 여러 건이 정상
- 결과 0건이면 키워드를 1~2단어로 줄여 재시도
