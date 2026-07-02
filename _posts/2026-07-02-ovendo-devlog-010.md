---

layout: post
title: "[Ovendo] 개발일지 #010 — TypeScript Rule Engine을 Python으로 포팅하기"
date: 2026-07-02
categories: [ovendo]
tags: [devlog, rule-engine, typescript, python, porting, rag]
---

## 오늘의 개발 주제

Ovendo Chapter 3에서는 기존에 TypeScript로 구현했던 Rule Engine v1을 Python 기반으로 포팅했다.

이번 작업의 목표는 새로운 기능을 추가하는 것이 아니었다.
이미 TypeScript에서 검증한 Rule Engine 구조를 Python으로 안정적으로 옮기고, 이후 Embedding, Retriever, RAG, LLM 연동을 실험할 수 있는 기반을 만드는 것이 목적이었다.

즉, 이번 작업의 핵심은 기능 확장이 아니라 **동일한 Rule Engine을 Python에서도 재현하는 것**이었다.

---

## 왜 Python으로 포팅했나

처음 Rule Engine v1은 TypeScript로 구현했다.

TypeScript는 Rule 구조를 빠르게 설계하고, JSON 기반 규칙을 검증하며, 사용자의 질문에 따라 어떤 Rule과 Variant가 선택되는지 확인하기에 좋았다.

TypeScript 구현을 통해 먼저 검증한 흐름은 다음과 같다.

```text
Rule JSON
↓
Validation
↓
Registry
↓
Rule Search
↓
Variant Selection
↓
Answer Generation
```

이 단계에서 TypeScript Rule Engine은 충분히 Reference Implementation 역할을 했다.

하지만 Ovendo의 장기 방향을 생각하면 Python 기반 구현도 필요했다.

앞으로 실험하고 싶은 기능들이 Python 생태계와 잘 맞기 때문이다.

```text
Embedding
Retriever
RAG
LLM orchestration
Data processing
Evaluation
```

특히 Rule Engine의 결과를 기반으로 Embedding 검색이나 RAG 실험을 이어가려면, Python 쪽에서도 Rule Engine Core가 동작하는 편이 확장하기 쉽다고 판단했다.

그래서 이번 Chapter의 목표는 TypeScript 구현을 버리는 것이 아니라,
TypeScript 구현을 기준으로 삼아 Python에서도 같은 결과를 내는 Rule Engine을 만드는 것이었다.

---

## 포팅의 기준

이번 포팅에서 가장 중요한 기준은 “개선”보다 “일치”였다.

Python 구현이 TypeScript 구현보다 더 똑똑해지는 것이 목표가 아니었다.
같은 Rule JSON을 읽고, 같은 입력에 대해 같은 결과를 반환하는 것이 우선이었다.

기준은 다음과 같이 잡았다.

```text
같은 Rule JSON
같은 입력
같은 Rule 선택
같은 Variant 선택
같은 Recommendation 반환
같은 AnswerResult 구조
```

Rule Engine v1의 핵심 원칙도 그대로 유지했다.

> 코드는 일반화하고, 지식은 데이터화한다.

베이킹 지식은 Python 코드 안에 하드코딩하지 않았다.
기존 Rule JSON을 그대로 사용하고, Python 코드는 이를 읽고 검증하고 검색하고 결과를 조립하는 역할만 담당하도록 했다.

---

## TypeScript 구현의 역할

TypeScript 구현은 이번 작업에서 기준점 역할을 했다.

기존 TypeScript Rule Engine에는 다음 기능이 포함되어 있었다.

```text
Rule JSON 구조 검증
Rule Registry
Rule Search
Variant Selection
Answer Builder
CLI
테스트
```

현재 사용 중인 Rule은 다음 5개다.

```text
cookie_spread
cookie_texture
cookie_color
cookie_shape
cookie_cracking
```

Python으로 포팅하는 동안 TypeScript 테스트도 계속 유지했다.

```text
TypeScript: 56 passed
```

즉, Python 구현을 추가하면서 기존 TypeScript Reference Implementation이 깨지지 않는지도 함께 확인했다.

---

## Python 구현 범위

Python에서는 Rule Engine Core가 독립적으로 동작할 수 있는 범위까지 구현했다.

이번에 포팅한 주요 범위는 다음과 같다.

```text
Rule type 정의
Rule validation
Rule registry
Rule JSON loader
Rule search
Variant selection
Answer generation
answer_question() API
CLI
Python tests
```

Rule JSON은 새로 만들지 않고 기존 파일을 공유했다.

```text
rules/problems/
  cookie-spread.json
  cookie-texture.json
  cookie-color.json
  cookie-shape.json
  cookie-cracking.json
```

즉, TypeScript와 Python이 같은 Rule 데이터를 읽는다.

이 구조 덕분에 지식은 여전히 JSON에 남고, 구현 언어만 달라져도 Rule 데이터는 재사용할 수 있다.

---

## 포팅하면서 주의한 부분

단순히 TypeScript 코드를 Python 문법으로 바꾸는 것만으로는 충분하지 않았다.

결과에 직접 영향을 주는 기준들이 있었기 때문이다.

첫 번째는 **Rule 등록 순서**였다.

여러 Rule이 동시에 매칭될 때 점수가 같으면 먼저 등록된 Rule이 선택된다.
따라서 Python에서도 TypeScript와 같은 순서로 Rule을 등록해야 했다.

