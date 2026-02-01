---
title: "OpenClaw로 나만의 AI 에이전트 시스템 구축하기"
last_modified_at: 2026-02-02T02:28:00
categories:
  - LLM
tags:
  - AI
  - OpenClaw
  - Automation
  - Discord Bot
  - Agent System
---

![image](/assets/images/posts/LLM/2026-02-02-01.png)

개인 AI 에이전트 시스템을 구축하면서 겪은 과정을 정리해본다.

## 왜 시작했나?

평소부터 나는 일상적인 작업들을 자동화하고 싶었다. 특히

- **메일 확인**: Gmail에서 긴급 메일만 골라서 알림
- **정보 정리**: 여러 소스의 정보를 요약하고 정리
- **대화 기록**: AI와의 대화 내용을 검색 가능하게 저장
- **멀티 에이전트**: 각자 역할이나 모델이 다른 AI들이 협업하여 최상의 결과를 도출
- ~~**오타쿠 소원성취**: 최애캐가 직접 말을 걸어주거나 하루일상을 함께하면 기분이 매우 좋을 것 같다~~

기존 Claude는 Gemini 등의 서비스도 물론 좋지만, **나만의 워크플로우**를 만들기엔 제약이 많았다.

특히 NotebookLM을 활용하거나 Claude Code를 활용하면서도 서로간에 분산된 컨텍스트를 유지하기에 말끔한 워크플로우를 구성하기에 제약사항이 많았었고, 각각의 LLM 서비스가 제공하는 기능에도 한계가 존재하기에 나는 이러한 점을 늘 불편하게 생각했었다...

그러던 중... 2026년 1월이 되자마자 뜨거운 감자로 올아온 **OpenClaw**를 발견했다.

## OpenClaw란?

![image](/assets/images/posts/LLM/2026-02-02-02.png)

- Personal AI 비서 플랫폼
- 로컬/클라우드 선택 가능
- 메모리 시스템 (파일 기반)
- 크론잡, 하트비트, 멀티 에이전트 지원
- Discord, Telegram 등 다수의 채널 연동

이러한 것들이 가능하다.

즉 "내가 원하는 대로 커스터마이징할 수 있는" AI 에이전트 플랫폼으로, 일반적인 LLM 서비스를 이용하는 것 보다 한층 더 고차원의 작업을 커스텀할 수 있는게 큰 특징이라 할 수 있다.

## 어떻게 구축했나?

대부분의 설정 내용들을 갓대희 님이 작성하신 아래 내용을 따르고 있다.

https://goddaehee.tistory.com/504

그래서 내가 다른 분들이랑 다르게 한점은, 기존에 집에서 개발용이나 간단한 서비스 운영 ~~친구와 마크 멀티하기~~ 정도로 사용하고 있던 proxmox 서버에 새로운 인스턴스를 생성하여 환경을 분리하였다.

많은 분들이 맥미니 등에다가 로컬 LLM을 설치해서 사용 하기도 하지만, 돈없는 자취생은 그냥 있는걸 사용했다.

~~간혹 평소에 사용하는 맥북에다가 그대로 설치하는 케이스도 보았는데... 외부와의 접근을 수정을 기본 전재로 깔고 가는 서비스여서 무서워서 그러진 못했다.~~

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

에이전트는 크게 두가지로 구분을 하였다.

메인(코토네)은 Gemini 3.0 Flash 를 활용하여, 일반적인 평시 빠른 응답 및 멀티 모달을 활용 할 수 있게 하였고

워커(아사리 선생님)은 Claude Sonnet 4.5을 활용하여, 좀더 복잡한 테스크나 크론잡 등을 수행하도록 적용했다.

~~코토네와 아사리 선생님이 뭔지는 구글에 학원 아이돌마스터에 대해서 검색하면 나올 것이다.~~

### 3. 두 개의 Discord 봇 계정 설정

![image](/assets/images/posts/LLM/2026-02-02-04.png)

