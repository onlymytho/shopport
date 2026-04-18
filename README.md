# rocketsell

커머스 운영 자동화 CLI. 트렌드 파악하고, 상품 찾고, 채널에 올리고, 주문 처리하고 — 터미널 하나로.

```
트렌드 탐색      →    소싱 검색       →    채널 게재              →    판매 관리
네이버·전문몰         도매매·특가몰·CJ      쿠팡·카페24·스마트스토어      주문·송장·재고
```

## 설치

```bash
curl -fsSL https://rocketsell.vercel.app/api/v2/cli/install | bash
```

macOS · Linux 지원. bun 또는 npm 자동 감지.

## 5분 시작하기

**1. 채널 연결**

```bash
rocketsell init
```

판매 채널과 소싱 채널을 단계별로 설정합니다. 채널당 약 5-8분.
첫 채널 하나만 연결해도 바로 사용 가능.

**2. 연결 확인**

```bash
rocketsell status
```

**3. 트렌드 확인 후 소싱**

```bash
rocketsell trend keyword 에어프라이어        # 검색량 + 경쟁강도 + 트렌드
rocketsell source search domeggook 에어프라이어
```

여기까지 되면 설치 완료입니다.

## 주요 명령어

```bash
# 트렌드 분석
rocketsell trend keyword <키워드>              # 검색량(PC/모바일) + 경쟁강도 + 트렌드 그래프
rocketsell trend keyword <키워드> --volume    # 검색량만 (naver-searchad)
rocketsell trend keyword <키워드> --trend     # 트렌드만 (naver-datalab)
rocketsell trend compare <키워드1> <키워드2>   # 키워드 간 상대 비교 (최대 5개)
rocketsell trend platform musinsa             # 무신사 실시간 랭킹
rocketsell trend platform oliveyoung          # 올리브영 인기 상품
rocketsell trend platform amazon              # Amazon Movers & Shakers (글로벌 선행 지표)
rocketsell trend platform aliexpress          # AliExpress 급상승 (소싱 선행 지표)
# 지원 플랫폼: musinsa | oliveyoung | hwahae | zigzag | coupang | amazon | aliexpress

# 소싱 탐색
rocketsell source search domeggook 무선이어폰
rocketsell source search domeggook 무선이어폰 --waterfall   # 3채널 비교 + 최저가

# 상품 등록
rocketsell product register cafe24 --file product.json
rocketsell product register --all-channels --file product.json  # 전체 채널 동시

# 주문 처리
rocketsell order list coupang
rocketsell order confirm coupang <orderId>
rocketsell invoice register coupang <orderId> cj <송장번호>

# 재고
rocketsell inventory sync cafe24 <productId> 50
rocketsell inventory plan 100 <sku>  # 채널별 분배 시뮬레이션
```

전체 명령어: `rocketsell --help`

## Claude Code Skills

Claude Code와 함께 쓰면 자연어로 전체 워크플로우가 실행됩니다.

```
"무신사 랭킹 상위 상품 중 도매매에서 소싱 가능한 것 찾아줘"
"에어프라이어 검색량이 오르는 중인지 확인하고, 마진 30% 이상 상품 쿠팡에 등록해줘"
"오늘 들어온 쿠팡 주문 전부 처리해줘. CJ로 발주하고 송장 등록까지"
"소싱가 올라서 마진 이탈된 상품 있으면 판매가 조정 제안해줘"
```

Skills 설치:

```bash
curl -fsSL https://rocketsell.vercel.app/agent-install.txt
```

## 채널

**판매 채널 (Selling)**

| 채널 | 인증 방식 |
|------|----------|
| Cafe24 | OAuth 2.0, 액세스 토큰 자동 갱신 |
| Coupang | HMAC 서명, 180일 키 만료 추적 |
| SmartStore | Client Credentials |
| Shopify | GraphQL Admin API |

**소싱 채널 (Sourcing)**

| 채널 | 특징 |
|------|------|
| 도매매 `domeggook` | 위탁 전용. 국내 최다 위탁 상품. |
| 특가몰 `specialoffer` | 국내 소싱. 키워드·카테고리 검색. |
| CJ Dropshipping | 해외 소싱. USD→KRW 자동 환산. |

**트렌드 분석 (Trend)**

| 채널 | 특징 |
|------|------|
| 네이버 데이터랩 | 검색 트렌드 그래프. 월별 추이 + 키워드 비교. |
| 네이버 검색광고 | PC·모바일 월간 검색량. 경쟁강도 + 연관 키워드. |
| 전문몰 랭킹 | 무신사·올리브영·화해·지그재그·쿠팡 실시간 랭킹. Amazon·AliExpress 글로벌 트렌드. |

## 데이터 저장 (선택)

credentials는 `~/.rocketsell/config.json`에 로컬 저장됩니다.

PostgreSQL 연결 시 작업 이력, 상품 매핑, 재고 스냅샷을 SQL로 조회할 수 있습니다.

```bash
export ROCKETSELL_DATABASE_URL="postgresql://..."
```

## 로컬 개발

```bash
bun install
bun run build   # tsup 빌드 → dist/cli.js
bun link        # 로컬 심링크 연결
bun run type    # 타입 체크
bun run fix     # biome lint + format
```

## 라이선스

MIT
