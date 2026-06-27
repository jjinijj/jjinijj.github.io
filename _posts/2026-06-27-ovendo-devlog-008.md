---

layout: post
title: "[Ovendo] 개발일지 #008 — Rule Coverage 확장과 다음 구조 정리"
date: 2026-06-27
categories: [ovendo]
tags: [devlog, rule-engine, rule-coverage, metadata, rag, testing, architecture]
---

## 오늘의 개발 주제

이번 작업에서는 Ovendo의 Rule Coverage를 확장했다.

이전까지는 Rule Engine v1이 안정적으로 동작하는지, 기존 Rule을 검색하고 답변으로 연결할 수 있는지가 중심이었다.

이번에는 그 구조 위에 실제 Rule을 추가하면서, Rule이 늘어나도 현재 구조가 유지되는지 확인했다.

핵심 목표는 단순히 Rule 개수를 늘리는 것이 아니었다.

> 새로운 Rule을 추가해도 기존 Rule과의 경계가 유지되는지 확인하기

이번 작업은 Rule Engine v1 위에서 Rule Coverage를 확장하고, 다음 구조 정리를 준비하는 단계였다.

---

## 이번에 추가한 Rule

이번 작업에서 새로 추가한 Rule은 다음 세 가지다.

```text
cookie_color
cookie_shape
cookie_cracking
```

기존 Rule은 다음 두 가지였다.

```text
cookie_spread
cookie_texture
```

따라서 현재 Ovendo는 총 5개의 Cookie Rule을 가진다.

```text
cookie_spread
cookie_texture
cookie_color
cookie_shape
cookie_cracking
```

다룰 수 있는 증상 축도 조금 더 넓어졌다.

```text
퍼짐
식감
색
모양
갈라짐
```

이전에는 주로 퍼짐과 식감 문제를 중심으로 Rule Engine이 동작하는지 확인했다.
이번에는 색, 모양, 갈라짐까지 추가하면서 Rule Coverage가 실제로 확장되기 시작했다.

---

## Rule 경계 확인

Rule이 늘어나면서 가장 중요하게 본 것은 Rule 사이의 경계였다.

새로운 Rule을 추가할수록 비슷한 증상을 다루는 Rule끼리 겹칠 가능성이 생긴다.

예를 들어 `cookie_shape`를 추가할 때는 `cookie_spread`와의 경계를 확인해야 했다.

```text
spread
→ 퍼짐 정도, 크기, 두께

shape
→ 형태, 윤곽, 둥글기
```

쿠키가 납작해지는 문제는 spread와 연결될 수 있지만,
쿠키의 형태가 흐트러지거나 모양이 일정하지 않은 문제는 shape로 볼 수 있다.

또 `cookie_cracking`을 추가할 때는 `cookie_texture`와의 경계를 확인했다.

```text
cracking
→ 표면 갈라짐

texture
→ 건조함, 퍽퍽함, 식감
```

갈라짐은 표면에서 관찰되는 증상이고, texture는 먹었을 때의 식감 문제에 가깝다.

이처럼 Rule Coverage를 확장할 때는 단순히 새로운 Rule을 추가하는 것보다, 기존 Rule과의 경계를 유지하는 것이 중요했다.

---

## Rule Metadata 추가

Rule이 늘어나면서 Rule 자체를 설명하는 정보도 필요해졌다.

그래서 Rule JSON에 Metadata를 추가했다.

```text
title
domain
category
status
searchDescription
```

이 중 `searchDescription`은 향후 Rule Index나 RAG 구조에서 사용할 수 있는 자연어 설명 필드다.

예를 들어 Rule이 어떤 증상을 다루는지, 어떤 상황에서 검색되어야 하는지 설명하는 역할을 할 수 있다.

다만 이번 단계에서 RAG를 구현한 것은 아니다.

현재 Rule Search는 여전히 keyword 기반으로 동작한다.

```text
현재:
사용자 질문 → keyword 기반 Rule Search

아직 하지 않음:
사용자 질문 → Embedding → Retrieval → Rule 선택
```

지금은 RAG를 먼저 붙이는 것보다, Rule을 설명 가능한 단위로 정리해두는 것이 우선이라고 판단했다.

즉, 이번 Metadata 추가는 당장 Retrieval을 구현하기 위한 작업이라기보다, 이후 Rule Index와 RAG 구조로 확장할 수 있도록 Rule의 설명 정보를 정리한 작업이다.

---

## 테스트 결과

Rule을 추가할 때마다 기존 Rule과 충돌하지 않는지도 테스트로 확인했다.

현재 테스트 결과는 다음과 같다.

```text
npm test
→ 49 pass
```

이번 단계에서 중요하게 본 것은 두 가지다.

```text
1. 새 Rule이 정상적으로 검색되는가
2. 새 Rule이 기존 Rule을 침범하지 않는가
```

