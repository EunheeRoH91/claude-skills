# claude-skills

Claude Code 플러그인 마켓플레이스입니다.

## 설치

```bash
claude plugin marketplace add EunheeRoH91/claude-skills
claude plugin install weekly-sales-report@claude-skills
```

Claude 앱에서는 [`dist/weekly-sales-report.skill`](dist/weekly-sales-report.skill) 파일을 받아 스킬로 추가할 수 있습니다.

## 포함된 스킬

### weekly-sales-report (위클리 세일즈 리포트)

Google Sheet `판매관리`의 `판매내역` 시트에서 지난주(월–일) 매출을 집계해 전주와 비교하고, Slack 채널에 한 줄 헤더와 스레드 상세 리포트로 올립니다.

**필요한 커넥터:** Google Drive / Google Sheets, Slack

**사용 전 설정:** `SKILL.md`의 0번 섹션에서 아래 자리표시자를 본인 값으로 바꾸세요.

- `<YOUR_CHANNEL>`, `<YOUR_CHANNEL_ID>`: 리포트를 올릴 Slack 채널
- `<SHEET_OWNER_EMAIL>`, `<YOUR_SHEET_ID>`: 원본 Google Sheet
