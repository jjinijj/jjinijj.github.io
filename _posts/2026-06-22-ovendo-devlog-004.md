---

layout: post
title: "[Ovendo] 개발일지 #004 — Rule 엔진을 안정화하기"
date: 2026-06-22
categories: [ovendo]
tags: [devlog, rule-engine, testing, validation, architecture, typescript]
---

## 오늘의 개발 주제

이번 작업에서는 새로운 기능을 많이 추가하지 않았다.

대신 지금까지 만든 Rule 기반 구조가 계속 유지될 수 있도록 구조를 정리하고, 안전장치를 추가했다.

이번 단계의 목표는 하나였다.

> Rule이 늘어나도 코드를 계속 수정하지 않는 구조 만들기

Ovendo가 단순한 조건 분기 코드가 아니라, 실제 Rule 엔진처럼 확장될 수 있도록 구조를 조금 더 안정화하는 작업이었다.

---

## 작업 배경

이전 단계에서는 `cookie_spread`, `cookie_texture` Rule을 추가하면서 Rule이 하나에서 둘로 늘어났다.

Rule이 하나뿐일 때는 코드 안에서 조건을 직접 관리해도 큰 문제가 없었다.

하지만 Rule이 늘어나기 시작하자 문제가 보이기 시작했다.

같은 지식이 코드와 JSON 양쪽에 흩어질 수 있었다.

예를 들어 texture 문제를 추가하면 다음처럼 중복이 생겼다.

```text
classifyQuestion()
→ texture 관련 키워드 관리

cookie_texture.json
→ texture 관련 Rule 정의
```

이렇게 되면 Rule을 수정할 때 TypeScript 코드도 같이 수정해야 한다.

처음에는 괜찮아 보여도 Rule이 늘어날수록 유지보수가 어려워질 수밖에 없다.

그래서 이번 작업에서는 지식을 최대한 Rule JSON 쪽으로 옮기는 방향으로 구조를 정리했다.

---

## Variant 판별 책임 이동

처음 구조에서는 variant 판별이 코드 안에 있었다.

```text
Question
↓
classifyQuestion()
↓
variant 결정
```

하지만 이 구조에서는 Rule이 늘어날수록 `classifyQuestion()`이 점점 많은 지식을 알아야 한다.

그래서 variant 판별 책임을 Rule Search 쪽으로 이동했다.

현재 방향은 다음과 같다.

```text
Question
↓
Intent 분류
↓
Rule Search
↓
Variant 결정
↓
Answer
```

`classifyQuestion()`은 질문의 큰 의도를 판단하고,
세부 variant는 Rule Search가 Rule 데이터를 기반으로 판단하도록 역할을 나누었다.

---

## Variant 키워드를 JSON으로 이동

다음으로 variant 키워드를 TypeScript 코드에서 Rule JSON으로 옮겼다.

이전 구조는 다음과 같았다.

```text
TypeScript
└ variant 키워드 관리
```

현재 구조는 다음과 같다.

```text
Rule JSON
├ keywords
├ symptoms
└ variantKeywords
```

예시는 다음과 같다.

```json
{
  "match": {
    "variantKeywords": {
      "hard_texture": [
        "딱딱",
        "단단"
      ]
    }
  }
}
```

이제 새로운 Rule을 추가할 때 TypeScript 코드를 직접 수정하는 일이 줄어들었다.

앞으로 Rule 추가 흐름은 다음처럼 가져갈 수 있다.

```text
Note 작성
↓
Rule JSON 생성
↓
Registry 등록
```

즉, 지식은 코드보다 Rule 데이터에 더 가까운 위치에서 관리된다.

---

## Rule JSON 검증 추가

Rule이 늘어날수록 가장 걱정되는 부분은 잘못된 JSON이다.

예를 들어 variant 이름에 오타가 있을 수 있다.

```text
hard_textuer
```

