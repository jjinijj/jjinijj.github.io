---

layout: post
title: "[Ovendo] 개발일지 #002 — 질문 분류에서 Rule Search로"
date: 2026-06-20
categories: [ovendo, development-log]
tags: [rule-engine, typescript, ai]
---

# Ovendo 개발일지 #002 - 질문 분류에서 Rule Search로

이번 작업에서는 질문을 받아 답변을 만드는 흐름을 조금 더 구조화했다.

처음에는 질문을 분류하는 코드 안에서 직접 Rule을 결정했다.

예를 들면 질문에 특정 키워드가 들어오면 코드에서 바로 `cookie_spread`를 반환하는 방식이었다.

처음 구조를 검증하기에는 편했지만, Rule이 늘어나기 시작하면 결국 코드 수정도 같이 늘어나는 구조였다.

그래서 이번에는 Rule 선택 기준을 코드에서 꺼내 JSON으로 옮겼다.

---

## 이전 구조

```text
Question
↓
classifyQuestion()
↓
ruleId 직접 결정
↓
buildAnswer()
```

질문 분류와 Rule 선택이 같은 단계에 있었다.

결과적으로 Rule이 늘어나면 classifier도 계속 수정해야 했다.

---

## 변경한 구조

```text
Question
↓
classifyQuestion()
↓
findRule(question)
↓
buildAnswer()
```

질문 분류는 질문 자체를 이해하는 역할만 남기고,

Rule 선택은 별도 단계로 분리했다.

---

## Rule을 JSON 기준으로 찾기

Rule 선택 기준은 `cookie-spread.json` 안의 `match.keywords`로 이동했다.

예시:

```json
{
  "match": {
    "keywords": [
      "쿠키",
      "퍼",
      "안 퍼",
      "납작",
      "크기",
      "달라"
    ]
  }
}
```

이제 새로운 Rule을 추가할 때 TypeScript 코드를 수정하기보다 Rule 데이터를 추가하는 방향으로 갈 수 있게 됐다.

아직은 단순 키워드 매칭이지만 구조 자체는 이후 확장할 수 있도록 바뀌었다.

---

## Variant 도입

같은 Rule이라도 결과를 다르게 만들기 위해 Variant 개념도 추가했다.

예를 들어 모두 `cookie_spread`로 연결되더라도 내부적으로는 다른 증상으로 분류한다.

```text
cookie_spread

├─ no_spread
├─ over_spread
└─ inconsistent_spread
```

현재 예시:

```text
쿠키가 안 퍼져요
→ no_spread

쿠키가 너무 퍼져요
→ over_spread

쿠키마다 크기가 달라요
→ inconsistent_spread
```

---

## 답변도 Variant에 따라 분기

Variant가 추가되면서 진단 순서와 추천도 분리했다.

이전에는 어떤 질문이 들어와도 같은 답을 반환했다.

지금은 Variant별로 다른 diagnostics / recommendations를 사용한다.

아직 데이터는 간단하지만 구조 자체는 만들어졌다.

---

## 이번 작업에서 얻은 것

이번 작업의 목표는 답변 품질 개선보다 구조 분리였다.

* 질문 분류와 Rule 선택 분리
* Rule을 JSON 기반으로 관리
* Variant 기반 분기 추가
* 이후 Embedding / Retrieval 확장을 위한 기반 마련

지금은 여전히 키워드 기반이다.

하지만 이제 Rule을 찾는 방식은 바꿀 수 있어도 Answer 구조는 유지할 수 있는 형태가 됐다.

다음 단계에서는 Rule Search 결과를 더 설명 가능하게 만들거나, Embedding 기반 탐색으로 확장해볼 예정이다.
