---

layout: post
title: "[Ovendo] 개발일지 #007 — 기존 Rule 재정립하기"
date: 2026-06-23
categories: [ovendo]
tags: [devlog, rule-engine, rule-audit, validation, testing, knowledge-quality]
---

## 오늘의 개발 주제

이번 작업에서는 새로운 Rule을 추가하지 않았다.

원래는 Chapter 2에서 `cookie_color` Rule을 추가하며 Rule Coverage를 확장할 계획이었다.
하지만 새 Rule을 만들기 전에, 기존 Rule이 현재 정리된 Note 기준과 잘 맞는지 먼저 확인할 필요가 있었다.

이번 작업의 목적은 기능 확장이 아니라 **기존 Rule을 다시 정렬하는 것**이었다.

대상 Rule은 다음 두 개였다.

```text
cookie_spread
cookie_texture
```

---

## 1. 왜 바로 새 Rule을 만들지 않았나

새 Rule을 추가하려면 먼저 기존 Rule이 안정적인 기준 위에 있어야 한다.

현재 Ovendo의 Rule 관리 원칙은 다음과 같다.

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

그런데 기존 Rule 일부가 최신 Note의 표현과 완전히 일치하지 않거나,
이름과 기준이 조금 추상적으로 남아 있는 부분이 있었다.

이 상태에서 `cookie_color` Rule을 추가하면 Rule Coverage는 늘어나지만,
기존 Rule과 새 Rule 사이의 기준이 흐려질 수 있다고 판단했다.

그래서 이번에는 확장을 잠시 멈추고 기존 Rule을 먼저 재정립했다.

---

## 2. Note와 Rule JSON의 역할 분리

이번 작업에서 다시 확인한 기준은 다음이다.

```text
Note는 지식 원본
Rule JSON은 실행용 데이터
```

Note에는 조사 내용과 판단 근거가 남는다.
Rule JSON은 그중 실제 엔진에서 사용할 수 있는 형태로 변환된 데이터다.

그래서 이번 작업에서는 다음 원칙을 적용했다.

```text
updated note에 없는 내용은 Rule JSON에 추가하지 않는다.
유용해 보이더라도 note에 없는 해석은 보류한다.
Rule JSON은 새로운 해석을 저장하는 공간이 아니다.
```

즉, Rule을 더 똑똑하게 만드는 것보다
Rule이 Note를 정확히 반영하도록 만드는 데 집중했다.

---

## 3. 기존 Rule에서 발견한 문제

기존 Rule은 동작 자체에는 큰 문제가 없었다.

하지만 점검하면서 몇 가지 정리할 부분이 보였다.

첫째, 일부 variant 이름이 updated note의 표현과 다르게 남아 있었다.
예를 들어 `no_spread`는 실제 의미상 “퍼짐 부족”에 가까웠고, `inconsistent_spread`는 “결과 편차”에 가까웠다.

둘째, diagnostic id 중 일부가 note에 명시된 원인보다 추상적인 이름으로 남아 있었다.

셋째, texture Rule에는 updated note에서 더 이상 직접적인 Diagnostic Signal로 남아 있지 않은 표현이 포함되어 있었다.

예를 들어 `hard_texture`, `딱딱`, `질김` 관련 처리는 이번 정리에서 보류했다.

현재 원칙은 명확하다.

```text
Note에 없는 내용은 JSON에 추가하지 않는다.
```

---

## 4. spread / texture Rule 재정렬

### cookie_spread

`cookie_spread`는 updated spread note 기준에 맞춰 version을 올렸다.

```text
0.2 → 0.3
```

variant 이름은 다음처럼 정리했다.

```text
no_spread → under_spread
inconsistent_spread → result_variation
over_spread → 유지
```

diagnostic id도 note에 명시된 cause 기준으로 맞췄다.

```text
ingredient_structure_balance → ingredient_ratio
fat_state_control → fat_state
sugar_hydration → dough_rest
creaming_variation → creaming
measurement_variation → measurement
thermal_setting → 유지
```

recommendation id도 updated note 기준에 맞게 정리했다.

```text
manage_fat_state → manage_fat_temperature
manage_dough_rest → test_rest_duration
stabilize_heat → stabilize_bake_environment
```

---

### cookie_texture

`cookie_texture`도 updated texture note 기준으로 version을 올렸다.

```text
0.1 → 0.2
```

