---

layout: post
title: "[Ovendo] 개발일지 #003 — 지식을 Rule로 바꾸기"
date: 2026-06-22
categories: [ovendo]
tags: [devlog, rule-engine, knowledge, typescript, baking]
---

## 오늘의 개발 주제

이번 작업에서는 Ovendo의 Rule을 추가하면서, **지식을 Rule로 변환하는 흐름**을 정리했다.

이전까지는 이미 정리된 내용을 기반으로 Rule을 만들었다.
하지만 이번에는 직접 내용을 조사하고, Markdown Note로 정리한 뒤, 실행 가능한 Rule JSON으로 변환하는 과정을 처음부터 경험했다.

작업하면서 느낀 점은 명확했다.

> 중요한 건 JSON을 만드는 일이 아니라, 어떤 지식을 Rule로 남길지 결정하는 과정이었다.

---

## 작업 배경

처음에는 새로운 Rule을 바로 추가하려고 했다.

하지만 작업을 진행하다 보니 한 가지 질문이 생겼다.

> 이 Rule은 무엇을 근거로 만들어지는 걸까?

기존에 있던 `cookie_spread` Rule은 이미 정리된 지식을 바탕으로 만들었다.
반면 새 Rule은 근거 없이 바로 만들면 나중에 수정 이유를 추적하기 어려울 수 있었다.

그래서 이번부터는 Rule을 바로 만들지 않고, 먼저 지식을 정리한 뒤 Rule로 변환하는 방식으로 진행하기로 했다.

---

## 새로 정한 흐름

이번 작업부터 아래 흐름을 기준으로 잡았다.

```text
Research
↓
Note (.md)
↓
Rule (.json)
↓
Search
↓
Answer
```

조사한 내용은 Markdown Note로 남긴다.
그리고 실제 프로그램에서 사용할 수 있는 구조는 Rule JSON으로 변환한다.

이렇게 분리하면 Rule이 수정되더라도 다음 내용을 추적할 수 있다.

* 어떤 자료를 바탕으로 만들었는지
* 왜 이런 조건이 들어갔는지
* 어떤 증상과 연결되는지
* 이후 어떤 Rule과 충돌하거나 연결되는지

---

## cookie_spread Rule 보강

먼저 기존 `cookie_spread` 관련 내용을 다시 정리했다.

`cookie-spread-control.md` Note를 다시 보면서 Rule도 함께 보강했다.

변경한 내용은 다음과 같다.

* 진단 순서 재정리
* 추천 항목 정리
* symptom별 진단 우선순위 조정
* 제약 조건 추가
* texture Rule과의 관계 정의

기존에는 쿠키가 퍼지지 않는 현상 자체에 집중했다.
하지만 작업을 하다 보니 “퍼짐 문제”와 “식감 문제”는 분리해서 다루는 편이 더 자연스럽다는 생각이 들었다.

예를 들어 쿠키가 딱딱한 문제와 쿠키가 퍼지지 않는 문제는 함께 나타날 수 있다.
하지만 두 문제는 같은 원인으로만 설명할 수 없다.

그래서 `cookie_spread`는 퍼짐 문제를 중심으로 유지하고, 식감 관련 문제는 별도의 Rule로 분리하기로 했다.

---

## cookie_texture Rule 추가

새로운 Note를 바탕으로 `cookie_texture` Rule을 추가했다.

현재 `cookie_texture` Rule에서 다루는 문제는 다음과 같다.

* 딱딱한 식감
* 퍽퍽한 식감
* 너무 부드러운 식감
* 케이크 같은 식감

현재는 Rule Search 단계에서 매칭까지 동작한다.

예를 들어 다음 질문을 입력하면,

```text
쿠키가 딱딱해요
```

`cookie_texture` Rule이 매칭된다.

```text
Matched Rule: cookie_texture
```

다만 아직 classifier는 texture 문제의 세부 variant를 이해하지 못한다.

현재 출력은 다음과 같다.

```text
Intent: unknown
Variant: none
Matched Rule: cookie_texture
```

Rule Search는 동작하지만, 질문 분류 단계에서 식감 문제를 충분히 해석하지 못하고 있는 상태다.

이 부분은 다음 작업에서 개선할 예정이다.

---

## 구조 변화

Rule이 하나뿐일 때는 구조 분리의 필요성이 크게 느껴지지 않았다.

하지만 두 번째 Rule이 추가되면서 Rule 관리 방식도 조금 정리할 필요가 생겼다.

현재 흐름은 다음과 같다.

```text
Question
↓
classifyQuestion()
↓
searchRules()
↓
RuleSearchResult
↓
buildAnswer()
↓
AnswerResult
↓
CLI
```

이번 작업에서는 Rule을 Registry를 통해 관리하도록 정리했다.

Search는 개별 Rule 파일을 직접 바라보는 것이 아니라, Registry를 통해 Rule 목록에 접근한다.

이렇게 분리하면 앞으로 Rule이 늘어나도 Search 로직이 특정 Rule에 직접 의존하지 않게 된다.

---

## 발견한 문제

이번 작업에서 발견한 문제는 classifier와 Rule Search의 역할 차이다.

현재는 `cookie_texture` Rule이 검색 단계에서는 매칭되지만, classifier는 여전히 다음과 같이 출력한다.

```text
Intent: unknown
Variant: none
```

즉, 시스템은 관련 Rule을 찾을 수 있지만, 질문의 의도와 세부 유형은 아직 제대로 분류하지 못한다.

이 문제를 해결하려면 classifier가 texture 관련 질문을 이해할 수 있도록 확장되어야 한다.

예를 들어 앞으로는 다음과 같은 분류가 가능해야 한다.

```text
Question: 쿠키가 딱딱해요

Intent: troubleshooting
Variant: hard_texture
Matched Rule: cookie_texture
```

---

## 이번 작업에서 배운 점

이번 작업에서 가장 크게 느낀 것은 Rule 자체보다 **지식 관리 방식**이 더 중요하다는 점이었다.

JSON Rule은 실행을 위한 데이터다.
반면 Markdown Note는 Rule의 근거를 남기는 기록이다.

둘을 분리하면 Rule이 단순한 설정 파일이 아니라, 근거를 가진 지식 구조로 관리될 수 있다.

정리하면 다음과 같다.

| 항목             | 역할                  |
| -------------- | ------------------- |
| Note           | 조사 내용과 판단 근거 기록     |
| Rule           | 프로그램에서 사용하는 실행용 데이터 |
| Search         | 질문과 관련된 Rule 찾기     |
| Answer Builder | Rule 기반 답변 구조 만들기   |

Rule이 늘어날수록 중요한 것은 단순히 Rule을 많이 추가하는 것이 아니다.

> 왜 이 Rule이 만들어졌는지 설명할 수 있는 구조를 유지하는 것

이 점이 앞으로 Ovendo에서 중요해질 것 같다.

---

## 다음 작업

다음 작업에서는 `cookie_texture` 관련 질문을 classifier가 더 잘 이해하도록 개선할 예정이다.

예정 작업은 다음과 같다.

* texture 관련 intent 분류 추가
* hard / dry / soft / cakey 같은 variant 정의
* `cookie_texture` Rule과 classifier 결과 연결
* CLI 출력에서 intent, variant, matchedRule 관계 확인

현재는 Rule Search가 먼저 가능성을 보여준 단계다.
다음 단계에서는 질문 분류와 Rule Search가 더 자연스럽게 이어지도록 정리할 예정이다.
