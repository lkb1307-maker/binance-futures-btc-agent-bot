# AGENT.md — Binance Futures BTC Never-Die Bot

## 1. 목적 / 범위

### 목적

이 프로젝트는 Binance Futures BTCUSDT Perpetual 자동매매 봇을 구축한다.
목표는 최대 수익이 아니라 생존 확률을 극대화하는 것이다.

### 포함 범위

- Binance Futures Testnet 연동
- Paper Trading Simulator
- 전략 엔진 (exchange-agnostic)
- 리스크 관리 시스템
- 실행 계층 (order routing)
- sqlite 기반 로그 저장
- CI 기반 PR 자동 검증

### 제외 범위

- 알트코인 거래
- 고레버리지 전략
- 손절 없는 전략
- 실계좌 자동 운용 (명시적 승인 전 금지)

---

## 2. 역할 분리

Product Owner: Alvin
Architect: CODEX AGENT
Implementer: CODEX AGENT
Reviewer: CI + Self Review
Approver: CI 통과 시 자동 승인

규칙:

- 설계 없이 구현 금지
- ADR 없이 구조 변경 금지
- CI 실패 상태에서 머지 금지
- Alvin 승인 없이 실거래 금지

---

## 3. 우선순위 규칙

1. 보안 > 자본 보호 > 시스템 안정성 > 정확성 > 성능 > 속도
2. 리스크 통제 > 전략 수익률
3. Fail-Closed > Fail-Open
4. 테스트 통과 > 기능 추가

---

## 4. Trigger → Mandatory Action

연속 손실 3회 → 1시간 거래 중단
일 손실 한도 초과 → 당일 거래 중지
API 오류 3회 → 신규 주문 차단
포지션 불일치 감지 → 즉시 Reconciliation 실행

코드 변경 발생 → 반드시 브랜치 생성
PR 생성 → CI 자동 실행
CI 실패 → 수정 후 동일 PR 업데이트

---

## 5. 품질 게이트 (Definition of Done)

모든 PR은 다음을 만족해야 한다:

- ruff check 통과
- pytest 통과
- 신규 기능 테스트 포함
- 전략 변경 시 관련 테스트 수정
- 로그 포맷 유지

Demo Acceptance Criteria:

- Testnet 7일 무중단
- Daily Loss Limit 정상 작동
- API 오류 시 주문 폭주 없음
- 재시작 시 포지션 동기화 정상
- 모든 주문 sqlite 기록

---

## 6. 금지사항

- .env 자동 수정 금지
- API 키 코드 하드코딩 금지
- 실거래 모드 자동 활성화 금지
- Force push 금지
- CI 우회 머지 금지
- Stop-loss 없는 주문 생성 금지

---

## 7. 기록 규칙

세션 시작 시:

- 작업 목표 기록
- 변경 범위 기록

커밋 메시지 형식:

type(scope): short description

- Why:
- Risk impact:
- Test:

로그는 JSON 구조로 남긴다.
이벤트 타입:
DECISION / ORDER / FILL / ERROR / RISK_BLOCK

---

## 8. 예외 / 실패 대응

CI 실패 → 수정 후 재실행
전략 손실 누적 → 거래 자동 중지 후 분석
API 장애 → 신규 주문 차단

롤백:
마지막 안정 버전으로 복원
