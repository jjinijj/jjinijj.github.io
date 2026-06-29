---

layout: post
title: "[Ovendo] 개발일지 #009 — Rule만으로는 부족했다. knowledge.md를 추가한 이유"
date: 2026-06-29
categories: [ovendo]
tags: [devlog, rule-engine, knowledge-management, research, rag, document-rag]
---

## 오늘의 개발 주제

이번 작업에서는 Rule을 추가하기보다, Research 결과를 어떻게 관리할지 다시 정리했다.

이전까지 Ovendo에서는 Research 결과를 주로 두 종류의 문서로 관리하고 있었다.

```text
note.md
→ Rule 생성을 위한 구조화된 정보

hold.md
→ 현재 Rule에 반영하지 않는 내용
```

이 구조만으로도 Rule Engine을 만드는 데는 문제가 없었다.

실제로 `cookie_spread`, `cookie_texture`, `cookie_color`, `cookie_shape`, `cookie_cracking`까지 이 방식으로 Rule을 구축했고, Rule Engine도 안정적으로 동작했다.

하지만 Rule Coverage가 늘어나면서 한 가지 고민이 생겼다.

> Research 결과를 Rule 생성에만 사용하는 것이 맞을까?

---

## 기존 구조의 한계

Ovendo에서 Rule은 진단을 위해 존재한다.

Rule에는 주로 다음과 같은 정보가 들어간다.

* 어떤 증상을 판단할 것인지
* 가능한 원인은 무엇인지
* 어떤 순서로 확인할 것인지
* 다음에는 무엇을 조정해야 하는지

즉, Rule JSON은 실행 가능한 구조화 데이터에 가깝다.

하지만 Research 과정에서 얻는 정보는 항상 Rule에 바로 들어갈 수 있는 형태가 아니다.

예를 들어 다음과 같은 내용이 있다.

* 왜 이런 현상이 발생하는지
* 재료가 어떤 역할을 하는지
* 오븐 안에서 어떤 변화가 일어나는지
* 여러 조건이 결과에 어떤 영향을 주는지

이런 내용은 Rule로 사용하기에는 설명 중심이고, 진단 우선순위가 정해져 있지 않은 경우가 많다.

하지만 그렇다고 버리기에는 중요한 지식이었다.

즉, 기존 구조에는 이런 문제가 있었다.

```text
Rule에 넣기에는 너무 설명적이다.
하지만 hold에 넣고 보류하기에는 장기적으로 가치가 있다.
```

그래서 Rule 생성용 문서와 별도로, 사람이 읽을 수 있는 지식 문서가 필요하다고 판단했다.

---

## knowledge.md를 추가하기로 했다

앞으로 Research 결과는 다음 세 가지 문서로 나누어 관리하려고 한다.

```text
knowledge.md
→ 사람이 읽는 베이킹 지식

note.md
→ Rule 생성을 위한 구조화된 정보

hold.md
→ 현재 Rule에 포함하지 않는 내용
```

가장 중요한 것은 역할을 명확히 분리하는 것이다.

`knowledge.md`는 설명을 위한 문서다.
Research 과정에서 정리한 배경 지식, 원리, 맥락을 사람이 읽을 수 있는 형태로 남긴다.

반면 `note.md`는 Rule을 만들기 위한 문서다.
진단에 사용할 수 있는 원인, 증상, 추천 조정 방향을 구조화한다.

같은 Research에서 나온 내용이라도 목적에 따라 저장 위치가 달라진다.

```text
설명과 맥락이 중심이면 knowledge.md
진단 기준과 구조화가 중심이면 note.md
지금 Rule에 넣지 않을 내용이면 hold.md
```

---

## Note와 Rule의 역할은 유지한다

`knowledge.md`를 추가한다고 해서 기존 원칙이 바뀌는 것은 아니다.

Ovendo의 Rule 관리 원칙은 여전히 같다.

```text
Research
↓
Note
↓
Rule JSON
↓
Search
↓
Answer
```

여기서 `note.md`는 여전히 Rule 생성의 기준이다.
`rule.json`은 여전히 실행용 데이터다.

새로 추가되는 `knowledge.md`는 Rule을 직접 대체하지 않는다.

즉, 앞으로의 구조는 다음에 가깝다.

```text
Research
↓
knowledge.md
↓
note.md
↓
rule.json
```

`knowledge.md`에서 바로 Rule을 만들기보다, 사람이 이해할 수 있는 지식으로 먼저 정리하고, 그중 진단에 사용할 수 있는 내용을 `note.md`로 구조화한 뒤 Rule JSON으로 변환한다.