Rule이 적을 때는 “검색이 되는가”만 확인해도 충분했다.

하지만 Rule이 늘어나기 시작하면 “어떤 Rule이 검색되는가”만큼이나 “잘못된 Rule이 검색되지 않는가”도 중요해진다.

이번 작업을 통해 Boundary Test의 중요성을 다시 확인했다.

---

## 이번에 하지 않은 것

이번 작업에서 의도적으로 하지 않은 것도 있다.

```text
Embedding 생성
Vector DB 구축
Retriever 구현
RAG 연결
LLM 진단 결정
UI 구현
```

아직 Rule 수가 많지 않기 때문에 Retrieval 구조를 먼저 구현할 필요는 없다고 판단했다.

지금 단계에서는 다음 순서가 더 적절하다고 보았다.

```text
Rule Coverage 확장
↓
Rule Boundary 검증
↓
Rule Specification 정리
↓
Rule Index 설계
```

즉, RAG는 지금 바로 붙일 기능이 아니라, Rule이 충분히 쌓이고 검색 기준이 더 복잡해졌을 때 검토할 다음 구조에 가깝다.

---

## 현재 상태

`cookie_cracking`까지 추가하면서 Ovendo는 단순히 몇 개의 증상만 처리하는 단계를 조금 넘어섰다.

현재는 쿠키 실패를 여러 증상 축으로 나누어 진단할 수 있는 구조를 갖추기 시작했다.

현재 Rule 상태는 다음과 같다.

| Rule              | 담당 증상        |
| ----------------- | ------------ |
| `cookie_spread`   | 퍼짐, 크기, 두께   |
| `cookie_texture`  | 식감, 건조함, 퍽퍽함 |
| `cookie_color`    | 색, 갈변        |
| `cookie_shape`    | 모양, 윤곽       |
| `cookie_cracking` | 표면 갈라짐       |

Rule Engine v1 자체를 크게 바꾸지 않고도 Rule을 추가할 수 있었고, 테스트를 통해 기존 Rule과의 경계도 확인할 수 있었다.

---

## 이번 작업에서 얻은 것

이번 작업을 통해 확인한 것은 다음과 같다.

| 항목                | 확인한 내용                   |
| ----------------- | ------------------------ |
| Rule Coverage 확장  | Cookie Rule이 2개에서 5개로 증가 |
| Rule Engine 안정성   | 신규 Rule 추가에도 기존 구조 유지    |
| Boundary Test 필요성 | Rule이 늘어날수록 경계 검증이 중요    |
| Metadata 필요성      | Rule 자체를 설명하는 정보가 필요     |
| RAG 적용 시점         | 아직 구현보다 준비 단계가 적절        |

이번 작업에서 가장 중요한 점은 Rule Engine v1이 신규 Rule 추가에 대응할 수 있다는 것을 확인한 것이다.

또한 Rule이 늘어날수록 단순한 keyword matching보다, Rule의 역할과 경계를 명확히 관리하는 일이 더 중요해진다는 점도 확인했다.

---

## 다음 단계

다음 작업에서는 Rule을 계속 늘리기 전에, 늘어난 Rule을 관리하기 위한 구조를 먼저 정리할 예정이다.

예정 작업은 다음과 같다.

```text
1. Rule Specification 문서 정리
2. Rule Index 설계 문서 작성
3. cookie_doneness Research 및 Boundary 검토
```

특히 `cookie_doneness`는 바로 Rule로 만들지 않고, 먼저 Research와 Boundary 검토를 진행할 예정이다.

익음 판단은 색, 식감, 굽기 정도와 연결되기 쉬워 기존 Rule과 경계가 겹칠 수 있기 때문이다.

따라서 다음 Rule을 추가하기 전에는 먼저 다음 질문을 확인해야 한다.

```text
doneness는 color와 어떻게 다른가?
doneness는 texture와 어떻게 다른가?
doneness는 baking condition과 어떻게 연결되는가?
```

---

## 정리

이번 작업은 Rule Coverage를 실제로 확장한 첫 단계에 가깝다.

기존 Rule Engine v1 위에 새로운 Rule을 추가했고, 테스트를 통해 기존 Rule과의 경계가 유지되는지 확인했다.

정리하면 다음과 같다.

```text
Rule Engine v1은 신규 Rule 추가에 대응할 수 있다.
Rule이 늘어날수록 Boundary Test가 중요하다.
RAG는 아직 구현하지 않고, Metadata와 searchDescription으로 준비만 한다.
다음 단계에서는 Rule Specification과 Rule Index 설계가 필요하다.
```

Ovendo의 방향은 여전히 같다.

> 코드는 일반화하고, 지식은 데이터화한다.

이제는 이 원칙을 유지하면서, 늘어난 Rule을 더 잘 관리할 수 있는 구조를 정리할 차례다.
