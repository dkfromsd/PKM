# Vibe Investing

Vibe Investing은 금융과 투자 분석을 위한 재사용 가능한 `SKILL.md` 기반 스킬 저장소입니다. 이 저장소의 목적은 일회성 프롬프트를 모으는 것이 아니라, 반복 가능한 리서치 워크플로, 시장 해석 프레임, 포트폴리오 판단, 보고서 포맷을 모듈 형태로 정리하는 데 있습니다.

Claude에서 이 저장소를 사용할 때는 `AGENTS.md`와 각 스킬의 `SKILL.md`를 먼저 읽고, 필요한 경우에만 `references/`, `scripts/`, `assets/`를 추가로 로드하는 방식으로 해석하면 됩니다.

## 저장소 아키텍처

이 저장소는 다음 4개 계층으로 보면 가장 이해하기 쉽습니다.

1. 저장소 규칙 계층
   - 루트 `AGENTS.md`는 전체 저장소의 편집 규칙을 정의합니다.
   - `skills/AGENTS.md`는 스킬 폴더 전체의 구조 원칙을 정의합니다.

2. 기능 계층
   - `skills/<category>/<skill-name>/SKILL.md`가 실제 실행 단위입니다.
   - 각 스킬은 하나의 투자 능력에 집중합니다.

3. 근거 계층
   - `references/`에는 긴 설명, 체크리스트, 출력 계약, 검증 규칙을 둡니다.
   - `scripts/`에는 반복 실행이 필요한 결정적 작업만 둡니다.
   - `assets/`에는 스킬이 직접 참조하는 보조 자료를 둡니다.

4. 출력 계층
   - 분석 결과는 각 스킬의 `output-contract.md` 또는 `SKILL.md`가 정의한 형식으로 정리됩니다.
   - 이 저장소는 “무엇을 분석할지”보다 “어떤 절차로 검증하고 어떤 형식으로 내보낼지”에 더 무게를 둡니다.

## Claude에서의 사용 흐름

Claude가 이 저장소를 해석할 때의 권장 순서는 다음과 같습니다.

1. 루트 `AGENTS.md`를 확인해 저장소 전역 규칙을 읽습니다.
2. 요청에 맞는 최상위 카테고리를 찾습니다.
3. 해당 스킬의 `SKILL.md`를 읽어 작업 절차를 결정합니다.
4. 필요할 때만 `references/`를 추가로 확인합니다.
5. 결과를 스킬이 요구하는 출력 계약에 맞춰 작성합니다.

예를 들어:

- 기업 분석 요청이면 `skills/fundamental-analysis/company-analysis`
- 시장 국면, 매크로, 유동성 해석이면 `skills/market-analysis/traditional-market-analysis`
- 시그널 설계, 백테스트, 오버피팅 방어는 `skills/quantitative-analysis/quant-research`
- 데이터 수집은 `skills/data-access/openbb-data-fetcher`
- 최종 보고서 포맷팅은 `skills/output-formats/financial-report`

## 현재 스킬 구성

### 핵심 분석

- `skills/fundamental-analysis/company-analysis`
- `skills/data-access/openbb-data-fetcher`
- `skills/market-analysis/traditional-market-analysis`
- `skills/quantitative-analysis/quant-research`
- `skills/output-formats/financial-report`

### 투자자 페르소나

- `skills/investor-personas/aswath-damodaran`
- `skills/investor-personas/ben-graham`
- `skills/investor-personas/bill-ackman`
- `skills/investor-personas/cathie-wood`
- `skills/investor-personas/charlie-munger`
- `skills/investor-personas/michael-burry`
- `skills/investor-personas/mohnish-pabrai`
- `skills/investor-personas/nassim-taleb`
- `skills/investor-personas/peter-lynch`
- `skills/investor-personas/phil-fisher`
- `skills/investor-personas/rakesh-jhunjhunwala`
- `skills/investor-personas/stanley-druckenmiller`
- `skills/investor-personas/warren-buffett`

### 분석 결과 (Stock Analysis)

- `stock_analysis/`: 개별 기업 및 시장 분석 보고서 저장소
  - `2026-04-27-AMKR-CLS-Analysis.md`: AMKR, CLS 상세 분석
  - `2026-04-27-Multi-Stock-Analysis.md`: CRDO, MSFT, PLTR 등 8종목 복합 분석
- `portfolio/outputs.md`: 포트폴리오 분석 및 전략 실행 이력

## 디렉터리 구조

```text
vibe-investing/
├── README.md
├── AGENTS.md
├── stock_analysis/
└── skills/
    ├── AGENTS.md
    ├── data-access/
    ├── fundamental-analysis/
    ├── investor-personas/
    ├── market-analysis/
    ├── output-formats/
    └── quantitative-analysis/
```

각 스킬은 독립 폴더로 구성됩니다.

```text
skills/<category>/<skill-name>/
├── SKILL.md
├── references/   # 선택사항
├── scripts/      # 선택사항
└── assets/       # 선택사항
```

