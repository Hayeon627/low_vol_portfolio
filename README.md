# 변동성 예측을 결합한 블랙-리터만 저변동성 포트폴리오 전략

**Low-Volatility Black–Litterman Portfolio Strategy with Volatility Forecasts**

LSTM–HAR 앙상블 기반 변동성 예측을 블랙-리터만(Black-Litterman, BL) 뷰로 결합하여, 저변동(Low-Volatility) anomaly를 포트폴리오 전략으로 구현한 퀀트 리서치 프로젝트입니다. 이 폴더는 발표/제출용 **논문**과 **최종 코드·데이터·산출물**을 하나로 정리한 스냅샷입니다.

- **저자**: 서윤범, 김하연, 김재천, 김윤서, 서정욱
- **대상 유니버스**: S&P 500 617개 종목, 2010–2025년 (192개월)

---

## 한 줄 요약

> Pyo & Lee (2018)의 ANN 기반 BL 저변동 전략을 (1) LSTM–HAR 앙상블 변동성 예측, (2) BL 4대 입력 슬롯($\mathbf{p}, q, \omega, \mathbf{w}_{mkt}$)의 90개 조합 체계적 탐색이라는 두 축으로 확장하여, 편측 20bp 거래비용 차감 후에도 SPY·1/N·Risk Parity·ANN-anchor를 모두 상회하는 슬롯 구성을 제시합니다.

### 핵심 결과

| 구분 | 결과 |
|---|---|
| 변동성 예측 | LSTM–HAR 앙상블 RMSE 0.3822 (HAR 단독 0.3914, LSTM 단독 0.5185 대비 우위, 특히 COVID-19 위기 구간 R3에서 격차 최대) |
| 저변동 anomaly (횡단면 EDA) | 저변동 Sharpe 0.96 vs 고변동 0.73, MDD -16.7% vs -34.1% |
| **최종 제안** (논문) | 단일 최적 슬롯 대신 **두 옵션** — Ensemble-defensive($q^{lam}$, 전 국면 저변동 뷰 유지) / Ensemble-adaptive($q^{ff3}$, 강세장 적응) |
| 벤치마크 대비 (20bp 거래비용 차감 후) | SPY 0.93 / 1N 0.86 / Risk Parity 0.89 / ANN-anchor 0.95 대비 우위 (Ensemble-defensive 1.04–1.07, Ensemble-adaptive 1.05–1.10) |
| 강건성 | Q 민감도 [0.001, 0.010] 전 구간 통계적 동등 (Memmel JK z-test), 3-레짐(HMM) 전환점 안정성 확인 |
| **Out-of-sample 한계** (1단계) | 단일 winner 선정 방식(`mat_eq_eq_raw_pap`)은 hold-out 24개월에서 **실패** — Sortino 0.516 (90개 중 88위), SPY(2.310) 대비 -1.79. 이 negative result가 두 옵션 제안으로의 설계 전환 근거 |

---

## 폴더 구조

```
├── paper_docs/                 논문
│   ├── paper/                      한글 원고 (paper.md) + 부록·상세 수식(detail.md, appendix_grid_tables.md) + JIIS 변환본(.docx)
│   ├── Low_Volatility_Portfolio_.../ 한글 풀 논문 LaTeX (main.tex, Figure/, references.bib)
│   ├── kjas_paper/                  KJAS(한국응용통계학회) 투고 포맷 — 영문 제목/저자, kjas3.cls
│   ├── kjas_guide/                  KJAS 서식 가이드 원본
│   └── root_files/                  블랙-리터만 절 초안(.docx), JIIS 서식 가이드 국문 번역
│
├── final_code/                 최종 분석 코드 (노트북 7단계 파이프라인 + 모듈)
│   ├── 01_DataCollection.ipynb              데이터 수집 (S&P500 멤버십·가격·패널)
│   ├── 02a_EDA_Returns_Volatility.ipynb     시계열 EDA — 수익률 vs 변동성 예측 가능성
│   ├── 02b_LowVol_PortfolioSort.ipynb       횡단면 EDA — 저변동 anomaly 6단 포트폴리오 정렬
│   ├── 03a_LSTM_Optuna_GridSearch.ipynb     LSTM 하이퍼파라미터 탐색 (Optuna 12-trial)
│   ├── 03b_Volatility_Forecasting.ipynb     LSTM + HAR + 성과가중 앙상블 (617종목 stockwise)
│   ├── 04_BL_Walkforward.ipynb              BL walk-forward 백테스트 (90개 슬롯 매트릭스)
│   ├── 05a_HMM_Regime.ipynb                 HMM 국면 분류 — K_CUT 계열 [1단계]
│   ├── 05a_HMM_Regime_full.ipynb            HMM 국면 분류 — 전체 기간 [2단계, 논문]
│   ├── 05b_Analyze.ipynb                    성과 분석 (민감도 → BL α 분해) [1단계]
│   ├── 06_Regime_Analysis.ipynb             hold-out winner 검증 [1단계]
│   ├── 99_main_analysis.ipynb               논문 본문·부록 결과 산출 [2단계]
│   ├── 99_analyze_ann(_full).ipynb          ANN 비교 — (_full) 이 논문 반영본 [2단계]
│   ├── 99_lstm_statistics / 99_slot_effects  통계 부록 분석
│   ├── appendix/                            부록 노트북 (슬롯 효과, 집중도 등)
│   ├── *.py                                 timeseries_lib / lstm_pipeline / bl_config(_ann) / bl_functions / bl_runner / master_table / analyze_plots
│   └── docs/                                단계별 상세 문서 (아래 표 참고)
│
├── final_data/                 노트북 산출 데이터 (data/03b_lstm/ 이하 LSTM 변동성 예측치)
├── final_outputs/               차트·표 PNG 산출물
│   └── outputs/                     단계별 폴더(01_data ... 06_Regime_Analysis) + 레짐/누적수익 차트
│
└── root_html/                   대시보드 프로토타입 정적 export
    └── root_files/                  Adaptive VolControl Fund.html
```

