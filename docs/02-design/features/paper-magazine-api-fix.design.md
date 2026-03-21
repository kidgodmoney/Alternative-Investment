# Design: paper-magazine-api-fix

**Date**: 2026-03-21
**Phase**: Design → Do
**Ref**: docs/01-plan/features/paper-magazine-api-fix.plan.md

---

## 1. 수정 대상 파일

| 파일 | 변경 유형 | 주요 내용 |
|------|-----------|-----------|
| `index.html` | 수정 | S2 쿼리 버그 수정 + ALT_META에 queries 필드 추가 |
| `news.html` | 수정 | RSS 프록시 교체 + 피드 URL 정비 + SNS 소스 추가 |

---

## 2. index.html 설계

### 2-1. ALT_META queries 필드 추가

각 대체투자 카테고리에 `queries` 배열 추가:

```js
'alt.PE': {
  queries: [
    'private equity buyout leveraged fund performance',
    'PE secondary market LP portfolio',
    'ESG private equity value creation',
  ],
  ...
}
'alt.RE': {
  queries: [
    'real estate investment trust REIT returns',
    'commercial real estate data center logistics',
    'green building sustainable real estate ESG',
  ],
  ...
}
'alt.INF': {
  queries: [
    'infrastructure fund energy transition renewable',
    'digital infrastructure data center investment',
    'PPP public private partnership emerging markets',
  ],
  ...
}
'alt.HF': {
  queries: [
    'hedge fund strategy alpha factor long short',
    'quantitative systematic trading machine learning',
    'global macro hedge fund geopolitical risk',
  ],
  ...
}
'alt.VC': {
  queries: [
    'venture capital startup AI funding valuation',
    'early stage investment unicorn exit IPO',
    'climate tech deep tech venture fund portfolio',
  ],
  ...
}
```

### 2-2. fetchPapers() 버그 수정

```js
// 변경 전
const queries = filterCat === 'all'
  ? SEARCH_QUERIES
  : isAlt
    ? (CAT_META[filterCat]?.queries || ['alternative investment'])  // 버그
    : [CAT_META[filterCat]?.query || filterCat.replace('cs.', '')];

// 변경 후
const queries = filterCat === 'all'
  ? SEARCH_QUERIES
  : isAlt
    ? (ALT_META[filterCat]?.queries || ALT_QUERIES)  // 수정
    : [filterCat.replace('cs.', '')];
```

### 2-3. Rate limit 캐시 (localStorage)

```js
const CACHE_TTL = 30 * 60 * 1000; // 30분

function getCached(key) {
  const raw = localStorage.getItem('s2_' + key);
  if (!raw) return null;
  const { ts, data } = JSON.parse(raw);
  if (Date.now() - ts > CACHE_TTL) { localStorage.removeItem('s2_' + key); return null; }
  return data;
}

function setCache(key, data) {
  localStorage.setItem('s2_' + key, JSON.stringify({ ts: Date.now(), data }));
}
```

---

## 3. news.html 설계

### 3-1. RSS 프록시 교체

```js
// 변경 전
const PROXY = 'https://api.allorigins.win/get?url=';

// 변경 후 — rss2json (안정적, JSON 직접 반환)
const RSS2JSON_API = 'https://api.rss2json.com/v1/api.json?rss_url=';

// fetchFeed() 파싱 방식도 변경
async function fetchFeed(feedUrl) {
  const res = await fetch(RSS2JSON_API + encodeURIComponent(feedUrl));
  const json = await res.json();
  if (json.status !== 'ok') return [];
  return json.items.map(item => ({
    title:   item.title,
    link:    item.link,
    desc:    stripHtml(item.description || item.content || '').slice(0, 280),
    pubDate: item.pubDate,
    source:  json.feed.title,
  })).filter(i => i.title);
}
```

### 3-2. RSS 피드 URL 정비 (글로벌)

| 카테고리 | 기존 (폐기됨) | 신규 (검증된 URL) |
|---------|-------------|-----------------|
| PE | Reuters M&A | Google News: `PE buyout` |
| PE | FT Alphaville | Seeking Alpha RSS |
| RE | MarketWatch RE | Google News: `real estate investment` |
| INF | Reuters Finance | Google News: `infrastructure investment` |
| HF | Reuters Finance | Google News: `hedge fund` |

**Google News RSS 패턴:**
```
https://news.google.com/rss/search?q={query}&hl=en&gl=US&ceid=US:en
```

### 3-3. 한국 RSS 피드 정비

```js
// 네이버 뉴스 검색 RSS (키워드 기반)
'alt.PE': [
  { label: '네이버 사모펀드', url: 'https://news.naver.com/rss/003' },
  { label: '한국경제',        url: 'https://www.hankyung.com/feed/finance' },
  { label: '매일경제',        url: 'https://www.mk.co.kr/rss/30200030/' },
],
```

