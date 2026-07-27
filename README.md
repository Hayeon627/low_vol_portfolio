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
| Winner BL 슬롯 (`mat_eq_eq_raw_pap`) | Sharpe 1.096 / Sortino 1.826 / CAGR 16.2% / MDD -13.6% |
| 벤치마크 대비 (20bp 거래비용 차감 후) | SPY 0.93 / 1N 0.86 / Risk Parity 0.89 / ANN-anchor 0.95 대비 우위 (Ensemble-defensive 1.04–1.07, Ensemble-adaptive 1.05–1.10) |
| 강건성 | Q 민감도 [0.001, 0.010] 전 구간 winner와 통계적 동등 (Memmel JK z-test), 3-레짐(HMM) 안정성 확인 |

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
│   ├── 05a_HMM_Regime(.._full).ipynb        3-레짐 HMM 국면 분류
│   ├── 05b_Analyze.ipynb                    성과 분석 (K_CUT → 민감도 → BL α 분해)
│   ├── 06_Regime_Analysis.ipynb             4-레짐(hold-out 포함) winner 검증
│   ├── 99_*.ipynb                           ANN 비교, 통계 부록 분석
│   ├── appendix/                            부록 노트북 (슬롯 효과, 집중도 등)
│   ├── *.py                                 timeseries_lib / lstm_pipeline / bl_config(_ann) / bl_functions / bl_runner / master_table / analyze_plots
│   ├── docs/                                단계별 상세 문서 (아래 표 참고)
│   └── _dev/, _evidence/                    개발용 스크립트, Optuna 캐시 (참고용)
│
├── final_data/                 노트북 산출 데이터 (data/03b_lstm/ 이하 LSTM 변동성 예측치)
├── final_outputs/               차트·표 PNG 산출물
│   ├── outputs/                     단계별 폴더(01_data ... 06_Regime_Analysis) + 레짐/누적수익 차트
│   └── results_backup_pre_spy_fix/  SPY 단위 수정 이전 백업
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
6. 레짐 안정성         3-레짐(HMM) + 4-레짐(hold-out) 민감도 검증
7. 대시보드            Streamlit 프로토타입 (root_html/에 정적 export 포함)
```

---

## 논문 버전 안내

- **`paper_docs/paper/paper.md`**: 한글 원고 마크다운 버전 (논문 본문 + 수식)
- **`paper_docs/Low_Volatility_Portfolio_via_Black_Litterman_with_Volatility_Forecasts/`**: 위 원고의 LaTeX 풀버전 (그림 포함, 최종 발표용)
- **`paper_docs/kjas_paper/`**: 한국응용통계학회(KJAS) 투고 규격 버전 (영문 제목/저자 표기, 학회 템플릿 `kjas3.cls` 적용)
- **`paper_docs/root_files/`**: 학회지(JIIS) 양식 변환본(.docx)

동일한 연구 내용을 제출처 양식에 맞게 재구성한 버전들이며, 실질적인 최종 결과·수치는 모두 동일합니다.
