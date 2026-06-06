---
name: blog-write
description: |
  큐레이션·블로그 글 작성 풀 라우틴 (sppt-contents). _picks.md queued 픽 1개에서
  PHASE 0~5 자동 진입 — 본문 + hero + 도식 + 사이드카 + 메일리 발행 인계까지.
  Use when asked to "글 써줘", "글 작성", "픽 진행해줘", "새 글", "큐레이션 글", "/blog-write".
---

# blog-write

> sppt-contents 콘텐츠 라우틴 풀 진입. 큐레이션 1편을 픽에서 메일리 발행까지
> 한 사이클로 끝내는 워크플로우.

## When to use

- "글 써줘", "글 작성", "픽 진행해줘", "새 글 만들어줘", "큐레이션 글 써줘" 등
- `research/pillars/global-builder/_picks.md` queued 픽 1개를 메일리 발행까지

## Prerequisites

- 위치: `sppt-contents/reports/` 라우틴 디렉토리
- 가이드: [`_routine-report.md`](../../../sppt-contents/reports/_routine-report.md),
  [`_conclusion-framework.md`](../../../sppt-contents/reports/_conclusion-framework.md),
  [content-syndication 스펙](../../../sppt-wiki/pages/product/spec/content-syndication.md)

## Steps

### 0. 픽 선택 (G1 게이트 — 🔴 사용자)

```
/blog-write <slug>           # 명시
/blog-write                  # _picks.md queued 첫 줄 자동 + 사용자 확인
```

`research/pillars/global-builder/_picks.md` 의 queued 섹션에서 픽 메타 읽기:
한글 제목, 영어 원제, Apple Podcasts URL, 발행일, 환원성·인사이트 깊이 메모,
인용 후보, 결론 아키타입 후보.

### 1. PHASE 0~0.5 자동 통과

- raw 항목 (`research/_raw/global-builder/{date}.md` 의 해당 에피) 인용 + 셀러
  맥락 추출
- 큐레이션 타입은 *에피 본문 자체*가 소스 — deep research 없음
- 인용 한도 사전 체크 (영문 ≤2문장·각 ≤50단어)

### 2. PHASE 1 결론 컨펌 (🔴 컨펌 1)

마무리 한 줄 방향을 사용자에게 묻는다 (AskUserQuestion):
- A. 셀러 환원 1줄 ("여기서 셀러는 ___")
- B. 메이커 시험 1줄 ("내가 다음에 시험할 것은 ___")
- C. 둘 다 넘나들기 (디폴트)

원본 에피의 결을 우선 본다. 인위적 환원 다양화는 식상 — 원본이 자연 가져오는
방향 그대로 따라가도 OK ([[_routine-report]] PHASE 3 "한국 환원 H2").

### 3. PHASE 2 SEO 설계 (가벼운 확인)

frontmatter 새 파일 만들고 본문 outline + cover hero 메타:
- `title`, `meta_description`, `target_keyword`, `secondary_keywords`
- `content_type: 큐레이션`, `persona: maker`
- `source_episode` (channel/title_en/url/published)
- `syndication_channels: [maily, eo-planet, threads, x, linkedin]`
- `maily_category` (1편은 "AI 네이티브 브랜드")
- `subscriber_gate: true` (메일리 디폴트, cliffhanger 구성 필수)
- `hero.kicker`·`hero.headline`(`|` 줄바꿈)·`hero.headline_accent_words`·`hero.caption`
- `images: [...]` (도식 4장 예상)

### 4. PHASE 3 본문 작성

[[_routine-report]] PHASE 3 큐레이션 전용 톤 적용:
- **Q&A 구조**: 메인 H2 4개 `## Q. ...` + 본문 보조 `**Q. ...**` 4~5개
- **해요체 단일화** (T5 큐레이션 예외, T4 합니다체 디폴트 아님)
- **저자 투명**: 1인칭은 도입·한국 환원 H2·마무리만
- **단문 폭탄** 1~2개 (`**진짜 무서운 건 그다음이에요.**` 식)
- **굵게** H2당 1~2 주장 한 줄
- **연결구**: 그런데·그래서·여기서·근데·차이라면
- **영문 인용 3단계**: `> "원문"` 다음 `> (한국어 의미 괄호)` 다음 단락 풀이
- **em dash(—) 0개** — 쉼표·줄바꿈·괄호로 대체
- **cliffhanger**: H2-1 보조 Q 끝 지점에 호기심 정점 (메일리 무료 노출 끝점)
- **마무리** = 메이커 시험 약속 1줄 + 이탤릭 2~3줄 인터뷰어 소회 (+ 옵션 ` :)`)

