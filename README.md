# 대체투자 인사이트 매거진

> Alternative Investment Research Magazine — 리서치·뉴스·SNS 인사이트를 한 곳에서

[![GitHub Pages](https://img.shields.io/badge/GitHub%20Pages-live-brightgreen)](https://kidgodmoney.github.io/Alternative-Investment/)

---

## 페이지 구성

| 페이지 | 파일 | 설명 |
|--------|------|------|
| 리서치 | `index.html` | 글로벌 대체투자 리서치 아카이브 + RSS 피드 |
| 뉴스피드 | `news.html` | 실시간 대체투자 뉴스 + 키워드 필터 |
| SNS 인사이트 | `social.html` | Twitter · Mastodon · Reddit · 네이버 블로그 통합 피드 |

---

## 주요 기능

- **RSS 자동 수집** — 금융·대체투자 전문 매체 피드 통합
- **키워드 필터링** — PE, VC, 헤지펀드, 부동산, 인프라 등 카테고리 탐색
- **SNS 피드 통합** — 글로벌(Twitter/Nitter, Mastodon, Reddit) + 국내(네이버 블로그) 모드
- **플랫폼 필터** — SNS 소스별 필터링 + 글로벌/국내 토글
- **다크/라이트 테마** — 시스템 설정 연동 + 수동 전환
- **Abstract 펼치기** — 리서치 요약 인라인 확장

---

## 디자인 시스템

Supanova 스타일 기반 Glassmorphism 다크 테마

```
배경:    #09090f (near-black)
강조:    #a78bfa → #60a5fa (Violet → Blue gradient)
폰트:    Pretendard Variable + JetBrains Mono
카드:    backdrop-filter blur + 반투명 보더
애니메이션: Blob background, fade-in, ticker scroll
```

라이트 테마에서는 보라/파랑 계열을 유지하되 배경·텍스트 반전 적용.

---

## 기술 스택

- **순수 HTML / CSS / Vanilla JS** — 빌드 도구·프레임워크 없음
- **RSS → JSON 프록시** — `allorigins.win` / `rss2json.com`
- **폰트** — Pretendard Variable (CDN), JetBrains Mono (Google Fonts)
- **GitHub Pages** — 정적 호스팅

---

## 파일 구조

```
.
├── index.html        # 리서치 매거진 메인
├── news.html         # 뉴스피드
├── social.html       # SNS 인사이트
├── favicon.svg       # 브랜드 아이콘
└── docs/
    └── archive/      # PDCA 개발 이력
```

---

## 개발 이력

| 버전 | 내용 | Match Rate |
|------|------|-----------|
| redesign | 앰버-골드 단색 다크 테마 적용 | 100% |
| supanova-redesign | Pretendard + Violet/Blue 글래스모피즘 전환 | 92% |
| social-sns-insight | SNS 인사이트 페이지 신규 추가 | 98% |

---

## 로컬 실행

별도 서버 불필요. 파일을 브라우저에서 직접 열거나:

```bash
# Python 내장 서버 사용
python3 -m http.server 8080
# → http://localhost:8080
```

---

*PDCA 방법론으로 개발 · bkit Vibecoding Kit 활용*
