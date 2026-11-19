---

layout: post
title: "[TripBranch] 개발일지 #005 — 팀 계정 인프라 세팅 기록"
date: 2026-08-24
categories: TripBranch
---

## 1. 왜 이걸 하게 됐나

### 출발점
4명이 Claude를 함께 쓰려고 했다. 처음 떠올린 방법은 "Max 계정 하나 결제해서 같이 쓰기"였지만 불가능했다.

- Anthropic 소비자 약관상 계정 공유 금지
- 대화 기록·프로젝트가 전부 한 계정에 섞임
- 여러 IP 동시 접속 시 이상 사용으로 감지될 수 있음
- 세션 한도를 4명이 나눠 갉아먹음

### 결제 방식 검토

| 옵션 | 4명 월 비용 | 문제점 |
|---|---|---|
| Max 공유 | $100~200 | 약관 위반 |
| API 키 종량 | $400~500 (본격 개발 시) | 예측 불가, 웹 UI 사용 불가 |
| Pro 개별 4개 | $80 | 한도 부족, **개인 결제 불인정** |
| Team 플랜 | $100~200 | **회사 도메인 이메일 필요** |

### 결정적 제약
교육과정 페이백 조건이 **"개인 구독 상품(Pro/Max)은 불인정, 조직 결제만 인정"**.
→ Team 플랜이 유일한 선택지.
→ 그런데 Team은 관리자 계정이 반드시 업무용 도메인 이메일이어야 함. gmail·yahoo·hotmail 등 공용 도메인 불가.
→ **도메인을 사야 한다.**

### 예산
- 페이백 상한: 20만 원
- 실제 예상: 15만 원 (다른 비용 포함)
- Claude에 쓸 수 있는 금액: 약 $95

---

## 2. 최종 구성

```
tripbranch.co.kr (가비아 등록)
        ↓ 네임서버 위임
   Cloudflare (DNS 관리)
        ↓ Email Routing
teamroot@tripbranch.co.kr → genai.teamroot@gmail.com
        ↓
   Claude Team 플랜 관리자 계정
```

### 계정 구조

| 구분 | 용도 | 결제 |
|---|---|---|
| 진 개인 Max | 본인 개발 작업 | 개인 부담 (페이백 대상 아님) |
| Team Standard 시트 | 팀원 3명 | 조직 결제 (페이백 대상) |
| Console API 키 | 에이전트 실습·자동화 | 조직 결제 |

**시트 등급 참고**
- Standard = Pro 대비 1.25배
- Premium = Pro 대비 6.25배
- 진은 Pro 한도는 다 쓰고 Max 5x는 남는 편이라, 실사용량은 Pro의 2~3배 정도

---

## 3. 진행 과정

### 3-1. 도메인 구매
- 원하던 `tripbranch.com`은 선점됨 → `tripbranch.co.kr`로 결정
- `.co.kr`은 **국내 등록기관에서만** 구매 가능 (Cloudflare에서 직접 등록 불가)
- 가비아에서 구매, 연 1~2만 원

**체크 사항**
- 자동 갱신 여부 확인 (계속 쓸 게 아니면 끄기)
- 첫 해 할인가 ≠ 갱신가

### 3-2. 메일 설정 (여기서 제일 헤맴 → 4장 참조)
최종적으로 Cloudflare Email Routing 사용.

1. Cloudflare 가입 → **Connect a domain** (Transfer 아님)
2. `tripbranch.co.kr` 입력, Free 플랜 선택
3. DNS 레코드 없음 상태로 **Add records later → Confirm**
4. Cloudflare가 네임서버 2개 발급
   ```
   dee.ns.cloudflare.com
   nero.ns.cloudflare.com
   ```
5. 가비아 → 도메인 관리 → 네임서버 설정
   - 기존 `ns.gabia.co.kr`, `ns1.gabia.co.kr` 삭제
   - Cloudflare 주소 2개 입력
   - 소유자 본인인증 (휴대폰) 필요