```text
cookie_spread
cookie_texture
cookie_color
cookie_shape
cookie_cracking
```

두 번째는 **검색 대상 제한**이었다.

Rule JSON에는 `searchDescription` 필드가 있지만, 현재 Rule Search는 이 필드를 사용하지 않는다.
검색은 오직 `match.keywords`를 기준으로 수행한다.

만약 Python에서 Rule JSON 전체 문자열을 검색 대상으로 삼으면 TypeScript와 다른 결과가 나올 수 있다.
그래서 Python 구현에서도 검색 범위를 기존과 동일하게 유지했다.

세 번째는 **Variant 선택 순서**였다.

Variant는 선택된 Rule 안에서 결정된다.
이때 `match.symptoms` 배열 순서가 중요하기 때문에 Python에서도 같은 순서를 유지했다.

마지막으로 **AnswerResult 구조**도 맞췄다.

TypeScript v1에서 아직 사용하지 않는 필드가 있더라도 Python에서 생략하지 않았다.

```text
summary: ""
causes: []
followUpQuestions: []
```

현재는 비어 있어도 AnswerResult 구조의 일부이기 때문이다.

---

## 테스트 결과

포팅은 한 번에 전체를 옮기기보다 작은 단계로 나누어 진행했다.

```text
1. Python project scaffold
2. Type definitions
3. Validator
4. Registry / Rule JSON Loader
5. Search / Variant Selection
6. Answer Builder / answer_question
7. CLI
```

각 단계마다 테스트를 추가했고, 이전 단계가 깨지지 않는지 확인했다.

최종 테스트 결과는 다음과 같다.

```text
Python: 64 passed
TypeScript: 56 passed
```

Python 구현이 독립적으로 동작하는 것뿐 아니라, 기존 TypeScript 구현도 그대로 유지되는 것을 확인했다.

---

## CLI 실행

Python Rule Engine은 CLI에서도 실행할 수 있다.

예를 들어 다음 명령어로 JSON 결과를 확인할 수 있다.

```bash
cd python
python -m ovendo_rule_engine.cli.main --json "쿠키가 안 퍼져요"
```

질문을 입력하지 않으면 기본 질문을 사용하도록 했다.

```text
쿠키가 안 퍼져요
```

CLI는 새로운 진단 로직을 가지지 않는다.
단순히 `answer_question()`을 호출하고, 결과를 formatted text 또는 JSON으로 출력하는 얇은 실행 레이어다.

---

## 이번 Chapter에서 하지 않은 것

이번 작업에서는 의도적으로 다음 기능을 구현하지 않았다.

```text
Embedding
Vector DB
Rule Index
Retriever
RAG
LLM 연동
Multi-turn
UI
```

이 기능들은 Ovendo의 장기 방향에서는 중요하다.

하지만 이번 Chapter의 목표는 Python Rule Engine Core를 안정적으로 포팅하는 것이었다.

아직 Core가 안정화되지 않은 상태에서 RAG나 LLM을 붙이면, 문제가 생겼을 때 원인을 구분하기 어렵다.

그래서 이번에는 기반 구현에 집중했다.

---

## 이번 작업에서 얻은 것

이번 Chapter를 통해 TypeScript Rule Engine v1을 Python 기반으로 포팅했다.

완료된 항목은 다음과 같다.

```text
Rule JSON 로드
Validation
Registry
Rule Search
Variant Selection
Answer Generation
answer_question() API
CLI
Python tests
```

이번 작업의 의미는 단순히 언어를 바꾼 것이 아니다.

TypeScript에서 검증한 Rule Engine 구조를 Python에서도 동일하게 재현했고, 이후 AI 실험을 이어갈 수 있는 실행 기반을 마련했다.

정리하면 다음과 같다.

| 항목             | 결과                            |
| -------------- | ----------------------------- |
| TypeScript 구현  | Reference Implementation으로 유지 |
| Python 구현      | Ported Implementation 추가      |
| Rule JSON      | 기존 데이터 공유                     |
| Python 테스트     | 64 passed                     |
| TypeScript 테스트 | 56 passed                     |
| RAG / LLM      | 아직 구현하지 않음                    |

---

## 다음 단계

다음 단계에서는 Python Rule Engine을 기반으로 AI 실험 구조를 단계적으로 검토할 수 있다.

후보 작업은 다음과 같다.

```text
1. Rule Index 설계
2. Embedding 대상 필드 정리
3. Retriever 구조 검토
4. RAG 연결 가능성 실험
5. Rule Engine 결과와 LLM 설명 생성의 역할 분리
```

다만 바로 RAG를 붙이기보다는, 먼저 Rule Index와 검색 대상 필드를 정리하는 것이 우선일 것 같다.

---

## 정리

Ovendo Chapter 3의 핵심은 다음과 같다.

```text
새 기능 추가보다 결과 일치
구조 변경보다 안정적 포팅
AI 확장보다 Rule Engine Core 완성
```

이번 작업을 통해 Ovendo는 TypeScript Reference Implementation과 Python Ported Implementation을 함께 갖게 되었다.

Ovendo의 방향은 여전히 같다.

> 코드는 일반화하고, 지식은 데이터화한다.

이제 이 구조를 Python 환경에서도 사용할 수 있게 되었고, 다음 단계에서는 Rule Index, Embedding, Retriever, RAG 구조를 조금씩 검토할 수 있다.
