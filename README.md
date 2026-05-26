# stock-slack-report

매일 아침 한국 주식 시장 데이터를 수집해 **Slack**으로 보내주는 자동화 리포트.
KOSPI TOP 50 시가총액 / 외국인·기관 순매수 / Claude(AWS Bedrock) AI 분석 / 모의투자 수익률을 한 번에.

> 모든 작업은 **GitHub Actions** 위에서 동작합니다. 서버를 띄울 필요가 없습니다.

---

## 기능

### 1. 일일 리포트 (`.github/workflows/stock-report.yml`)
- **스케줄**: 평일 매일 오전 07:10 KST (`cron: '10 22 * * 0-4'`)
- KOSPI 시가총액 TOP 50 + 종가 + 등락률
- 외국인 순매수 TOP 10 (KOSPI)
- 기관 순매수 TOP 10 (KOSPI)
- **Claude Sonnet 4.5** (AWS Bedrock) 3줄 핵심 분석
- 일일 스냅샷을 `data/YYYY-MM/YYYY-MM-DD.json`에 저장
- **모의투자**: 매일 시총 TOP 5를 1주씩 매수해 `portfolio.json`에 기록

### 2. 주간 분석 (`.github/workflows/weekly-analysis.yml`)
- **스케줄**: 월요일 07:10 KST (`cron: '10 22 * * 0'`)
- 지난 5거래일 데이터 집계
- 일별 TOP3 등락 / 주간 평균 상승·하락 TOP5
- **Claude Sonnet 4.5** 주간 인사이트 (불릿 5개)
- **모의투자 누적 수익률** — 종목별/전체 손익 계산

---

## Repo 구조

```
stock-slack-report/
├── .github/workflows/
│   ├── stock-report.yml      ← 일일 리포트 (평일 07:10 KST)
│   └── weekly-analysis.yml   ← 주간 분석 (월요일 07:10 KST)
├── data/
│   └── YYYY-MM/
│       └── YYYY-MM-DD.json   ← 일일 시총 스냅샷 (자동 커밋)
└── portfolio.json            ← 모의투자 기록 (자동 갱신)
```

---

## 필요한 Secrets

GitHub repo → Settings → Secrets and variables → Actions 에 등록:

| Secret | 용도 |
|---|---|
| `SLACK_WEBHOOK_URL` | Slack Incoming Webhook URL |
| `AWS_ACCESS_KEY_ID` | AWS Bedrock 호출용 |
| `AWS_SECRET_ACCESS_KEY` | 〃 |
| `AWS_REGION` | Bedrock 리전 (코드 내 `us-east-1` 고정 호출, 환경변수는 인증용) |
| `KRX_ID`, `KRX_PW` | (예비) KRX 계정 |

---

## 스택

- **데이터**: [`pykrx`](https://github.com/sharebook-kr/pykrx) (KRX 일별 시세 / 시가총액 / 순매수)
- **AI**: AWS Bedrock — `us.anthropic.claude-sonnet-4-5-20250929-v1:0`
- **알림**: Slack Incoming Webhook
- **런타임**: GitHub Actions (Python 3, `pykrx requests boto3`)

---

## 수동 실행

GitHub → Actions 탭 → 워크플로 선택 → **Run workflow** (workflow_dispatch).

---

## 데이터 포맷

`data/YYYY-MM/YYYY-MM-DD.json`:
```json
{
  "date": "YYYY-MM-DD",
  "kospi": [
    { "rank": 1, "ticker": "...", "name": "...",
      "close": 0, "market_cap": 0, "change_pct": 0.0 }
  ],
  "foreign_net_buy": [ { "name": "...", "amount": 0 } ],
  "inst_net_buy":    [ { "name": "...", "amount": 0 } ]
}
```

`portfolio.json`:
```json
[
  { "date": "YYYY-MM-DD", "rank": 1, "name": "...",
    "ticker": "...", "buy_price": 0, "shares": 1 }
]
```

---

## 비용 메모

- Bedrock Claude Sonnet 4.5: 일일 ~300 tokens / 주간 ~600 tokens 출력 — 월 수십원 단위
- GitHub Actions Public repo는 무료
