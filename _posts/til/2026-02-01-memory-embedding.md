---
title: "TIL: 로컬 임베딩으로 AI 기억력 향상시키기"
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

**사용한 모델**

- `hf:ggml-org/embeddinggemma-300M-GGUF/embeddinggemma-300M-Q8_0.gguf`
- 300M 파라미터 (Q8 양자화)
- 로컬 실행 (외부 API 불필요!)

## 🧪 테스트 결과

다양한 주제로 검색 테스트를 진행했다

| 검색어                            | 결과    | 점수 |
| --------------------------------- | ------- | ---- |
| "크론잡 멀티봇"                   | ✅ 완벽 | 0.46 |
| "Gmail 10분 긴급 체크"            | ✅ 완벽 | 0.46 |
| "Proxmox LXC 브라우저"            | ✅ 완벽 | 0.67 |
| "ㅁㅁㅁ(에이전트 이름) 칭찬 감사" | ✅ 양호 | 0.47 |

정확한 키워드를 쓰지 않아도, **의미**만으로 에이전트는 과거의 맥락을 찾아 낼 수 있었다.

## 🎯 장점

1. **시맨틱 검색:** 정확한 단어 없이도 의미로 검색 가능
2. **과거 메모리 검색:** 여러 날의 메모리를 한 번에 검색
3. **관련도 점수:** score로 정확도 판단
4. **로컬 실행:** API 비용 없음

## 💭 느낀 점

AI 에이전트가 **기억을 잘한다**는 것의 핵심은 단순히 저장하는 게 아니라, **필요할 때 꺼낼 수 있어야 한다**는 것!

벡터 임베딩이 그 해결책이었다.

이제 에이전트는

- "오늘 뭐 했더라?" ❌
- "비슷한 상황이 언제였지?" ✅

이렇게 질문할 수 있게 되었다!

## ⚙️ 세팅 방법

OpenClaw에서 메모리 임베딩을 활성화하는 방법은 매우 간단하다!

### 1. OpenClaw 설정 파일 수정

`~/.openclaw/openclaw.json` 파일을 열어서 다음 설정을 추가

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

**주요 옵션:**

- `enabled: true` - 메모리 검색 활성화
- `provider: "local"` - 로컬 임베딩 모델 사용 (무료!)
- `local.modelPath` - 사용할 임베딩 모델 경로 (HuggingFace GGUF)
- `query.hybrid` - 하이브리드 검색 설정 (Vector + BM25)
- `vectorWeight: 0.7` - 의미 검색 가중치 70%
- `textWeight: 0.3` - 키워드 검색 가중치 30%
- `cache.enabled` - 임베딩 캐시 활성화 (성능 향상)
- `watch: true` - 파일 변경 자동 감지

### 2. Gateway 재시작

설정 변경 후 OpenClaw Gateway를 재시작

```bash
openclaw gateway restart
```

### 3. 임베딩 모델 다운로드

처음 실행 시 자동으로 모델이 다운로드됩니다. (약 300MB)

```bash
# 각 에이전트별 메모리 상태 확인  (모델 자동 다운로드)
openclaw memory status --deep --agent main
openclaw memory status --deep --agent worker

# 인덱싱 실행
openclaw memory index --agent main --verbose
openclaw memory index --agent worker --verbose
```

### 4. 테스트

이후 에이전트에 의뢰해서 `memory_search` 도구를 사용하여 검색 테스트

```javascript
// 에이전트에서 사용
memory_search({
  query: "크론잡 설정",
  maxResults: 5,
});
```

~~사실 이러한 설정내용을 사용자가 모르더라도 AI들한테 해달라고하면 대부분은 해준다~~

~~해야하는 내용은 위의 메모리 다운로드와 인덱싱 정도~~

## 📚 참고 자료

- [OpenClaw Memory Search 도구](https://docs.openclaw.ai/concepts/memory#memory)
- [Embedding 모델: embeddinggemma-300M](https://huggingface.co/ggml-org/embeddinggemma-300M-GGUF)
