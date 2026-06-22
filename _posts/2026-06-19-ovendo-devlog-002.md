---

layout: post
title: "[Ovendo] 개발일지 #002 — 질문 분류에서 Rule Search로"
date: 2026-06-19
categories: [ovendo]
tags: [devlog, rule-engine, rule-search, typescript, ai]
---

## 오늘의 개발 주제

이번 작업에서는 질문을 받아 답변을 만드는 흐름을 조금 더 구조화했다.

이전까지는 질문을 분류하는 코드 안에서 직접 Rule을 결정했다.

예를 들어 질문에 특정 키워드가 들어오면 코드에서 바로 `cookie_spread`를 반환하는 방식이었다.

처음 구조를 검증하기에는 편했지만, Rule이 늘어나면 classifier 코드도 계속 수정해야 하는 구조였다.

그래서 이번 작업에서는 **Rule 선택 기준을 코드에서 분리하고, JSON 기반 Rule Search로 옮기는 것**을 목표로 했다.

---

## 작업 배경

처음 구조는 단순했다.

```text
Question
↓
classifyQuestion()
↓
ruleId 직접 결정
↓
buildAnswer()
```

이 구조에서는 `classifyQuestion()`이 두 가지 역할을 동시에 하고 있었다.

* 질문의 의도 파악
* 어떤 Rule을 사용할지 결정

Rule이 하나뿐일 때는 문제가 크지 않았다.
하지만 앞으로 Rule이 늘어나면 질문 분류 코드가 계속 커질 수밖에 없다.

예를 들어 새로운 Rule을 추가할 때마다 TypeScript 코드 안에 조건문을 추가해야 한다면, Rule 기반 구조의 장점이 줄어든다.

그래서 질문 분류와 Rule 선택을 분리하기로 했다.

---

## 변경한 구조

이번 작업 이후 흐름은 다음과 같이 바뀌었다.

```text
Question
↓
classifyQuestion()
↓
findRule(question)
↓
buildAnswer()
```

`classifyQuestion()`은 질문 자체를 이해하는 역할에 집중한다.
Rule을 찾는 일은 `findRule()` 단계로 분리했다.

이렇게 하면 이후 Rule Search 방식을 바꾸더라도 전체 Answer 구조는 유지할 수 있다.

예를 들어 지금은 키워드 기반으로 Rule을 찾지만, 나중에는 Embedding이나 Retrieval 방식으로 확장할 수 있다.

---

## Rule을 JSON 기준으로 찾기

Rule 선택 기준은 TypeScript 코드가 아니라 Rule JSON 안으로 이동했다.

`cookie-spread.json`에는 `match.keywords`를 추가했다.

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

이제 Rule Search는 질문과 Rule의 `keywords`를 비교해서 관련 Rule을 찾는다.

아직은 단순 키워드 매칭이지만, 중요한 점은 Rule을 찾는 기준이 코드 밖으로 이동했다는 것이다.

앞으로 새로운 Rule을 추가할 때는 TypeScript 코드를 수정하기보다 Rule 데이터를 추가하는 방향으로 갈 수 있다.

---

## Variant 도입

이번 작업에서는 Variant 개념도 추가했다.

같은 Rule로 연결되는 질문이라도 세부 증상이 다를 수 있기 때문이다.

예를 들어 모두 `cookie_spread` Rule과 관련되지만, 실제 문제 유형은 다를 수 있다.

```text
cookie_spread

├─ no_spread
├─ over_spread
└─ inconsistent_spread
```

현재 예시는 다음과 같다.

```text
쿠키가 안 퍼져요
→ no_spread

쿠키가 너무 퍼져요
→ over_spread

쿠키마다 크기가 달라요
→ inconsistent_spread
```

즉, Rule은 같지만 질문의 세부 유형에 따라 다른 진단 흐름을 사용할 수 있게 되었다.

---

## Variant에 따른 답변 분기

Variant가 추가되면서 답변 구성도 함께 바뀌었다.

이전에는 어떤 질문이 들어와도 같은 Rule에 연결되면 거의 같은 답변을 반환했다.

하지만 이제는 Variant에 따라 다른 diagnostics와 recommendations를 사용할 수 있다.

예를 들어 `cookie_spread` Rule 안에서도 다음처럼 나눌 수 있다.

| Variant               | 의미                    |
| --------------------- | --------------------- |
| `no_spread`           | 쿠키가 충분히 퍼지지 않는 문제     |
| `over_spread`         | 쿠키가 너무 많이 퍼지는 문제      |
| `inconsistent_spread` | 쿠키마다 크기나 퍼짐 정도가 다른 문제 |

아직 데이터는 간단하지만, Variant 기반으로 답변을 분기할 수 있는 구조는 만들어졌다.

---

## 현재 구조

현재 질문 처리 흐름은 다음과 같다.

```text
Question
↓
classifyQuestion()
↓
findRule(question)
↓
Rule
↓
Variant
↓
buildAnswer()
↓
AnswerResult
```

이번 작업의 핵심은 질문 분류, Rule Search, 답변 생성을 분리한 것이다.

각 단계의 역할은 다음과 같다.

| 단계                   | 역할                           |
| -------------------- | ---------------------------- |
| `classifyQuestion()` | 질문의 큰 의도 파악                  |
| `findRule()`         | 질문과 관련된 Rule 검색              |
| Variant 결정           | 같은 Rule 안에서 세부 증상 분류         |
| `buildAnswer()`      | Rule과 Variant를 기반으로 답변 구조 생성 |

---

## 이번 작업에서 얻은 것

이번 작업의 목표는 답변 품질을 바로 높이는 것이 아니라, 구조를 분리하는 것이었다.

이번에 정리한 내용은 다음과 같다.

* 질문 분류와 Rule 선택 분리
* Rule 선택 기준을 JSON으로 이동
* Variant 기반 세부 분기 추가
* 이후 Embedding / Retrieval 확장을 위한 기반 마련

지금은 여전히 키워드 기반이다.
하지만 이제 Rule을 찾는 방식을 바꾸더라도 Answer 구조는 유지할 수 있는 형태가 되었다.

이 점이 이번 작업에서 가장 중요한 변화였다.

---

## 다음 작업

다음 단계에서는 Rule Search 결과를 더 설명 가능하게 만들 예정이다.

예정 작업은 다음과 같다.

* Rule Search 결과 구조 정리
* 어떤 키워드로 Rule이 매칭되었는지 표시
* Variant 판단 기준 정리
* Rule이 늘어났을 때의 관리 방식 고민
* Embedding 기반 탐색 가능성 검토

이번 작업을 통해 Ovendo는 단순한 조건 분기에서 벗어나, Rule을 검색하고 선택하는 구조로 한 단계 이동했다.