### 5. PHASE 3.5 hero (🔴 컨펌 1.5)

사용자에게 baseline 사진 출처 묻기 (X 프로필·YouTube·공식 사이트 등).

사용자가 사진 받아 `reports/assets/{slug}/_baseline/{filename}` 에 두면:

```bash
# 누끼 (sppt-gogh 활용)
cd ../sppt-gogh && bun scripts/cutout-one.ts \
  ../shopport/sppt-contents/reports/assets/{slug}/_baseline/{filename} \
  ../shopport/sppt-contents/reports/assets/{slug}/_baseline/{name}-cutout.png

# hero 합성 (네이비 + 오렌지 + 흑백 누끼 + 헤드라인 텍스트)
cd ../shopport && python3 sppt-contents/reports/scripts/build-hero-photo.py {slug}
```

### 6. PHASE 3.7 도식 + 사이드카

**도식**: `build-diagram.py` 의 `DIAGRAMS[slug]` 딕셔너리에 함수 4개 등록 후:

```bash
python3 sppt-contents/reports/scripts/build-diagram.py {slug}
```

도식 4종 패턴 (`diagram-style-guide.md`): 화살표 전환·4박스·3원·인용 카드.
키 컬러 `#FFB23D` 오렌지, 배경 `#0E2A57` 네이비, 박스 radius 32, 화살표 굵게.

**사이드카 2종**:
- `{date}-{slug}.long-syndication.md` — EO플래닛 (도입 1단락 + SOT 본문 reference)
- `{date}-{slug}.sns.md` — Threads (200~400자) · X (280자/스레드) · LinkedIn (800~1500자)

### 7. PHASE 4 자동 점검

체크리스트 자동 확인:
- em dash 0개 (`grep -c "—"`)
- Q&A 구조 (메인 4·보조 4+)
- 인용 한도 (영문 ≤2문장·각 ≤50단어)
- 인터뷰어 소회 2~3줄 이탤릭
- 저자 등장 절제
- 사이드카 정합

### 8. PHASE 5 발행 인계 (🔴 컨펌 2)

```bash
# 메일리 호환 HTML 변환 + 클립보드 복사
python3 sppt-contents/reports/scripts/prepare-maily.py {slug}
```

사용자에게:
1. 메일리 build.hyuk 에디터 열고 새 글 작성
2. 본문 빈 곳에 ⌘V 붙여넣기 (변환된 HTML)
3. cover 이미지 (`hero.jpg`) 업로드
4. 도식 4장 (`diagram-01~04.png`) 본문 placeholder 자리에 직접 업로드
5. 제목·meta_description·카테고리·subscriber_gate 위치 설정
6. 발행 → URL 받기

### 9. 발행 후 동기화

사용자가 메일리 URL 알려주면:
- frontmatter `published_urls.maily` + `status: published` + `publish_date` 실제 발행일
- `_schedule.md` 발행 완료 표에 행 추가 (메일리 ✓ + URL, 나머지 4채널 ☐)
- `_picks.md` queued → published 섹션 이동

이후 사용자가 다른 채널(EO·Threads·X·LinkedIn) 인계하며 URL 받으면 동일 동기화.

## Related

- 라우틴 SOT: [`_routine-report.md`](../../../sppt-contents/reports/_routine-report.md)
- 결론 프레임워크: [`_conclusion-framework.md`](../../../sppt-contents/reports/_conclusion-framework.md)
- 픽: [`pillars/global-builder/_picks.md`](../../../sppt-contents/research/pillars/global-builder/_picks.md)
- 스펙: [content-syndication](../../../sppt-wiki/pages/product/spec/content-syndication.md)
- 브랜드: [`_brand/README.md`](../../../sppt-contents/reports/assets/_brand/README.md), [`diagram-style-guide.md`](../../../sppt-contents/reports/assets/_brand/diagram-style-guide.md)
- 빌더: `build-hero-photo.py`, `build-diagram.py`, `prepare-maily.py`, sppt-gogh `cutout-one.ts`
