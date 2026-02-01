---
title: "OpenClaw로 나만의 AI 에이전트 시스템 구축하기"
last_modified_at: 2026-02-02T02:28:00
categories:
  - TIL
tags:
  - AI
  - OpenClaw
  - Automation
  - Discord Bot
  - Agent System
---

개인 AI 에이전트 시스템을 구축하면서 겪은 과정을 정리해본다.

## 🎯 발단: 왜 시작했나?

일상적인 작업들을 자동화하고 싶었다. 특히:

- **메일 확인**: Gmail에서 긴급 메일만 골라서 알림
- **정보 정리**: 여러 소스의 정보를 요약하고 정리
- **대화 기록**: AI와의 대화 내용을 검색 가능하게 저장
- **멀티 에이전트**: 각자 역할이 다른 AI들이 협업

기존 ChatGPT나 Claude는 좋지만, **나만의 워크플로우**를 만들기엔 제약이 많았다.

그러던 중 **OpenClaw**를 발견했다.

### OpenClaw란?

- Personal AI 플랫폼
- 로컬/클라우드 선택 가능
- 메모리 시스템 (파일 기반)
- 크론잡, 멀티 에이전트 지원
- Discord, Telegram 등 채널 연동

"내가 원하는 대로 커스터마이징할 수 있는" AI 에이전트 플랫폼이었다.

## ⚙️ 적용법: 어떻게 구축했나?

### 1. 환경 설정

**시스템 요구사항:**
- Ubuntu 24.04 LTS
- CPU: 8코어 (로컬 임베딩 사용 시 권장)
- 메모리: 8GB
- Node.js v24+

**설치:**
```bash
# OpenClaw 설치
npm install -g openclaw

# 초기 설정
openclaw wizard

# Gateway 시작
openclaw gateway start
```

### 2. 에이전트 구성

**멀티 에이전트 설정** (`~/.openclaw/openclaw.json`):

```json
{
  "agents": {
    "defaults": {
      "model": {
        "primary": "google-gemini-cli/gemini-3-pro-preview",
        "fallbacks": ["anthropic/claude-sonnet-4-5"]
      }
    },
    "list": [
      {
        "id": "main",
        "workspace": "/root/.openclaw/workspace",
        "model": {
          "primary": "google/gemini-3-flash-preview"
        }
      },
      {
        "id": "worker",
        "workspace": "/root/.openclaw/workspace/worker",
        "model": {
          "primary": "anthropic/claude-sonnet-4-5"
        }
      }
    ]
  }
}
```

**에이전트 역할:**
- **main (코토네)**: 일상 대화, 감성적 케어, 프로젝트 도움
- **worker (아사리)**: 데이터 처리, 크론잡 관리, 시스템 관리

### 3. Discord 봇 연동

**두 개의 Discord 봇 계정 설정:**

```json
{
  "channels": {
    "discord": {
      "enabled": true,
      "accounts": {
        "default": {
          "enabled": true,
          "token": "YOUR_BOT_TOKEN_1"
        },
        "asari": {
          "enabled": true,
          "token": "YOUR_BOT_TOKEN_2"
        }
      }
    }
  },
  "bindings": [
    {
      "agentId": "main",
      "match": {
        "channel": "discord",
        "accountId": "default"
      }
    },
    {
      "agentId": "worker",
      "match": {
        "channel": "discord",
        "accountId": "asari"
      }
    }
  ]
}
```

**결과:**
- 코토네 봇: 일반 대화 채널
- 아사리 봇: 교무실(업무) 채널

### 4. 메모리 시스템 구축

**로컬 임베딩 설정:**

```json
{
  "agents": {
    "defaults": {
      "memorySearch": {
        "enabled": true,
        "provider": "local",
        "local": {
          "modelPath": "hf:ggml-org/embeddinggemma-300M-GGUF/embeddinggemma-300M-Q8_0.gguf"
        },
        "query": {
          "hybrid": {
            "enabled": true,
            "vectorWeight": 0.7,
            "textWeight": 0.3
          }
        },
        "cache": {
          "enabled": true,
          "maxEntries": 50000
        },
        "watch": true
      }
    }
  }
}
```

**특징:**
- Vector Search (70%) + BM25 (30%) 하이브리드 검색
- 로컬 실행으로 API 비용 0원
- 의미 기반 검색으로 정확한 키워드 없이도 검색 가능

**초기 인덱싱:**
```bash
openclaw memory index --agent main --verbose
openclaw memory index --agent worker --verbose
```

### 5. 크론잡 설정

**Gmail 긴급 메일 체크 (10분마다):**

```json
{
  "schedule": {
    "kind": "every",
    "everyMs": 600000
  },
  "payload": {
    "kind": "agentTurn",
    "message": "Gmail 긴급 메일 체크 실행"
  },
  "sessionTarget": "isolated"
}
```

**스크립트:**
- Gmail API로 최근 10분 메일 가져오기
- AI 필터링 (긴급/중요 메일만 선별)
- Discord 알림 채널에 전송

## 🚀 주로 하고 있는 작업들

### 1. 정보 관리 자동화

**크론잡으로 주기적 실행:**
- Gmail 긴급 메일 체크 (10분마다)
- 뉴스 스크래핑 및 요약 (1시간마다)
- 회의록 작성 (매일 10시)