### 코드 문서 가이드 (`final_code/docs/`)

| 목적 | 문서 |
|---|---|
| 90개 실험 슬롯 수식·명명 체계·실행법 | `BL_EXPERIMENT_GUIDE.md` |
| 데이터 수집 파이프라인 | `DATA_COLLECTION.md` |
| 저변동 anomaly 검증 EDA | `ANOMALY_ANALYSIS.md` |
| Q·PCT 민감도 분석 (winner 슬롯) | `SENSITIVITY_ANALYSIS.md` |
| Winner BL 슬롯 시계열 추이 | `WINNER_SLOT_TIMESERIES.md` |
| 선행연구 요약 (Pyo & Lee, 2018) | `Exploiting_LowRisk_Anomaly_BL_Summary.md` |
| 전체 파이프라인·파일 구조 개요 | `PROJECT_OVERVIEW.md` |

---

## 파이프라인 개요

```
1. 데이터 수집        S&P500 멤버십 + 일별 가격 + 월별 패널 + 매크로/FF 팩터
2. EDA               시계열(수익률 vs 변동성 예측성) + 횡단면(저변동 6단 정렬)
3. 변동성 예측 모델    LSTM(Optuna HPO) + HAR baseline → 성과가중 앙상블
4. Black-Litterman   π/Σ/P/Q/Ω 슬롯 조합(90개) walk-forward 백테스트
5. 성과 분석          MVO 위험성향 매핑, Sharpe/Sortino/MDD/α 분해
6. 레짐 분해·검증      HMM 국면 분류 → hold-out 검증(1단계) / 4-국면 성과 분해(2단계)
7. 대시보드            Streamlit 프로토타입 (root_html/에 정적 export 포함)
```

---

## 분석 계보 — 2단계 설계

본 저장소에는 **두 가지 평가 설계**가 병존합니다. 실수나 중복이 아니라, 1단계의 negative result에 대응해 설계를 바꾼 결과이며 두 결과 모두 의도적으로 보존합니다.

### 1단계 — Hold-out 검증 설계 (K_CUT)

- **구성**: TEST(2010-01 ~ 2023-12, 168m) + HOLD-OUT(2024-01 ~ 2025-12, 24m) 분리
- **국면**: R1 30m / R2 90m / R3 48m(=K_CUT) + R4 = hold-out 24m
- **코드**: `05a_HMM_Regime.ipynb` → `05b_Analyze.ipynb` → `06_Regime_Analysis.ipynb`, 국면 상수는 `master_table.REGIMES`
- **방식**: TEST 구간의 `sortino_ir` 기준으로 90개 슬롯 중 단일 winner 자동 선정 → hold-out으로 검증
- **결과**: winner `mat_eq_eq_raw_pap` 가 **hold-out에서 실패**. Sortino 0.516(90개 중 88위), SPY 2.310 대비 -1.79.
  AI 랠리 강세장에서 저변동 anomaly의 cyclical 약점이 드러남 (Frazzini–Pedersen 2014 §5와 정합)

### 전환 판단

hold-out 실패는 특정 슬롯의 문제가 아니라 **"전 국면에서 통하는 단일 최적 슬롯을 찾는다"는 접근 자체의 한계**로 해석했습니다. 이에 단일 winner 선정을 폐기하고, 국면별로 강점이 다른 복수 옵션을 제안하는 구조로 전환했습니다.

### 2단계 — 국면 분해 설계 (full)

