# Plan: paper-magazine-api-fix

**Date**: 2026-03-21
**Status**: Plan
**Feature**: API 오류 수정 + SNS 뉴스 수집 기능 추가

---

## 1. 문제 정의 (Problem Statement)

### 1-1. index.html — Semantic Scholar API 오류

| # | 위치 | 오류 내용 | 원인 |
|---|------|-----------|------|
| A | `fetchPapers()` L690 | `CAT_META[filterCat]?.queries` → `undefined` | `index.html`에 `CAT_META` 없음, `ALT_META` 사용해야 함 |
| B | `ALT_META` 각 항목 | `queries` 필드 없음 | 탭 클릭 시 대체투자 전용 쿼리 미정의 |
| C | S2 API 호출 | 429 Rate Limit | API key 없이 100 req/5min 제한 |
| D | 대체투자 탭 | AI 논문만 노출 | `SEARCH_QUERIES`(AI 쿼리)가 fallback으로 사용됨 |

### 1-2. news.html — RSS Feed 오류

| # | 위치 | 오류 내용 | 원인 |
|---|------|-----------|------|
| E | `allorigins.win` | 간헐적 503/429 | 무료 공용 CORS 프록시 불안정 |
| F | Reuters RSS | 피드 없음/오류 | `feeds.reuters.com` 다수 URL 폐기됨 |
| G | 한국 언론사 RSS | 일부 오류 | 한경·이데일리·매경 RSS 경로 변경됨 |
| H | 에러 핸들링 | 전체 빈 화면 | 한 피드 실패 시 전체 실패 UI 표시 |

### 1-3. 신규 기능 — SNS 뉴스 수집

현재 뉴스 소스가 RSS에만 의존하여 SNS(소셜미디어) 기반 실시간 대체투자 정보가 누락됨.

---

## 2. 목표 (Goals)

### Goal A: API 오류 수정
- [ ] Semantic Scholar: 대체투자 탭별 전용 쿼리 맵 추가
- [ ] Semantic Scholar: Rate limit 대응 (캐시 + 재시도 로직)
- [ ] RSS: CORS 프록시를 안정적인 대안으로 교체
- [ ] RSS: 작동하는 피드 URL로 교체
- [ ] RSS: 부분 실패 허용 (graceful degradation)

### Goal B: SNS 뉴스 수집 추가
- [ ] 네이버 뉴스 검색 연동 (국내 대체투자)
- [ ] Google News RSS 연동 (글로벌 무료)
- [ ] Reddit 관련 서브레딧 피드 연동
- [ ] 선택적: GNews API / NewsAPI (API key 필요)

---

## 3. 솔루션 설계 방향

### 3-1. Semantic Scholar 수정

```
ALT_META 객체에 queries 필드 추가:
  'alt.PE' → ['private equity buyout', 'LBO leveraged buyout fund']
  'alt.RE' → ['real estate investment trust REIT', 'commercial real estate']
  'alt.INF' → ['infrastructure fund energy transition', 'renewable energy investment']
  'alt.HF' → ['hedge fund strategy alpha', 'systematic trading quantitative']
  'alt.VC' → ['venture capital startup AI funding', 'early stage investment unicorn']

fetchPapers() 수정:
  CAT_META[filterCat]?.queries → ALT_META[filterCat]?.queries

Rate limit 대응:
  - localStorage 캐시 (TTL 30분)
  - 429 응답 시 exponential backoff
```

### 3-2. RSS 프록시 교체

```
현재: allorigins.win (불안정)
대안 우선순위:
  1. rss2json.com API (무료 100 req/day, JSON 변환, 안정적)
  2. api.rss2json.com → JSON 반환으로 파싱 간소화
  3. 다중 프록시 fallback (allorigins → rss2json → corsproxy.io)
```

### 3-3. RSS 피드 URL 정비

**글로벌 (교체 목록)**
| 현재 | 교체 |
|------|------|
| `feeds.reuters.com/reuters/mergersNews` | `reuters.com` RSS 미지원 → GNews API |
| `feeds.reuters.com/reuters/businessNews` | → Google News RSS |
| `www.ft.com/rss/home/uk` | 유료 구독 필요 → Bloomberg RSS |

