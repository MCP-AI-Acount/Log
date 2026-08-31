# VM 스케줄러 자동화 실패 방지

- Cloud Scheduler VM start 성공만으로 자동화 성공으로 판단하지 않는다.
- 실패 원인 구분:
  1. Cloud Scheduler가 VM start 호출 실패
  2. VM startup-script / vm_boot_services 실패
  3. n8n 워크플로 미기동 또는 브리지 미기동
  4. 카드 생성/Canva/SNS 단계 실패
- 장애 조사 순서:
  1. `gcloud scheduler jobs list --filter='name~mcp-auto-vm'`
  2. Cloud Scheduler 로그 확인
  3. VM `~/vm-boot-services.log`, `~/daily-ops-bridge.log` 확인
  4. 브리지 health `8792,8795,8796,8798` 확인
  5. 실제 파이프라인 dry/skip-canva 검증
- 정규 운영 부팅 시에만 daily ops state reset 한다.
- 실패 상태가 남으면 VM이 무기한 켜지지 않도록 fail self-stop을 켠다.
- 저녁 토픽 뉴스 실패 시 `tech` 기사 fallback이 4건 확보되는지 먼저 확인한다.
- 카드뉴스 길이는 공백 포함: 제목 10~15자, 3줄요약 각 줄 12~20자.
