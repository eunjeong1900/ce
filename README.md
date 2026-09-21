# DART 재무분석 Agent — 서울대학교병원

GitHub repository: `eunjeong1900/ce`

이 프로젝트는 OpenDART API를 이용해 서울대학교병원의 정기보고서/재무제표를 자동 수집하고, 첨부된 **재무제표 및 재무비율 실무 가이드**의 핵심 지표를 계산해 GitHub Pages 대시보드로 제공합니다.

## 구성

- `src/dart_agent.py` — DART 기업코드 조회, 사업보고서 검색, 재무제표 수집, 원문 저장, 비율 계산
- `data/financials.csv` — 정규화된 연도별 재무 데이터
- `data/ratios.csv` — 계산된 재무비율
- `data/dashboard.json` — 대시보드용 데이터
- `data/raw/` — DART 원문 XML/ZIP 보관
- `dashboard/index.html` — GitHub Pages 정적 대시보드
- `.github/workflows/update-and-deploy.yml` — 매일 자동 업데이트 + Pages 배포

## 자동화 흐름

1. GitHub Actions가 매일 실행
2. OpenDART `corpCode.xml`에서 `서울대학교병원`의 DART 고유번호를 확인
3. 최근 사업보고서(기본 5개 연도)를 조회
4. 보고서 원문과 재무제표 데이터를 저장
5. 주요 재무수치와 재무비율을 계산
6. `data/`에 변경분을 commit
7. 같은 workflow에서 GitHub Pages를 배포

## API Secret

GitHub 저장소에서 **Settings → Secrets and variables → Actions → New repository secret**으로 아래 Secret을 추가하세요.

- Name: `DART_API_KEY`
- Value: OpenDART에서 발급받은 40자리 인증키

API 키는 코드에 절대 넣지 않습니다.

## 로컬 실행

```bash
python -m venv .venv
# Windows: .venv\\Scripts\\activate
# macOS/Linux: source .venv/bin/activate
pip install -r requirements.txt

# PowerShell
$env:DART_API_KEY="발급받은키"
python src/dart_agent.py
```

## GitHub Pages

저장소의 **Settings → Pages → Source**를 `GitHub Actions`로 선택합니다. Workflow에는 Pages 배포 권한이 포함되어 있습니다.

## 주요 계산 지표

첨부 가이드 기준으로 다음을 포함합니다.

- 매출액, 매출총이익, 영업이익, 세전이익, 당기순이익
- 총자산, 현금및현금성자산, 매출채권, 재고자산, 유형자산
- 총부채, 이자부차입금, 자본총계
- 영업활동현금흐름, 투자활동현금흐름, 재무활동현금흐름, CAPEX, FCF
- 매출총이익률, 영업이익률, 순이익률, EBITDA 마진
- ROA, ROE, 유동비율, 당좌비율
- 부채비율, 자기자본비율, 차입금의존도, 이자보상배율
- 순차입금/EBITDA, 총자산회전율, 매출증가율, CFO/순이익, CCC
- 데이터가 충분할 경우 DSO/DIO/DPO

일부 항목은 DART/XBRL에서 직접 제공되지 않거나 계정과목 명칭이 회사별로 달라질 수 있습니다. 그런 경우 `null`로 두고 대시보드에서 `N/A`로 표시합니다.

## 중요: 서울대학교병원 데이터 특성

서울대학교병원은 일반 상장기업과 달리 `기타법인`일 수 있으며, DART의 XBRL 재무정보 제공 대상 여부에 따라 `fnlttSinglAcntAll`에서 재무데이터가 반환되지 않을 수 있습니다. Agent는 이를 오류로 숨기지 않고 로그에 남기며, 사업보고서 검색/원문 저장과 별도로 처리합니다.
