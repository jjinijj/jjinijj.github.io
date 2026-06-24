---

layout: post
title: "[Ovendo] 개발일지 #006 — Rule Audit으로 지식 품질 보강하기"
date: 2026-06-23
categories: [ovendo]
tags: [devlog, rule-audit, knowledge-quality, llm, validation, rule-engine]
---

## 오늘의 개발 주제

이번 작업은 새로운 기능을 추가하는 작업이 아니었다.

오히려 기능 확장을 잠시 멈추고, 지금까지 만든 Rule의 근거와 품질을 다시 점검하는 작업이었다.

새로운 Rule인 `cookie_color`를 작성하기 위해 리서치를 진행하던 중, 기존 Rule 일부에 LLM의 해석이 섞여 있을 가능성을 발견했다.

그래서 이번에는 Rule을 더 추가하기보다, 기존 Rule이 신뢰 가능한 기준으로 관리되고 있는지 확인하는 **Rule Audit**을 진행했다.

---

## 작업 배경

Ovendo는 홈베이킹 실패 원인을 진단하고, 다음 개선 방향을 제안하는 Rule 기반 베이킹 진단 서비스이다.

현재 MVP에서는 Rule Engine v1을 구현했고, 다음 두 Rule을 중심으로 진단 흐름을 만들었다.

```text
cookie_spread
cookie_texture
```

초기 Rule 생성 흐름은 다음과 같았다.

```text
Reference
↓
Research Note
↓
Rule JSON
```

하지만 점검 과정에서 일부 항목이 실제로는 다음 흐름에 가까울 수 있다는 점을 발견했다.

```text
Reference
↓
LLM 해석 포함
↓
Research Note
↓
Rule JSON
```

문제는 단순히 “틀렸는가”가 아니었다.

더 중요한 문제는 아래 질문에 명확히 답하기 어렵다는 점이었다.

* 이 Rule은 어떤 근거에서 만들어졌는가?
* 이 원인은 실제 출처에서 확인 가능한가?
* Rule을 수정할 때 어떤 기준으로 판단할 수 있는가?

Rule 기반 구조를 선택한 이유는 설명 가능한 진단을 만들기 위해서였다.
그런데 Rule의 근거가 흐려지면, 구조 자체의 장점이 약해질 수 있다.

그래서 기능 확장을 잠시 멈추고 기존 Rule 품질을 먼저 점검하기로 했다.

---

## Rule Audit으로 방향 전환

이번 작업에서는 새 Rule 추가를 중단하고, 기존 Rule을 점검하는 Audit Sprint로 방향을 바꾸었다.

점검 범위는 다음 두 Rule이었다.

```text
cookie_spread
cookie_texture
```

Audit 원칙은 다음과 같이 정했다.

* 새 Rule 추가 금지
* 구조 변경 금지
* Research → Note → Rule 추적
* confidence 재평가 허용
* 근거가 약한 항목은 유지 / 수정 / 제거 여부 판단

즉, 이번 작업의 목표는 기능 확장이 아니라 **이미 존재하는 Rule의 신뢰도 확인**이었다.

---

## 점검 기준

각 cause는 다음 기준으로 확인했다.

```text
cause
↓
Research Note 존재 여부 확인
↓
출처 존재 여부 확인
↓
과도한 해석 여부 확인
↓
유지 / 수정 / 제거 결정
```

이 과정에서 중요하게 본 것은 “그럴듯한가”가 아니라 “추적 가능한가”였다.

Rule은 답변을 만들기 위한 실행 데이터이지만, 동시에 진단 기준이기도 하다.
그래서 각 항목이 어떤 근거에서 나왔는지 확인할 수 있어야 한다.

---

## Audit 결과

### cookie_spread

`cookie_spread` Rule의 주요 cause는 유지하기로 했다.

```text
유지
- ingredient_structure_balance
- fat_state_control
- sugar_hydration
- creaming_variation
- thermal_setting
- measurement_variation
```

각 항목은 기존 Research Note와 Rule의 연결이 유지 가능하다고 판단했다.

다만 일부 표현은 실제 출처에서 확인 가능한 범위를 넘지 않도록 해석 강도를 조정했다.

