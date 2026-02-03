# Fotonet Blog

개발 + 일상 블로그

[![GitHub Pages](https://img.shields.io/badge/GitHub%20Pages-deployed-brightgreen)](https://fotoner.github.io)

## Tech Stack

- **Static Site Generator**: Jekyll 4.x
- **Theme**: [Minimal Mistakes](https://mmistakes.github.io/minimal-mistakes/)
- **Hosting**: GitHub Pages
- **Comments**: Utterances

## Features

- 게시글 목록에 썸네일 자동 표시 (본문 첫 이미지 추출)
- SEO 최적화 (Open Graph, Twitter Card)
- 반응형 디자인
- 카테고리/태그 아카이브
- 검색 기능

## Local Development

### Requirements

- Ruby 3.0+
- Bundler

### Installation

```bash
# 의존성 설치
bundle install

# 로컬 서버 실행
bundle exec jekyll serve
```

브라우저에서 http://localhost:4000 접속

### macOS (Homebrew Ruby)

시스템 Ruby 대신 Homebrew Ruby 사용 시:

```bash
export PATH="/opt/homebrew/opt/ruby/bin:$PATH"
bundle install
bundle exec jekyll serve
```

## Directory Structure

```
├── _config.yml          # 사이트 설정
├── _posts/              # 블로그 게시글
│   ├── LLM/
│   ├── python/
│   ├── spring/
│   ├── til/
│   └── vue/
├── _pages/              # 정적 페이지
├── _includes/           # 재사용 컴포넌트
├── _layouts/            # 페이지 레이아웃
├── _sass/               # 스타일시트
└── assets/              # 이미지, JS, CSS
```

## Writing Posts

새 게시글 작성:

```markdown
---
title: "게시글 제목"
last_modified_at: 2026-02-03T12:00:00
categories:
  - 카테고리명
tags:
  - 태그1
  - 태그2
header:
  teaser: /assets/images/thumbnail.png # 선택사항 (없으면 본문 첫 이미지 사용)
---

게시글 내용...
```

파일명 형식: `YYYY-MM-DD-title.md`

## Customizations

### Thumbnail Display

- 게시글 목록에서 썸네일 자동 표시
- 우선순위: `header.teaser` > 본문 첫 이미지 > `site.og_image`

### SEO

- Open Graph 메타 태그 자동 생성
- Twitter Card 지원
- 게시글 이미지 자동 추출

## Deployment

GitHub에 push하면 자동으로 GitHub Pages에 배포됩니다.

```bash
git add .
git commit -m "Add new post"
git push origin master
```

## License

MIT License