이런 오타는 TypeScript 컴파일 단계에서 바로 잡히지 않을 수 있다.
하지만 런타임에서는 Rule Search나 Answer Builder 동작에 영향을 줄 수 있다.

그래서 Rule 등록 시 자동 검증을 추가했다.

현재 검증하는 항목은 다음과 같다.

* 필수 필드 존재 여부
* 중복 rule id 여부
* `symptoms`와 `variantKeywords`의 일관성
* `variantMappings`와 `symptoms`의 일관성

잘못된 Rule이 등록되면 바로 실패하도록 했다.

예시는 다음과 같다.

```text
Invalid Rule [cookie_texture]:
missing variantKeywords
```

이 검증을 통해 Rule 파일이 늘어나더라도 최소한의 구조적 안전성을 확보할 수 있다.

---

## 자동 테스트 도입

이전까지는 CLI로 직접 질문을 입력하면서 동작을 확인했다.

```text
npm run dev
```

하지만 Rule이 늘어나면 매번 직접 확인하는 방식은 한계가 있다.

그래서 최소한의 자동 테스트를 추가했다.

```text
npm test
```

현재 테스트하는 항목은 다음과 같다.

* spread 질문
* texture 질문
* 매칭 실패 케이스
* 복합 질문

예를 들어 다음 질문을 테스트한다.

```text
쿠키가 작아요 딱딱해요
```

이 경우 현재 기대 흐름은 다음과 같다.

```text
cookie_spread 우선
↓
cookie_texture 후보 유지
```

테스트가 통과하면 기존 Rule Search 동작이 유지되고 있다는 것을 확인할 수 있다.

---

## 현재 구조

현재 Ovendo의 질문 처리 흐름은 다음과 같다.

```text
Question
↓
Intent
↓
Rule Search
↓
Variant
↓
Answer
↓
CLI
```

Rule 관리 흐름은 다음과 같다.

```text
Research
↓
Note
↓
Rule JSON
↓
Validation
↓
Search
↓
Answer
```

이번 작업을 통해 Rule은 단순히 JSON 파일로 존재하는 것이 아니라,
검증 과정을 거쳐 검색과 답변 생성에 사용되는 구조로 정리되었다.

---

## 이번 작업에서 얻은 것

이번 작업에서 얻은 것은 새로운 기능보다 유지보수성이었다.

Rule이 하나일 때는 코드로 관리해도 괜찮았다.
하지만 두 번째 Rule이 생기면서 지식이 코드 밖으로 이동해야 한다는 점이 분명해졌다.

이번 정리를 통해 Ovendo는 단순한 조건 분기 구조에서 조금 더 Rule 엔진에 가까운 구조로 이동했다.

특히 다음 세 가지가 중요했다.

| 항목                  | 의미                       |
| ------------------- | ------------------------ |
| Variant 키워드 JSON 이동 | 지식을 코드가 아닌 Rule 데이터에서 관리 |
| Rule Validation 추가  | 잘못된 Rule 등록을 초기에 차단      |
| 자동 테스트 추가           | 기존 동작이 유지되는지 반복 확인       |

Rule이 늘어날수록 중요한 것은 단순히 Rule을 추가하는 속도가 아니다.

> Rule이 늘어나도 구조가 무너지지 않게 만드는 것

이번 작업은 그 기반을 만드는 단계였다.

---

## 다음 작업

다음 단계에서는 테스트를 더 늘리거나, Rule 생성 흐름을 더 정리할 예정이다.

예정 작업은 다음과 같다.

* Rule Search 테스트 케이스 추가
* 복합 질문 처리 방식 개선
* Rule 생성 흐름 정리
* Note에서 Rule JSON으로 변환하는 기준 정리
* Rule Registry 구조 점검

이번에는 눈에 보이는 기능보다 내부 구조를 다듬은 작업이었다.
하지만 앞으로 Rule이 많아질수록 이번 안정화 작업이 중요한 기반이 될 것 같다.
