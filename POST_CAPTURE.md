# A1 잡힌 직후 — 이걸로 뭘 할까

> 본 문서는 A1.Flex 4 OCPU / 24 GB 인스턴스를 캡처한 직후 의사결정용 가이드.
> 텔레그램 성공 알림에서 링크되어 들어온다. 결론부터:

## 한 줄 결론

**NTR 매매 VM은 절대 이전하지 말고 그대로 둔다.** 본 A1은 "보너스 자원"으로 신규 워크로드에 할당.

## 왜 NTR을 옮기지 않는가

`news-trade-runner` 실측 워크로드:

| 항목 | 실제 부하 |
|---|---|
| Python deps | `httpx`, `pydantic-settings` 2개 — ~100~200MB |
| 실행 모델 | systemd timer 5개 (fetch/review/session/report/manual). 상주 데몬 아님 |
| WebSocket | KIS WS, **후보 종목 한정** — 동시 연결 수 작음 |
| Redis | self-host, 키 6종 + 짧은 TTL — ~50~100MB |
| LLM | Gemini Flash API 1~2회/일, 로컬 추론 0 |
| 가동 | 평일 09:00–15:30만 활성, 나머지는 idle |

→ 피크에서도 **~500–700MB / CPU<1코어**. 1 OCPU + 1GB AMD + swap 2GB로 충분.

매매 VM 이전은 항상 리스크 (KIS 토큰·systemd·Redis 데이터·네트워크 재검증). 이미 잘 돌고 있는 걸 바꿀 이득이 작다.

## A1 활용 옵션 (NTR 시너지 순)

### ① 시뮬·그리드 백테스트 전용 머신 ⭐ 최우선

지금 로컬 PC에서 `scripts/sim_v2_grid_*.py` 그리드 서치를 돌리고 있는데, 4 OCPU ARM에 옮기면:
- 24/7 병렬 실행 가능
- 노트북 자원 회수
- `gemini-analysis-step.md`의 파라미터 5개 학습 사이클 가속
- `/ntr §C` 시뮬·그리드 메뉴와 직접 결합

**셋업 골자:**
- `news-trade-runner` 일부(`engine/src/engine/sim*`, `scripts/sim_*.py`)만 clone
- venv + Supabase read-only 키 (trades 읽기)
- systemd timer 또는 수동 트리거 (사이클 부담 적음)

### ② NTR 스테이징 / 카나리 VM

운영 VM과 동일한 systemd 구성을 A1에 미러로 깔고:
- `DRY_RUN=true`
- 별도 모의계좌 또는 같은 계좌 + 주문 차단 플래그
- 새 룰을 1주 검증한 뒤 운영 `.env` 반영

5/15 같은 incident 재발 방지용. `_snapshot_balance` 류 결함이 운영 도달 전에 잡힘.

### ③ 자체 호스팅 모니터링 백엔드

- Grafana + Loki + Prometheus
- NTR `journalctl`·`trades` 테이블을 끌어다 시각화
- Vercel·Supabase 무료 한도 절약
- 텔레그램 외 자체 알림 채널

### ④ 보조 데이터 파이프라인

- 한투/공시/뉴스 raw 데이터 상시 수집 → Supabase 적재
- `news-trading-bot` strategy repo가 사용할 피처 누적
- 무료 티어 한도가 가장 빠듯한 부분 보강

## 비추 옵션

**로컬 LLM (Ollama Qwen/Llama 등) 자체 호스팅.**
ARM Ampere 추론 속도가 Gemini Flash 무료 한도 대비 가성비 안 나옴.

## 우선순위 / 액션

1. SSH 첫 접속·기본 보안 (timezone, ufw, security list 22만)
2. **① 시뮬 머신**부터 셋업 (가장 즉시적 가치)
3. 안정화되면 **② 스테이징** 추가
4. ③·④는 운영 안정성 / 무료 한도 압박 발생 시 도입

## 참고

- 본 분석 출처: `C:\Users\kook0\AI\CC` 세션 2026-05-19
- NTR 기획: `C:\Users\kook0\AI\CC\vault\50_Projects\news-trade-runner\PLAN.md`
- NTR repo: `C:\Users\kook0\AI\news-trade-runner`
- VM 셋업 가이드: `news-trade-runner/docs/planning/setup/P0.3-oracle-vm.md` §3