- **국면**: R1 30m / R2 90m / **R3 42m**(2020-01~2023-06) / **R4 30m**(2023-07~2025-12) — 전 기간 HMM 전환점(2023-06-05)에 align, R1과 R4 길이 대칭
- **코드**: `05a_HMM_Regime_full.ipynb` → `99_main_analysis.ipynb` · `99_analyze_ann_full.ipynb` (국면 상수 직접 hardcode)
- **결과**: Ensemble-defensive / Ensemble-adaptive 두 옵션 제안. 각각 R1·R3(변동성 상승 국면)와 R2·R4(강세장)에서 상호 보완적 강점
- **논문 본문·부록 수치는 전부 이 계열에서 산출**

### 두 설계의 관계와 한계

| | 1단계 (K_CUT) | 2단계 (full) |
|---|---|---|
| 슬롯 선택의 out-of-sample 검증 | ✅ 있음 (24m hold-out) | ❌ 없음 (R4가 평가에 포함) |
| 국면 길이 대칭성 | ❌ R4가 24m로 짧음 | ✅ R1=R4=30m |
| 산출물 | 단일 winner + 실패한 hold-out | 두 옵션 + 국면별 분해 |

2단계 설계에서도 walk-forward 백테스트 자체는 각 시점 예측이 out-of-sample이지만, **90개 중 어느 슬롯을 추천할지 고르는 단계에는 hold-out이 없습니다.** 1단계 결과를 함께 제시하는 이유가 이것입니다.

`master_table.REGIMES`는 1단계 정의를 유지하며 갱신하지 않습니다. 2단계 노트북들은 이 상수를 import하지 않고 국면을 직접 정의합니다.

### 재현성 검증 — HMM (2026-08-07)

`05a_HMM_Regime_full.ipynb`를 재실행하여 논문 부록 A와 일치함을 확인했습니다.

| n=3 | 논문 표 9 (5,512일) | 재실행 (5,534일, seed 0) |
|---|---|---|
| Bull | 52.9% / 0.653% / VIX 14.0 | 52.7% / 0.653% / 14.0 |
| Neutral | 34.1% / 1.070% / 20.7 | 34.2% / 1.068% / 20.7 |
| Bear | 13.1% / 2.271% / 35.0 | 13.1% / 2.264% / 35.0 |
| posterior n=3 (표 8) | 0.984 / 94.5% | 0.983 / 94.4% |
| 구조적 전환점 (표 11) | 4개 | 날짜·지속일수까지 완전 일치 |

미세 차이는 논문 실행분의 데이터가 22거래일 짧았기 때문입니다(논문 A1: 2026-02까지 5,512일 / 현재 `data/`: 2026-03-31까지 5,534일). 시드 0~9 스윕 결과 전부 같은 계열 해로 수렴해 안정적입니다.

> 🔒 `random_state=0`을 바꾸지 마세요. log-likelihood 최대해는 seed 2~9의 Bull 45.7%이고, 논문값에 가장 가까운 것은 seed 0입니다. multi-restart를 도입하면 오히려 논문에서 멀어집니다.

### ⚠️ 데이터 재수집 주의 — `01_DataCollection.ipynb`

`build_monthly_membership()`이 Wikipedia **현재** 페이지에서 과거 S&P 500 편입/편출 이력을 재구성하므로, **실행 시점에 따라 과거 유니버스가 달라집니다.**

| | 논문 | 2026-08-07 재수집 |
|---|---|---|
| universe.csv | 833종목 | 837종목 |
| daily_returns.pkl | 824종목 | 836종목 |
| monthly_panel.csv | **617종목** | **609종목** |

논문은 "S&P 500 617종목" 기준입니다. **01을 재실행하면 04_BL_Walkforward 이후 성과 수치를 재현할 수 없습니다.** HMM(05a)은 FF·매크로만 사용해 유니버스와 무관하므로 영향받지 않습니다.

원본 `data/`는 저장소에 포함되어 있지 않습니다(용량). 논문 수치 재현이 필요하면 원 저자 환경의 데이터가 필요합니다.

---

## 논문 버전 안내

- **`paper_docs/paper/paper.md`**: 한글 원고 마크다운 버전 (논문 본문 + 수식)
- **`paper_docs/Low_Volatility_Portfolio_via_Black_Litterman_with_Volatility_Forecasts/`**: 위 원고의 LaTeX 풀버전 (그림 포함, 최종 발표용)
- **`paper_docs/kjas_paper/`**: 한국응용통계학회(KJAS) 투고 규격 버전 (영문 제목/저자 표기, 학회 템플릿 `kjas3.cls` 적용)
- **`paper_docs/root_files/`**: 학회지(JIIS) 양식 변환본(.docx)

동일한 연구 내용을 제출처 양식에 맞게 재구성한 버전들이며, 실질적인 최종 결과·수치는 모두 동일합니다.
