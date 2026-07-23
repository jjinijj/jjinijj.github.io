---
layout: post
title: "[TripBranch] 개발일지 #002 — TripBranch AI 기능 패키지 분리와 팀 역할 재배분"
date: 2026-07-22
categories: [TripBranch]
tags: [Planning, AI-Agent]
---

## 배경

TripBranch의 AI 기능을 세부 업무로 바로 나누자 기능 수가 지나치게 많아졌다.
이에 연관된 기능을 네 개의 책임 패키지로 재구성했다.

## AI 기능 패키지

- A: Request Intelligence·Agent Runtime
- B: Agent State·Memory·LLMOps
- C: Tool Intelligence·External Context
- D: Recommendation Intelligence·AI Quality

## 역할 배분 과정

역할은 직책에 따라 일방적으로 배정하지 않고,
팀원들이 희망하는 패키지를 먼저 선택하도록 했다.

- 임**: A
- 이**: B
- 나**: D
- 김**: C

## 고민한 점

C는 외부 API와 Tool 구현 비중이 높아
처음에는 팀장 역할과 거리가 있는 영역처럼 보였다.

하지만 팀장의 횡단 책임과 개인의 개발 책임은
반드시 같은 기능 영역일 필요가 없다고 판단했다.

## 최종 책임 정의

김**의 개발 책임은 Tool 구조와 External Context 구축이다.

팀장으로서의 횡단 책임은 다음과 같다.

- 전체 우선순위 관리
- 패키지 간 인터페이스 합의
- 의존성과 통합 일정 조정
- 체크포인트 완료 판단
- 최종 통합 품질 확인

## 배운 점

팀장이 반드시 가장 중심적인 기능을 맡아야 하는 것은 아니다.
팀원의 희망과 전문성을 존중하면서,
각 영역의 책임 경계를 명확히 하고 전체 결과를 연결하는 것도
팀장의 중요한 역할이라고 판단했다.