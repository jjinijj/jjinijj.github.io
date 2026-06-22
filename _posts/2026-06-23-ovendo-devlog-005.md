---

layout: post
title: "[Ovendo] 개발일지 #005 — 코드는 일반화하고 지식은 데이터로 이동하기"
date: 2026-06-23
categories: [ovendo]
tags: [devlog, rule-engine, architecture, testing, validation, typescript]
---

## 오늘의 개발 주제

이번 작업은 새로운 기능을 많이 추가하는 것보다, Ovendo의 구조를 더 명확하게 정리하는 데 가까웠다.

처음에는 질문을 코드가 직접 이해하고 분기하는 구조였다.
하지만 Rule이 늘어나기 시작하면서 점점 도메인 지식이 TypeScript 코드 안으로 들어가기 시작했다.

이번 사이클의 목표는 하나였다.

> 코드는 일반화하고, 지식은 데이터로 이동한다.

즉, TypeScript 코드는 특정 베이킹 문제를 직접 알기보다, Rule 데이터를 읽고 처리하는 역할에 집중하도록 만드는 것이 목표였다.

---

## 이전 구조의 문제

초기 구조에서는 classifier가 도메인 지식을 직접 가지고 있었다.

```text
Question
↓
classifyQuestion()
↓
ruleId / variant
↓
Answer
```

예전에는 분류기가 이런 판단을 직접 했다.

```text
쿠키 문제인가?
안 퍼지는 문제인가?
식감 문제인가?
```

Rule이 하나뿐일 때는 이 방식도 괜찮았다.

하지만 Rule이 늘어나면 문제가 생긴다.

```text
새 Rule 추가
↓
TypeScript 수정
↓
Classifier 비대화
```

새로운 문제 유형을 추가할 때마다 코드가 수정된다면, Rule 기반 구조의 장점이 줄어든다.

그래서 이번 작업에서는 도메인 지식을 코드에서 빼내고, Rule JSON 쪽으로 옮기는 방향을 확정했다.

---

## 현재 구조

현재 Ovendo는 질문을 바로 classifier가 해석해서 답을 만드는 구조가 아니다.

질문이 들어오면 Rule Search를 통해 관련 Rule과 Variant를 찾고, 그 결과를 기반으로 답변을 구성한다.

```text
Question
↓
searchRules()
↓
Rule 선택
↓
Variant 선택
↓
buildAnswer()
↓
CLI
```

각 역할은 다음처럼 나누었다.

| 구성 요소              | 역할                                           |
| ------------------ | -------------------------------------------- |
| `classifyQuestion` | 도메인 판단을 최소화하고 타입 정의 수준으로 유지                  |
| `ruleRegistry`     | Rule 등록, 검증, 목록 제공                           |
| `searchRules`      | 후보 Rule 수집, score 계산, 최적 Rule 선택, variant 결정 |
| `Rule JSON`        | 도메인 지식 보관                                    |
| `buildAnswer`      | Rule과 Variant를 기반으로 최종 응답 생성                 |

---

## classifyQuestion의 역할 축소

이전에는 `classifyQuestion()`이 질문을 직접 이해하고, 어떤 Rule을 사용할지 결정했다.

하지만 현재 방향에서는 classifier가 도메인 지식을 많이 가지지 않도록 했다.

현재 `classifyQuestion()`의 역할은 다음에 가깝다.

```text
질문 자체를 깊게 이해하지 않음
도메인 판단을 직접 하지 않음
현재는 타입 정의 수준으로 유지
```

질문에 대한 실제 도메인 판단은 Rule Search 단계에서 처리한다.

이렇게 하면 새로운 Rule이 추가되더라도 classifier를 계속 수정하지 않아도 된다.

---

## ruleRegistry의 역할

Rule은 registry를 통해 관리한다.

```text
Rule 등록
↓
Rule 검증
↓
Rule 목록 제공
```

새 Rule을 추가하는 흐름은 다음과 같다.

```text
Rule JSON 작성
↓
Registry 등록
↓
끝
```

이 구조 덕분에 Rule을 추가할 때 Search나 Answer Builder 코드를 직접 수정하는 일을 줄일 수 있다.

---

## searchRules의 역할

`searchRules()`는 질문과 Rule 데이터를 비교해서 후보 Rule을 찾는다.

현재 역할은 다음과 같다.

```text
Rule 탐색
↓
후보 수집
↓
score 계산
↓
최적 Rule 선택
↓
variant 결정
```

현재는 `Candidates[]`를 수집하고, 그중 선택된 Rule을 반환하는 구조를 지원한다.

```text
Candidates[]
↓
Selected Rule
```

이 구조는 앞으로 Rule이 늘어났을 때도 확장하기 쉽다.

지금은 키워드 기반이지만, 나중에 score 계산 방식이나 Retrieval 방식을 바꾸더라도 전체 흐름은 유지할 수 있다.

---

## Rule JSON의 역할

이제 Rule JSON은 단순한 키워드 파일이 아니다.

현재 Rule JSON은 도메인 지식을 담는 핵심 데이터다.

포함하는 정보는 다음과 같다.

```text
match
variantKeywords
variantMappings
constraints
evidence
```

예를 들어 현재 Rule은 다음과 같이 관리된다.

```text
cookie_spread
cookie_texture
```

