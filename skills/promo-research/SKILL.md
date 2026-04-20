# promo-research

소싱 키워드 또는 등록 상품을 바탕으로 경쟁 광고 분석 → 바이럴 강도 확인 → AI 광고 카피 생성까지 자동으로 진행하는 홍보 리서치 워크플로우.

## When to use

- "이 상품 광고 어떻게 하면 좋아?" 라고 물을 때
- 소싱 후 상품 등록 직전/직후 홍보 전략이 필요할 때
- 경쟁사 광고를 분석하고 싶을 때
- 인플루언서 마케팅을 시작하려 할 때
- "카피 써줘", "광고 문구 만들어줘" 요청

## Prerequisites

- `sppt` CLI 설치
- Apify API Token 설정 (ad-spy, viral-check, influencer에 필요):
  ```bash
  sppt config set apify apiToken <YOUR_APIFY_TOKEN>
  ```
  발급: https://console.apify.com/settings/integrations

## Steps

### 1. 트렌드 기반 키워드 검증 (선택)

소싱 전이라면 먼저 키워드 수요를 확인합니다.

```bash
sppt trend keyword <키워드>
```

검색량이 월 1,000 이상, 최근 트렌드 상승 중인 키워드를 선택합니다.

---

### 2. 경쟁 광고 분석 (Facebook Ad Library)

```bash
sppt promo ad-spy "<키워드>" --country KR --max 30
```

결과에서 주목할 포인트:
- **Winning Ads**: 집행 기간이 긴 광고 = 수익성 있는 광고 (오래 돌릴수록 ROI 높다는 신호)
- **반복 문구**: 여러 광고주가 공통으로 쓰는 단어/각도
- **광고주 목록**: 주요 경쟁사 파악

사용자에게 Winning Ad의 카피 내용을 보여주고, 어떤 각도(pain point / benefit / social proof)를 주로 사용하는지 분석해줍니다.

---

### 3. TikTok 바이럴 강도 확인

```bash
sppt promo viral-check "<키워드>" --max 30
```

결과 해석:
- **high** (평균 50만뷰↑): 이미 바이럴 중 — 빠르게 진입해야 함
- **medium** (5만~50만뷰): 성장 중 — 지금이 적기
- **low** (5만뷰 미만): TikTok에서 아직 미미 — SNS 광고보다 검색 광고 집중

---

### 4. AI 광고 카피 생성

위 분석 결과를 바탕으로 플랫폼별 카피를 생성합니다.

#### 4a. CLI 기본 카피 (템플릿 기반)

```bash
# 소싱 상품 기반
sppt promo copy "<키워드>" \
  --platform smartstore \
  --price <판매가> \
  --original-price <정가> \
  --features "특징1,특징2,특징3"

# 소싱 채널 상품 직접 연동
sppt promo copy "<키워드>" \
  --source domeggook \
  --product-id <ID> \
  --platform cafe24
```

#### 4b. AI 카피 최적화 (Claude 직접 생성)

CLI 결과 + ad-spy Winning Ad 카피를 참고해서 다음 프롬프트로 더 나은 카피를 생성합니다:

```
아래 정보를 바탕으로 [플랫폼]용 광고 카피를 작성해줘:

상품: [상품명]
판매가: [가격]원
주요 특징: [특징들]

경쟁 광고에서 효과적인 각도:
[ad-spy Winning Ad 카피 내용]

요청:
1. 검색 최적화 제목 (40자 이내)
2. 상품 상세 설명 (SEO 키워드 포함)
3. SNS 짧은 카피 3가지
4. 해시태그 15개
```

---

### 5. 인플루언서 발굴 (선택)

마이크로 인플루언서(1K~50K) 협업을 원할 때:

```bash
sppt promo influencer "<키워드>" \
  --min-followers 3000 \
  --max-followers 30000 \
  --csv influencers.csv
```

CSV 저장 후 사용자에게 상위 10명 목록을 보여주고, DM 템플릿을 제안합니다:

```
안녕하세요 @[username]님!
[상품명] 관련 콘텐츠를 만드시는 분을 찾고 있어요.
협업 의향이 있으시면 연락 부탁드립니다 🙏
```

---

### 6. 결과 요약

사용자에게 다음을 제공합니다:

1. **광고 각도 추천**: ad-spy 분석에서 도출한 효과적인 접근법
2. **플랫폼 우선순위**: 바이럴 강도에 따른 채널 전략 (TikTok high → 인스타/TikTok 먼저, low → 네이버/쿠팡 검색광고)
3. **최종 카피**: 바로 사용 가능한 플랫폼별 광고 문구
4. **인플루언서 리스트**: CSV 파일 경로 + 상위 10명

## 전략 가이드

### 커머스 광고 접근법

| 상황 | 추천 전략 |
|---|---|
| 신규 상품, 검증 필요 | ad-spy로 경쟁사 winning ad 찾기 → 그대로 따라하기 |
| 바이럴 강도 high | TikTok/Instagram 인플루언서 먼저 → SNS 광고 |
| 바이럴 강도 low | 네이버 쇼핑/쿠팡 검색 최적화 먼저 |
| 마진이 낮은 상품 | 카카오 단톡방/커뮤니티 바이럴 전략 |
| 브랜드 상품 | Instagram 마이크로 인플루언서 + 리뷰 마케팅 |

### Winning Ad 판별 기준
- 집행 기간 60일 이상: 수익 나는 광고
- 집행 기간 30~60일: 테스트 중
- 집행 기간 30일 미만: 아직 검증 전

## 주의사항

- Apify 크레딧 소모: actor 1회 실행당 약 $0.5~2 (Facebook Ad Library는 무거움)
- Facebook Ad Library는 최근 7일 이내 게재 시작 광고가 누락될 수 있음
- Instagram Scraper는 공개 계정/해시태그만 수집 가능
- TikTok Scraper 결과는 한국 IP 기준이 아닐 수 있으므로 참고용으로만 사용