**검증된 무료 글로벌 피드**
- `https://news.google.com/rss/search?q=private+equity&hl=en` (Google News RSS, 무료)
- `https://techcrunch.com/category/venture/feed/` (VC, 작동 확인)
- `https://gnews.io/api/v4/search?q=...` (GNews API, 100 req/day 무료)

**국내 (재검증)**
- 한국경제: `https://www.hankyung.com/feed/finance` → 확인 필요
- 이데일리: URL 형식 변경됨 → 재검증

### 3-4. SNS 뉴스 수집 아키텍처

```
소스별 접근 방법:

1. 네이버 뉴스 (국내 최적)
   - 방법: Naver Search API (클라이언트 직접 호출 불가, CORS)
   - 해결: 네이버 뉴스 RSS 사용 또는 서버사이드 프록시
   - URL: https://news.naver.com/main/rss
   - 키워드: "사모펀드", "부동산투자", "헤지펀드", "벤처캐피탈"

2. Google News RSS (글로벌, 무료, 키 불필요)
   - URL: https://news.google.com/rss/search?q={query}&hl=en&gl=US&ceid=US:en
   - CORS 프록시 필요 (rss2json 활용)

3. Reddit RSS (커뮤니티 인사이트)
   - URL: https://www.reddit.com/r/privateequity/.rss
   - URL: https://www.reddit.com/r/investing/.rss
   - CORS 없음 (직접 접근 가능 — User-Agent 설정 필요)

4. Twitter/X (현실적으로 어려움)
   - Basic API: $100/month
   - → 제외 또는 임베드 위젯 방식

5. LinkedIn
   - 공개 API 없음 → 제외

추천 구현 순서: Google News RSS → Reddit → 네이버 RSS → GNews API
```

---

## 4. 구현 우선순위

| 우선순위 | 항목 | 난이도 | 효과 |
|---------|------|-------|------|
| P0 | Semantic Scholar queries 버그 수정 | 쉬움 | 즉시 |
| P0 | RSS 프록시 교체 (rss2json) | 쉬움 | 즉시 |
| P1 | Google News RSS 연동 | 쉬움 | 높음 |
| P1 | RSS URL 정비 (Reuters 대체) | 보통 | 높음 |
| P2 | Reddit RSS 연동 | 보통 | 보통 |
| P2 | 네이버 뉴스 RSS 연동 | 보통 | 국내 높음 |
| P3 | Rate limit 캐시 로직 | 보통 | 안정성 |
| P3 | GNews API 연동 | 보통 | 선택사항 |

---

## 5. 파일 영향 범위

```
paper-magazine/
├── index.html        ← S2 API 쿼리 수정, 캐시 추가
├── news.html         ← RSS 프록시 교체, 피드 URL 정비, SNS 소스 추가
└── (선택) proxy/     ← 네이버 API 프록시가 필요할 경우 Node.js 서버
```

---

## 6. 비용 및 제약

| 소스 | 비용 | 제약 |
|------|------|------|
| Semantic Scholar | 무료 | 100 req/5min (key 없이) |
| Google News RSS | 무료 | CORS 프록시 필요 |
| rss2json.com | 무료 100 req/day | 트래픽 많으면 유료 |
| Reddit RSS | 무료 | User-Agent 필요 |
| GNews API | 무료 100 req/day | API key 필요 |
| Naver News API | 무료 25,000 req/day | API key 필요, 서버사이드 |
| Twitter/X API | $100/month | 현실적으로 제외 |

---

## 7. 성공 기준

- [ ] 모든 대체투자 탭에서 관련 논문 정상 표시
- [ ] 뉴스 피드 로드 성공률 90% 이상
- [ ] SNS 소스 최소 2개 추가 (Google News + Reddit)
- [ ] 오류 발생 시 graceful fallback (빈 화면 아닌 메시지)

---

*Plan Created: 2026-03-21*