이 Rule들은 각각 쿠키의 퍼짐 문제와 식감 문제에 대한 도메인 지식을 가진다.

즉, TypeScript 코드가 “쿠키가 왜 안 퍼지는지”를 직접 아는 것이 아니라,
Rule JSON이 그 지식을 가지고 있고 코드는 이를 읽고 처리한다.

---

## buildAnswer의 역할

`buildAnswer()`는 Rule Search 결과를 바탕으로 최종 응답 구조를 만든다.

```text
Rule
+
Variant
↓
Answer
```

현재는 다음 정보를 조립한다.

```text
Diagnostics
Recommendations
Evidence
```

이 구조 덕분에 질문 처리와 답변 생성의 역할도 분리된다.

Rule Search는 “어떤 Rule과 Variant가 적절한가”를 찾고,
Answer Builder는 “그 결과를 어떻게 답변으로 구성할 것인가”를 담당한다.

---

## 지식 이동

이번 작업에서 가장 큰 변화는 지식의 위치가 바뀐 것이다.

이전에는 TypeScript 코드 안에 키워드와 증상 판단이 들어 있었다.

```text
TypeScript
↓
COOKIE_KEYWORDS
↓
SYMPTOM_KEYWORDS
```

현재는 Rule JSON 안에 들어간다.

```text
Rule JSON
↓
keywords
↓
variantKeywords
```

이제 새로운 문제 유형을 추가할 때의 흐름은 다음과 같다.

```text
Note 작성
↓
Rule 생성
↓
Registry 등록
```

즉, 새 지식을 추가하기 위해 TypeScript 조건문을 늘리는 것이 아니라,
Note와 Rule 데이터를 추가하는 방향으로 바뀌었다.

---

## 안정성 추가

구조를 바꾸면서 안전장치도 함께 추가했다.

Rule이 늘어나면 가장 위험한 것은 잘못된 Rule 데이터다.

그래서 Rule 등록 시 검증 과정을 추가했다.

### Rule Validation

현재 검증하는 항목은 다음과 같다.

* 필수 필드 존재 여부
* 중복 id 여부
* `symptoms`와 `variantKeywords`의 일관성
* `variantMappings`와 `symptoms`의 일관성

잘못된 Rule이 있으면 즉시 중단한다.

```text
Invalid Rule
```

이렇게 하면 잘못된 JSON이 런타임까지 넘어가기 전에 문제를 확인할 수 있다.

---

## 자동 테스트 추가

구조 변경 이후 기존 동작이 깨지지 않았는지 확인하기 위해 자동 테스트도 추가했다.

```bash
npm test
```

현재 테스트하는 항목은 다음과 같다.

```text
Rule Search
Variant
Validation
Ambiguous Question
Unknown Input
```

현재 총 12개의 테스트가 통과한다.

테스트가 생기면서 CLI로 매번 직접 확인하지 않아도, 기본 동작이 유지되는지 빠르게 확인할 수 있게 되었다.

---

## 현재 Ovendo 구조

현재 Ovendo의 전체 흐름은 다음과 같다.

```text
Research
↓
Note (.md)
↓
Rule (.json)
↓
Registry
↓
Validation
↓
Search
↓
Answer
↓
CLI
```

이 구조에서 Markdown Note는 지식의 근거를 남기고, Rule JSON은 실행 가능한 데이터 역할을 한다.

그리고 TypeScript 코드는 특정 지식을 직접 품기보다, Rule을 검증하고 검색하고 답변으로 조립하는 일반화된 역할을 맡는다.

---

## 이번 작업에서 얻은 것

Rule이 하나일 때는 코드 중심 구조도 충분했다.

하지만 두 번째 Rule부터는 지식이 코드 밖으로 이동해야 한다는 점이 분명해졌다.

이번 구조 변경을 통해 Ovendo는 단순한 조건 분기에서 조금 더 Rule 기반 엔진 형태로 이동했다.

이번 작업에서 얻은 것은 다음과 같다.

| 항목        | 의미                                              |
| --------- | ----------------------------------------------- |
| 지식의 위치 변경 | 도메인 지식을 TypeScript에서 Rule JSON으로 이동             |
| 코드 일반화    | Search, Registry, Answer Builder가 특정 Rule에 덜 의존 |
| 안정성 확보    | Rule Validation으로 잘못된 데이터 차단                    |
| 테스트 기반 확보 | 총 12개 테스트로 기존 동작 검증                             |

이번 작업의 핵심은 기능 추가가 아니라 구조를 오래 유지할 수 있게 만드는 것이었다.

> Rule이 늘어나도 코드가 계속 커지지 않는 구조

이 방향이 Ovendo의 중요한 기반이 될 것 같다.

---

## 다음 작업

다음 단계에서는 Rule Coverage를 늘려 실제 확장 비용을 확인할 예정이다.

예정 작업은 다음과 같다.

* 새로운 Rule 추가
* Rule 추가 시 TypeScript 수정이 필요한지 확인
* Rule Coverage 확장
* 테스트 케이스 추가
* Note → Rule 변환 기준 정리

이제 구조는 어느 정도 준비되었다.
다음에는 실제 Rule을 더 늘려보면서 이 구조가 얼마나 잘 버티는지 확인해볼 예정이다.