---

### cookie_texture

`cookie_texture` Rule도 대부분 유지했다.

```text
유지
- moisture_balance
- fat_behavior
- aeration_control
- thermal_response
- dough_aging
- gluten_inhibition
```

다만 `protein_structure`는 그대로 삭제하지 않고 confidence를 낮추는 방향으로 수정했다.

```text
수정
- protein_structure
  → 삭제하지 않고 confidence 하향
```

이 항목은 완전히 제거할 정도는 아니지만, 현재 근거만으로 강하게 판단하기에는 조심스러운 부분이 있었다.

그래서 Rule에서 제외하기보다는 신뢰도 수준을 조정하는 방식으로 처리했다.

---

## 이번 작업에서 정리한 원칙

이번 Audit을 통해 Ovendo의 지식 관리 원칙을 다시 정리했다.

LLM은 빠르게 초안을 만드는 데 도움을 줄 수 있다.
하지만 LLM이 생성한 내용을 그대로 제품 지식으로 사용하는 것은 위험할 수 있다.

앞으로의 Rule 생성 흐름은 다음과 같이 가져가기로 했다.

```text
AI
↓
Draft
↓
Human Review
↓
Research Note
↓
Rule JSON
↓
Application
```

즉, AI는 초안을 만들 수 있지만, 최종 Rule은 검토 가능한 근거를 거쳐야 한다.

Ovendo에서 Rule은 절대 정답이 아니라, **검증 가능한 진단 기준**으로 관리되어야 한다.

---

## 이번 작업에서 얻은 것

이번 작업은 단순한 데이터 수정이 아니었다.

새로운 기능을 추가하지는 않았지만, Rule 기반 시스템을 계속 유지하기 위한 품질 관리 기준을 세운 작업이었다.

이번에 얻은 것은 다음과 같다.

| 항목             | 의미                         |
| -------------- | -------------------------- |
| Rule Audit     | 기존 Rule의 근거와 신뢰도 점검        |
| confidence 재평가 | 애매한 항목을 삭제 대신 신뢰도 조정       |
| Research 추적    | Rule이 어떤 Note와 연결되는지 확인    |
| 지식 품질 관리       | LLM 해석과 검증된 근거를 분리         |
| 확장 전 점검        | 새 Rule 추가 전에 기존 구조의 신뢰성 확인 |

기능을 더 추가하는 것보다, 지금 있는 Rule이 믿을 수 있는 상태인지 확인하는 것이 더 중요하다고 판단했다.

---

## 이번 작업의 의미

이번 작업은 구현보다 시행착오와 보강에 가까운 작업이었다.

하지만 오히려 이 과정이 Ovendo의 방향을 더 명확하게 만들었다.

Ovendo는 LLM이 답을 바로 생성하는 구조가 아니다.
검증 가능한 진단 기준을 만들고, 그 기준을 운영하는 구조를 목표로 한다.

이번 경험을 통해 보여줄 수 있는 것은 다음과 같다.

* AI 결과를 그대로 사용하지 않는 판단
* Rule 기반 지식 구조 설계
* 품질 검증 및 Audit 프로세스 구축
* 테스트 가능한 도메인 지식 관리
* 제품 요구사항과 구현 구조 연결

핵심 메시지는 다음과 같다.

> Ovendo는 LLM이 답을 생성하는 구조가 아니라, 검증 가능한 진단 기준을 운영하는 구조로 설계한다.

---

## 다음 작업

Audit을 통해 기존 Rule의 품질을 다시 확인했으므로, 다음 단계에서는 Rule 확장을 다시 진행할 수 있다.

예정 작업은 다음과 같다.

* `cookie_color` Rule 리서치 재개
* Research Note 작성 기준 정리
* Rule 생성 시 confidence 기준 명확화
* LLM Draft와 Human Review 단계 분리
* Rule Coverage 확장

이번 작업은 잠깐 멈춰서 구멍을 확인하고 보강한 과정이었다.
기능은 늘어나지 않았지만, 앞으로 Rule이 늘어날 수 있는 기반은 더 단단해졌다.
