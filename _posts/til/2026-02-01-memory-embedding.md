---
title: "TIL: 시맨틱 검색으로 AI 기억력 향상시키기"
last_modified_at: 2026-02-02T01:42:00
categories:
  - TIL
tags:
  - AI
  - OpenClaw
  - Embedding
  - Vector Search
  - Memory System
---

오늘 OpenClaw에 메모리 임베딩(Memory Embedding) 시스템을 적용했다!

## 🧠 기존 메모리 시스템의 한계

기존에는 AI 에이전트가 메모리를 텍스트 파일로만 저장하고 있었다.

```
memory/2026-02-01.md
memory/2026-01-31.md
MEMORY.md
```

문제는 **검색**이었다. 정확한 키워드를 알아야만 찾을 수 있었고, 비슷한 상황이나 관련된 내용을 찾기 어려웠다.

## 💡 해결책: Vector Embedding + 시맨틱 검색

메모리 파일들을 **벡터 임베딩**으로 변환하고, 의미 기반 검색이 가능하도록 개선했다!

**사용한 모델:**

- `hf:ggml-org/embeddinggemma-300M-GGUF/embeddinggemma-300M-Q8_0.gguf`
- 300M 파라미터 (Q8 양자화)
- 로컬 실행 (외부 API 불필요!)

## 🧪 테스트 결과

다양한 주제로 검색 테스트를 진행했다:

| 검색어                            | 결과    | 점수 |
| --------------------------------- | ------- | ---- |
| "크론잡 멀티봇"                   | ✅ 완벽 | 0.46 |
| "Gmail 10분 긴급 체크"            | ✅ 완벽 | 0.46 |
| "Proxmox LXC 브라우저"            | ✅ 완벽 | 0.67 |
| "ㅁㅁㅁ(에이전트 이름) 칭찬 감사" | ✅ 양호 | 0.47 |

**특히 놀라운 점:**

정확한 키워드("AppArmor", "Seccomp")를 쓰지 않아도, **의미**만으로 찾아냈다!

## 🎯 장점

1. **시맨틱 검색:** 정확한 단어 없이도 의미로 검색 가능
2. **과거 메모리 검색:** 여러 날의 메모리를 한 번에 검색
3. **관련도 점수:** score로 정확도 판단
4. **로컬 실행:** API 비용 없음

## 💭 느낀 점

AI 에이전트가 **기억을 잘한다**는 것의 핵심은 단순히 저장하는 게 아니라, **필요할 때 꺼낼 수 있어야 한다**는 것!

벡터 임베딩이 그 해결책이었다.

이제 에이전트는:

- "오늘 뭐 했더라?" ❌
- "비슷한 상황이 언제였지?" ✅

이렇게 질문할 수 있게 되었다!

## 📚 참고 자료

- [OpenClaw Memory Search 도구](https://docs.openclaw.ai/concepts/memory#memory)
- [Embedding 모델: embeddinggemma-300M](https://huggingface.co/ggml-org/embeddinggemma-300M-GGUF)