6. 전파 대기 (안내상 1~2시간, 최대 24시간)
7. zone이 **Active**가 된 후 → Email Routing → Settings → **Add missing records**
   - MX 3개 + TXT 2개(DKIM, SPF) 자동 등록
8. Routing rule 생성: `teamroot` @ `tripbranch.co.kr` → Send to an email → gmail
9. Destination Address 인증 (gmail로 온 확인 링크 클릭)

### 3-3. Team 플랜 개설
- 시크릿 창에서 claude.ai/login (개인 Max 계정과 섞이지 않게)
- `teamroot@tripbranch.co.kr`로 신규 가입
- **개인 Max를 업그레이드하지 않음** ← 중요, 사유는 4장

---

## 4. 시행착오

### ❌ 가비아 "포워딩" 메뉴는 메일과 무관
`포워딩` 메뉴는 **웹 포워딩**(URL 리디렉션)이다. 화면에 `http://` 입력란과 "연결 주소"가 나오면 이것.
저장하면 A 레코드가 가비아 파킹 서버(121.254.178.231)로 박혀서 나중에 배포할 때 걸리적거린다. **누르지 말 것.**

### ❌ "DNS 호스트" 메뉴도 아님
자체 네임서버를 직접 운영할 때 쓰는 기능. 호스트명 + IP 입력란이 나온다. 메일과 무관.

### ❌ 가비아 무료 웹메일(하이웍스)은 1년치 결제
과거 자료에는 "무료 웹메일 도메인당 1계정 제공"이라고 나오는데, 실제로는 1년 단위 결제 상품이었다. 한 달만 쓸 거면 부적합.

### ❌ zone이 Active 되기 전엔 Email Routing 활성화 불가
```
This zone must be active before you can enable Email Service.
```
네임서버 전파가 끝나야 함. "Check nameservers now" 버튼으로 즉시 재확인 가능.

### ❌ MX 레코드 등록 전 발송 → 부정 캐시에 걸림
```
DNS type 'mx' lookup of tripbranch.co.kr responded with code NOERROR
DNS type 'mx' lookup of tripbranch.co.kr had no relevant answers
```
`NOERROR + no relevant answers` = 도메인은 존재하지만 MX가 비어 있다는 뜻.
한 번 이 응답을 받은 발신 서버(gmail 등)는 "MX 없음"을 몇 시간 캐싱한다. 진 쪽 설정을 고쳐도 발신 서버가 옛날 정보를 보고 계속 반송함.

**대응**
- 다른 메일 서비스(네이버·다음)에서 보내보면 캐시가 없어 바로 감 → 캐시 문제 확진
- 근본적으로는 시간이 답
- **교훈: MX 등록이 끝난 걸 확인한 뒤에 테스트 메일을 보낼 것**

### ⚠️ 개인 Max를 Team으로 마이그레이션하면 안 되는 이유
- 기존 개인 구독이 자동 해지됨
- 개인 워크스페이스로 콘텐츠를 **되돌릴 수 없음**
- 기존 프로젝트·대화 기록이 걸려 있으면 위험

→ 별도 계정으로 Team을 새로 개설하는 것이 안전하다.

### ⚠️ `ANTHROPIC_API_KEY` 환경변수 함정
셸 프로필에 이 변수가 남아 있으면 Claude Code가 구독(OAuth) 대신 API 키를 **조용히 우선 사용**한다. Max를 결제해두고 API 요금이 따로 나가는 사고가 흔하다.

**확인**: `claude` 실행 후 `/status`
- `CLAUDE_CODE_OAUTH_TOKEN` → 구독
- `ANTHROPIC_API_KEY` → 종량 과금

**되돌리기**: `unset ANTHROPIC_API_KEY` 후 터미널/VS Code 완전 재시작
**권장**: 셸 프로필 전역 설정 금지. direnv로 프로젝트 폴더 단위 적용.

---

## 5. 유용했던 진단 명령