### 3-4. SNS 소스 추가 (신규)

#### Google News RSS (글로벌, 무료)

```js
// news.html FEEDS 객체에 추가
'alt.PE': [
  { label: 'Google News',
    url: 'https://news.google.com/rss/search?q=private+equity+buyout&hl=en&gl=US&ceid=US:en' },
  { label: 'Google News KR',
    url: 'https://news.google.com/rss/search?q=사모펀드&hl=ko&gl=KR&ceid=KR:ko' },
],
```

#### Reddit RSS (커뮤니티 인사이트)

```js
// Reddit 서브레딧별 RSS (직접 접근, 프록시 없이)
const REDDIT_FEEDS = {
  'alt.PE':  'https://www.reddit.com/r/privateequity+investing/.rss?limit=10',
  'alt.RE':  'https://www.reddit.com/r/realestateinvesting+REInvesting/.rss?limit=10',
  'alt.INF': 'https://www.reddit.com/r/investing+RenewableEnergy/.rss?limit=10',
  'alt.HF':  'https://www.reddit.com/r/algotrading+quant/.rss?limit=10',
  'alt.VC':  'https://www.reddit.com/r/startups+venturecapital/.rss?limit=10',
};

// Reddit은 CORS 없이 직접 fetch (User-Agent 헤더 필요)
async function fetchReddit(cat) {
  const url = REDDIT_FEEDS[cat];
  const res = await fetch(url, { headers: { 'User-Agent': 'AltInvestMag/1.0' } });
  const xml = new DOMParser().parseFromString(await res.text(), 'application/xml');
  const entries = [...xml.querySelectorAll('entry')];
  return entries.map(e => ({
    title:   e.querySelector('title')?.textContent?.trim() || '',
    link:    e.querySelector('link')?.getAttribute('href') || '#',
    desc:    stripHtml(e.querySelector('content')?.textContent || '').slice(0, 280),
    pubDate: e.querySelector('updated')?.textContent || '',
    source:  'Reddit',
  })).filter(i => i.title);
}
```

#### UI: 소스 뱃지 구분

```html
<!-- 뉴스 카드에 소스별 색상 뱃지 추가 -->
<span class="source-badge source-reddit">Reddit</span>
<span class="source-badge source-google">Google News</span>
<span class="source-badge source-naver">네이버</span>
```

```css
.source-badge { font-size: 9px; padding: 1px 6px; border-radius: 2px; font-weight: 700; letter-spacing: 0.06em; text-transform: uppercase; }
.source-reddit { background: #FF4500; color: #fff; }
.source-google { background: #4285F4; color: #fff; }
.source-naver  { background: #03C75A; color: #fff; }
```

### 3-5. 병렬 로딩 + Graceful Degradation

```js
// 현재: 실패 시 전체 빈 화면
// 개선: 성공한 소스만 표시

async function loadCat(cat, mode) {
  showLoading();
  const feedList = (mode === 'ko' ? FEEDS_KO : FEEDS)[cat] || [];
  const redditItems = await fetchReddit(cat).catch(() => []);

  // 병렬 fetch, 실패해도 빈 배열 반환
  const rssResults = await Promise.allSettled(feedList.map(f => fetchFeed(f.url)));
  const rssItems = rssResults
    .filter(r => r.status === 'fulfilled')
    .flatMap(r => r.value);

  const allItems = [...rssItems, ...redditItems]
    .sort((a, b) => new Date(b.pubDate) - new Date(a.pubDate));

  renderNews(allItems, cat);
}
```

---

## 4. 구현 체크리스트

### index.html
- [ ] ALT_META 각 항목에 `queries` 배열 추가 (5개 카테고리)
- [ ] `fetchPapers()` CAT_META → ALT_META 참조 수정
- [ ] localStorage 캐시 함수 추가 (`getCached`, `setCache`)
- [ ] `fetchPapers()` 캐시 적용

### news.html
- [ ] `PROXY` → `RSS2JSON_API` 교체
- [ ] `fetchFeed()` rss2json 파싱 방식으로 수정
- [ ] 글로벌 FEEDS 객체: Google News RSS URL로 교체
- [ ] 한국 FEEDS_KO: URL 검증 및 정비
- [ ] `REDDIT_FEEDS` 객체 추가
- [ ] `fetchReddit()` 함수 추가
- [ ] `loadCat()` Promise.allSettled + Reddit 병합
- [ ] 소스 뱃지 CSS 추가
- [ ] 뉴스 카드 렌더링에 소스 뱃지 반영

---

*Design Created: 2026-03-21*
