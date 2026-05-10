# OCI A1 Hunter

GitHub Actions가 10분마다 Oracle Cloud Free Tier의 A1.Flex 4 OCPU / 24 GB 인스턴스를 시도.

## 동작
- `launch.yml`: 10분마다 launch 시도. 성공하면 텔레그램 알림 + `.a1_captured` 마커 commit + 워크플로 자동 비활성화.
- `daily-status.yml`: 매일 KST 09:00 (UTC 00:00) 진행 상황 텔레그램 요약. 이미 잡혔으면 송신 안 함.

## 멈추기
잡힌 후엔 `gh workflow disable` 자동 호출되므로 수동 정리 불필요.
다시 돌리려면 `.a1_captured` 파일 제거 + `gh workflow enable launch.yml`.

## 리전
- ap-chuncheon-1 (춘천)
- AD: AP-CHUNCHEON-1-AD-1

## 사양
- VM.Standard.A1.Flex
- 4 OCPU / 24 GB / Ubuntu 22.04 ARM
- Public IP 자동 할당