variant는 다음처럼 정리했다.

```text
hard_texture → 제거
soft_texture → too_soft_texture
low_chewy_texture → 추가
cakey_texture → 유지
dry_texture → 유지
```

`hard_texture`는 현재 updated note에 명시적인 Diagnostic Signal로 남아 있지 않기 때문에 제거했다.

반대로 `low_chewy_texture`는 “쫀득함 부족”을 다루기 위해 추가했다.

초기 검증에서는 다음 문제가 있었다.

```text
쿠키가 쫀득하지 않아요
→ cookie_texture matched
→ variant: null
```

이후 `쫀득` 단독 표현은 general keyword로 두고,
부정 또는 부족 표현만 `low_chewy_texture` variant로 연결되도록 정리했다.

최종 동작은 다음과 같다.

```text
쿠키가 쫀득하지 않아요
→ cookie_texture
→ low_chewy_texture
```

중요한 결정은 다음이다.

```text
"쫀득" 단독 표현은 low_chewy_texture variant keyword로 넣지 않는다.
```

`쫀득한 쿠키를 만들고 싶어요` 같은 일반 질문까지 실패 증상으로 오분류할 수 있기 때문이다.

---

## 5. 테스트 결과

이번 작업에서 matching 알고리즘 자체는 변경하지 않았다.

변경은 대부분 Rule JSON 정렬과 variant type 동기화에 가까웠다.

최종 테스트 결과는 다음과 같다.

```text
npm test
→ 13/13 pass
```

CLI로 확인한 결과도 정상적으로 동작했다.

| 질문            | Rule           | Variant           |
| ------------- | -------------- | ----------------- |
| 쿠키가 안 퍼져요     | cookie_spread  | under_spread      |
| 쿠키가 너무 퍼져요    | cookie_spread  | over_spread       |
| 쿠키 크기가 제각각이에요 | cookie_spread  | result_variation  |
| 쿠키가 퍽퍽해요      | cookie_texture | dry_texture       |
| 쿠키가 케이크 같아요   | cookie_texture | cakey_texture     |
| 쿠키가 쫀득하지 않아요  | cookie_texture | low_chewy_texture |

현재 `딱딱해요` 관련 질문은 match되지 않는다.

```text
쿠키가 딱딱해요
→ no match
```

이 케이스는 버그라기보다 현재 Note 기준에 따른 보류 상태로 본다.
향후 필요하면 먼저 texture note를 업데이트한 뒤 Rule JSON에 반영할 예정이다.

---

## 6. 이번 작업에서 얻은 것

이번 작업은 기능을 추가한 것이 아니라, 기존 Rule을 기준에 맞게 다시 정렬한 작업이었다.

작업하면서 가장 크게 확인한 것은 다음이다.

> Rule을 늘리기 전에, 기존 Rule이 어떤 근거를 기준으로 만들어졌는지 먼저 정리해야 한다.

이번 작업에서 얻은 것은 다음과 같다.

| 항목          | 결과                                    |
| ----------- | ------------------------------------- |
| 기존 Rule 점검  | `cookie_spread`, `cookie_texture` 재정렬 |
| Rule 기준 정리  | updated note 기준으로 ID와 variant 정렬      |
| 지식 관리 원칙 확인 | Note에 없는 내용은 JSON에 추가하지 않음            |
| 엔진 영향       | matching 알고리즘 변경 없음                   |
| 테스트         | 13/13 pass                            |

이번 작업을 통해 Rule Engine v1 구조가 기존 Rule 재정립 상황에서도 안정적으로 동작하는 것을 확인했다.

---

## 7. 다음 단계

다음 단계에서는 다시 Rule Coverage 확장을 진행할 수 있다.

예정 작업은 다음과 같다.

```text
1. 이번 Rule 재정립 작업 커밋
2. 개발 로그에 작업 내용 반영
3. cookie_color 리서치 재개 여부 결정
4. hard_texture 지원 필요 여부는 note 업데이트 후 별도 판단
```

추천 커밋 메시지는 다음과 같다.

```text
refine existing cookie rules from updated notes
```

이번 작업은 새 기능을 추가한 로그라기보다, 확장 전에 기준을 다시 세운 작업이었다.

앞으로 Rule이 늘어나더라도 이 원칙은 유지하려고 한다.

```text
Note는 지식 원본이고,
Rule JSON은 실행용 데이터다.
```