이렇게 하면 지식의 흐름을 더 명확하게 추적할 수 있다.

---

## 장기적으로는 Ovendo Lab과 연결된다

현재 개발 중인 Ovendo는 Rule 기반 진단 시스템이다.

하지만 장기적으로는 AI 실험용 프로젝트인 `Ovendo Lab`도 함께 운영해보고 싶다.

Ovendo는 Rule Engine을 사용하고,
Ovendo Lab은 Document RAG 방식도 실험해볼 예정이다.

이때 `knowledge.md`는 Document RAG의 원본 문서가 될 수 있다.

```text
Research
        │
        ▼
knowledge.md
        │
        ├──────────────┐
        ▼              ▼
note.md         Ovendo Lab
        │        (Document RAG)
        ▼
rule.json
        │
        ▼
Ovendo
(Rule Engine)
```

이 구조를 만들면 같은 Research 결과를 기반으로 두 가지 접근을 비교할 수 있다.

```text
Ovendo
→ Rule 기반 진단

Ovendo Lab
→ Document RAG 기반 진단
```

즉, `knowledge.md`는 지금 당장 Rule Engine에 필요한 문서이기도 하지만, 장기적으로는 RAG 실험을 위한 기반 문서가 될 수 있다.

---

## 지금 바로 RAG를 구현하지 않는 이유

이번 작업에서 Document RAG를 바로 구현하지는 않는다.

현재 Ovendo의 우선순위는 여전히 Rule Engine을 안정적으로 확장하는 것이다.

아직 하지 않는 작업은 다음과 같다.

```text
Embedding 생성
Vector DB 구축
Retriever 구현
LLM 진단 결정
Document RAG 연결
```

지금 단계에서 중요한 것은 RAG 구현이 아니라, Research 결과를 나중에 재사용할 수 있는 형태로 정리해두는 것이다.

따라서 현재 판단은 다음과 같다.

```text
knowledge.md 추가
→ 지금 적용

Document RAG
→ 장기 실험으로 보류
```

---

## 이번 작업에서 얻은 것

처음에는 Rule만 잘 만들면 된다고 생각했다.

하지만 Rule Coverage가 늘어나면서, 지식을 어떻게 관리할지도 중요한 문제가 되었다.

이번 작업을 통해 정리한 것은 다음과 같다.

| 항목             | 역할                        |
| -------------- | ------------------------- |
| `knowledge.md` | 사람이 읽는 베이킹 지식             |
| `note.md`      | Rule 생성을 위한 구조화된 근거       |
| `hold.md`      | 현재 Rule에 넣지 않는 보류 지식      |
| `rule.json`    | Rule Engine에서 사용하는 실행 데이터 |

이번 구조 정리의 핵심은 Research 결과를 하나의 목적에만 묶어두지 않는 것이다.

Rule에 바로 들어가지 않는 지식도 `knowledge.md`에 남겨두면, 나중에 설명 문서나 RAG 실험에 활용할 수 있다.

---

## 다음 단계

다음 작업에서는 이 문서 구조를 실제 Research 흐름에 적용해볼 예정이다.

예정 작업은 다음과 같다.

```text
1. 기존 Research 문서 구조 점검
2. knowledge.md / note.md / hold.md 역할 기준 정리
3. 다음 Rule Research부터 knowledge.md 작성 적용
4. Rule Specification과 Rule Index 설계에 반영
5. Ovendo Lab의 Document RAG 실험 가능성 검토
```

당장은 Rule Engine을 위한 정리지만, 장기적으로는 다양한 AI 접근 방식을 실험할 수 있는 기반이 될 수 있다.

---

## 정리

이번 작업은 새로운 Rule을 추가한 작업은 아니었다.

대신 Research 결과를 Rule 생성에만 사용하지 않고, 사람이 읽을 수 있는 지식 문서와 실행용 Rule 근거로 나누어 관리하기로 했다.

정리하면 다음과 같다.

```text
Rule은 진단을 위한 실행 데이터다.
Note는 Rule 생성을 위한 구조화된 근거다.
Knowledge는 사람이 읽고 재사용할 수 있는 지식 문서다.
```

Ovendo의 방향은 여전히 같다.

> 코드는 일반화하고, 지식은 데이터화한다.

이제는 여기에 한 가지 기준을 더 추가하려고 한다.

> 지식은 실행용 데이터와 설명용 문서로 나누어 관리한다.
