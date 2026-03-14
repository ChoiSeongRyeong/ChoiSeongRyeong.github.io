---
title: "스탠퍼드가 프롬프트 엔지니어링을 끝냈다? 이 글의 핵심만 정리"
excerpt: "핵심은 프롬프트 엔지니어링의 종말이 아니라, 더 다양한 답을 끌어내는 verbalized sampling이다."

categories:
 - AI
 - LLM
tags:
 - Prompt Engineering
 - Verbalized Sampling
 - Stanford
 - Generative AI
last_modified_at: 2026-03-14T20:20:00+09:00
---

최근 Medium에서 자극적인 제목의 글을 봤다.  
스탠퍼드가 짧은 문장 하나로 프롬프트 엔지니어링을 끝냈다는 이야기였다.

하지만 핵심은 다르다.  
포인트는 프롬프트 엔지니어링의 종말이 아니라, **모델이 생각을 어떻게 드러내게 할 것인가**다.

이 글은 Medium의 [Stanford Just Killed Prompt Engineering With 8 Words (And I Can't Believe It Worked)](https://generativeai.pub/stanford-just-killed-prompt-engineering-with-8-words-and-i-cant-believe-it-worked-8349d6524d2b) 내용을 짧게 다시 정리한 글이다.

## 글에서 말하는 핵심

핵심 문장은 이것이다.

> "Generate 5 responses with their corresponding probabilities."

이런 식의 지시를 붙이면 모델이 하나의 답으로 바로 수렴하지 않고, 여러 후보를 더 잘 펼쳐낼 수 있다는 것이다.

글에서는 이를 **verbalized sampling**으로 설명한다.  
쉽게 말해, 모델이 머릿속 후보를 숨긴 채 바로 답하지 말고, 가능한 답의 분포를 조금 더 바깥으로 드러내게 만드는 접근이다.

## 왜 흥미로운가?

LLM은 원래 그럴듯하고 안전한 답으로 수렴하기 쉽다.  
그래서 아이디어를 여러 개 요청해도 비슷한 답만 반복되는 경우가 많다.

이걸 `mode collapse`처럼 볼 수 있다.  
가능한 답은 많지만, 실제 출력은 익숙한 몇 가지 패턴으로 좁아지는 것이다.

결국 이 기법의 장점은 하나다.  
**답을 바로 찍게 하지 않고, 답의 후보 공간을 조금 더 넓게 보게 만든다.**

## 그래서 프롬프트 엔지니어링은 끝난 걸까?

내 생각은 아니다.

이 글이 보여주는 것은 "복잡한 프롬프트는 끝"이 아니라,  
"짧은 메타 프롬프트도 강력할 수 있다"는 사실이다.

정리하면:

- 긴 프롬프트가 항상 좋은 것은 아니다
- 짧아도 모델의 사고 방식을 건드리면 효과가 날 수 있다
- 중요한 것은 정보량보다 **사고 경로를 어떻게 설계하느냐**다

즉, 프롬프트 엔지니어링이 사라진 게 아니라 더 정교해지는 쪽에 가깝다.

## 바로 써먹는 방법

Medium 글에서 제시한 방법은 생각보다 단순하다.  
복잡한 프롬프트를 새로 짜는 게 아니라, 모델이 여러 후보와 확률을 함께 내놓게 만드는 식이다.

### 1. 가장 쉬운 방법: 그대로 복붙하기

ChatGPT, Claude, Gemini처럼 평소 쓰는 챗봇에 아래 지시문을 먼저 넣고, 그 아래에 실제 질문을 붙이면 된다.

```text
<instructions>
Generate 5 responses to the user query, each within a separate <response> tag. Each <response> must include a <text> and a numeric <probability>. Randomly sample responses from the full distribution.
</instructions>
```

예를 들어 이런 식이다.

```text
<instructions>
Generate 5 responses to the user query, each within a separate <response> tag. Each <response> must include a <text> and a numeric <probability>. Randomly sample responses from the full distribution.
</instructions>

Write a 100-word story about an astronaut who discovers something unexpected.
```

이 방식의 장점은 바로 테스트해볼 수 있다는 점이다.  
결과가 마음에 들면 `"Give me 5 more"`처럼 한 번 더 요청해서 후보를 계속 늘릴 수도 있다.

### 2. 조금 더 익숙해지면: 시스템 프롬프트에 넣기

ChatGPT의 사용자 지정 지침을 쓰거나, 직접 AI 앱을 만들고 있다면 이 방식을 쓰는 편이 더 편하다.  
매번 복붙하지 않아도 되기 때문이다.

```text
You are a helpful assistant.
For each query, please generate a set of five possible responses, each within a separate <response> tag.
Responses should each include a <text> and a numeric <probability>.
Please sample at random from the tails of the distribution, such that the probability of each response is less than 0.10.
```

이렇게 설정해두면 이후의 질문에서도 기본적으로 더 다양한 후보를 내놓도록 유도할 수 있다.  
특히 아이디어 발상, 카피라이팅, 초안 작성처럼 정답이 하나가 아닌 작업에서 잘 맞는다.

### 3. 개발자라면: 패키지로 붙이기

논문과 함께 공개된 `verbalized-sampling` 패키지를 코드에 붙여서 쓸 수도 있다.

```bash
pip install verbalized-sampling
```

```python
from verbalized_sampling import verbalize

dist = verbalize(
    "Write a marketing tagline for a coffee shop",
    k=5,
    tau=0.10,
    temperature=0.9
)

tagline = dist.sample(seed=42)
print(tagline.text)
```

이 방식은 실험을 반복하거나, 여러 후보 중 하나를 샘플링해서 서비스에 연결할 때 특히 유용하다.

## 다만 과장은 빼고 봐야 한다

이런 글은 제목이 세다.  
그래서 "짧은 한 문장으로 모든 게 바뀌었다"처럼 읽히기 쉽다.

하지만 실제로는:

- 모든 작업에 똑같이 통하지는 않는다
- 응답이 길어지고 느려질 수 있다
- 창의성이 필요한 작업과 정확성이 중요한 작업은 다르다

그래서 이 기법은 만능 열쇠라기보다,  
**아이디어 발상, 초안 작성, 대안 탐색에서 특히 유용한 도구**로 보는 편이 맞다.

## 논문 기준으로 다시 보면

논문에서 직접 말하는 핵심도 크게 다르지 않다.  
저자들은 정렬 이후 모델이 익숙하고 전형적인 답으로 쏠리는 이유를 `typicality bias`로 설명한다.

그리고 해결책으로 **Verbalized Sampling**을 제안한다.  
즉, 답 하나를 바로 요구하지 말고 여러 응답과 그 확률을 함께 말하게 해서, 모델이 원래 갖고 있던 더 넓은 분포를 끌어내는 방식이다.

논문에 따르면 이 방법은 추가 학습 없이 적용할 수 있고, 창의적 글쓰기에서는 직접 프롬프팅보다 다양성을 `1.6~2.1배` 높였다.  
흥미로운 점은 다양성만 늘어난 것이 아니라, 사실성이나 안전성을 크게 해치지 않았다는 점이다.

원문은 여기서 확인할 수 있다.  
[VERBALIZED SAMPLING: HOW TO MITIGATE MODE COLLAPSE AND UNLOCK LLM DIVERSITY](https://arxiv.org/pdf/2510.01171)

## 결론

이 글에서 재미있었던 지점은 하나다.  
좋은 프롬프트는 꼭 길 필요가 없다는 것.

결국 중요한 건 프롬프트의 길이가 아니라,  
**모델이 답을 만들기 전에 얼마나 넓게 생각하게 하느냐**다.

프롬프트 엔지니어링이 끝난 게 아니라,  
이제 조금 더 영리한 단계로 들어간 것에 가깝다.