```bash
# 네임서버 전환 확인
nslookup -type=ns tripbranch.co.kr
# → cloudflare.com이 나오면 전환 완료

# MX 레코드 확인
nslookup -type=mx tripbranch.co.kr
# → route1/2/3.mx.cloudflare.net 나오면 정상

# 구글 DNS로 직접 조회 (캐시 우회)
nslookup -type=mx tripbranch.co.kr 8.8.8.8
```

웹으로 확인하려면 `dnschecker.org`에서 전 세계 전파 상황을 볼 수 있다.

**Cloudflare Activity Log 읽는 법**
- 기록 없음 → 메일이 Cloudflare까지 도달 못 함 (DNS/캐시 문제)
- 기록 있는데 실패 → 목적지 주소 문제 (인증 미완료 등)

---

## 6. 증빙 자료 정리

### 비용 증빙
| 서류 | 위치 |
|---|---|
| 도메인 결제 영수증 | 가비아 마이페이지 |
| Team 플랜 인보이스 | claude.ai → Settings → Billing → Invoices |
| Console API 인보이스 | platform.claude.com → Billing |
| 해외결제 원화 환산 | 카드사 앱 명세서 |

### 사용 증빙
- Console → Usage / Cost 페이지에서 워크스페이스·기간 필터 후 **CSV 내보내기**
- 키 이름을 `이름-용도` 형식으로 지어두면 사람별·용도별로 나눠 볼 수 있음
- git 커밋 히스토리
- `~/.claude/projects/` 아래 프로젝트별 세션 기록

### 주의
- Anthropic은 해외 사업자라 **한국 세금계산서 발행 불가**. 카드 매출전표/인보이스가 전부.
- 워크스페이스를 아카이브하면 소속 API 키가 즉시 폐기되고 복구 불가. **정산 완료 전 건드리지 말 것.**
- 인보이스 청구지 정보는 **발행 전에** 수정해야 반영됨. 소급 불가.

---

## 7. 운영 시 체크리스트

### API 비용 관리
- [ ] Console 워크스페이스 **Spend limits** 설정 (처음엔 낮게)
- [ ] 자동 충전(auto-reload) **끄기** — 충전액이 곧 하드 상한
- [ ] 분류·추출은 Haiku, 어려운 추론만 Sonnet으로 라우팅
- [ ] 프롬프트 캐싱 활용 (캐시 읽기 = 기본 입력가의 10%)
- [ ] 서브에이전트 병렬 실행 주의 — 인스턴스마다 컨텍스트가 따로 잡혀 비용이 인스턴스 수만큼 늘어남

### 키 관리
- [ ] 팀원을 Console 멤버로 초대해 **각자 키를 직접 발급**하게 함 (키를 사람끼리 주고받지 않는 게 가장 안전)
- [ ] `.env`, `.envrc`, `~/.claude/settings.json`을 `.gitignore`에 추가
- [ ] 키 유출 시 즉시 폐기·재발급 (숨기는 게 더 위험하다고 미리 공지)
- [ ] "키별 사용량이 관리자에게 보인다"는 점 팀원에게 사전 고지 (내용은 안 보임)

### 갱신·해지
- [ ] 도메인 자동 갱신 설정 확인
- [ ] 해지 순서: **Team 플랜 먼저 해지 → 이후 도메인 만료**
  - 도메인이 먼저 죽으면 관리자 메일이 끊겨 비밀번호 재설정·청구 알림 수신 불가

---

## 8. 남은 활용

도메인을 샀으므로 프로젝트 서비스에 그대로 붙일 수 있다. MX(메일)와 A/CNAME(웹)은 레코드 종류가 달라 충돌하지 않는다.

```
tripbranch.co.kr        → 랜딩 페이지
app.tripbranch.co.kr    → 앱
api.tripbranch.co.kr    → 백엔드
teamroot@tripbranch.co.kr → 관리자 메일
```

서브도메인은 무제한, 추가 비용 없음.
DNS가 이미 Cloudflare에 있으므로 Cloudflare Pages로 배포하면 도메인 연결이 클릭 몇 번으로 끝난다. 무료 SSL 자동 발급.