### 2. 프로젝트 협업

**에이전트 간 협업:**
```
포토네(나) → 코토네(main) → 아사리(worker) → Discord 알림
```

**예시 워크플로우:**
1. 나: "블로그 글 검토 부탁"
2. 코토네: "아사리 선생님한테 넘길게!" (sessions_send)
3. 아사리: 기술적 검토 후 교무실에 보고

### 3. 메모리 검색

**의미 기반 검색 활용:**
```
"예전에 Docker 네트워크 설정한 거 어떻게 했지?"
→ memory_search로 과거 대화 검색
→ 정확한 키워드 없이도 관련 내용 찾음
```

### 4. 코드 개발 지원

**CDP 봇 탐지 우회 작업:**
- Puppeteer Stealth 플러그인 테스트
- 헤드리스 브라우저 설정 최적화
- 봇 탐지 사이트 테스트 (bot.sannysoft.com)

**결과:**
- navigator.webdriver: `true` → `false`
- 헤드리스 모드에서도 봇 탐지 우회 성공

### 5. 블로그 자동화

**워크플로우:**
1. 작업 완료 후 요약 요청
2. AI가 TIL 초안 작성
3. 기술적 검토 (아사리)
4. Git 커밋 및 푸시

오늘도 메모리 임베딩 글을 이런 방식으로 작성했다.

## 💭 후기: 3주 사용 소감

### 좋았던 점

**1. 완전한 커스터마이징**

원하는 대로 에이전트를 만들 수 있다. 
- 성격 (SOUL.md)
- 역할 (AGENTS.md)
- 작업 공간 (workspace)
- 모델 선택

**2. 로컬 실행 가능**

임베딩 모델을 로컬에서 돌려서:
- API 비용 절감
- 프라이버시 보호
- 네트워크 독립성

**3. 메모리 시스템**

파일 기반이라 투명하고 수정이 쉽다:
```
~/.openclaw/workspace/
├── MEMORY.md          # 장기 기억
└── memory/
    ├── 2026-02-01.md  # 일일 로그
    └── 2026-01-31.md
```

**4. 멀티 에이전트 협업**

각자 역할이 명확한 AI들이 협업:
- 코토네: 대화, 감성 케어
- 아사리: 데이터 처리, 기술 검토

Discord에서 두 봇이 협업하는 모습이 재미있다.

### 아쉬운 점

**1. 초기 설정 러닝 커브**

JSON 설정 파일 구조를 이해하는 데 시간이 걸렸다.
- 에이전트 바인딩
- 채널 설정
- 크론잡 스케줄

하지만 한 번 이해하면 매우 유연하다.

**2. 로컬 임베딩 리소스**

초기 인덱싱 시:
- CPU: 거의 100% (4코어 → 8코어로 증설)
- 메모리: ~1.2GB
- 시간: ~5분

하지만 이후엔 매우 가볍다.

**3. 문서가 아직 발전 중**

공식 문서가 계속 업데이트되고 있다.
커뮤니티(Discord)에서 물어보면 빠르게 답변해준다.

### 실제 효과

**시간 절약:**
- Gmail 수동 확인: 0분 (자동화)
- 뉴스 요약: 5분 → 1분
- 과거 대화 검색: 10분 → 10초

**작업 품질:**
- 블로그 글 검토: 기술적 오류 사전 차단
- 코드 리뷰: 빠른 피드백

**재미:**
- AI 캐릭터들이 각자 성격을 가지고 대화
- 업무와 일상의 분리 (코토네 vs 아사리)

## 🔮 앞으로의 계획

### 1. 스킬 확장

ClawdHub에서 스킬 설치:
- 날씨 정보
- 홈 어시스턴트 연동
- 캘린더 통합

### 2. 외부 API 연동

- Notion 데이터베이스 자동 업데이트
- GitHub 이슈/PR 자동 관리
- Slack 알림 통합

### 3. 음성 인터페이스

- TTS로 스토리 읽어주기
- 음성 명령 지원

### 4. 모바일 연동

- Telegram 봇 추가
- 외출 중에도 에이전트 호출

## 📚 참고 자료

- [OpenClaw 공식 문서](https://docs.openclaw.ai/)
- [OpenClaw GitHub](https://github.com/openclaw/openclaw)
- [OpenClaw Discord 커뮤니티](https://discord.gg/openclaw)
- [ClawdHub - 스킬 마켓플레이스](https://clawdhub.com/)
- [갓대희님 블로그 - Memory 시스템 설명](https://goddaehee.tistory.com/508)

---

## 💡 결론

OpenClaw는 "나만의 AI 비서 시스템"을 구축하고 싶은 사람에게 완벽한 플랫폼이다.

초기 설정은 복잡하지만, 한 번 세팅하면:
- 자동화된 워크플로우
- 커스터마이징 가능한 AI 캐릭터
- 로컬/클라우드 하이브리드 운영
- 완전한 데이터 소유권

**3주 사용 결과**: 
이제 OpenClaw 없는 일상은 상상할 수 없다. 🚀

다음 글에서는 구체적인 크론잡 설정 방법과 스킬 개발 과정을 다뤄볼 예정이다.