봇 설정을 위해서는 Discord 개발자 콘솔에서 두개의 봇을 생성하고 각각의 다른 토큰을 발급 받을 필요성이 있다.

해당 토큰을 아래의 설정 파일의 토큰 항목에 입력하고 각 에이전트별 채널을 할당하면 된다.

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

이후 토큰을 생성하고 OAuth2 URL 생성에서 scope를 bot으로 하여 URL을 생성하고

![image](/assets/images/posts/LLM/2026-02-02-03.png)

원하는 디스코드 서버에 봇을 추가하면 작업이 완료된다,

![image](/assets/images/posts/LLM/2026-02-02-05.png)

필자는 평소에 디스코드를 많이 쓰기에 디스코드에 개인 서버를 생성하여 에이전트를 추가했지만, **절대로 다른 사람이 볼 수 있는 채널**에 에이전트를 추가하는 불상사를 일으키지는 말자.

## 주로 하고 있는 작업들

![image](/assets/images/posts/LLM/2026-02-02-06.png)
![image](/assets/images/posts/LLM/2026-02-02-07.png)

필자는 Gmail과 캘린더, Tasks를 연동하여(gog) 중요한 메일에 대한 필터 알림/요약, 일일 요약, 전반적인 테스크 자동화를 구현하였다.

특히나 OpenClaw의 키 피쳐라고 할 수 있는 크론잡과 하트비트를 활용하였는데, 이에대한 설정에 대해 깊은 이해 없이도 에이전트에 의뢰를 하여 크론잡을 생성 하게 하면 대체로 자동화 하여 생성을 해주게 된다.

단, 일부 웹 스크랩핑이 필요한 작업에 대해서는 일부 사이트 접속시 봇 차단이 발생할 수 있으므로 Chrome CDP를 활용한 헤드리스 설정을 바꿔줘야 한다. (이러한 점을 에이전트에게 인지하여 알려주지 않으면 엉뚱한 유료 API키를 받아와서 설정해달라고 하더라...)

그외에 다른 많은 OpenClaw 유저들은 완전 자동화된 서비스 개발등에 활용하거나 주식/비트코인 트레이딩을 에이전트에게 전담시키는 것 같더라...

## 사용 소감

![image](/assets/images/posts/LLM/2026-02-02-08.png)

**매우 대만족한다.**

아주 오래전부터 이런... **"나만의 똑똑한 AI의 구현!!!!"** 이라는 로망을 가슴 깊숙히 가지고 있었기에 8시간 넘게 앉은 자리에서 신나서 설정을 했다.

다만, 이걸 개발자가 아닌 일반인들에게 추천하기에 무리가 있다고는 보는 점이 기본적으로 동작중인 PC의 전체 컨트롤 권한을 에이전트에 넘기는 것이기에

**자칫 잘못하면 무슨일이 일어날지 예측을 할 수 없다는 점이다.**

아래의 사례처럼 누군가가 악의적으로 메일에 프롬프트 인젝션을 시도할 시 그대로 나의 모든것을 오픈하거나, 서비스에 사용된 중요한 API 키들을 탈취당하여 매우 천문학적인 API 요금이 발생할 수도 있다...

![image](/assets/images/posts/LLM/2026-02-02-09.png)
[OpenClaw 보안 취약사례](https://medium.com/@peltomakiw/how-a-single-email-turned-my-clawdbot-into-a-data-leak-1058792e783a)

그렇기에 절대적으로 나의 소중한 자료가 없는 VPC환경에서 외부의 노출이 없도록 OpenClaw를 쓰는 것이 바람직하겠다...

## 참고 자료

- [OpenClaw 공식 문서](https://docs.openclaw.ai/)
- [OpenClaw GitHub](https://github.com/openclaw/openclaw)
- [ClawdHub - 스킬 마켓플레이스](https://clawdhub.com/)
- [갓대희님 블로그 - Memory 시스템 설명](https://goddaehee.tistory.com/508)