`SKILL.md`만 필수이며, 나머지는 스킬 성격에 따라 추가합니다.

## 분석 아키텍처 요약

이 저장소의 분석 아키텍처는 “입력 -> 스킬 선택 -> 검증 -> 출력”으로 요약할 수 있습니다.

- 입력: 회사명, 시그널 아이디어, 시장 질문, 데이터 요청, 보고서 요청
- 스킬 선택: 문제 유형에 맞는 전용 스킬 선택
- 검증: 각 스킬의 리서치 순서, 데이터 품질 규칙, 오버피팅 방어 규칙 적용
- 출력: 스킬별 출력 계약에 맞는 구조화된 결과 생성

즉, 이 저장소는 단순한 문서 모음이 아니라, Claude가 투자 분석을 일관된 절차로 수행하도록 만드는 작업 규격 집합입니다.

## 설계 원칙

- 재사용 가능한 역량 기준으로 스킬을 분리합니다.
- 구조는 얕게 유지하고, 스킬은 독립적으로 동작하게 만듭니다.
- `SKILL.md`는 짧고 작업 중심으로 유지합니다.
- 상세한 검증 규칙, 수식, 템플릿은 `references/`로 이동합니다.
- 반복 실행이 분명한 작업에만 `scripts/`를 추가합니다.
- 스킬 이름은 소문자 하이픈 표기를 사용합니다.
- 폴더별 유지보수 규칙은 추가 `README.md`보다 `AGENTS.md`로 관리합니다.

## 기여 원칙

새 스킬을 추가하거나 기존 스킬을 수정할 때는 다음 원칙을 따릅니다.

- `skills/`의 가장 적절한 최상위 카테고리에 둡니다.
- 구조는 얕게 유지합니다.
- 추측성 골격보다 작은, 검증 가능한 추가를 우선합니다.
- 중첩된 추가 `README.md`는 만들지 않습니다.
- 특정 호스트 제품에 종속된 설명은 피합니다.
- 편집 중인 디렉터리의 `AGENTS.md` 규칙을 우선합니다.

## 참고 연결

- `skills/fundamental-analysis/company-analysis`: 나래티브 우선, 시장이 이미 반영한 기대치 분석, Reverse DCF, FCFF 기반 내재가치, 재투자와 마진 분해 중심의 기업 분석 프레임
- `skills/quantitative-analysis/quant-research`: 팩터 프레임, 신호 검증, 백테스트, 오버피팅 방어, 리스크/어트리뷰션 분석 중심의 퀀트 리서치 프레임
- `skills/market-analysis/traditional-market-analysis`: 국면, 유동성, 포지셔닝, 내러티브 반사성까지 포함하는 시장 배경 해석 프레임

## 외부 참고

- [virattt/ai-hedge-fund](https://github.com/virattt/ai-hedge-fund): `skills/investor-personas/`의 일부 설계 참고 소스



**예시**
: **이 저장소는 실행 가능한 CLI 스크립트가 아니라, Claude가 읽고 따를 수 있는 작업 규격 모음입니다.**

## 사용 방식

### Claude에서의 사용 (권장)

- "company-analysis 스킬로 Apple을 분석해줘"

그러면 Claude가:

1. [SKILL.md](vscode-file://vscode-app/c:/Users/Dokyu/AppData/Local/Programs/Microsoft%20VS%20Code/10c8e557c8/resources/app/out/vs/code/electron-browser/workbench/workbench.html)를 읽음
2. 그 안의 절차(Narrative → Reverse DCF → Forward DCF 등)를 따름
3. 결과를 `references/output-contract.md`가 정의한 형식으로 정리함

### CLI에서는

직접 "실행"할 수 없습니다. 대신:

**# 폴더 열어서 SKILL.md를 읽으면 됨**
```txt
cat skills/fundamental-analysis/company-analysis/SKILL.md
```

이런 식으로 파일 내용을 확인한 후, **그 절차를 직접 수행**하거나 **Claude에게 그 규칙을 링크로 건넵니다**.

## 현재 저장소의 실행 가능 부분

`scripts/` 폴더를 보면:

```txt
skills/data-access/openbb-data-fetcher/scripts/fetch_openbb.py
```

이건 실제 Python 스크립트라서 CLI에서 실행 가능합니다:

```txt
python skills/data-access/openbb-data-fetcher/scripts/fetch_openbb.py
```
## 권장 워크플로

1. **Claude와 대화**: "company-analysis로 TSLA 분석해" → Claude가 스킬 규칙 적용
2. **데이터 필요**: "openbb-data-fetcher로 TSLA 실시간 데이터 가져와" → Python 스크립트 실행
3. **최종 보고**: "financial-report 스킬로 결과 정리해" → Claude가 출력 형식 적용

즉, 이 저장소 자체는 **Claude를 위한 일관된 분석 작업 규격**이고, 
CLI는 데이터 수집 같은 보조 작업에만 쓰는 방식. 
CLI 기반 프레임워크로 확장하거나, 각 스킬을 실제 Python/Node 모듈로 변환할 수도 있음.